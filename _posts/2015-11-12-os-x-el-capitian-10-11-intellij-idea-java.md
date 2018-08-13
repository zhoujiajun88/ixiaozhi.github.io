---
title: OS X EL Capitian 10.11 升级后启动 Intellij IDEA 14 报"您需要安装旧java se 6 运行环境才能打开"
key: 20151112
tags: macOS Intellij
---
## 描述
升级 OS X EL Capitian 10.11 后，启动 Intellij IDEA 14 报"您需要安装旧Java SE 6 运行环境才能打开"
## 解决
首先，先查看本机的 Java 环境的版本

    $ java -version

我这边的 Java 版本是`javac 1.7.0_40`。

到Finder-->应用程序中找到Intellij IDAE，右键-->显示包内容-->Contents-->Info.plist，用文本编辑器编辑。找到 `JVMVersion` 节点，如下：

    <key>JVMVersion</key>
    <string>1.6*</string>

修改为上面版本值为本机的 Java 环境版本：(如果本机 Java SE 环境为 1.8，则修改为 `1.8*` 或者为具体的版本号)

    <key>JVMVersion</key>
    <string>1.7*</string>

保存，重新打开Intellij IDEA即可启动。
