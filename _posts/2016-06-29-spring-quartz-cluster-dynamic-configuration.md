---
title: Spring 与 Quartz 动态配置
key: 20160629
tags: spring quartz
---
因为项目的需求，需要有动态配置计划任务的功能。
本文在 Quartz JobBean 中获取配置的 Quartz cronExpression 时间表达式及 Spring Bean 的对象名、方法名并运行。

## 准备

### 环境

* quartz : 2.2.2
* spring : 4.2.3.RELEASE

### 配置

假设已经配置好数据源，且在数据库中已经建好相关的 Quartz 表。

Spring 配置文件配置好单机器的 Quartz 任务。

```
<bean id="localQuartzScheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean"></bean>
```

去除原有的 quartz 的 jobDetail 等其他设置，下面我们将把这些改为动态设置。

## 集群

Spring 增加 cluster quartz 配置。

```
<!-- Quartz集群Scheduler -->
<bean id="clusterQuartzScheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
   <!-- quartz配置文件路径-->
<property name="configLocation" value="classpath:quartz.properties"/>
<!-- 启动时延期3秒开始任务 -->
<property name="startupDelay" value="3"/>
<!-- 保存Job数据到数据库所需的数据源 -->
<property name="dataSource" ref="dataSource"/>
<!-- Job接受applicationContext的成员变量名 -->
<property name="applicationContextSchedulerContextKey" value="applicationContext"/>
<property name="overwriteExistingJobs" value="true"/> </bean>
```

配置中使用的 dataSource 数据源，需要提前配置；quartz.properties 属性文件自行配置。集群定时任务的任务会序列化后储存至数据库，在某机器 crash 后，可以快速的切换到新的机器去运行，并且保证有仅只有一台机器运行计划任务。

先写 QuartzRunnable 文件，这个文件我用来启动 Quartz 定时任务。

```
import org.quartz.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.ApplicationContext;

/**
 * Created by ixiaozhi on 16/6/27.
 */
public class QuartzRunnable {
    private static final Logger logger = LoggerFactory.getLogger(QuartzRunnable.class);

    private ApplicationContext context;

    /**
     * 构造函数, 传入 applicationContext
     *
     * @param context
     */
    public QuartzRunnable(ApplicationContext context) {
        this.context = context;
    }

    public void work() throws SchedulerException {
        logger.info("quartz is running ...");
        // scheduler 对象
        Scheduler schedulerCluster = (Scheduler) context.getBean("clusterQuartzScheduler");
        Scheduler schedulerLocal = (Scheduler) context.getBean("localQuartzScheduler");

        List<ScheduleJob> allQuartzJobs = ......; // 从数据库或者配置文件或者其他任何地方取得 Quartz 任务的配置文件， ScheduleJob 对象为自定义的 Quartz 任务设置，对象的属性见下文

        // 启动定时任务
        for (ScheduleJob job : allQuartzJobs) {
            // 区分本机运行或集群运行
            Scheduler scheduler;
            if (job.getIsCluster() == 1) {
                scheduler = schedulerCluster;
            } else {
                scheduler = schedulerLocal;
            }

            TriggerKey triggerKey = TriggerKey.triggerKey(job.getJobName(), job.getJobGroup());
            CronTrigger trigger = (CronTrigger) scheduler.getTrigger(triggerKey);
            //不存在，创建一个
            if (null == trigger) {
                JobDetail jobDetail = JobBuilder.newJob(MyDetailQuartzJobBean.class).withIdentity(job.getJobName(), job.getJobGroup()).build();
                JobDataMap dataMap = jobDetail.getJobDataMap();
                dataMap.put("scheduleJob", job); // 传递 job 对象至执行的方法体

                //表达式调度构建器
                CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(job.getCronExpression());
                //按新的cronExpression表达式构建一个新的trigger
                trigger = TriggerBuilder.newTrigger().withIdentity(job.getJobName(), job.getJobGroup()).withSchedule(scheduleBuilder).withDescription(job.getDescription()).build();
                scheduler.scheduleJob(jobDetail, trigger);
            } else {
                // Trigger已存在，那么更新相应的定时设置
                //表达式调度构建器
                CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(job.getCronExpression());
                //按新的cronExpression表达式重新构建trigger
                trigger = trigger.getTriggerBuilder().withIdentity(triggerKey).withSchedule(scheduleBuilder).build();
                //按新的trigger重新设置job执行
                scheduler.rescheduleJob(triggerKey, trigger);
            }
        }
    }
}
```

 从 Spring ApplicationContext 上下文对象中取得本文上面配置的两个 Quartz Scheduler，分别用于启动本机的定时任务与集群定时任务。再根据自己的配置，构造对应的 Trigger 与 Job，加入不同的计划任务中执行。
 
 对于计划任务，都使用同一个类 `MyDetailQuartzJobBean` 进行启动。在配置中可以根据配置反射启动相应的方法。
 
 我的 ScheduleJob 计划任务配置的属性有以下。 `targetObject` 为 Spring 中注入的 bean 名称， `targetMethod` 用于定时任务启动的方法入口。
 
