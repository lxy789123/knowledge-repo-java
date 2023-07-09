# HTTPS从理论到应用

[TOC]

## 一、没有HTTPS的世界

​	在正式介绍HTTPS之前，先复习下网络协议的五个层次：物理层、数据链路层、网络层、传输层、应用层，后面三层是软件开发人员比较熟悉的。其中网络层协议的代表是IP(Internet Protocol)协议；传输层的代表是TCP (Transmission Control Protocol) /UDP(User Datagram Protocol)协议；应用层的协议就比较五花八门，比较公共且具有代表性的有HTTP(Hypertext Transfer Protocol), SMTP(Simple Mail Transfer Protocol)和FTP(File Transfer Protocol)三个协议。

​    位于应用层的HTTP协议仅仅解决了“温饱”——完成数据传输。但http协议本身并不能保证“安全”，请求与响应的内容都以**明文**形式传输，不利于保密工作开展！信息交互的双方当然可以自行

将报文加密，然后将密文明文发送。但对于服务提供方来说非常不方便——难以要求所有人都遵循自己的加解密规范，密钥的管理、发放都是问题。于是在业务数据之外的协议层面开展统一的加密就显得十分有必要。https协议就是在这个需求之下诞生的。

​	那么https协议究竟位于五层协议哪一层呢？答案是应用层。这里又引申出另一个问题，如果https也在应用层，那么**https和http的关系**是什么呢？如果它们相互独立，也就意味着使用https时可以不使用http；如果https依赖于http，意味着有https存在的地方，就一定会使用http。对于这个问题，相信你读过本文后能给出自己的解答。


## 二、从零开始创造HTTPS

​    `HTTPS`这个名称其实就是`HTTP Secure`的缩写。本质上就是`HTTP`+`SSL(Secure Sockets Layer)`/`TLS(Transport Layer Security)`。其中又可以将`TLS`看做`SSL`的演进版本。之前我们已经说到了网络协议的五个层次，并说明了HTTPS和HTTP一样同处应用层，下图展示了它们之间的关系。

\----------------                     \---------------- 
\|      http     \|                     \|      http     \|
\----------------                     \|     ssl/tls   |
\|      tcp      \|                     \----------------
\----------------                     \|      tcp      \|
\|       ip       \|                     \----------------
\----------------                     \|       ip       \|
\|     mac     \|                     \----------------
\----------------                     \|     mac     \| 
                                        \---------------- 

​    TLS其实是一系列子协议组成的一个密码套件。密码套件的命名规范，格式固定，如，`ECDHE-RSA-AES256-GCM-SHA384`：`密钥交换算法-签名算法-对称加密算法-摘要算法`。

​	既然HTTPS协议的作用是信息加密，那么它具体是如何建立，又如何保护你的请求的呢?接下来我们开始进行一次技术探险，尝试从零开始搭建一个可行的加密方案。

**【对称加密】**

​    对称加密就是解密和加密使用同一个密钥的加密方法。理论上讲，如果相互通信的两方，在线下提前约定好密钥，并好好保存，那么通信就是安全的。

​    但互联网上的服务供应商形形色色，数量多如繁星，用户不可能提前和每一个服务提供商约定好密钥。 因此密钥只能在**使用时传递**。如果密钥是明文传递，那么加密就失去了意义。如果用另一个密钥给使用密钥加密，那么只是给密钥使用增加一个步骤而已。就好像一把锁有一把钥匙，为了不让钥匙被盗或者遗失，又把这把钥匙放进另一个抽屉。可另一个抽屉的钥匙怎么保存又成了问题。

​    因此，使用对称加密不能保证安全。

**【非对称加密】**

​    非对称加密的加密和解密使用两个不同的密钥，一个公开，一个私密保存，分别称为公钥和私钥。公钥加密的内容只有私钥才能解开，反过来，私钥加密的内容只有公钥可以解开。非对称加密好像可以解决安全问题——解决一边，即**客户端**向**服务端**发送信息的安全问题被解决了。那我们可以给客户端也准备相应的公钥私钥是否可行呢？

​    从理论上讲，答案是可行的。不过我们并没有将这种想法投入到现实中，因为`非对称加密`的计算量相较于`对称加密`大得多，既会大量消耗系统计算资源，耗时又十分长，这对客户端和服务端都是十分差劲的体验。

**【非对称加密+对称加密】**

