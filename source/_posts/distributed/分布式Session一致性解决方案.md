---
title: 分布式Session一致性解决方案
date: 2018-11-10 19:12:04
tags: [distributed]
categories: distributed
---
## Session复制
   - 在支持Session复制的Web服务器上，通过修改Web服务器的配置，可以实现将Session同步到其它Web服务器上，达到每个Web服务器上都保存一致的Session
     - ### 优点
       - 代码上不需要做支持和修改
     - ### 缺点
       - 需要依赖支持的Web服务器，一旦更换成不支持的Web服务器就不能使用了，在数据量很大的情况下不仅占用网络资源，而且会导致延迟
     - ### 使用场景
       - 只适用于Web服务器比较少且Session数据量少的情况
     - ### 可用方案
       - 开源方案tomcat-redis-session-manager，暂不支持Tomcat8
   
## Session粘滞
   - 将用户的每次请求都通过某种方法（比如：nginx ip_hash等）强制分发到某一个Web服务器上，只要这个Web服务器上存储了对应Session数据，就可以实现会话跟踪
     - ### 优点
       - 使用简单，没有额外开销
     - ### 缺点
       - 一旦某个Web服务器重启或宕机，相对应的Session数据将会丢失，而且需要依赖负载均衡机制
     - ### 使用场景
       - 对稳定性要求不是很高的业务情景
                                                    
## Session集中管理
   - 在单独的服务器或服务器集群上使用缓存技术，如Redis存储Session数据，集中管理所有的Session，所有的Web服务器都从这个存储介质中存取对应的Session，实现Session共享
     - ### 优点
       - 可靠性高，减少Web服务器的资源开销
     - ### 缺点
       - 实现上有些复杂，配置较多
     - ### 使用场景
       - Web服务器较多、要求高可用性的情况
     - ### 可用方案
       - 开源方案Spring Session，也可以自己实现，主要是重写HttpServletRequestWrapper中的getSession方法
       
## 基于Cookie管理
   - 这种方式每次发起请求的时候都需要将Session数据放到Cookie中传递给服务端
     - ### 优点
       - 不需要依赖额外外部存储，不需要额外配置
     - ### 缺点
       - 不安全，易被盗取或篡改；Cookie数量和长度有限制，需要消耗更多网络带宽
     - ### 使用场景
       - 数据不重要、不敏感且数据量小的情况

## 总结
   - 经过分析对比以上4种方案，相对来说，Sessioin集中管理更加可靠，使用也是最多