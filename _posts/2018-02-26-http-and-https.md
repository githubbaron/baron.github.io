---
layout: post
title: "HTTP and HTTPS"
date: 2018-02-26 02:35:03
image: 'http://res.cloudinary.com/degzyaemb/image/upload/c_scale,w_750/v1518871929/The_Deer_and_the_Cauldron1_wfxxzg.png'
description: 《图解HTTP》关于HTTP和HTTPS的总结与实践(包括了HTTP和HTTPS的差别以及如何在站点中实现HTTPS)。
category: 'HTTP'
tags:
- HTTP
- HTTPS
twitter_text:
introduction: 《图解HTTP》关于HTTP和HTTPS的总结与实践。
---

## HTTP协议
要理解HTTP协议，首先要从TCP/IP协议讲起，TCP/IP协议族按层次分别为：应用层、传输层、网络层、数据链路层。

### TCP/IP协议族分层
**应用层**：决定了向用户提供应用服务时通信的活动。*协议包括：`FTP、DNS、HTTP`*。

**传输层**：传输层对上层应用层，提供处于网络连接中的两台计算机之间的数据传输。
运输层协议在端系统中而不是在路由器中实现*协议包括：`TCP、UDP`。*

**网络层**：又称IP层，用来处理在网络上流动的数据包。数据包是网络传输的最小数据单位。
该层规定了通过怎样的路径（所谓的传输路线）到达对方计算机，并把数据包传送给对方。
该层提供了路由和寻址的功能。*协议包括：`IP、ARP`。*

**数据链路层**：用来处理连接网络的硬件部分。包括控制操作系统、硬件的设备驱动、NIC（Network Interface Card，网络适配器，即网卡），
及光纤等物理课件部分（还包括连接器等一切传输媒介）。硬件上的范畴均在链路层的作用范围之内。*协议包括：`ARP`。*

#### tips
1. 网络层提供了主机之间的逻辑通信，而运输层为运行在不同主机上的进程提供逻辑通信。
2. 每个进程好比是一座房子，该进程的套接字好比是一扇门。

### IP、TCP和DNS

**在TCP/IP协议族中存在与HTTP相关的三个协议（IP、TCP和DNS）。**

**IP协议**：通过IP地址和MAC（网卡所属的固定地址）确保数据包能够准确传输给对方。

**TCP协议**：按层次分，TCP位于传输层，提供可靠的字节流服务。
（将大块数据分割成以报文段为单位的数据包进行管理）为了能准确的将数据送达目标处，
TCP协议采用了<a href="https://github.com/jawil/blog/issues/14">三次握手策略</a>。

**DNS协议**：提供通过域名查找IP地址或者逆向从IP地址反查域名的服务。

#### TCP和UDP之间的区别

1. TCP是面向连接的而UDP不是，其理由是*UDP在发送数据前不需要建立连接。*<br>
这一点通过对TCP、UDP套接字的编程可以看出。（Python代码）<br>
对于**UDP**来说，在发送信息的时间写入serverName和serverPort即可。
`clientSocket.sendto(message, (serverName, serverPort))`<br>
而对于**TCP**来说，在发送之前要先建立连接。
`clientSocket.connect((serverName, serverPort))`
`clientSocket.send(sentence)`

2. TCP是面向可靠的数据传输，而UDP则是不可靠的。
3. TCP面向字节流传输，而UDP是面向报文。
4. TCP传输速度很慢，但是UDP传输的速度很快。