​    我们可以考虑使用`非对称加密`来传递`对称加密`的密钥，这样只有第一次传递的时候需要使用大量资源，之后的每次普通通信都是对称加密。这种方案兼顾了安全和性能，就是我们的解决方案，了吗？

**【中间人攻击】**

​    假设我们目前就使用【对称+非对称】的这种方案，有一个`中间人`从中作梗，会发生什么呢？

​    首先，在服务端向客户端发送公钥A的时候，中间人将公钥A截获，并将其替换成自己的公钥B。

```
server ======[明文公钥A]=======> 中间人 ======[明文公钥B]======> client
```

​    然后client发送使用的**公钥B**加密的对称加密密钥被中间人截获，并替换成中间人用自己的私钥B加密的对称密钥。

```
client =======[公钥B加密的对称密钥X]======> 中间人(解密得到对称密钥X) ======[公钥A加密的对称密钥X]======> server
```

​    之后服务端和客户端之间用对称密钥加密的信息，中间人都可以轻松获取。

​    出现这种问题的根本原因是，客户端无法确定收到的服务端公钥是否是真实的，或者说能否“人证合一”。

​    中间人攻击之外，还可能出现另一种问题：否认问题。即由于中间人的存在，数据的发送方否认数据由自己发送。

**【数字证书】**

​    为了解决信任问题，我们引入了可信的第三方机构**CA(Certificate Authority)**来颁发数字证书。CA机构就像数学中的公理一样，是“不证自明”的。由它们颁发证书给服务供应商，并在具体的业务请求中将证书发送给客户端，以验明正身和保证信息不被篡改。

​    证书的生成及使用过程如下：

1. 证书申请方（一般为服务提供方）将自己的机构、公钥等信息发送到CA。CA将这些信息以**明文**形式写在证书上，然后将这些明文信息经过散列计算后，利用自己（CA机构）的**私钥**将散列后的信息加密，再追加在证书的明文信息之后当做签名。
2. 客户端安装CA机构根证书。通常浏览器出厂就会自带一些常用的CA机构的证书。
3. 客户端向服务端发送请求，服务端返回证书。
4. 客户端利用CA机构的公钥解密证书上的密文，并将它同证书上的明文信息散列计算得到的散列信息对比看是否一致。如果一致，则认可该服务端，并继续后续的https协议握手。

​    由于有CA机构的保证，证书本身及证书内容不可能作假；而证书上又写了证书申请机构的信息，因此证书也无法被其他机构挪用。

​    **【总结】**

​    本节内容主要讲述使用数字证书的必要性，也介绍了在实际业务中，数字证书的实际使用方式。对于为何HTTPS协议需要数字证书以及为何需要对称-非对称方式进行密钥通信，读者已经有了相当的了解。

​    通过对HTTPS整个“升级”历程的探索可以发现，整个HTTPS协商周期中，涉及到使用算法有：对称加密算法、非对称加密算法、信息摘要算法

## 三、HTTPS握手流程

​    虽然有了HTTPS协议，但是很多问题依然没有解决。比如很多已有服务依然在使用HTTP协议，不能及时完成转换或者没有必要使用加密协议，双方如何完成HTTPS协议的协商；HTTPS的TLS或者SSL的版本很多，如何确定具体使用的版本；双方如何确认已经协商好的协议细节被正确传递了（如对称密钥是否正确传递）。

​    为了完整的展示这些细节，本文将以一个实例为切入点，方便和读者共同学习这些实现的细节。以下介绍将以单边认证（仅认证服务端身份）为主，双边认证作为补充。

​    1、三次握手

​    我们先来看下三次握手的标准流程是什么：

- 客户端发送建立连接请求：SYN=1, seq=J
- 服务端针对客户端的SYN确认应答并请求建立连接：SYN=1, ACK=1, ack=J+1, seq=K
- 客户端针对服务器端的SYN的确认应答：ACK=1, ack=K+1

​    其中使用`UpperCase`书写的字段(`SYN`与`ACK`)只有0和1两个状态。

​    在我们的实例可以看到客户端请求的发起端口是`24077`，服务器的服务端口是`9898`。

<p align="center">​    
<img src=".\attachment\[HTTPS]三次握手.png" alt="[HTTPS]client_hello" style="zoom:60%;"  />
</p>


