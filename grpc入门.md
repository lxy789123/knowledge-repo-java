# GRPC入门

[TOC]

## 1.背景

​    在了解`GRPC`之前，我们先认识一下什么是`RPC(Remote Procedure Call)`。

​    对于Java微服务架构的应用来说，调用一段‘代码’可以分成两种情况。一种是这段‘代码’位于同一个应用中，直接通过`对象名.方法名`或者`类名.方法名`调用；另一种情况就是应用相互隔离，此时需要通过http调用来实现‘代码’调用。

​    回忆一下我们完成一次http调用需要做的准备工作：

- URL。包括`域名/ip`,`端口`(可能有), `路径`等。
- HTTP METHOD。 包括`POST`/`GET`等。
- HTTP连接相关参数。如`超时时间`等。
- 请求与响应结构的构造。请求响应的结构必须调用者自行构造，如果结构或者字段名称不正确，则在请求时会报错。

​    以上的工作，你需要针对每一次http调用都完成一次设置。为了隐藏这些调用细节，让远端调用“看起来”更像一个本地调用，RPC通常会提前规定好相关“协议”，即上文提到的连接参数等信息，让用户只关心需要提供/调用的方法本身。

​	除此之外，RPC框架往往还有服务注册与发现、熔断限流与降级等服务治理功能，让框架的使用者摆脱自行构建这些服务的麻烦，让RPC看起来更‘本地’。

## 2.gRPC远程调用方式

​    了解了RPC，我们再来看看什么是`gRPC`。以下内容翻译自[gRPC官方介绍](https://grpc.io/docs/what-is-grpc/introduction/)。
```
在gRPC中，一个客户端应用可以直接以调用本地方法的方式对位于不同机器的服务端应用的方法进行调用。这能让你更轻松地创建分布式应用和服务。和其他RPC系统一样，gRPC也是一个明确了方法的参数与返回的预定义服务。在服务端，服务实现接口并处理gRPC客户端调用。在客户端，客户端拥有一个存根(stub)，该存根提供和服务端相同的方法。
```

​    gRPC的调用方式如图所示：

![grpc结构](.\attachment\[grpc]struct.png)

从图中可以看出gRPC的一些特点：

- 跨语言。目前gRPC已经支持数种语言。
- 使用框架自定义的通信协议`Protobuf(Protocol Buffers)`，该协议目前已经更新到3.0版本(Proto3)。 Q: 该协议属于哪个层次的协议？
- 客户端通过`Stub(存根)`访问服务。

## 3. 开启gRPC之旅

​    本文不会涉及gRPC框架的java项目搭建的细节，读者可以阅读从gRPC官方网站下载相关demo。(请查看： [gRPC首页](https://grpc.io/docs/what-is-grpc/); [gRPC官方仓库](https://github.com/grpc/grpc-java))

### 3.1 定义Protobuf文件

​    要提供或者调用一个gRPC接口，首先要做的就是定义一个`Protobuf`文件。这个文件就是一个gRPC接口的“身份证”，称为`IDL(Interface Definition Language)`——通过它才能区分不同的gRPC的接口，以及接口到底有什么参数。Protobuf文件可以分为四个部分：

- 服务
- 方法
- 请求类型
- 响应类型

```protobuf
/*
* HelloWorld入门示例
*/

//使用proto3语法
syntax = "proto3";

//proto包名
package hello;
//生成多个Java文件
option java_multiple_files = true;
//指定Java包名
option java_package = "com.psbc.cpufp.grpc.hello";
//指定Java输出类名
option java_outer_classname = "HelloProto";

//gRPC服务定义
service Hello {
  //gRPC服务方法定义 - Unary
  rpc sayHello (HelloRequest) returns (HelloReply) {}

  //gRPC服务方法定义 - Server Streaming - 服务端流
  rpc sayHelloServerStream (HelloRequest) returns (stream HelloReply) {}

  //gRPC服务方法定义 - Client Streaming - 客户端流
  rpc sayHelloClientStream (stream HelloRequest) returns (HelloReply) {}

  //gRPC服务方法定义 - BiDirection Streaming - 双向流
  rpc sayHelloBiStream (stream HelloRequest) returns (stream HelloReply) {}
}

//请求参数定义
message HelloRequest {
  string name = 1;
}

//响应结果定义
message HelloReply {
  string message = 1;
}
```

​    gRPC支持的方法类型就像上面定义的那样：

- **Unary**: 一元的RPC，单独的request和response。
- **Server streaming**： 服务端流，仅发送一个request，然后由server返回response流（源源不断的response）直到再无response，gRPC会保证消息顺序。
- **Client streaming**: 客户端流，发送request消息流知道再无request，然后server端仅返回一个response消息，gRPC保证消息顺序。
- **Bidirectional streaming**: 双向流，双向读写，且request和response流可以各自独立保持消息顺序。例如：server可以接受全部request后才同意发送response，或者街道一条request后发送response，又或者是其他任意组合。也就是说，双方的request和response发送可以在任意时刻，没有限制。

### 3.2 完成代码填充

​    定义完成后，使用idea上的`protobuf`插件，它根据该文件生成代码文件。

![proto](.\attachment\[grpc]proto.png)

- `HelloGrpc`为gRPC服务的代码定义，其中包括`HelloGrpc.HelloImplBase`，**服务端**继承实现该类以提供对应的具体的业务逻辑；`HelloGrpc.newBlockingSub`和`HelloGrpc.newSub`分别为**客户端**提供接口的阻塞和非阻塞调用API。
- `HelloRequest`为定义的请求类，`HelloReply`是定义的响应类。`xxxOrBuilder`为用户提供了对应类的`Builder`，这些接口在代码生成的时候就会被对应的参数类实现。

​    需要说明的是，每个方法并不是强制把四种形态的API都定义完整，用户根据需要定义即可。

## 4. gRPC的软肋

​    gRPC目前最大的问题就是不原生支持**服务治理**。

​    对比成熟的`dubbo`框架。`dubbo`提供了一个`Registry`即`注册中心`，当服务启动时，将向注册中心注册本应用下的所有服务；同时客户端向注册中心订阅服务信息，以便注册中心可以及时将服务列表变更情况同步到客户端。客户端再从列表中选择一个服务端地址进行访问。

​    虽然`dubbo`和`gRPC`的调用都是点对点的直接调用（即请求不会通过第三个应用转发），但dubbo注册中心的存在使得所有的客户端和服务端都不需要在服务启动前就得知所有的服务提供信息。gRPC则不行，同一个服务（指相同的IDL），一个客户端只能指定一个address。是的，你没有看错！地址只能指定一个address，也就是说，如果要完成负载均衡，需要自行单独搭建相关应用，不支持客户端一次性配置。

## 5. 总结

   本文介绍了gRPC基本结构及创建服务，请求服务的关键步骤。