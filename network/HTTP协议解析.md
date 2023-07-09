# HTTP协议解析

## 1. HTTP协议地址格式

   http是一种**无状态**协议。所谓无状态，就是不同的http请求之间，没有协议层面之间的关联。例如，你先登录某银行，然后又查看自己的银行账户余额。这两次请求在业务逻辑上是有依赖关系的。从逻辑上说，你得先登录，才能查到“你”的银行账户余额（而不是查不到余额或者查到别人的余额）。但是作为http请求，服务器并不认为它们两者之间有什么关系，除了路径、参数不同，怎么看都是一样的。

​    服务器如何识别一个http请求？答案是通过url。http请求的url格式如下：

​    `http://host[:port][/path]`

##  2. HTTP请求
[深入理解HTTP协议](https://zhuanlan.zhihu.com/p/45173862)

​    http请求由请求行、消息报头和请求正文三个部分构成。

\-----------   ----------------  -----------------  ------------------
\| Method | RequestUrl | Http-version | CRLF(换行)  |
\-----------   ----------------  -----------------  ------------------
\|                               消息报头                                   |
\-----------   ----------------  -----------------  ------------------
\|                     空行(只有CRLF换行)                          |
\-----------   ----------------  -----------------  ------------------
\|                               请求正文                                   |
\-----------   ----------------  -----------------  ------------------

### 2.1 请求行

​    请求行由请求`Method`, `URL` 字段和`HTTP Version`三部分构成, 总的来说请求行就是定义了本次请求的请求方式, 请求的地址, 以及所遵循的HTTP协议版本例如：

```text
GET /example.html HTTP/1.1 (CRLF)
```

先说`method`, HTTP协议的方法有：

- GET: 请求**获取**Request-URI所标识的资源
- POST: 在Request-URI所标识的资源后**增加**新的数据
- HEAD:  请求获取由Request-URI所标识的资源的**响应消息报头**
- PUT: 请求服务器**存储或修改**一个资源，并用Request-URI作为其标识
- DELETE: 请求服务器**删除**Request-URI所标识的资源
- TRACE: 请求服务器回送收到的请求信息，主要用于**测试或诊断**
- OPTIONS: 请求查询服务器的性能，或者查询与资源相关的选项和需求
- CONNECT: 保留将来使用

​    `URL`表示资源路径与格式，用来唯一定位一个资源或者说是数据。它可以是一张图片，一段视频，一个js文件。实际上，java程序员开发的后端HTTP接口，接口路径就是一个提供‘数据’资源的URL。

### 2.2 请求头

​    请求头可以成为消息报头，是由一些列键值对组成，允许客户端向服务端发送一些附加信息或者客户端自身信息，主要有：

​    

|     Header      |                             意义                             |                       示例                        |
| :-------------: | :----------------------------------------------------------: | :-----------------------------------------------: |
|     Accept      |                 指定客户端能够接受的内容类型                 |           Accept: text/plain, text/html           |
| Accept-Charset  |                  浏览器可以接受的字符集编码                  |         Accept-Charset: iso-8859-5,utf-8          |
| Accept-Encoding |      指定浏览器可以支持的web服务器返回内容压缩编码类型       |          Accept-Encoding: compress, gzip          |
| Accept-Language |                      浏览器可接受的语言                      |              Accept-Language: en,zh               |
|  Accept-Ranges  |           可以请求网页实体的一个或者多个子范围字段           |               Accept-Ranges: bytes                |
|  Authorization  |                    HTTP授权的授权证书类型                    | Authorization: Basic QWxhZGRpbjpvcGVulHNlc2FtZQ== |
|  Cache-Control  |                 指定请求和响应遵循的缓存机制                 |              Cache-Control: no-cache              |
|   Connection    |        表示是否需要持久连接(HTTP1.1 默认进行持久连接)        |                 Connection: close                 |
|     Cookie      | HTTP请求发送时，会把保存在该请求域名下的所有cookie值<br/>一起发送给web服务器 |            Cookie:$Version=1;Skin=new;            |
| Content-Length  |                        请求的内容长度                        |                Content-Length: 348                |
|  Content-Type   |                  请求的与实体对应的MIME信息                  |  Content-Type: application/x-www-form-urlencoded  |

### 2.3 HTTP请求报文

```
GET /a/path	HTTP/1.1
Host: google.com
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Accept: text/html, application/xhtml+xml, application/xml;q=0.9, image/webp,*/*;q=0.8
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0(Macintosh; Intel Mac OS X 10_10_0) AppleWebKit/537.36(KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36
Referer: https://segmentfault.com/u/_xian_zjun/articles
Accept-Encoding: gzip, deflate, sdch, br
Accept-Language: zh-CN, zh;q=0.8;en;q=0.6,ja;q=0.4
Cookie: mp_18fe57584af9657dea233
空行(CR+LF)
```

## 3. HTTP响应

​    响应的格式与请求相似：
\------------------   -----------------  ---------------------   ------------------
\| Http-Version | Status-Code | Reason-Phrase | CRLF(换行)  |
\------------------   -----------------  ---------------------   ------------------
\|                                      消息报头                                           |
\------------------   -----------------  ---------------------   ------------------
\|                              空行(只有CRLF换行)                                |
\------------------   -----------------  ---------------------   ------------------
\|                                       响应正文                                          |
\------------------   -----------------  ---------------------   ------------------

​    HTTP响应也由三个部分组成，包括状态行、消息报头、响应正文。

### 3.1 响应状态行

​    HTTP协议版本，状态码以及对应状态码的文本描述

```text
HTTP/1.1 200 OK (CRLF)
```

### 3.2 HTTP响应状态码

​    一般由三维数字组成。第一个数字定义相应的**类别**，且有五种可能得取值:

- 1XX: 指示信息， 表示请求已经接受，继续处理，不常使用

- 2XX: 成功，表示清气已被成功接收、理解、接收

- 3XX: 重定向，要完成请求必须进行更进一步的操作

- 4XX: 客户端请求错误，请求有语法错误或请求无法实现

- 5XX: 服务端错误，服务器未能实现合法的请求

  9414

​    需要注意，响应状态码除了少数几个标准状态码外，各个机构、组织、应用可能定义自己的。例如http响应状态码499就是nginx定义的。详细的标准状态码及其含义此处不再列出，可以自行搜索。

### 3.3 HTTP响应报文


```
HTTP/1.1 200 OK

Date: Mon, 30 June 2023 10:29:61 GMT
Content-Type: text/html;charset=UTF-8
Transfer-Encoding: chunked
Expires: Mon, 31 June 2023 10:30:44 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
X-Hit: sf-web2
Content-Encoding: gzip
Strict-Transport-Security: max-age=15768000; preload

空行(CR+LF)

<html>
......
```

## 4. HTTP协议特点

   HTTP有如下特点：

- 支持客户/服务器模式。即一个请求是**由客户端**发起，并由服务器返回一个响应。
- 无连接。其含义是限制每次连接只处理一个请求，待服务器处理完一个请求并收到客户端应答后断开连接。不过现在通过`Connection: Keep-Alive`实现长连接。
- 无状态。指协议对于事务处理没有记忆能力。缺少状态就意味着如果后续处理需要前面的信息，则必须再重传。好处是不需要先验信息可以快速应答；坏处是导致每次连接传送的数据量增大。