​    上图展示的三次握手流程中，`24077`先发送seq=0，SYN=1；`9898`随后回复SYN=1，ACK=1, ack=1(实际上就是上一个seq+1), seq=0; `24077`再回复ACK=1, ack = 1 (上一个seq+1)。整个过程和标准流程一致。值得一提的是三次握手是TCP协议的流程，因此整个过程都是TCP层面的交互，从`Protocol`一栏可以看出。

2、客户端发送client_hello

<p align="center">​    
<img src=".\attachment\[HTTPS]client_hello.png" alt="[HTTPS]client_hello" style="zoom:60%;"  />
</p>


​    其中，`Version`为使用的TLS版本(此处为1.2)；`Random`随机数，用于用户后续的**密钥协商**，暂且记为Rc；`Cipher Suites`加密套件候选列表；`Compression Method`压缩算法候选列表；`Extension`为扩展字段；`applicaito_layer_protocol_negotiation`应用层协议协商，这里的值表明客户端后续希望使用http/2协议。

​    加密套件需要拎出来单独**提一下。在第二节，我们展示了一个加密套件`ECDHE-RSA-AES256-GCM-SHA384`，它是一种可用的组合。在`client_hello`中，`Cipher Suites`字段会罗列出**客户端所有可用的算法组合供服务端选择。

3、服务端发送server_hello

<p align="center">​   
<img src=".\attachment\[HTTPS]server_hello.png" alt="[HTTPS]server_hello" style="zoom:60%; text-align: center;" />
</p>

​    首先需要说明一下，图中展示的`Server Hello`，`Certificate`, `Server Key Exchange`, `Server Hello Done`并不要求必须在同一个报文中发送。协商如此进行和具体的实现相关。如果读者自行使用`wireshark`抓包时发现报文内容与本文图示不一致也不用觉得惊讶，这是正常的。

​    `Server Hello`发送的主要内容包括：`Version`选择使用的TLS协议版本；`Random`随机数，用于后续**密钥协商**，暂且记为Rs；`Cipher Suite` 服务端最终从客户端提供的加密套件列表中选择的加密套件，此处就是`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`，其中`Tls`表示具体加密协议类型，`ECDHE`表示**密钥协商算法**，`RSA`是证书中用于签名信息加密的算法，`AES-128-GCM`是对称加密算法，`SHA-256`是证书中用明文得到散列值的算法。`Compression Method`压缩算法，此处没有选择压缩算法，即没有压缩。

4、服务端发送Certificate

<p align="center">​   
<img src=".\attachment\[HTTPS]certificate.png" alt="[HTTPS]server_hello" style="zoom:80%; text-align: center;" />
</p>
​    可以看到证书的报文长度是783。

5、服务端发送Server Key Exchange

<p align="center">   
<img src=".\attachment\[HTTPS]server key exchange.png" alt="[HTTPS]server_hello" style="zoom:80%; text-align: center;" />
</p>

​    这一阶段的主要用途就是发送经过**服务器私钥加密**的公钥`Pubkey`（暂且记为Ps）和椭圆曲线名称`x25519`给客户端。这里的公钥是服务方临时(ephemeral)生成的（公私钥对中的），和CA办法的数字证书中的公钥往往不是同一个。

6、服务器发送Server Hello Done

<p align="center">   
<img src=".\attachment\[HTTPS]server_hello_done.png" alt="[HTTPS]server_hello" style="zoom:80%; text-align: center;" />
</p>

​    通知客户端server_hello信息发送结束。

7、客户端发送Client Key Exchange, Change Cipher Spec, Encrypted Handshake Message

<p align="center">   
<img src=".\attachment\[HTTPS]client_key_exchange.png" alt="[HTTPS]server_hello" style="zoom:60%; text-align: center;" />
</p>

​    

​    可以看到，此处的`client_key_exchange`层级下也表明了**密钥协商算法**类型为`EC Diffie-Hellman`，客户端给出的使用服务端公钥加密的`PubKey`（暂且记为Pc）和Ps一样，也是临时生成的。

​    在前一节中我们提到'使用非对称加密来传递对称加密算法的密钥'时是这样描述的：客户端用服务端的公钥加密一个私钥， 然后服务端通过私钥解密后获取密钥。这样的描述给了我们一种错觉，仿佛对称密钥是由客户端单方面决定，然后经过加密后'通知'客户端的。这样的描述是为了简化抽象整个协商流程而隐藏了一些协商的细节。如果就采用客户端单方面决定对称加密密钥的方案，会导致什么问题？既无法保证本次通信的安全，还可能危害之后通信的安全。

