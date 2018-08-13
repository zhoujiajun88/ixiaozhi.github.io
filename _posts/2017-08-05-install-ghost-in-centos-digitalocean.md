---
title: Install Ghost in DigitalOcean
key: 20170805
tags: digitalocean ghost
---
## 前言
惯例，换博客平台第一件事便是发篇搭建过程。本次使用的服务器是 DigitalOcean,CentOS，Blog 程序使用开源的 Ghost。
## 下载 Ghost
截止至目前，最高版本为0.7.0，那就它了。
文件是一个 zip 包，因为我的 DigitalOcean 上没有安装 Unzip，所以解压前我还得安装 Unzip。

    $ sudo yum install unzip
    $ unzip ghost-0.7.0.zip

## 下载 Node.js
Ghost 0.7.0 需要 Node.js v0.10~v0.12 版本，因些我们去 nodejs.org previous releases 下载旧版本（下载地址见`参考`）。这里下载的是 v0.12.7 版本。之后便是解压：

    $ tar zxvf node-v0.12.7-linux-x64.tar.gz

配置环境变量：

    $ sudo vi /etc/profile

文件末尾添加：

    PATH=$PATH:/root/ghost/node-v0.12.7-linux-x64/bin

生效配置并验证：

    $ source /etc/profile
    $ echo $PATH
    $ node -v
    $ npm -v

PATH 中已经有 nodejs/bin 且 node、npm返回正确的版本号 v0.12.7 便是配置成功了。

## 运行 Ghost
Ghost 需要依赖 sqlite 数据库，在安装时，sqlite 需要依赖 gcc,g++。所以先安装g++库：

    $ sudo yum install gcc gcc-c++

npm install

    $ npm install --production

修改配置

    $ cd /root/ghost/ghost
    $ vi config.js

修改 Production 节点，添加 MailServer 等配置，示例如下：

    production: {
        url: 'http://www.ixiaozhi.com',
        mail: {      
        transport: 'SMTP',
                options: {
                    service: 'smtp.xxxx.com',
                    auth: {
                        user: 'service@ixiaozhi.com',
                        pass: 'xxxxxxxxxx'
                    }
                }
        },
        database: {
            client: 'sqlite3',
            connection: {
                filename: path.join(__dirname, '/content/data/ghost.db')
            },
            debug: false
        },

        server: {
            host: 'xxxxxx',
            port: '80'
        }
    }

运行

    $ npm start --production

访问并进行管理员账号密码设置

    http://www.ixiaozhi.com/ghost

访问

    http://www.ixiaozhi.com

至此，程序已经在运行了。
## Forever 运行
为了 Node.js 以 daemon 运行并能在错误时自动重启，我选择使用 Forever 组件，当然也有其他的选择~

安装

    $ sudo npm install -g forever

运行并记录日志（ /root/ghost/log 目录要求先存在）

    $ NODE_ENV=production forever -l /root/ghost/log/forever.log  start index.js

---
*参考*

* Node.js 下载： https://nodejs.org/en/download/releases/
* Ghost 下载： https://ghost.org/download/
* Ghost Git：https://github.com/TryGhost/Ghost


