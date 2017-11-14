# 前端基础知识篇（一）
<span style="position: absolute; left: 160px; font-size:20px;">—— Http协议及Restful设计架构</span><br />

## 1. Http协议

Http协议，从服务端到浏览器端的传输协议。它属于应用层（OSI第七层）的面向对象的协议。

### 1.1 主要特点
- 简单快速：请求服务时，仅需传递请求方法（POST/GET等）和URL。规模小，通信快。
- 灵活：HTTP允许传输任意类型的数据对象，类型用Content-Type加以标记。
- 无连接：即每次连接服务端仅处理一个请求，处理完后，服务端收到客户端应答即断开连接，用于节省传输时间。
- 无状态：HTTP协议是无状态协议，对于事务处理没有记忆，若本次请求需要前面的信息则必须将数据同本次数据一同传回。这种情况会导致一次请求数据量变大，所以应尽量避免。如果不需要前置信息处理本次信息，意味着服务端每次处理应答就较快。
- 支持B/S和C/S模式

### 1.2 请求报文格式

- 请求行：方法符号开头，使用空格分隔。格式如下：Method Request-URI HTTP-Version CRLF
    - Method
        - GET：请求获取Request-URI所标识的资源
        - POST：在Request-URI所标识的资源后附加新数据（新建）
        - PUT：请求服务器存储一个资源，并用Request-URI作为其标识（全部更新）
        - PATCH：局部更新
        - DELETE：删除
        - HEAD：返回Request-URI所标识资源的响应消息报头
        - OPTIONS：查询服务器性能，或者查询与资源相关的选项或需求
        - CONNECT：保留将来使用
        - TRACE: 请求服务器返回收到的请求信息，用于测试和诊断
    - Requset-URI ：资源的唯一标识符
    - HTTP-Version：HTTP协议版本
    - CRLF：回车和换行，除了结尾使用外，不允许单独出现CR或LF字符
- 请求报头
- 请求正文

### 1.3 响应报文格式

- 状态行：HTTPVersion StatusCode Reason-Phrase CRLF
    - StatusCode：返回的状态码，如404 302 304 200等
    - Reason-Phrase：状态码的文本描述
- 响应报头
- 响应正文：服务器返回的资源内容

### 消息报头
报头信息典型的key:value形式组成，且key不区分大小写

- 普通报头：只用于传输消息，不用于传输实体。（就了解了大概，详细需翻阅相关资料）
- 请求报头：由客户端向服务端传输自身信息及附加信息
    - Accept：接受的类型（image/gif，text/html等）
    - Accept-Charset：接受的字符集（iso-8859-1,gb2312等）
    - Accept-Encoding：接受内容的编码
    - Accept-Language：接受的自然语言
    - Authorization：授权查看某个资源
    - Host：主机和端口号
    - User-Agent：客户环境信息（如操作系统，浏览器等等，主要用于用户分布，行为分析等），非必需
- 响应报头
    - Location：用于更换域名使用，作用为重定向接受者到一个新的位置
    - Server：服务器用来处理软件请求的相关信息，与请求报头中的User-Agent对应
- 实体报头
请求和相应消息都可以传送一个实体，一个完整的实体有实体报头和实体正文组成，可以单独发送实体报头。实体报头定义了关于实体正文（有无实体正文）和请求所标识的资源的元信息。
    - Content-Encoding：实体内容的编码形式（gzip等），用于Content-Type找到相应的解码机制。
    - Content-Language：资源所使用的自然语言
    - Content-Length：实体正文长度
    - Content-Type：实体的媒体类型（text/html; charset=ISO-8859-1）
    - Last-Modified：最后修改日期时间
    - Expires：响应过期日期和时间

上述是Http协议的概要内容，还有许多规则没有阐述，主要因为我觉得目前这些内容应该是够用了，了解这些规则，对于我们处理请求方面的问题，完成恶心的上传，下载等等工作有了一定帮助。

## 2. RESTful设计架构

高度依赖Http协议的设计架构，Representational State Transfer 直译为 表现层状态转化。这里面表现层表示的是资源（Resources）的表现层。

- 资源（Resources）：即网络上的一个实体，或者说是具体信息，比如一段文本，一张图片等等。通过URI可以获取这个资源。
- 表现层（Representation）：资源具体呈现出来的形式就叫表现层。例如一段文本，它可以是HTML格式，JSON格式甚至是二进制格式。所以URI严格来说不需要加入资源的格式后缀，应由HTTP协议中的Accept和Content-Type来描述。
- 状态转化（State Transfer）：HTTP协议是无状态协议，这意味着，所有的状态都保存在服务器端。如果客户端想要操作服务器，必须通过某种手段让服务端发生状态转化。而这种转化是建立在表现层之上的，所以就是表现层状态转化。这个手段就是HTTP协议，具体的就是POST，GET等等操作。

综合上面的阐述，可以得到如下的结论：
1. 每个URI代表一种资源；
2. 客户端和服务器之间，传递这种资源的某种表现层；
3. 客户端通过四个HTTP动词，对服务端资源进行操作，实现表现层的状态转化。

RESTful简单高效，以资源为中心，极端依赖Http协议。与SOAP协议的区别在于，REST面向资源，SOAP面向活动。