---
title: Gradle 打包实现生产环境与测试环境配置分离
key: 20160512
tags: java gradle
---
# Gradle 打包实现生产环境与测试环境配置分离

前篇：[Maven 打包实现生产环境与测试环境配置分离](https://ixiaozhi.com/java-maven-archive-different-profile/)

前篇是使用 Maven 进行的包管理，这次我们使用 Gradle 进行 Java Web Server 的包管理的配置。

## 配置 Gradle 配置文件

build.gradle 中配置相关的 resources 配置文件的目录。不同的资源文件放置在 `src/main/filters/$env` 目录下，其中 $env 目录为环境名，例如：dev、test、product 等等。且定义了默认环境为 dev 环境。

```
def env = System.getProperty("profile") ?: "dev"

sourceSets {
    main {
        resources {
            srcDirs = ["src/main/resources", "src/main/filters/$env"]
        }
    }
}
```

把不同环境的 properties 的文件，分别放在 filters 目录下的不同的环境文件中，如下图。

![alt](https://raw.githubusercontent.com/zhoujiajun88/zhoujiajun88.github.io/images/2016/gradle-filters-properties.png)

在使用 Gradle 编译的时候，添加参数 `-Dprofile=dev` 来指定编译的最终代码为何环境。如：

```
# 把程序编译成生产环境
./gradlew bootRepackage -Dprofile=product
```
## 使用 Intellij IDEA 启动不同的 Gradle 环境

这里的方式同本文前篇所讲述的方式，可以直接参见 maven 的使用方式。
