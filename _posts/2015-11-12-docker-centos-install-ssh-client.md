---
title: Docker CentOS 安装 SSH
key: 20151112
tags: Docker SSH CentOS
---
## 安装 CentOS 镜像

```
docker pull centos  #使用当前最新的 centos-7.1503 (2015-11-12)
```

然后以交互方式进入容器，命令的`-i`表示以交互模式运行容器，`-t`表示容器启动后会进入其命令行

```
docker run -i -t centos /bin/bash
```

如果我们要挂上本地的硬盘进进入容器，可以使用`-v <宿主机目录>:<容器目录>`挂载本地目录进容器

```
docker run -i -t -v /User/ixiaozhi:/mnt/software centos /bin/bash
```

## 安装必要软件及配置
一次性安装 openssh 及 vim 等一些一会儿要用的 sshd 及其他工具软件。

```
yum install -y openssh openssh-server openssh-clients httpd vim passwd
```
看到 Complete 后，可以查一下 sshd 是否已经安装

```
which sshd
```
 然后，修改 ssh 的密码，以便可以远程登录。如果是 ubuntu 系统，直接用 `passwd ` 修改即可，centos 需要多做一些事。

```
ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key

vim /etc/pam.d/sshd   #修改其中的pam_loginuid.so修改为optional

mkdir /var/run/sshd
passwd   #设置一个密码
```

## 保存及启动
查看当前的容器，并 commit 更改

```
docker ps -l
docker commit e197 centos
```

现在便可以以后台长期运行的方式运行该容器，通过 `-p <对外端口>:<对内端口>` 可以指定端口映射。

```
docker run -d -p 22 -p 80:2368 centos /usr/sbin/sshd -D
docker ps -l
```

port 显示：    
0.0.0.0:32770->22/tcp, 0.0.0.0:80->2368/tcp    
因此，我们 ssh 的 22 端口被随机分配的端口为 32770，之后使用 32770 进行 ssh 连接。（PS:2368 是我将使用的 Node.js  的 Ghost 使用的端口）

打开电脑的 iTerm 或其他终端软件，就可以进行连接了。

```
ssh root@192.168.99.100 -p 32270
``` 
---
