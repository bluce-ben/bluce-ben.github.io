---
title: HTTP 详解
date: 2018-04-10 16:23:25
categories:
- 计算机网络
tags:
- 计算机网络
---
### 一、基础概念篇
#### 1、介绍
HTTP是Hyper Text Transfer Protocol（超文本传输协议）的缩写。它的发展是万维网协会（World Wide Web Consortium）和Internet工作小组IETF（Internet Engineering Task Force）合作的结果，（他们）最终发布了一系列的RFC，RFC 1945定义了HTTP/1.0版本。其中最著名的就是RFC 2616。RFC 2616定义了今天普遍使用的一个版本——HTTP 1.1。

HTTP协议（HyperText Transfer Protocol，超文本传输协议）是用于从WWW服务器传输超文本到本地浏览器的传送协议。它可以使浏览器更加高效，使网络传输减少。它不仅保证计算机正确快速地传输超文本文档，还确定传输文档中的哪一部分，以及哪部分内容首先显示(如文本先于图形)等。

HTTP是一个应用层协议，由请求和响应构成，是一个标准的客户端服务器模型。HTTP是一个无状态的协议。
<!--more-->

#### 2、在TCP/IP协议栈中的位置
HTTP协议通常承载于TCP协议之上，有时也承载于TLS或SSL协议层之上，这个时候，就成了我们常说的HTTPS。如下图所示：
![](/uploads/2018/04/network_http_01.jpg)
默认HTTP的端口号为80，HTTPS的端口号为443。


#### 3、HTTP的请求响应模型
HTTP协议永远都是客户端发起请求，服务器回送响应。见下图：
![](/uploads/2018/04/network_http_02.jpg)
这样就限制了使用HTTP协议，无法实现在客户端没有发起请求的时候，服务器将消息推送给客户端。
HTTP协议是一个无状态的协议，同一个客户端的这次请求和上次请求是没有对应关系。


#### 4、工作流程
一次HTTP操作称为一个事务，其工作过程可分为四步：
>1）首先客户机与服务器需要建立连接。只要单击某个超级链接，HTTP的工作开始。
2）建立连接后，客户机发送一个请求给服务器，请求方式的格式为：统一资源标识符（URL）、协议版本号，后边是MIME信息包括请求修饰符、客户机信息和可能的内容。
3）服务器接到请求后，给予相应的响应信息，其格式为一个状态行，包括信息的协议版本号、一个成功或错误的代码，后边是MIME信息包括服务器信息、实体信息和可能的内容。
4）客户端接收服务器所返回的信息通过浏览器显示在用户的显示屏上，然后客户机与服务器断开连接。

如果在以上过程中的某一步出现错误，那么产生错误的信息将返回到客户端，有显示屏输出。对于用户来说，这些过程是由HTTP自己完成的，用户只要用鼠标点击，等待信息显示就可以了。


#### 5、客户端一个请求流程（详细版）
待补充。。。


### 二、协议详解篇
#### 1、HTTP/1.0 与 HTTP/1.1 比较

异同点        | HTTP/1.0     | HTTP/1.1
--------------|--------------|-----------------------
建立连接方面  | 连接不能复用 | 可复用，减少TCP三次握手的开销
请求方式      | GET、POST、HEAD | GET、POST、HEAD、OPTIONS、PUT、DELETE、TRACE、CONNECT
Request消息头 |              | 新增Host域 


#### 2、HTTP请求消息
##### （1）请求消息格式
HTTP请求消息实例：
```
GET /hello.htm HTTP/1.1
Accept: */*
Accept-Language: zh-cn
Accept-Encoding: gzip, deflate
If-Modified-Since: Wed, 17 Oct 2007 02:15:55 GMT
If-None-Match: W/"158-1192587355000"
User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)
Host: 192.168.2.162:8080
Connection: Keep-Alive
```
请求消息格式如下所示：
>请求行
通用信息头|请求头|实体头
CRLF(回车换行)
实体内容

其中“请求行”为：请求行 = 方法 [空格] 请求URI [空格] 版本号 [回车换行]

##### （2）请求方法
HTTP的请求方法包括如下几种：
GET | POST | HEAD | PUT | DELETE | OPTIONS | TRACE | CONNECT


#### 3、HTTP响应消息
##### （1）响应消息格式
HTTP响应消息实例如下所示：
```
HTTP/1.1 200 OK
ETag: W/"158-1192590101000"
Last-Modified: Wed, 17 Oct 2007 03:01:41 GMT
Content-Type: text/html
Content-Length: 158
Date: Wed, 17 Oct 2007 03:01:59 GMT
Server: Apache-Coyote/1.1
```
HTTP响应消息的格式如下所示：
>状态行
通用信息头|响应头|实体头
CRLF
实体内容

