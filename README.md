# HighConcurrence
深入扩展先前做的电商项目，添加秒杀商品的模块
***
# 一 高并发会导致哪些问题

## 1. 服务器请求数过载

- 每台服务器能够承载的请求的个数是一定的，请求数太多，服务器处理不过来。（单台服务器能够承受多少的并发量？C10K问题。具体数目不太确定，但肯定是无法满足百万以上的量级的）

## 2. 快速响应

- 一个Web请求可以简单地理解成从浏览器发出，经过应用服务器处理，再请求数据库服务器。这之间有网络延迟（北京-上海约15ms），还有应用服务器可能发生的GC操作（一次约50ms），这些对于TPS来说影响都很大。

## 3. 数据一致性

- 多个请求并发访问数据库，可能出现脏读、不可重复读、幻读等问题。

# 二 各个问题的可选解决方案

## 1. 服务器请求数过载

### (1) 尽量使用静态资源或动静分离

**减少.jsp文件的使用，减少服务器对jsp文件的解析；也减少jvm内存占用**

**用Nginx做动静分离：使用NFS文件服务器，当访问静态资源时则访问NFS，当访问动态资源时则访问服务器**

### (2) 负载均衡

当单台服务器无法满足高并发请求要求时，可以考虑将应用拆分部署到多台服务器上，这时就需要考虑在集群设备前加负载均衡处理，实现流量分发。

**负载均衡是指将负载（如请求等）进行平衡，分摊到各个处理单元上执行。**

负载均衡技术主要有两类：

#### &nbsp; &nbsp; **1）软件负载均衡**

&nbsp; &nbsp; 软件负载均衡常用的软件是**Nginx**，底层用的技术包括DNS负载均衡、HTTP负载均衡、IP负载均衡、链路层负载均衡等。

#### &nbsp; &nbsp; 2）硬件负载均衡

&nbsp; &nbsp; 一般软件负载均衡支持到5万级并发已经很困难了，可以考虑使用负载均衡服务器。较昂贵。

### (3) 控制请求数量

为访问量较大的API接口设置信号量，能够拿到信号量的就执行，不能的就返回异常。

## 2. 快速响应

先前说到应用服务器和数据库服务器之间会有网络延迟的问题，这一处可以采用数据库优化（使用存储过程）、使用缓存来解决；应用服务器的GC操作也会大量降低TPS，这一处可以用JVM优化来解决；另外浏览器和应用服务器之间的网络延迟也可以降低，通过使用CDN服务器（华为就有）。

### (1) 数据库优化

使用存储过程可以减少数据库IO次数，降低网络延迟。

数据库优化包含很多方面，不仅能解决网络延迟的问题，在其他方面优化还可以降低数据库本身操作的时间：

#### &nbsp; &nbsp; 1）SQL语句优化 

#### &nbsp; &nbsp; 2）索引优化

#### &nbsp; &nbsp; 3）表结构优化

### (2) 使用缓存

缓存主要有两类：

#### &nbsp; &nbsp; 1）本地缓存：不需要序列化，速度快，缓存数量和大小受本机内存限制。

&nbsp; &nbsp; Google guava cache

&nbsp; &nbsp; Ehcache

&nbsp; &nbsp; Oscache

#### &nbsp; &nbsp; 2）分布式缓存：需要序列化，速度相对本地缓存慢，但缓存数量和大小理论上不限（缓存机器可以扩展）。

&nbsp; &nbsp; Memcached

&nbsp; &nbsp; **Redis**

### (3) JVM优化

无论选择什么GC算法，Stop-the-world都是不可避免的，JVM调优的主要目的就是减少Stop-the-world的时间。

### (4) CDN服务器

浏览器直接向源服务器请求资源，一是距离可能较远，而是不同运营商之间跨网访问易拥堵，故有较大网络时延。

没有CDN服务器时，直接由DNS服务器解析服务器IP地址给浏览器，浏览器向该IP发送请求；有了CDN服务器，浏览器输入的域名会由CDN专用的DNS服务器解析，得到CDN负载均衡设备的IP，浏览器接着向负载均衡服务器发送请求，由负载均衡服务器选择最合适（最近/有相应内容/负荷较小）的缓存服务器为用户提供服务。

所以，只要将一些html、图片等静态资源缓存至CDN服务器即可减少网络时延。

## 3. 数据一致性

保证数据一致性主要是从事务和加锁两方面考虑（单是事务只能保证一个事务是原子性的，并不能保证并发场景下的数据安全，必须依赖数据库层面的加锁机制）。

（事务）

加锁分为三个层面上的：java代码层面、数据库层面、第三方插件

### (1) java代码层面

加synchronized关键词，锁住对象。在高并发场景显然会造成性能急剧下降，不采用。

### (2) 数据库层面（包括数据库和NoSQL：Redis）

#### [数据库锁](https://github.com/Yutoti/HighConcurrence/blob/master/Topic3/%E6%82%B2%E8%A7%82%E9%94%81%E4%B8%8E%E4%B9%90%E8%A7%82%E9%94%81)

&nbsp; &nbsp; 悲观锁：可实现分布式锁，但造成资源浪费，性能下降；

&nbsp; &nbsp; 乐观锁：降低锁开销，但无法避免脏读。

#### Redis实现分布式锁

### (3) 第三方插件：Zookeeper

# 三 各个解决方案的优缺点，适用场合

# 四 最终选定的方案