​    通过之前的分析我们知道，非对称加密只会在https协商阶段使用，之后的正常通信阶段都使用对称加密。试想这种情况：攻击者截获了发送**对称加密密文**的报文，然后替换成自己的对称加密密钥，此时整个非对称加密都会变成'裸奔'——攻击者完全不需要对之前的非对称加密阶段做任何手脚，只等着对称加密的密钥发出时截获修改即可，之前所做的一切前功尽弃。服务端公钥是公开的，虽然攻击者不能解密，但是使用公钥加密却是可以的，服务方也没有办法辨认这个密钥来源的真实性，此次通信的安全性岌岌可危。

​    另一方面，如果允许客户端单方面决定密钥。一旦泄露一次对称密钥，之后中间攻击者伪装成该客户端发送密钥重设请求，之后的通信都会变‘透明’。这对**前向安全性**(Forward Secrecy)造成威胁。前向安全性也称为未来保密性或密钥自动生成，是指在当前通信中使用的密钥泄露后，以后的通信内容仍然保持机密性。因此，https使用了`密钥协商`的机制。

​    https中的`ECDHE`密钥协商的算法原理我们在附录中详细介绍，此处隐藏算法细节，仅做大概说明。之前说过，双方已经相互交换了Rc，Rs，Pc，Ps。该机制让客户端和服务端分别利用前文已经出现的Pc、Ps和算法`ECDHE`计算出一个称为`pre-master secret(预主密钥)`的随机值，然后再利用Rs、Rc、pms三者和伪随机算法`PRF(pseudo-random function)`计算出48字节（384位）的`master secret（主密钥）`，最后再利用PRF算法传入Rs、Rc、ms得到由数个对称密钥。

​    到此就完成了共享密钥的交换。对称密钥需要每次协商，保证了密钥的更换，即使某一次密钥泄露，也不会影响到之后的通信。同时，使用双方协商保证了不会有攻击者通过更换单方面密钥的方式来实现重放攻击。

​    `change_cipher_spec`是一种密钥规格变更协议，表示客户端已经生成加密密钥，并切换到加密模式，并通知服务端后续的通信都采用协商的通信密钥和加密算法进行通信。

​    `encrypted_handshake_message`是将所有握手数据做一个摘要，再用最后协商好的对称加密算法对数据加密，通过该消息发送到服务器进行校验，以测试这个对称加密密钥是否成功。

8、服务端发送Change Cipher Spec, Encrypted Handshake Message

​    `change_cipher_spec`说明服务端解密客户端发送的参数，然后按照前述方法计算出了会话密钥，并通过客户端发送的`encrypted_handshake_message`验证其有效性。待验证通过后，发送报文通知客户端，后续可以使用此密钥进行通信。

​    `encrypted_handshake_message`同样是为了测试密钥的有效性。客户端发送的报文是为了服务端能够验证密钥可以用来解密，客户端能够正常加密；反过来就是验证服务端密钥可以加密，客户端可以解密。

​    此步骤结束后，https的协议协商正式结束，可以开始传递业务数据报文了。

​     


## 四、眼花缭乱的HTTPS证书文件

​    上文我们详细阐释了https的保密原理、协商过程。但还有一个重要的问题——在我的服务中，我该怎么开启https，需要做哪些工作。

​     本节我们将从各种证书文件出发，介绍这些文件的产生过程与作用。

### 4.1 证书格式

​    在看具体的格式之前，先介绍两种编码方式：`PEM`和`DER`。

- PEM: Privacy Enhanced Mail. 以文本形式存储，使用Base64编码，以`------BEGIN XXX------`开始，`------END XXX------`结尾。可以存储公钥、私钥、证书签名请求。PEM格式的文件，后缀可以为`.pem`。
- DER: Distinguished Encoding Rules, 二进制，可以用软件`OPENSSL`查看其内容。DER格式的文件，其后缀可以是`.der`。

​    这两种格式的证书可以相互转换。

### 4.2 扩展名

​    扩展名才是重点，因为它们的类型十分繁多。我们先简单罗列说明，然后通过一个生成实例来说明他们的作用和差别。