其中：状态行 = 版本号 [空格] 状态码 [空格] 原因 [回车换行]

##### （2）响应状态码
**1\*\*：请求收到，继续处理**
100——客户必须继续发出请求
101——客户要求服务器根据请求转换HTTP协议版本

**2\*\*：操作成功收到，分析、接受**
<font color="red">200——交易成功</font>
201——提示知道新文件的URL
202——接受和处理、但处理未完成
203——返回信息不确定或不完整
204——请求收到，但返回信息为空
205——服务器完成了请求，用户代理必须复位当前已经浏览过的文件
206——服务器已经完成了部分用户的GET请求

**3\*\*：完成此请求必须进一步处理**
300——请求的资源可在多处得到
<font color="red">301——删除请求数据</font>
302——在其他地址发现了请求数据
303——建议客户访问其他URL或访问方式
304——客户端已经执行了GET，但文件未变化
305——请求的资源必须从服务器指定的地址得到
306——前一版本HTTP中使用的代码，现行版本中不再使用
307——申明请求的资源临时性删除

**4\*\*：请求包含一个错误语法或不能完成**
<font color="red">400——错误请求，如语法错误</font>
<font color="red">401——未授权</font>
HTTP 401.1 - 未授权：登录失败
　　HTTP 401.2 - 未授权：服务器配置问题导致登录失败
　　HTTP 401.3 - ACL 禁止访问资源
　　HTTP 401.4 - 未授权：授权被筛选器拒绝
　　HTTP 401.5 - 未授权：ISAPI 或 CGI 授权失败
402——保留有效ChargeTo头响应
<font color="red">403——禁止访问</font>
HTTP 403.1 禁止访问：禁止可执行访问
　　HTTP 403.2 - 禁止访问：禁止读访问
　　HTTP 403.3 - 禁止访问：禁止写访问
　　HTTP 403.4 - 禁止访问：要求 SSL
　　HTTP 403.5 - 禁止访问：要求 SSL 128
　　HTTP 403.6 - 禁止访问：IP 地址被拒绝
　　HTTP 403.7 - 禁止访问：要求客户证书
　　HTTP 403.8 - 禁止访问：禁止站点访问
　　HTTP 403.9 - 禁止访问：连接的用户过多
　　HTTP 403.10 - 禁止访问：配置无效
　　HTTP 403.11 - 禁止访问：密码更改
　　HTTP 403.12 - 禁止访问：映射器拒绝访问
　　HTTP 403.13 - 禁止访问：客户证书已被吊销
　　HTTP 403.15 - 禁止访问：客户访问许可过多
　　HTTP 403.16 - 禁止访问：客户证书不可信或者无效
　　HTTP 403.17 - 禁止访问：客户证书已经到期或者尚未生效
<font color="red">404——没有发现文件、查询或URl</font>
405——用户在Request-Line字段定义的方法不允许
406——根据用户发送的Accept拖，请求资源不可访问
407——类似401，用户必须首先在代理服务器上得到授权
408——客户端没有在用户指定的饿时间内完成请求
409——对当前资源状态，请求不能完成
410——服务器上不再有此资源且无进一步的参考地址
411——服务器拒绝用户定义的Content-Length属性请求
412——一个或多个请求头字段在当前请求中错误
413——请求的资源大于服务器允许的大小
414——请求的资源URL长于服务器允许的长度
415——请求资源不支持请求项目格式
416——请求中包含Range请求头字段，在当前请求资源范围内没有range指示值，请求也不包含If-Range请求头字段
417——服务器不满足请求Expect头字段指定的期望值，如果是代理服务器，可能是下一级服务器不能满足请求长。

**5\*\*：服务器执行一个完全有效请求失败**
<font color="red">HTTP 500 - 内部服务器错误</font>
　　HTTP 500.100 - 内部服务器错误 - ASP 错误
　　HTTP 500-11 服务器关闭
　　HTTP 500-12 应用程序重新启动
　　HTTP 500-13 - 服务器太忙
　　HTTP 500-14 - 应用程序无效
　　HTTP 500-15 - 不允许请求 global.asa
Error 501 - 未实现
<font color="red">HTTP 502 - 网关错误</font>


#### 4、请求头：
HTTP最常见的请求头如下：

