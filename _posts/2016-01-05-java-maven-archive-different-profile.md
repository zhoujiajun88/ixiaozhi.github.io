---
title: Maven 打包实现生产环境与测试环境配置分离
key: 20160105
tags: java maven
---
我们可以把环境变量配置在 maven 的配置文件中，然后打包发布或者运行时加上参数来区分运行的是测试环境或者是生产环境。这样在版本管理时只需要提交一份代码，能更方便的做版本管理。

## 配置 pom.xml

maven 配置文件中添加 resouces 资源文件的配置，并告知哪些配置文件需要动态替换或者哪些文件不需要替换。为了方便，我们设置需要替换的配置文件路径，在 build 节点中添加以下配置：

```
        <resources>
            <resource>
                <directory>${basedir}/resources</directory>
                <includes>
                    <include>**/*</include>
                </includes>
            </resource>
            <!--设置自动替换-->
            <resource>
                <directory>${basedir}/resources</directory>
                <includes>
                    <include>jdbc.properties</include>
                </includes>
                <!--也可以用排除标签-->
                <!--<excludes></excludes>-->
                <!--开启过滤-->
                <filtering>true</filtering>
            </resource>
        </resources>
```

其中 `${basedir}` 为预先配置的根路径，也可直接使用相对路径。 `jdbc.properties` 的内容为变量替换符，大概如下：

```
jdbc.url=${jdbc.url}
jdbc.username=${jdbc.username}
jdbc.password=${jdbc.password}
```

然后在 pom.xml 中的 properties 节点中添加每一个不同的运行参数要替换内容。可以直接把变量直接用 propertie 直接写在 pom.xml 里；也可以再引入不同的 xxx.properties 文件用来更方便的配置。这里使用第二种方法，配置如下：

```
    <profiles>
        <profile>
            <id>product</id>
            <build>
                <filters>
                    <filter>${basedir}/filters/jdbc-product.properties</filter>
                </filters>
            </build>
        </profile>
        <profile>
            <id>test</id>
            <build>
                <filters>
                    <filter>${basedir}/filters/jdbc-test.properties</filter>
                </filters>
            </build>
        </profile>
        <profile>
            <id>dev</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <build>
                <filters>
                    <filter>${basedir}/filters/jdbc-test.properties</filter>
                </filters>
            </build>
        </profile>
    </profiles>
```

这里用编译参数： dev、test、product 来区分开发、测试、生产不同环境的配置文件，并且去取不同的 properties 文件。这里把 properties 文件放置于 filter 目录下。

至此，maven 编译时，会读取 activeByDefault 为 true 的默认配置文件，即 dev 开发环境的文件。

## 使用 Intellij IDEA 启动不同的 maven 环境

这里不使用 maven 带的 tomcat 而使用自定义的 Tomcat。

打开 Intellij IDEA 的 Run/Debug Configurations，设置 maven 的 Parameters ，并为其添加 `-Pproduct` 或 `-Ptest` 编译不同环境的配置文件。

然后添加 Tomcat Server，在 Before launch 中，设置第一步，选择 Run Another Configuration，并选中先前配置的 product。

![](http://www.ixiaozhi.com/content/images/2016/01/A49B1B41-B9C0-44FC-87BB-5011638C817E.png)

之后 Deployment 中配置 Deploy 的 war 包便可以运行。

![](http://www.ixiaozhi.com/content/images/2016/01/2D824006-F547-4207-BD0A-86ECEBE08E50.png)

![](http://www.ixiaozhi.com/content/images/2016/01/31CFEDD7-C730-48DB-86D2-80665AF278AF.png)
