下面主要从几个不同的方面介绍HTTP/1.0与HTTP/1.1之间的差别，当然，更多的内容是放在解释这种差异背后的机制上。

## 1 可扩展性

可扩展性的一个重要原则：如果HTTP的某个实现接收到了自身未定义的头域，将自动忽略它。

Ø  在消息中增加版本号，用于兼容性判断。注意，版本号只能用来判断逐段（hop-by-hop）的兼容性，而无法判断端到端（end-to-end）的兼容性。  
例如，一台HTTP/1.1的源服务器从使用HTTP/1.1的Proxy那儿接收到一条转发的消息，实际上源服务器并不知道终端客户使用的是HTTP/1.0还是HTTP/1.1。因此，HTTP/1.1定义Via头域，用来记录消息转发的路径，它记录了整个路径上所有发送方使用的版本号。

Ø  HTTP/1.1增加了OPTIONS方法，它允许客户端获取一个服务器支持的方法列表。

Ø  为了与未来的协议规范兼容，HTTP/1.1在请求消息中包含了Upgrade头域，通过该头域，客户端可以让服务器知道它能够支持的其它备用通信协议，服务器可以据此进行协议切换，使用备用协议与客户端进行通信。

## 2 缓存

在HTTP/1.0中，使用Expire头域来判断资源的fresh或stale，并使用条件请求（conditional request）来判断资源是否仍有效。例如，cache服务器通过If-Modified-Since头域向服务器验证资源的Last-Modefied头域是否有更新，源服务器可能返回304（Not Modified），则表明该对象仍有效；也可能返回200（OK）替换请求的Cache对象。

此外，HTTP/1.0中还定义了Pragma:no-cache头域，客户端使用该头域说明请求资源不能从cache中获取，而必须回源获取。

HTTP/1.1在1.0的基础上加入了一些cache的新特性，当缓存对象的Age超过Expire时变为stale对象，cache不需要直接抛弃stale对象，而是与源服务器进行重新激活（revalidation）。

HTTP/1.0中，If-Modified-Since头域使用的是绝对时间戳，精确到秒，但使用绝对时间会带来不同机器上的时钟同步问题。而HTTP/1.1中引入了一个ETag头域用于重激活机制，它的值entity tag可以用来唯一的描述一个资源。请求消息中可以使用If-None-Match头域来匹配资源的entitytag是否有变化。

为了使caching机制更加灵活，HTTP/1.1增加了Cache-Control头域（请求消息和响应消息都可使用），它支持一个可扩展的指令子集：例如max-age指令支持相对时间戳；private和no-store指令禁止对象被缓存；no-transform阻止Proxy进行任何改变响应的行为。

Cache使用关键字索引在磁盘中缓存的对象，在HTTP/1.0中使用资源的URL作为关键字。但可能存在不同的资源基于同一个URL的情况，要区别它们还需要客户端提供更多的信息，如Accept-Language和Accept-Charset头域。为了支持这种内容协商机制\(content negotiation mechanism\)，HTTP/1.1在响应消息中引入了Vary头域，该头域列出了请求消息中需要包含哪些头域用于内容协商。

## 3 带宽优化

HTTP/1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了。例如，客户端只需要显示一个文档的部分内容，又比如下载大文件时需要支持断点续传功能，而不是在发生断连后不得不重新下载完整的包。

HTTP/1.1中在请求消息中引入了range头域，它允许只请求资源的某个部分。在响应消息中Content-Range头域声明了返回的这部分对象的偏移值和长度。如果服务器相应地返回了对象所请求范围的内容，则响应码为206（Partial Content），它可以防止Cache将响应误以为是完整的一个对象。

另外一种情况是请求消息中如果包含比较大的实体内容，但不确定服务器是否能够接收该请求（如是否有权限），此时若贸然发出带实体的请求，如果被拒绝也会浪费带宽。

HTTP/1.1加入了一个新的状态码100（Continue）。客户端事先发送一个只带头域的请求，如果服务器因为权限拒绝了请求，就回送响应码401（Unauthorized）；如果服务器接收此请求就回送响应码100，客户端就可以继续发送带实体的完整请求了。注意，HTTP/1.0的客户端不支持100响应码。但可以让客户端在请求消息中加入Expect头域，并将它的值设置为100-continue。

节省带宽资源的一个非常有效的做法就是压缩要传送的数据。Content-Encoding是对消息进行端到端（end-to-end）的编码，它可能是资源在服务器上保存的固有格式（如jpeg图片格式）；在请求消息中加入Accept-Encoding头域，它可以告诉服务器客户端能够解码的编码方式。

而Transfer-Encoding是逐段式（hop-by-hop）的编码，如Chunked编码。在请求消息中加入TE头域用来告诉服务器能够接收的transfer-coding方式，

## 4 长连接