|   扩展名   | 属性                        | 作用                                                         |
| :--------: | --------------------------- | ------------------------------------------------------------ |
|    .pem    | PEM编码格式文件             |                                                              |
|    .der    | DER编码格式文件             |                                                              |
|    .crt    | certificate                 | 常见于UNIX，大概率是PEM编码，也可以是DER                     |
|    .cer    | certificate                 | 常见于windows, 可以将其视为crt                               |
|    .p12    | PKCS#12                     | 也作.pfx。是公钥加密标准(PUBLIC KEY CRYPTOGRAPH STANDARDS)<br/>系列的一种可以理解为由X.509证书+私钥组成 |
|    .csr    | Certificate Signing Request | 证书签名请求，并非证书格式，而是服务器向CA获得签名证书的申请。<br/>内容为 RSA公钥+附带信息。生成csr时会生成对应私钥及机构信息。私钥应该严格保密。 |
|  keystore  |                             | 通常存储一个RSA公钥或私钥，非X.509证书格式，编码可以是PEM或DER |
| truststore |                             | 通常用来存储CA机构的根证书                                   |

​    `.pem`和`.der`本身是编码格式，其实不表示文件的业务类型。但你依然可以把生成的`.p12`,`crt`等扩展名的文件命名为`.pem`，这不会影响使用。从另一个角度说，当你看到数个`.pem`扩展文件时，它们可能只是有相同的编码格式，但文件作用、内容都完全不同。

​    `keystore`和`truststore`比较特殊，它们都是'容器'，通常在java web应用中用到。`keystore`往往用来存储客户端或服务端的

`crt`文件和私钥打包而成的本机https证书信息。`truststore`则是本机信任的ca机构的相关信息。

### 4.3 生成自己的证书

​    本节将示范手动生成你自己的https相关证书(自签证书)的过程。根据生成命令中指明的依赖关系，可以清楚的看到各种扩展文件的性质和作用。

​    **OPENSSL**

1. *生成根证书的私钥。*

​    这个根证书就是CA机构的根证书`root_private_key.pem`。因为是自我认证，因此CA机构就是自己。

​    Q: 回想上文中的描述，哪里需要用到CA机构的密钥？

    ```
openssl genrsa -out root_private_key.pem 2048
    ```

2. *生成根证书`root.crt`*

​    可以看到生成根证书需要用到`root_private_key`信息。

```
openssl req -x509 -new -key root_private_key.pem -out root.crt
```

3. *生成客户端的私钥*

​    客户端密钥生成不依赖其他文件。

```
openssl genrsa -out client_key.pem 2048
```

4. *生成客户端的csr文件*

​     生成的csr文件用来请求CA机构发放证书。需要将密钥`client_key.pem`信息写入`csr`文件。可以看到，私钥的生成和csr文件的生成都不用依赖CA机构——私钥本身就是私密信息，csr文件则是一个正式申请认证的‘申请书’，自然其本身的生成仍然和CA机构无关。

```
openssl req -new -key client_key.pem -out client.csr
```

5. *用CA机构根证书来签发客户端请求文件，生成客户端证书`client.crt`*

​    可以看到，这里为了生成crt文件，我们使用了客户端证书签名请求文件`client.csr`, CA机构根证书`root.crt`, CA机构私钥`root_privat_key.pem`。请注意，这里需要指明申请签发证书的机构的域名，此处为`localhost`，因为是我们自用，因此这里使用一个我们在测试中会请求的接口的域名。

```
openssl x509 -req -in client.csr -CA root.crt -CAkey root_private_key.pem -CAcreateserial -days 3650 -out client.crt -subj "/CN=localhost"
```

6. *生成服务端私钥*

```
openssl genrsa -out server_key.pem 2048
```

​    和3一致，不赘述。

​    这里为了使用，我们从服务端私钥中提取公钥供客户端使用。只需要私钥信息就可以生成相应的公钥。

```
openssl pkey -in server_key.pem -pubout -out server_public_key.pem
```

​    为了兼容spring应用对私钥格式的要求，将服务器私钥格式转换为`pkcs8`。`pkcs12`和`pkcs8`的区别在于``pkcs12``主要用于打包和传输多个密钥和证书，而`pkcs8`主要用于表示私钥的格式。

```
openssl pkcs8 -topk8 -inform PEM -outform PEM -nocrypt -in server_key.pem -out server_key_pkcs8.pem
```