名称					| 说明
------------------------|-------------------------------------------
Accept：				| 浏览器可接受的MIME类型；
Accept-Charset：		| 浏览器可接受的字符集；
Accept-Encoding：		| 浏览器能够进行解码的数据编码方式，比如gzip。Servlet能够向支持gzip的浏览器返回经gzip编码的HTML页面。许多情形下这可以减少5到10倍的下载时间；
Accept-Language：		| 浏览器所希望的语言种类，当服务器能够提供一种以上的语言版本时要用到；
Authorization：			| 授权信息，通常出现在对服务器发送的WWW-Authenticate头的应答中；
Connection：			| 表示是否需要持久连接。如果Servlet看到这里的值为“Keep-Alive”，或者看到请求使用的是HTTP 1.1（HTTP 1.1默认进行持久连接），它就可以利用持久连接的优点，当页面包含多个元素时（例如Applet，图片），显著地减少下载所需要的时间。要实现这一点，Servlet需要在应答中发送一个Content-Length头，最简单的实现方法是：先把内容写入ByteArrayOutputStream，然后在正式写出内容之前计算它的大小；
Content-Length：		| 表示请求消息正文的长度；
Cookie：				| 这是最重要的请求头信息之一；
From：					| 请求发送者的email地址，由一些特殊的Web客户程序使用，浏览器不会用到它；
Host：					| 初始URL中的主机和端口；
If-Modified-Since：		| 只有当所请求的内容在指定的日期之后又经过修改才返回它，否则返回304“Not Modified”应答；
Pragma：				| 指定“no-cache”值表示服务器必须返回一个刷新后的文档，即使它是代理服务器而且已经有了页面的本地拷贝；
Referer：				| 包含一个URL，用户从该URL代表的页面出发访问当前请求的页面。

User-Agent：			| 浏览器类型，如果Servlet返回的内容与浏览器类型有关则该值非常有用；
UA-Pixels，UA-Color，UA-OS，UA-CPU： | 由某些版本的IE浏览器所发送的非标准的请求头，表示屏幕大小、颜色深度、操作系统和CPU类型。


#### 5、响应头：
HTTP最常见的响应头如下所示：
<style type="text/css">
	table th:first-of-type{
		width: 24%;
	}
</style>

名称					| 说明
------------------------|-------------------------------------------
Allow：					| 服务器支持哪些请求方法（如GET、POST等）；
Content-Encoding：		| 文档的编码（Encode）方法。只有在解码之后才可以得到Content-Type头指定的内容类型。利用gzip压缩文档能够显著地减少HTML文档的下载时间。Java的GZIPOutputStream可以很方便地进行gzip压缩，但只有Unix上的Netscape和Windows上的IE 4、IE 5才支持它。因此，Servlet应该通过查看Accept-Encoding头（即`request.getHeader("Accept-Encoding")`）检查浏览器是否支持gzip，为支持gzip的浏览器返回经gzip压缩的HTML页面，为其他浏览器返回普通页面；
Content-Length：		| 表示内容长度。只有当浏览器使用持久HTTP连接时才需要这个数据。如果你想要利用持久连接的优势，可以把输出文档写入ByteArrayOutputStram，完成后查看其大小，然后把该值放入Content-Length头，最后通过`byteArrayStream.writeTo(response.getOutputStream()`发送内容；
Content-Type：			| 表示后面的文档属于什么MIME类型。Servlet默认为text/plain，但通常需要显式地指定为text/html。由于经常要设置Content-Type，因此HttpServletResponse提供了一个专用的方法setContentTyep。 可在web.xml文件中配置扩展名和MIME类型的对应关系；
Date：					| 当前的GMT时间。你可以用setDateHeader来设置这个头以避免转换时间格式的麻烦；
Expires：				| 指明应该在什么时候认为文档已经过期，从而不再缓存它。
Last-Modified：			| 文档的最后改动时间。客户可以通过If-Modified-Since请求头提供一个日期，该请求将被视为一个条件GET，只有改动时间迟于指定时间的文档才会返回，否则返回一个304（Not Modified）状态。Last-Modified也可用setDateHeader方法来设置；
Location：				| 表示客户应当到哪里去提取文档。Location通常不是直接设置的，而是通过HttpServletResponse的sendRedirect方法，该方法同时设置状态代码为302；
Refresh：				| 表示浏览器应该在多少时间之后刷新文档，以秒计。除了刷新当前文档之外，你还可以通过`setHeader("Refresh", "5; URL=http://host/path")`让浏览器读取指定的页面。注意这种功能通常是通过设置HTML页面HEAD区的`<META HTTP-EQUIV="Refresh" CONTENT="5;URL=http://host/path">`实现，这是因为，自动刷新或重定向对于那些不能使用CGI或Servlet的HTML编写者十分重要。但是，对于Servlet来说，直接设置Refresh头更加方便。注意Refresh的意义是“N秒之后刷新本页面或访问指定页面”，而不是“每隔N秒刷新本页面或访问指定页面”。因此，连续刷新要求每次都发送一个Refresh头，而发送204状态代码则可以阻止浏览器继续刷新，不管是使用Refresh头还是<META HTTP-EQUIV="Refresh" ...>。注意Refresh头不属于HTTP 1.1正式规范的一部分，而是一个扩展，但Netscape和IE都支持它。


