---
title: Nginx 配置 HTTP/2
key: 20160226
tags: nginx http2
---
前提条件，Nginx 已经配置好 SSL 的 HTTP/1 访问。

至于为何要使用 HTTP/2，我便不讨论了，可以参见文后给出的参与链接。

## 升级 Nginx 版本

当前(2016-02-26) nginx Stable version 最新为 `1.8.1`，Mainline version 最新为 `1.9.12` ，我们需要使用 `1.9.5+` 版本才能支持 HTTP/2 的配置访问。可以通过 `nginx -v` 查看系统中安装的 Nginx 版本，因为我的 Nginx 版本过低，所以我们先升级当前的 Nginx 版本。

修改 `/etc/yum.repos.d/nginx.repo` 的 nginx 仓库地址为 mainline 的版本

```
[nginx]
name=nginx repo  
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/  
gpgcheck=0  
enabled=1
```

CentOS 系统通过 `yum clean all && yum update nginx` 升级 Nginx 版本。 

## 配置 Nginx HTTP/2 支持

找到 nignx 的配置文件，并修改。

```
[root@ixiaozhi local]# whereis nginx
nginx: /usr/sbin/nginx /etc/nginx /usr/share/nginx
```

我服务器里的 nginx 配置文件位于 `/etc/nginx/conf.d/default.conf` ，找到 Server 节点，并修改。

```
server {
    listen  443  ssl http2; #增加 HTTP2
    server_name  www.ixiaozhi.com;

    ssl_certificate      /etc/nginx/ixiaozhi.com.crt;
    ssl_certificate_key  /etc/nginx/ixiaozhi.com.key;
    
    
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers   on;
    
    ... ...
```

这时候，重启 Nginx 服务器 `service nginx restart` ，浏览器无法访问，且 Chrome 会报 "ERR_SPDY_INADEQUATE_TRANSPORT_SECURITY" 错误。

通过资料查阅，HTTP/2 协议中对 TLS 有了更严格的限制：例如 HTTP/2 中只能使用 TLSv1.2+，还禁用了几百种 CipherSuite（详见：TLS 1.2 Cipher Suite Black List）。至此可以肯定，之所以出现这个错误，要么是服务端没有启用 TLSv1.2，要么是 CipherSuite 配置有问题。

因此我们上面的文件需要继续修改：

* 添加 ssl_protocols 支持 TLSv1.2
* 修改 ssl_ciphers ，我使用的是 Mozilla 推荐的 CipherSuite 配置，参见 [Mozilla 推荐配置](https://wiki.mozilla.org/Security/Server_Side_TLS#Recommended_configurations)

改后的文件：

```
server {
    listen  443  ssl http2; #增加 HTTP2
    server_name  www.ixiaozhi.com;

    ssl_certificate      /etc/nginx/ixiaozhi.com.crt;
    ssl_certificate_key  /etc/nginx/ixiaozhi.com.key;
    
    
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS; # 修改 CipherSuite 协议
    ssl_prefer_server_ciphers   on;
    ssl_protocols  SSLv3 TLSv1 TLSv1.1 TLSv1.2; # 增加 TLSv1.2 支持
    
    ... ...
```

重启 Nginx 服务器 `service nginx restart`

使用Chrome的网络工具，在地址栏中输入 `chrome://net-internals/#http2` ，这时便可以在 HTTP/2 sessions 中找到你的网站。

## 参考资料

* [HTTP/2 与 WEB 性能优化（一）](https://imququ.com/post/http2-and-wpo-1.html)
* [HTTP/2 与 WEB 性能优化（二）](https://imququ.com/post/http2-and-wpo-2.html)
* [HTTP/2 与 WEB 性能优化（三）](https://imququ.com/post/http2-and-wpo-3.html)
* [从启用 HTTP/2 导致网站无法访问说起](https://imququ.com/post/why-tls-handshake-failed-with-http2-enabled.html)
* [Mozilla 推荐 CipherSuite 配置](https://wiki.mozilla.org/Security/Server_Side_TLS#Recommended_configurations)


