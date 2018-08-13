---
title: 网站发送 Safari 推送通知
key: 20151112
tags: Safari
---
在 2013 年的 WWDC 上，Safari 增加了新的推送通知功能。虽然这功能用的人并不多，但确是很实用。

用户在访问网站后，网站会请求用户同意推送，之后就可以用 APNs 来推送通知了（不论用户是否开启 Safari，利用的 OS X 的推送）。具体在 IM web 版本等具体领域上可以应用。

具体开发文档参见 Apple Developer 文档，这里不重复：

https://developer.apple.com/notifications/safari-push-notifications/

![ Safari 推送过程](http://www.ixiaozhi.com/content/images/2015/11/9B08E895-6173-437E-A83B-676456B47436.png)

国内网站 [少数派](http://sspai.com) 有使用该功能，可以用 Safari 访问尝试。

