# Redis

[toc]



## 一、概况

<a href="https://blog.csdn.net/u014453898/article/details/112292028">参考链接</a>

### 1、String

最大能存储512MB.

- API:

| 方法 | 使用示例               | 备注                                                 |
| ---- | ---------------------- | ---------------------------------------------------- |
| set  | set name "hello world" | set key value                                        |
| get  | get name(hello world)  |                                                      |
| incr | incr money             | 如果money非数值，则会报错。<br/>已经存储的值不受影响 |
| decr | decr money             | 同上                                                 |
| mget | mget name money        | 一次性获取多个value                                  |

### 2、 List

- API：

| 方法  | 示例            | 备注                     |
| ----- | --------------- | ------------------------ |
| lpush | lpush list1 cat | 从列表最左边插入一个元素 |
| lpop  | lpop list1      | 从列表最左边移除一个元素 |
| rpush |                 | 从列表的右边插入一个元素 |
| rpop  |                 | 从列表的右边插入一个元素 |
| llen  | llen list1      | 返回当前列表的的元素个数 |

### 3、Set

- API：

| 方法      | 使用示例           | 备注                      |
| --------- | ------------------ | ------------------------- |
| sadd      | sadd set1 sky      | 往set中添加数据           |
| srem      | srem set1 sky      | 从set中删除数据           |
| scard     | scard set1         | 查看set中存在的元素个数   |
| sismember | sismember set1 sea | 查看set中是否存在某个数据 |

### 4、hash

- API：

| 方法   | 使用示例              | 备注                             |
| ------ | --------------------- | -------------------------------- |
| hget   | hget hash1 name       | 通过key值，从hash里取对应的value |
| hset   | hset hash1 name julia | 向hash中添加key-value            |
| hmeget | hmeget hash1 name age | 一次性获取多个key-value          |



### 5、zset

- API:

| 方法      | 使用示例                                                     | 备注                       |
| --------- | ------------------------------------------------------------ | -------------------------- |
| zadd      | zadd database 5 redis<br/>zadd database 4 mysql<br/>zadd database 3 mongodb | 添加数据                   |
| zrem      | zrem database redis<br/>也可以删除多个：<br/>zrem database redis mysql | 移除元素                   |
| zcard     | zcard database<br/>返回： 3                                  | 查询元素个数               |
| zrange    | zrange database 0 2 withscores<br/>其中：<br/>withscores表示会同时返回分数。0 2表示排序区间是第0个到第2个元素<br/>显示：<br/>"mysql"<br/>3<br/>"mongodb"<br/>4<br/>"redis"<br/>5 | 数据排序，根据分数从小到大 |
| zrevrange | zrevrange database 0 2 withscores<br/>0 2表示排序区间是第0个到第2个元素<br/>显示：<br/>"redis"<br/>"mysql"<br/>"mongodb" | 数据排序，根据分数从大到小 |



## 二、存储结构

### 1. 存储结构概况

一个redisObject的信息包括：数据类型（type）、编码方式（encoding）、数据指针（ptr）、虚拟内存（vm）、其它信息……

<img src="C:\Users\PainKiller_7\Desktop\redis\redis-数据存储结构.png" style="zoom:120%;" />

- type: 用来标识存储数据的数据类型
- encoding: 用来标识type的底层数据结构的具体实现。
- ptr: 指向底层数据结构的指针。
- vm: 虚拟内存。

### 2. String

三种编码方式：int, raw, embstr

#### 2.1 int

当string对象的值全部是数字，就会使用int编码。