HTTP 1.0规定浏览器与服务器只保持短暂的连接，浏览器的每次请求都需要与服务器建立一个TCP连接，服务器完成请求处理后立即断开TCP连接，服务器不跟踪每个客户也不记录过去的请求。此外，由于大多数网页的流量都比较小，一次TCP连接很少能通过slow-start区，不利于提高带宽利用率。

HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟。例如：一个包含有许多图像的网页文件的多个请求和应答可以在一个连接中传输，但每个单独的网页文件的请求和应答仍然需要使用各自的连接。

HTTP 1.1还允许客户端不用等待上一次请求结果返回，就可以发出下一次请求，但服务器端必须按照接收到客户端请求的先后顺序依次回送响应结果，以保证客户端能够区分出每次请求的响应内容，这样也显著地减少了整个下载过程所需要的时间。

在HTTP/1.0中，要建立长连接，可以在请求消息中包含Connection: Keep-Alive头域，如果服务器愿意维持这条连接，在响应消息中也会包含一个Connection: Keep-Alive的头域。同时，可以加入一些指令描述该长连接的属性，如max，timeout等。

事实上，Connection头域可以携带三种不同类型的符号：

1、一个包含若干个头域名的列表，声明仅限于一次hop连接的头域信息；

2、任意值，本次连接的非标准选项，如Keep-Alive等；

3、close值，表示消息传送完成之后关闭长连接；



客户端和源服务器之间的消息传递可能要经过很多中间节点的转发，这是一种逐跳传递（hop-by-hop）。HTTP/1.1相应地引入了hop-by-hop头域，这种头域仅作用于一次hop，而非整个传递路径。每一个中间节点（如Proxy，Gateway）接收到的消息中如果包含Connection头域，会查找Connection头域中的一个头域名列表，并在将消息转发给下一个节点之前先删除消息中这些头域。

通常，HTTP/1.0的Proxy不支持Connection头域，为了不让它们转发可能误导接收者的头域，协议规定所有出现在Connection头域中的头域名都将被忽略。

## 5 消息传递

HTTP消息中可以包含任意长度的实体，通常它们使用Content-Length来给出消息结束标志。但是，对于很多动态产生的响应，只能通过缓冲完整的消息来判断消息的大小，但这样做会加大延迟。如果不使用长连接，还可以通过连接关闭的信号来判定一个消息的结束。

HTTP/1.1中引入了Chunkedtransfer-coding来解决上面这个问题，发送方将消息分割成若干个任意大小的数据块，每个数据块在发送时都会附上块的长度，最后用一个零长度的块作为消息结束的标志。这种方法允许发送方只缓冲消息的一个片段，避免缓冲整个消息带来的过载。

在HTTP/1.0中，有一个Content-MD5的头域，要计算这个头域需要发送方缓冲完整个消息后才能进行。而HTTP/1.1中，采用chunked分块传递的消息在最后一个块（零长度）结束之后会再传递一个拖尾（trailer），它包含一个或多个头域，这些头域是发送方在传递完所有块之后再计算出值的。发送方会在消息中包含一个Trailer头域告诉接收方这个拖尾的存在。

## 6 Host头域

在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。

HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。此外，服务器应该接受以绝对路径标记的资源请求。

## 7 错误提示

HTTP/1.0中只定义了16个状态响应码，对错误或警告的提示不够具体。HTTP/1.1引入了一个Warning头域，增加对错误或警告信息的描述。

此外，在HTTP/1.1中新增了24个状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。

## 8 内容协商

为了满足互联网使用不同母语和字符集的用户，一些网络资源有不同的语言版本（如中文版、英文版）。HTTP/1.0定义了内容协商（contentnegotiation）的概念，也就是说客户端可以告诉服务器自己可以接收以何种语言（或字符集）表示的资源。例如如果服务器不能明确客户端需要何种类型的资源，会返回300（Multiple Choices），并包含一个列表，用来声明该资源的不同可用版本，然后客户端在请求消息中包含Accept-Language和Accept-Charset头域指定需要的版本。

就像有些人会说几门外语，但每种外语的流利程度并不相同。类似地，网络资源也可以有不同的表达形式，比如有母语版和各种翻译版本。HTTP引入了一个品质因子（quality values）的概念来表示不同版本的可用性，它的取值从0.0到1.0。例如一个母语是英语的人也能讲法语、甚至还学了点丹麦语，那么他的浏览器可用作如下配置：Accept-Language: en, fr;q=0.5, da;q=0.1。这时，服务器会优先选取品质因子高的值对应的资源版本作为响应。



## 参考资料

\[1\] Brian Totty, "HTTP: The DefinitiveGuide", O'REILLY & ASSOC INC, 27, Sep., 2002.

\[2\] B. Krishnamurthy, J. C. Mogul, and D.M. Kristol, "Key Differences between HTTP/1.0 and HTTP/1.1",[http://www2.research.att.com/~bala/papers/h0vh1.html](http://www2.research.att.com/~bala/papers/h0vh1.html).

