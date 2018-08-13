---
title: CORS JavaScript 跨域方案 Java 实现
key: 20160512
tags: cors javascript
---
## JSONP 方案

### 实现

在 Controller 中，直接返回 com.fasterxml.jackson.databind.util.JSONPObject 对象，前端代码中便可使用 JSONP 的方案接受。

```
@RequestMapping(value = "/test") 
@ResponseBody
public Object test(@RequestParam(required = false,name = "callback") String callback) {
    // 程序需要返回的数据
    Map<String,String> result = new HashMap<>();
    result.put("result","返回的结果");

     if (callback == null || "".equals(callback)) {
         return result; // 非 JSONP 请求，返回正常的 JOSN 数据
     } else { 
        return new JSONPObject(callback, result); // JSONP 请求，返回 JOSNP 数据
     }
 }
```

### 缺点

JSONP 仅能使用 GET 请求的方式。 对于 RESTful 的 API 来说，发送 POST/PUT/DELET 请求将成为问题，不利于接口的统一。

## 共享 CORS 方案

### 实现

CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。
前端代理无需作处理，后端代码中，Header 头中添加 `Access-Control-Allow-Origin: https://static.ixiaozhi.com` 来告知浏览器允许的域名。

CORS 有简单与非简单请求，其中区别可见参考资料中的阮一峰的网络日志。

Java 服务器端，我们使用 Filter 来批量添加相关的 Header 头信息。

CORSFilter 类代码：

```
package com.ixiaozhi.filter; 
import org.springframework.web.filter.OncePerRequestFilter;
import javax.servlet.FilterChain;
 import javax.servlet.ServletException;
 import javax.servlet.http.HttpServletRequest;
 import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

  /** 
 * Created by ixiaozhi  
*/

public class CORSFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // CORS 的域名白名单，不支持正规，允许所有可以用 *
        response.addHeader("Access-Control-Allow-Origin", "https://static.ixiaozhi.com"); 
        // 对于非简单请求，浏览器会自动发送一个 OPTIONS 请求，利用 Header 来告知浏览器可以使用的请求方式及 Header 的类型
        if (request.getHeader("Access-Control-Request-Method") != null && "OPTIONS".equals(request.getMethod())) {
                     response.addHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE");
                     response.addHeader("Access-Control-Allow-Headers", "Content-Type");
                     response.addHeader("Access-Control-Max-Age", "1");
                 }  
        filterChain.doFilter(request, response);
    }
  }
```

该过滤器在 `web.xml` 中配置：

```
 <filter>
    <filter-name>cors</filter-name>
    <filter-class>com.ixiaozhi.filter.CORSFilter</filter-class>
 </filter>
 <filter-mapping>
    <filter-name>cors</filter-name>
    <url-pattern>/*</url-pattern><!--需要允许CORS跨域的地址-->
 </filter-mapping>
```

其他，前端的代码使用正常的 Ajax 调用方式就可以，无需关心跨域问题。

## 参考资料

* [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
