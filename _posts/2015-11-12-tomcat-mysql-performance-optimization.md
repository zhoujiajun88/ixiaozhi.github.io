---
title: Tomcat 性能的一些优化手记
key: 20151112
tags: tomcat
---
## 集群
说能优化高并发，最先最开始当然是服务器自身硬件的升级及服务器集群了。之前的服务器设计上并没有考虑集群的习惯，所以第一件事在做集群的时候，要同步 Session。因为服务器在阿里云上，所以 Seesion 集群同步直接考虑阿里云的 OCS 方案(操作与 Memcached 几乎一致)。

Tomcat 的 Server.xml 配置文件中添加 Manager 节点，托管 Tomcat 的 Session 至 Memcached。

    <Context path="" docBase="/home/xxx/xxx.war" reloadable="true" > 
    <Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager" memcachedNodes="xxxxxxxxxxxx.m.cnhzaliqshpub001.ocs.aliyuncs.com:11211" username="xxxxxxxx" password="xxxxxxxx" memcachedProtocol="binary" sticky="false" lockingMode="auto" sessionBackupAsync="false" sessionBackupTimeout="1000" requestUriIgnorePattern=".*\.(gif|jpg|jpeg|png|bmp|swf|js|css|html|htm|xml|json)$" transcoderFactoryClass="de.javakaffee.web.msm.serializer.kryo.KryoTranscoderFactory" />
    </Context>

这里使用 Kryo 进行序列化。对应的，Tomcat/lib 下不仅需要 memcached 基础包外，还需要添加 memcached-session 相关的 Jar 包，注意 Tomcat 的版本下载不同版本的 Jar 包。

* memcached-session-manager-1.8.3.jar
* memcached-session-manager-`tc7`-1.8.3.jar
* spymemcached-2.8.4.jar
* kryo-1.04.jar
* kryo-serializers-0.11.jar
* minlog-1.2.jar
* msm-javolution-serializer-1.6.3.jar
* msm-kryo-serializer-1.6.3.jar
* msm-xstream-serializer-1.6.3.jar
* reflectasm-0.9.jar

## 数据库
通过压测下来，发现 CPU/内存未达到上限，但网站的响应时间已经下去了，发现瓶颈在数据库。继续升级数据库服务器的硬件。接下来更换数据库连接池。

原来使用的 DBCP 连接词，更换至 BoneCP 数据库连接池。

### BoneCP spring 配置更新

    <bean id="dataSource" class="com.jolbox.bonecp.BoneCPDataSource" destroy-method="close">
        <property name="driverClass" value="${jdbc.driverClassName}" />
        <property name="jdbcUrl" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="idleConnectionTestPeriod" value="${jdbc.idleConnectionTestPeriod}"/>
        <property name="idleMaxAge" value="${jdbc.idleMaxAge}"/>
        <property name="maxConnectionsPerPartition" value="${jdbc.maxConnectionsPerPartition}"/>
        <property name="minConnectionsPerPartition" value="${jdbc.minConnectionsPerPartition}"/>
        <property name="partitionCount" value="${jdbc.partitionCount}"/>
        <property name="acquireIncrement" value="${jdbc.acquireIncrement}"/>
        <property name="statementsCacheSize" value="${jdbc.statementsCacheSize}"/>
        <property name="releaseHelperThreads" value="${jdbc.releaseHelperThreads}"/>
    </bean>

### pom.xml maven 添加包

    <dependency>
          <groupId>com.jolbox</groupId>
          <artifactId>bonecp</artifactId>
          <version>0.8.0.RELEASE</version>
    </dependency>

实测下来，对比 dbcp 连接池，效率提升5倍左右，大并发下出错率降低明显。

继续更换连接池至 Druid，阿里系的 Druid 号称是Java语言中最好的数据库连接池。
### Druid spring 配置更新
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
    <property name="initialSize" value="${jdbc.initialSize}" />
    <property name="maxActive" value="${jdbc.maxActive}" />
    <property name="minIdle" value="${jdbc.minIdle}" />
    </bean>

### pom.xml maven 添加包

    <dependency>
          <groupId>com.alibaba</groupId>
          <artifactId>druid</artifactId>
          <version>1.0.15</version>
    </dependency>

### Druid 监控
Druid 的监控是一个 Servlet，直接在 `web.xml` 中开启。监控中会含有一些敏感的信息，因此可以设计一个 allow 的 IP 白名单。

    <servlet>
        <servlet-name>DruidStatView</servlet-name>
        <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
        <init-param>
            <param-name>allow</param-name>
            <param-value>192.168.1.1</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>DruidStatView</servlet-name>
        <url-pattern>/druid/*</url-pattern>
    </servlet-mapping>

访问 /druid 可以监测数据源信息。
![](http://www.ixiaozhi.com/content/images/2015/11/92E1F163-B8B5-4261-BE98-F6E5C26A4268.png)

## 其他监测工具
服务器的监控部署选择使用 probe 工具，probe.war 包直接置于 webapps 下即可。

用户名密码在 config/tomcat-users.xml 里设置

    <role rolename="probeuser" />
    <role rolename="poweruser" />
    <role rolename="poweruserplus" />
    <role rolename="manager-gui" />
    <user username="admin" password="123456789" roles="probeuser,poweruser,poweruserplus,manager-gui" />

![](http://www.ixiaozhi.com/content/images/2015/11/5318825B-0A25-4B1D-86FD-6D19B555CFC5-1.png)

然而关键我并不是运维啊，硬头皮上。

---
*参考*

* Maven http://search.maven.org/
* BoneCP http://jolbox.com/
* Druid https://github.com/alibaba/druid/
* psi-probe https://github.com/psi-probe/psi-probe