```
 public class ScheduleJob implements Serializable {
    private static final long serialVersionUID = -4166311089940333025L;
    private String jobId; // 任务 ID
    private String jobName; // 任务名称
    private String jobGroup; // 任务分组
    private String cronExpression; // 时间表达式
    private String description; // 任务描述

    private String targetObject; // Spring 注入的类名
    private String targetMethod; // 方法
    
    private int isCluster;// 是否集群运行
    
    // getter and setter
    ... ...
    
 }
```

 比如测试的 ScheduleJob 对象:
 jobId=1;
 jobName="Test";
 jobGroup="DEFAULT";
 cronExpression="1/30 * * * * ?"; // 从1秒开始，每30秒执行一次
 description="测试任务";
 targetObject="testService";
 targetMethod="quartzTest";
 isCluster=true;
 含义为，将从 1 秒开始，每 30 秒在集群中的某一台机器运行一次，从 targetObject 的注入对象中的 targetMethod 方法。
 
 现在来实现 JobBean 计划任务执行类，我命名为 `MyDetailQuartzJobBean.java`。
 
```
import org.quartz.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.quartz.QuartzJobBean;

import java.lang.reflect.Method;

/**
 * 动态运行方法
 */
//@PersistJobDataAfterExecution
//@DisallowConcurrentExecution //确保多个任务不会同时运行
public class MyDetailQuartzJobBean extends QuartzJobBean {
    private static final Logger logger = LoggerFactory.getLogger(MyDetailQuartzJobBean.class);

    private ScheduleJob scheduleJob;

    protected void executeInternal(JobExecutionContext context)
            throws JobExecutionException {
        try {
            Object targetObject = ApplicationContextUtil.getBean(scheduleJob.getTargetObject());
            Method m;
            try {
                m = targetObject.getClass().getMethod(scheduleJob.getTargetMethod(), new Class[]{});
                m.invoke(targetObject, new Object[]{});
            } catch (SecurityException e) {
                logger.error(e.getMessage(), e);
            } catch (NoSuchMethodException e) {
                logger.error(e.getMessage(), e);
            }
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
            throw new JobExecutionException(e);
        }

    }

    public void setScheduleJob(ScheduleJob scheduleJob) {
        this.scheduleJob = scheduleJob;
    }
}
```

JobBean 需要继承至 QuartzJobBean，并重写 executeInternal 方法。而且，Quartz 集群中运行的 QuartzJobBean 必须实现序列化。但是，applicationContext 并不支持序列化，在这里面直接注入对象会报 exception 且无法使用。

因此我使用静态化来保存 applicationContext 对象，实现类 `ApplicationContextUtil` 。

```
@Component
public class ApplicationContextUtil implements ApplicationContextAware {
    private static ApplicationContext applicationContext; // Spring应用上下文环境

    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        ApplicationContextUtil.applicationContext = applicationContext;
    }

    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    public static Object getBean(String beanName) {
        return applicationContext.getBean(beanName);
    }

    @SuppressWarnings("unchecked")
    public static <T> T getBeanDetail(String beanName) throws BeansException {
        return (T) applicationContext.getBean(beanName);
    }
}
```

Spring 配置文件中注册该 Bean。

```
<bean id="applicationContextUtil" class="com.ixiaozhi.util.ApplicationContextUtil"/>
```

利用 getBean 可直接从 Spring 上下文中取得注入的对象，如上述的 `MyDetailQuartzJobBean` 使用该方法绕过 Spring ApplicationContext 无法序列化的问题，且取得 Spring Bean 并反射调用其中的方法。

## 测试

```
    /**
     * 程序入口
     *
     * @param args
     */
    public void test() {
            // 初始化 Spring
            ApplicationContext applicationContext = new ClassPathXmlApplicationContext("/applicationContext.xml");

            // 启动定时任务
            QuartzRunnable quartz = new QuartzRunnable(applicationContext);
            quartz.work();
    }
```

其他更好的实现方案，欢迎留言一起探讨。

## 参考

* [Spring+quartz 实现动态管理任务](http://itindex.net/detail/53315-spring-quartz-%E7%AE%A1%E7%90%86)
* [Quartz应用与集群原理分析](http://tech.meituan.com/mt-crm-quartz.html)
* [Spring与Quartz Cluster备忘](http://shift-alt-ctrl.iteye.com/blog/2215983)