#### 6、实体头
实体头用坐实体内容的元信息，描述了实体内容的属性，包括实体信息类型，长度，压缩方法，最后一次修改时间，数据有效性等。

名称					| 说明
------------------------|-------------------------------------------
Allow：					| GET,POST
Content-Encoding：		| 文档的编码（Encode）方法，例如：gzip，见“2.5 响应头”；
Content-Language：      | 内容的语言类型，例如：zh-cn；
Content-Length：		| 表示内容长度，eg：80，可参考“2.5响应头”；
Content-Location：		| 表示客户应当到哪里去提取文档，例如：`http://www.dfdf.org/`dfdf.html，可参考“2.5响应头”；
Content-MD5：			| MD5 实体的一种MD5摘要，用作校验和。发送方和接受方都计算MD5摘要，接受方将其计算的值与此头标中传递的值进行比较。Eg1：Content-MD5: <base64 of 128 MD5 digest>。Eg2：dfdfdfdfdfdfdff==；
Content-Range：			| 随部分实体一同发送；标明被插入字节的低位与高位字节偏移，也标明此实体的总长度。Eg1：Content-Range: 1001-2000/5000，eg2：bytes 2543-4532/7898
Content-Type：			| 标明发送或者接收的实体的MIME类型。Eg：text/html; charset=GB2312       主类型/子类型；
Expires：				| 为0证明不缓存；
Last-Modified：			| WEB 服务器认为对象的最后修改时间，比如文件的最后修改时间，动态页面的最后产生时间等等。例如：Last-Modified：Tue, 06 May 2008 02:42:43 GMT.


#### 7、扩展头
在HTTP消息中，也可以使用一些再HTTP1.1正式规范里没有定义的头字段，这些头字段统称为自定义的HTTP头或者扩展头，他们通常被当作是一种实体头处理。

现在流行的浏览器实际上都支持Cookie,Set-Cookie,Refresh和Content-Disposition等几个常用的扩展头字段。

名称					| 说明
------------------------|-------------------------------------------
Refresh：1;`url=http://www.dfdf.org`  |  //过1秒跳转到指定位置；
Content-Disposition：	| 头字段,可参考“2.5响应头”；
Content-Type：			| WEB 服务器告诉浏览器自己响应的对象的类型。


### 三、协议高级篇
#### 1、HTTP传输及与HTTPS的区别：
TCP的数据是通过名为IP分组（或IP数据报）的小数据块来发送的。
HTTP就是“HTTP over TCP over IP”这个“协议栈”中的最顶层。其安全版本HTTPS就是在HTTP和TCP之间插入了一个（称为TLS或SSL的）密码加密层。
![](/uploads/2018/12/network_http_https.png)

HTTP要传送一条报文时，会以流的形式将报文数据的内容通过一条打开的TCP连接按序传输。TCP收到数据流之后，会将数据流砍成被称作段的小数据块，并将段封装在IP分组中，通过因特网进行传输。

每个TCP段都是由IP分组承载，从一个IP地址发送到另一个IP地址的。每个IP分组中都包括：
* 一个IP分组首部（通常为20字节）；
* 一个TCP段首部（通常为20字节）；
* 一个TCP数据块（0个或多个字节）。

IP首部包含了源和目的IP地址、长度和其它一些标记。TCP段的首部包含了TCP端口号、TCP控制标记，以及用于数据排序和完整性检查的一些数字值。

TCP连接是通过4个值来识别的：
	<源IP地址、源端口号、目的IP地址、目的端口号>
这4个值一起唯一地定义了一条连接。两条不同的TCP连接不能拥有4个完全相同的地址组件值（但不同连接的部分组件可以拥有相同的值）。

#### 2、HTTP事务的时延有以下几种主要原因。
（1）客户端首先需要根据URI确定Web服务器的IP地址和端口号。如果最近没有对URI中的主机名进行访问，通过DNS解析系统将URI中的主机名转换成一个IP地址可能要花费数十秒的时间。（注：大多数HTTP客户端都有一个小的DNS缓存，用来保存近期所访问站点的IP地址。）
（2）接下来，客户端向服务器发送一条TCP连接请求，并等待服务器回送一个请求接收应答。每条新的TCP连接都会有连接建立时延。这个值通常最多只有一两秒钟，但如果有数百个HTTP事务的话，这个值会快速地叠加上去。
（3）一旦连接建立起来了，客户端就会通过新建立的TCP管道来发送HTTP请求。数据到达时，Web服务器会从TCP连接中读取请求报文，并对请求进行处理。
（4）然后，Web服务器会回送HTTP响应，这也需要花费时间。

这些TCP网络时延的大小取决于硬件速度、网络和服务器的负载，请求和响应报文的尺寸，以及客户端和服务器之间的距离。TCP协议的技术复杂性也会对时延产生巨大的影响。
