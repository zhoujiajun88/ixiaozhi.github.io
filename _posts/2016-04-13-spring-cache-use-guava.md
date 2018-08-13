---
title: Spring Cache use Guava
key: 20160413
tags: spring cache
---
前篇：[Spring Cache 缓存注解实现](https://ixiaozhi.com/spring-cache-annotation-useage/)

## 需求

现在缓存需要指定的过期策略，比如某段时间后过期或者多久没有使用过后过期。

## 开始

同样的，spring 配置中文件中，需要启用 cache 相关的注解。

```
<cache:annotation-driven/>
```

TestCache 示例：

```
public class TestCache { 
    @Cacheable(value = "testCache")
     public String get(String name) {
         return System.currentTimeMillis() + "";
     }
 }
```

并注册该缓存的 Bean。

```
<bean id="testCache" class="com.ixiaozhi.TestCache"/>
```

对缓存管理的设置，改用 GuavaCache 并把配置移到 class 中进行配置。新建一个 CacheConfig 文件，并注册。

```
@Configuration
 @EnableCaching 
public class CacheConfig {
     @Bean
     public CacheManager cacheManager() {
         SimpleCacheManager cacheManager = new SimpleCacheManager();
        
        // 设置 testCache 的缓存有效期为创建后的1天
         GuavaCache testCache = new GuavaCache("testCache", CacheBuilder.newBuilder().expireAfterWrite(1, TimeUnit.DAYS).build());
        
        // 设置 testCache 的缓存有效期为访问过后的 1 个小时
        // GuavaCache testCache = new GuavaCache("testCache", CacheBuilder.newBuilder().expireAfterAccess(1, TimeUnit.HOURS).build()); 

        // 把缓存加入 cacheManager
        cacheManager.setCaches(Arrays.asList(testCache));
          return cacheManager; 
    }
```

GuavaCache 中可以设置像创建后固定时间过期或者是闲置固定时间后过期等缓存的策略，并可以对单个缓存类进行单独的配置。