7. *生成服务端证书的签名请求文件*

​    作用同4，不再赘述

```
openssl req -new -key server_key.pem -out server.csr
```

8. *用CA机构根证书来签发服务端请求文件，生成服务端证书`server.crt`*

```
openssl x509 -req -in server.csr -CA root.crt -CAkey root_private_key.pem -CAcreateserial -days 3650 -out server.crt -subj "/CN=localhost"
```

​    参见5，不赘述。

9. 打包客户端证书资料为pkcs12格式

​    需要输入密码，请记住。其中含有客户端证书、私钥。该文件的别名为undertow，可以自定义。

```
openssl pkcs12 -export -in client.crt -inkey client_key.pem -out client.p12 -name undertow
```

10. 打包服务端证书资料为pkcs12格式

​    同客户端

```
openssl pkcs12 -export -in server.crt -inkey server_key.pem -out server.p12 -name undertow
```

​    **keytool**

11. 生成客户端keystore

​    使用keytool的importkeystore指令。pkcs12转jks，需要p12密码和jks密码。

```
keytool -importkeystore -srckeystore client.p12 -destkeystore client.jks -srcstoretype pkcs12
```

12. 生成服务端keystore

​    同11.

```
keytool -importkeystore -srckeystore server.p12 -destkeystore server.jks -srcstoretype pkcs12
```



​    至此，我们的证书都已经生成完毕了，接下来我们看看怎么具体的使用它们。

### 4.4 在项目中使用

​    这里以一对grpc服务为例。grpc客户端想要访问grpc服务。这里读者可以不关心什么是`GRPC`，就把它看做是一对需要配置https证书的服务端与客户端就行了。

​    先来看客户端的配置文件

```yaml
grpc: 
    client: 
    	GLOBAL:
          security:
            client-auth-enabled: true
            trust-cert-collection: file:src/main/resources/certificate/root.crt
            key-store: file:src/main/resources/certificate/client.p12
            certificate-chain: file:src/main/resources/certificate/client.crt
            private-key: file:src/main/resources/certificate/client_key.pem
            protocols: TLSv1.2
```

    -  `file:`前缀表示配置的值表示一个文件。
    -  `trust-cert-collection`：就是`truststore，`用来存储CA机构根证书。它的生成方式同`keystore`，只是内容不同而已。如果需要数个CA机构根证书，把他们一起打包成一个`jks`文件就行。此处只有一个我们自己的CA机构，因此可以直接使用`.crt`根证书。
    -  `key-store`:  见名知意。这里是客户端的keystore文件。
    -  `certificate-chain`: 客户端证书。如果是单向验证，该字段可以不配。https可以支持单向验证，即客户端验证服务端；也支持双向验证，即客户端和服务端都需要证书证明自己的身份。                                                                                                                                                                                                                                                                                                                                                                    
    -  `private-key`: 见名知意。
    -  `protocols`: 指明`tls`协议版本号。

​    再看服务端文件：

    ```yaml
grpc:
  server:
    port: 9898
    enableKeepAlive: true
    keepAliveTimeout: 20s
    security:
      enabled: true
      private-key: file:src/main/resources/certificate/server_key.pem
      certificate-chain: file:src/main/resources/certificate/server.crt
      protocols: TLSv1.2
server:
  ssl:
    enabled: true
    key-store-password: 123456
    key-store-type: PKCS12
    key-store: src/main/resources/certificate/server.p12
    key-alias: undertow
    ```

​    除了`keysotre`信息更全面之外，没有其他新增的配置。

​    读者在阅读本节的实例的时候，没必要纠结于`GRPC`这样配，其他应用该怎么配这样的问题。不同的框架、组件对于同样的证书文件，配置的名字和路径可能不完全相同，甚至完全不相同。此时要想把‘萝卜’准确的放进‘坑’里，可以依靠搜索引擎和组件的官方文档。

​     本节想让你了解的内容：

- 证书的各种各样的内容、格式、后缀的意义。
- 在实际应用中，我们该怎么生成自己的证书。在生产环境几乎不可能让你自己生成任何证书文件，但这对自己调试代码的作用很大。
- java web应用中如何配置这些文件。熟练各种https证书的格式、内容，结合框架、中间件的文档，最后，一点一点尝试，一定能配好这些文件。



## 附录

### A. Diffie Hellman 密钥算法简述

TBC

