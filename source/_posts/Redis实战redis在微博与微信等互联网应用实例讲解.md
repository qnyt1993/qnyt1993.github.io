---
title: Redis实战redis在微博与微信等互联网应用实例讲解
tags: [redis]
categories: [后端,redis]
date: 2019-11-04 19:06:56
---

### 1. 对象缓存

|id|user|balance|
|---|---|---|
|1|john|1200|
|2|tom|3000|

对于这种存储，redis可以使用mset or hmset实现
    
    mset user:{userId}:name john user:{userId}:balance:1200
    mget user:{userId}:name user:{userId}:balance
    mset user:1:name john user:1:balance:1200
    mget user:1:name user:1:balance
    
    or
    
    hmset user {userId}:name john {userId}:balance 1888
    hmget user {userId}:name {userId}:balance
    hmset user 1:name john 1:balance 1888
    hmget user 1:name 1:balance
    
    
### 2. 计数器

    incr article:readcount:{文章id}    
    get article:readcount:{文章id}
    
### 3. web集群session共享

    spring session + redis实现session共享   
    
### 4. 分布式系统全局序列号

> 分库分表情况下，自行生成主键id,高并发情况下，每次批量取出1000个id存入内存中供使用，用完再取

    INCRBY orderId 1000   //redis批量生成序列号提升性能
    
### 5. redis实现电商购物车

![QQ截图20191104194310.png](https://i.loli.net/2019/11/04/HEqJpk3SeBGciXf.png)

![QQ截图20191104193724.png](https://i.loli.net/2019/11/04/Esj9roxkvXbVwRZ.png)

> 电商购物车
> 1. 以用户id为key
> 2. 商品id为field
> 3. 商品数量为value

    //向购物车添加商品
    hset cart:1001 10099 1
    hset cart:1001 20088 1
    hset cart:1001 30088 1
    //往存在购物车中的商品增加数量
    hincrby cart:1001 10099 1
    //获取购物车中商品的数目
    hget cart:1001 10099
    //获取购物车中存在的不同商品总数
    hlen cart:1001
    //删除商品
    hdel cart:1001 10099
    //获取所有商品
    hgetall cart:1001

## Hash结构优缺点

优点

    1. 同类数据归类整合存储，方便数据管理
    2. 相比string操作消耗内存与cpu更小
    3. 相比string存储更节省空间

缺点

    1. 过期功能不能使用在field，只能用在key上
    2. redis集群架构下不适合大规模使用

 
 
## 队列的使用

![QQ截图20191104200122.png](https://i.loli.net/2019/11/04/7Evrx9gh2qzoiyc.png)

 ![QQ截图20191104195602.png](https://i.loli.net/2019/11/04/64smEOFCHcTUifn.png)
 
 
![QQ截图20191104201553.png](https://i.loli.net/2019/11/04/zSvhRjodqPYrK9W.png)   
 
 
![QQ截图20191104201812.png](https://i.loli.net/2019/11/04/u7yVOxZUdN9b256.png)  

   

## set的使用

![QQ截图20191104202540.png](https://i.loli.net/2019/11/04/hPgcO9ST68apvIj.png)

![QQ截图20191104202755.png](https://i.loli.net/2019/11/04/12PcU3pBNx95aDr.png)

![QQ截图20191104203538.png](https://i.loli.net/2019/11/04/1OgNustcvXbFP3j.png)

![QQ截图20191104204144.png](https://i.loli.net/2019/11/04/l8eH5NxEfQvtbSY.png)

![QQ截图20191104205121.png](https://i.loli.net/2019/11/04/AlP8MBRJ2YsSwOF.png)

![QQ截图20191104205147.png](https://i.loli.net/2019/11/04/SKsmqYcXoz4iQkO.png)

![QQ截图20191104205202.png](https://i.loli.net/2019/11/04/3EtWFzD2ZscH7mJ.png)

![QQ截图20191104205332.png](https://i.loli.net/2019/11/04/F7PQhZVLAeTEOWr.png)

![QQ截图20191104205413.png](https://i.loli.net/2019/11/04/1YlfP3tQDynvKWa.png)















 