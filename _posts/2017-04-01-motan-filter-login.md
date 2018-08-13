---
title: motan 实现调用授权
key: 20170401
tags: motan rpc
---
motan 提供官方的 spi 扩展方法。 [点击查看](https://github.com/weibocom/motan/wiki/zh_userguide#%E4%BD%BF%E7%94%A8mock%E5%8D%8F%E8%AE%AE)

方式如下：

```
1、实现自定义mock协议类，继承 AbstractMockRpcProtocol，实现 processRequest 方法(自定义 mock 逻辑)。

2、添加spi声明 @SpiMeta(name = "your_mock_protocol") ，在 META-INF/services/com.weibo.api.motan.rpc.Protocol 文件中添加 mock 协议类的类全名。

3、配置 motan:protocol 为 SpiMeta 中声明的名字，即
 name=your_mock_protocol，如果在 client 端 mock，就在
 basicReferer 或 Referer 中设置对应 protocl；如果在 server
 端 mock，则在 export 中配置 ${mock协议的id}:port

```

## 扩展方式

### 扩展点
* Filter 发送/接收请求过程中增加切面逻辑，默认提供日志统计等功能
* HAStrategy 扩展可用性策略，默认提供快速失败等策略
* LoadBalance  扩展负载均衡策略，默认提供轮询等策略
* Serialization 扩展序列化方式，默认使用 Hession 序列化
* Protocol 扩展通讯协议，默认使用 Motan 自定义协议
* Registry 扩展服务发现机制，默认支持 Zookeeper、Consul 等服务发现机制
* Transport 扩展通讯框架，默认使用 Netty 框架

### 编写一个 Motan 扩展

1. 实现 SPI 扩展点接口
2. 实现类增加注解

```
@Spi(scope = Scope.SINGLETON)  // 扩展加载形式，单例或多例
@SpiMeta(name = "motan")  // name 表示扩展点的名称，根据name加载对应扩展
@Activation(sequence = 100) // 同类型扩展生效顺序，部分扩展点支持。非必填
```

## 实现

这里使用 Filter 过滤器扩展点，来现在对 Motan 接口登录授权。 其他的工作就简单了，大致代码如下：

```
@SpiMeta(name = "baseFilter")
@Activation(sequence = 99)
public class BaseFilter implements Filter {
    @Override
    public Response filter(Caller<?> caller, Request request) {
        Object[] args = request.getArguments();

        ApplicationContext applicationContext = 取出全局 Spring Context 对象
        ITokenService tokenService = (ITokenService) applicationContext.getBean("tokenServiceHandler");
        // 认证逻辑
        if (tokenService.isAllow(String.valueOf(args[0]))) {
            throw new RuntimeException("RPC manage token expired.\r\nRequest token: " + args[0]); // throw exception to client
        }

        Response response = caller.call(request);
        return response;
    }
}
```

然后，添加该扩展至 Motan。在 `resources` 下新建路径及文件 `META-INF.services/com.weibo.api.motan.filter.Filter`，并往中添加内容

```
com.ixiaozhi.motan.BaseFilter
```

至此，所有的工作已经完成。
