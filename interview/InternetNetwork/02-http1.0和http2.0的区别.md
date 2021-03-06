## HTTP1.0和HTTP1.1的一些区别

HTTP1.1也是当前使用最为广泛的HTTP协议。 主要区别主要体现在：

- 缓存处理
  
  在HTTP1.0中主要使用header里的If-Modified-Since,Expires来做为缓存判断的标准，HTTP1.1则引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since, If-Match, If-None-Match等更多可供选择的缓存头来控制缓存策略。

- 带宽优化及网络连接的使用
  
  HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。

- 错误通知的管理
  
  在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。

- Host头处理
  
  在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。

- 长连接
  
  HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，**在HTTP1.1中默认开启Connection： keep-alive**，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。

## HTTP1.1与HTTP2.0的区别

- **多路复用**

HTTP2.0使用了多路复用的技术，做到**同一个连接并发处理多个请求**，而且并发请求的数量比HTTP1.1大了好几个数量级。

当然HTTP1.1也可以多建立几个TCP连接，来支持处理更多并发的请求，但是创建TCP连接本身也是有开销的。

**TCP连接有一个预热和保护的过程，先检查数据是否传送成功，一旦成功过，则慢慢加大传输速度**。因此对应瞬时并发的连接，服务器的响应就会变慢。所以最好能使用一个建立好的连接，并且这个连接可以支持瞬时并发的请求。


- **数据压缩**
  
我们知道，http请求和响应都是由【状态行、请求/响应头部、消息主题】三部分组成的。 一般而言，消息主体都会经过gzip压缩，或者本身传输的就是压缩过后的二进制文件（如图片、音频等），但是状态行和头部多是没有经过任何压缩，而是直接以纯文本的方式进行传输的。

然而，随着web功能越来越复杂，请求数量越来越多，随之而来的就是头部的流量越来越多，并且在建立初次链接之后的链接也要发送user-agent等信息，是在是一种浪费。

因此，http2提出了**对请求和响应的头部进行压缩**，即不再只是压缩主题部分，这种压缩方式就是HAPCK 。

通过压缩，头部大小可以减少一半之多，如果后面重复发送请求，那么可能压缩后的头部大小只有原始大小的 1/10。

HTTP1.1不支持header数据的压缩，HTTP2.0使用HPACK算法对header的数据进行压缩，这样数据体积小了，在网络上传输就会更快。

- **服务器推送**
  
当代网页使用了许多资源:HTML、样式表、脚本、图片等等。在HTTP/1.x中这些资源每一个都必须明确地请求。这可能是一个很慢的过程。浏览器从获取HTML开始，然后在它解析和评估页面的时候，增量地获取更多的资源。因为服务器必须等待浏览器做每一个请求，网络经常是空闲的和未充分使用的。

**为了改善延迟，HTTP/2引入了server push，它允许服务端推送资源给浏览器**，在浏览器明确地请求之前。一个服务器经常知道一个页面需要很多附加资源，在它响应浏览器第一个请求的时候，可以开始推送这些资源。这允许服务端去完全充分地利用一个可能空闲的网络，改善页面加载时间。

服务器端推送的这些资源其实存在客户端的某处地方，客户端直接从本地加载这些资源就可以了，不用走网络，速度自然是快很多的。


链接：https://www.jianshu.com/p/7bfec28236c3