!["HTTP请求过程"](http://7xikfc.com1.z0.glb.clouddn.com/http-1.6.jpg)

从上述的解析图可以看出当用户需要访问 "http://facebook.com" 这个页面的时候，需要的几个步骤。
1. 浏览器会通过*DNS协议*解析成相应的IP地址。(其中DNS又会通过浏览器缓存、系统缓存、路由器缓存以及递归的查找缓存来进行解析)
2. 通过*HTTP*协议生成HTTP请求报文。
3. 请求发送至服务器之前将会通过*TCP协议*将报文分割成多个报文段。
4. 再通过*IP协议*搜索服务器的地址，一边中转一边传送。
5. **Web服务器软件（Nginx、Apache）接收到请求，根据请求的地址结构映射到类似于/httpdocs/facebook/index.php这样的文件，然后需求处理会生成一个HTML响应。**
6. 浏览器显示HTML文件
7. 继续请求嵌入在HTML中的文件。

#### tips
1. 负载平衡器：以一个特定IP地址进行侦听并将网络请求转发到集群服务器上的硬件设备。
2. 永久301重定向将会重新造成一次TCP连接（三次握手）。
3. 将facebook.com永久重定向到 www.facebook.com 会使搜索引擎将两个地址认定为同一个网站。

### 持久连接
HTTP协议的初始版本中，每进行一次HTTP通信就要断开一次TCP连接。
比如在一个页面加载中，就要进行多次的HTTP请求，其中包括了页面的各种资源（图片、js、css等样式）的加载。
而每一次的HTTP请求会带来多次的TCP重新连接，TCP重复的建立导致了资源的损失，造成了额外的开销。

!["HTTP/1.0之前"]({{ site.baseurl }}/assets/img/HTTP:1.0.jpg)

在HTTP/1.1中新增了持久连接。（只要任何一方没有明确提出断开连接，则保持TCP连接状态）

!["持久连接']({{ site.baseurl }}/assets/img/HTTP:1.1.png)

### 理解TCP的三次握手

> “已失效的连接请求报文段”的产生在这样一种情况下：client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，
以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段。但server收到此失效的连接请求报文段后，
就误认为是client再次发出的一个新的连接请求。于是就向client发出确认报文段，同意建立连接。假设不采用“三次握手”，那么只要server发出确认，
新的连接就建立了。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送数据。
但server却以为新的运输连接已经建立，并一直等待client发来数据。这样，server的很多资源就白白浪费掉了。
采用“三次握手”的办法可以防止上述现象发生。例如刚才那种情况，client不会向server的确认发出确认。server由于收不到确认，就知道client并没有要求建立连接。”

### HTTP和HTTPS的区别
HTTP缺点：
1. 通信使用明文（不加密），内容可能会被窃听
2. 不验证通信方的身份，因此有可能遭遇伪装
3. 无法证明报文的完整性，所以有可能已遭篡改

由于以上HTTP的不足，所以引进了HTTPS。（HTTP+加密+认证即证书+完整性保护 = HTTPS）

HTTPS不是应用层一个新协议，只是HTTP通信接口部分用SSL（Secure Socket Layer，安全套接层）和TLS（Transport Layer Security，安全传输层协议），
通常，HTTP直接和TCP通信。当使用了SSL时，则演变成先和SSL通信，再由SSL和TCP通信了。
简言之，所谓HTTPS，其实就是身披SSL协议这层外壳的HTTP。

HTTPS采用的*混合加密机制（公开加密方式和共享加密方式）*。
其中*共享加密方式*就是通过相同的密钥进行加密和解密，而*公开加密方式*就是通过一把公开密钥和一把私有密钥分别进行加密和解密。
由于公开密钥加密处理起来比共享密钥加密方式更加复杂，因此若在通信时使用公开密钥加密，效率会很低。
HTTPS的混合加密机制充分利用了两种加密方式，**通过使用公开加密方式来传送共享加密密钥来进行后续的请求和响应。**

服务器的私有密钥就是通过申请CA证书来获得，后面将会有申请证书来实现HTTPS的实例。

!["HTTPS']({{ site.baseurl }}/assets/img/HTTPS.jpg)

通过上述的图片可以看出，在申请CA证书的时候，将会获得专属的私有密钥，
用户在登录某个网站的时候，浏览器将会通过从该站点服务器中获取相应的公开密钥，
然后发送后续需要用到的共享密钥，服务器接收到该请求后，通过私有密钥来解密，
后面的请求和响应将会使用只有两者知道的共享密钥来通信。

### 在网站中实现HTTPS

在本人<a href="https://www.bnujob.cn">BNUJOB</a>这个站点中，我将会使用Let's Encrypt申请证书。

步骤1：登录<a href="https://letsencrypt.org">Let's Encrypt</a>的官网，
在Documents中可以看到他将会使用<a href="https://certbot.eff.org">certbot</a>来自动获取和配置站点的证书

步骤2：选择你服务器的操作系统以及服务器引擎，页面上将会显示一系列的操作指令（操作系统我使用的是Ubantu16.4 服务器引擎我使用的是Nginx）。
![certbot]({{ site.baseurl }}/assets/img/certbot.png)

步骤3：通过上面的指令进行操作，需要注意的是，在运行`sudo certbot`或者`certbot certonly --webroot -w /var/www/example/`指令的时候，
指定的站点目录需要是能够访问的到根目录，因为该插件将会生成新的文件并且访问。(因为我使用的Laravel框架，需要指定public目录才能运行成功)

步骤4：更改nginx的配置文件，修改监听443接口以及指定key文件路径（这一点certbot插件应该会帮助实现，但是由于中间的操作问题，我还得自己配置）

最后就能看到自己的网站配置成功，在Chrome上有一个绿色钥匙的标志啦！
![bnujob]({{ site.baseurl }}/assets/img/bnujob.png)

*注：虽然说是傻瓜式的操作，但是还是花了我一个早上的时间，所以说配置的过程还是要耐心和细心*




