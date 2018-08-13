---
title: Mac 安装 DokuWiki
key: 20151203
tags: DokuWiki
---
作为个人知识管理文库或者是团队知识文库，选择 wiki 都是一个不错的方案。DokuWiki 使用文本来存储 wiki 内容，也方便备份及转移。

## Mac 配置 PHP

这里不多说，Mac 自带 Apache 及 php 插件，直接开启就可以。

```
sudo vim /etc/apache2/httpd.conf 
#找到 #LoadModule php5_module libexec/apache2/libphp5.so 并开启注释

sudo apachectl start #开启 Apache
```

访问 `http://localhost/` 便可以看到 `It Works!`

系统自带的 Apache 的配置文件位于 `/etc/apache2/`，网站存放目录位于 `/Library/WebServer/Documents/` 。为了之后程序方便，可以直接把该目录的读写权限设置为

## 安装 DokuWiki 程序

直接把解压后的文件放入 `/Library/WebServer/Documents/` ，访问 `http://localhost/install.php` ，按照提示输入 Wiki name、Manage Info 等信息，便完成了安装。

如果这时提示文件没有写权限，更改 `/Library/WebServer/Documents/` 目录及其子目录的权限。

访问 `http://localhost/index.php` 开启自己的 Wiki 之旅咯。（假设你的 Apache 端口还是 80）

## 数据备份

DokuWiki 使用文本方式储存数据，所以备份数据就很容易了。所有的数据都位于 `安装目录/data/` 下。

data 下的目录解释：

`data/pages` 当前版本的Wiki页面，txt 直接可以打开查看

`data/meta` 页面的 meta 数据，例如页面的创建者等等

`data/media` 存放图片、PDF 等媒体文件

`data/media_meta` 媒体文件的 meta 数据

`data/attic` 历史版本的 Wiki 页面，gz 压缩文件，解压后就是原 txt 文件

`data/media_attic` 媒体文件的历史版本

`conf` wiki 的所有设置

## 其他

wiki 编辑器上有一些常用的格式文本，也可以使用一些通过的 wiki 格式文本来编辑。比如表格可以使用：

```
! 表头1 ! 表头2 ! 表头3
| 第一行单元格 | 第一行单元格 | 第一行单元格
| 第二行单元格 | 第二行单元格 | 第二行单元格
| 第三行单元格 | 第三行单元格 | 第三行单元格
```

