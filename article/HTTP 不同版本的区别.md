# HTTP 不同版本的区别

# http1.0

定义了最初了 HTTP 规范, 包括 header body 以及无状态等, 但是:

- 不支持持久连接

每次连接都要发起一次新的连接.

# http1.1

这个版本实在 http1.0 的基础上添加了内容:

- 首先是**长连接**，`HTTP1.1`增加了一个`Connection`字段，通过设置`Keep-Alive`可以保持`HTTP`连接不断开，避免了每次客户端与服务器请求都要重复建立释放建立`TCP`连接，提高了网络的利用率。如果客户端想关闭`HTTP`连接，可以在请求头中携带`Connection: false`来告知服务器关闭请求。
- 增加 host 字段
- 引入 Chunked transfer-coding , 实现断点续传
- 管线化(pipelining)传输, 这意味着可以同时发送多个请求. 原有的通信信道中只能存在一个请求. 可以同时发送多个请求后在大量的请求下可以并行请求. **但是存在一个严重问题, 就是接收的顺序需要和发送时候一致, 如果较前的请求的响应十分的缓慢会导致在这个请求之后的响应被阻塞, 这种状况被称为队头阻塞(Head of line blocking)**.
- 引入了 缓存处理
  - 强缓存
  - 协商缓存

**注意**: 浏览器没有实现管线化, 但是多个请求依然是并行处理的, 浏览器通过限制页面打开的 TCP 连接的数量, 来控制并发, 这个限制一般是 6-8 个.

# http2

HTTP2 和之前的版本不兼容所以 HTTP2 而不是 HTTP1.2, 是因为引入了**二进制分帧层**这个特性.

## 二级制分帧层

这里所谓的“层”，指的是位于套接字接口与应用可见的高级 HTTP API 之间一个经过优化的新编码机制：HTTP 的语义（包括各种动词、方法、标头）都不受影响，不同的是传输期间对它们的编码方式变了。

**HTTP/1.x 协议以换行符作为纯文本的分隔符，而 HTTP/2 将所有传输的信息分割为更小的消息和帧，并采用二进制格式对它们编码。**

新的二进制分帧机制改变了客户端与服务器之间交换数据的方式。 为了说明这个过程，我们需要了解 HTTP/2 的三个概念：

- *数据流*：已建立的连接内的双向字节流，可以承载一条或多条消息。
- *消息*：与逻辑请求或响应消息对应的完整的一系列帧。
- *帧*：HTTP/2 通信的最小单位，每个帧都包含帧头，至少也会标识出当前帧所属的数据流。

这些概念的关系总结如下：

- 所有通信都在一个 TCP 连接上完成，此连接可以承载任意数量的双向数据流。
- 每个数据流都有一个唯一的标识符和可选的优先级信息，用于承载双向消息。
- 每条消息都是一条逻辑 HTTP 消息（例如请求或响应），包含一个或多个帧。
- 帧是最小的通信单位，承载着特定类型的数据，例如 HTTP 标头、消息负载等等。 来自不同数据流的帧可以交错发送，然后再根据每个帧头的数据流标识符重新组装。

如果把 TCP 视为一条公路那么相似的包含关系如下, 则 "数据流" 就是 "车道", 车辆就是 "消息" 而 "帧" 就是乘客.

## 请求和响应复用

TCP 连接建立后内部可以存在多个双向数据流, 而数据流可以承载 N 个 HTTP 消息, 包含请求和响应, 这样可以实现请求与响应复用.

HTTP/2 中新的二进制分帧层突破 HTTP/1.x 的并行请求限制，实现了完整的请求和响应复用：客户端和服务器可以将 HTTP 消息分解为互不依赖的帧，然后交错发送，最后再在另一端把它们重新组装起来。

将 HTTP 消息分解为独立的帧，交错发送，然后在另一端重新组装是 HTTP 2 最重要的一项增强。事实上，这个机制会在整个网络技术栈中引发一系列连锁反应，从而带来巨大的性能提升，让我们可以：

- 并行交错地发送多个请求，请求之间互不影响。
- 并行交错地发送多个响应，响应之间互不干扰。
- 使用一个连接并行发送多个请求和响应。
- 不必再为绕过 HTTP/1.x 限制而做很多工作（请参阅[针对 HTTP/1.x 进行优化](https://hpbn.co/optimizing-application-delivery/#optimizing-for-http1x)，例如级联文件、image sprites 和域名分片。
- 消除不必要的延迟和提高现有网络容量的利用率，从而减少页面加载时间。
- *等等…*

HTTP/2 中的新二进制分帧层解决了 HTTP/1.x 中存在的队首阻塞问题，也消除了并行处理和发送请求及响应时对多个连接的依赖。 结果，应用速度更快、开发更简单、部署成本更低。

## 数据流优先级

由于数据被拆为多个帧, 帧可以组成消息, 在交错发送帧的时候某个消息是否优先处理, 也就是他们的传输顺序是一个关键性能因素:

- 高优先级的消息优先处理

HTTP/2 标准允许每个数据流都有一个关联的权重和依赖关系：

- 可以向每个数据流分配一个介于 1 至 256 之间的整数。
  - 数值越大可使用的资源越高
- 每个数据流与其他数据流之间可以存在显式依赖关系。
  - 当依赖的数据流传输完毕后依赖的数据流会被传输.

## 服务器推送

HTTP/2 新增的另一个强大的新功能是，服务器可以对一个客户端请求发送多个响应。 换句话说，除了对最初请求的响应外，服务器还可以向客户端推送额外资源，而无需客户端明确地请求。

为什么在浏览器中需要一种此类机制呢？一个典型的网络应用包含多种资源，客户端需要检查服务器提供的文档才能逐个找到它们。 那为什么不让服务器提前推送这些资源，从而减少额外的延迟时间呢？ 服务器已经知道客户端下一步要请求什么资源，这时候服务器推送即可派上用场。

 推送资源可以进行以下处理：

- 由客户端缓存
- 在不同页面之间重用
- 与其他资源一起复用
- 由服务器设定优先级
- 被客户端拒绝

## 标头(header)压缩

为了减少此开销和提升性能，HTTP/2 使用 HPACK 压缩格式压缩请求和响应标头元数据，这种格式采用两种简单但是强大的技术：

1. 这种格式支持通过静态霍夫曼代码对传输的标头字段进行编码，从而减小了各个传输的大小。
2. 这种格式要求客户端和服务器同时维护和更新一个包含之前见过的标头字段的索引列表（换句话说，它可以建立一个共享的压缩上下文），此列表随后会用作参考，对之前传输的值进行有效编码。

作为一种进一步优化方式，HPACK 压缩上下文包含一个静态表和一个动态表：静态表在规范中定义，并提供了一个包含所有连接都可能使用的常用 HTTP 标头字段（例如，有效标头名称）的列表；

动态表最初为空，将根据在特定连接内交换的值进行更新。 因此，为之前未见过的值采用静态 Huffman 编码，并替换每一侧静态表或动态表中已存在值的索引，可以减小每个请求的大小。

注：在 HTTP/2 中，请求和响应标头字段的定义保持不变，仅有一些微小的差异：所有标头字段名称均为小写，请求行现在拆分成各个 `:method`、`:scheme`、`:authority` 和 `:path` 伪标头字段。

# 参考

> https://segmentfault.com/a/1190000013028798
>
> https://www.cnblogs.com/shangyueyue/p/11041998.html
>
> https://developers.google.com/web/fundamentals/performance/http2