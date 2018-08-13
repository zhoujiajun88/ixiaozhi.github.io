---
title: API 签名简谈
key: 20170307
tags: api
---
介绍一种简单的 API 调用时，各参数的完整性效验。仅使用服务器签发 App Key 与 App Secret 和客户进行交互。

## 方式

Client 端使用 Server 提供的 App Secret 以及指定的方法进行签名，生成 signature 签名字符串，Server 收到请求先进行验签，确认请求完整。反之亦然。

## 服务器维护客户端列表

服务器端需要一个客户端、App Key、App Secret 的对应关系。

假使我们有以下客户端：

| Client | App Key | App Secret |
| --- | --- | --- |
| 官方客户端 | 0001 | 3F2504E0-4F89-11D3-9A0C-0305E82C3301 |

服务端提供 App Key 与 App Secret 给用户客户端程序。

假设使用参数名称以及当前时间的时间戳 `timestamp` （可以规定，服务器与客户端的时间差需要小于10分钟）先按字典序进行排列；然后再拼上 App Secret 进行 md5 signature 后，把 `appkey` 及 `signature` 附加在字段后传输。

## 客户端发送请求

某请求有三个参数


| 参数名 | 值 |
| --- | --- |
| ab | test |
| aa | 1234 |
| zz | haha |

1. request params 的字典序为(包含当时时间戳):  `aa=1234&ab=test&timestamp=1488857536&zz=haha`
2. 拼接上 App Secret 后为 `aa=1234&ab=test&timestamp=1488857536&zz=haha3F2504E0-4F89-11D3-9A0C-0305E82C3301` ; 对该字符串进行 md5 散列哈希后为 `C194D39F31D115B7F115631237E9CD5E`
3. 因此最终的请求 request params 为：`aa=1234&ab=test&timestamp=1488857536&zz=haha&appkey=0001&signature=C194D39F31D115B7F115631237E9CD5E`

## 服务器验签

先对接收到的参数进行拆解，分成四个部分

* 原请求再进行字典序 `aa=1234&ab=test&timestamp=1488857536&zz=haha`
* App Key `0001`
* signature `C194D39F31D115B7F115631237E9CD5E`
* timestamp `1488857536`

然后进行验签

1. 判断时间是否过期
2. 利用 App Key 查表得出 App Secret；把原请求进行相同的算法，把得到的 md5 哈希字符串与收到的 signature 字段串进行比较

## 反向

服务器返回给客户端的数据，可以进行同样的方式进行签名与验签。

