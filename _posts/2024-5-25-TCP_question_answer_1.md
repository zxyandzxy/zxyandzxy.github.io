---
layout: post
title: "TCP问题回答1"
date: 2024-5-25
tags: [network]
comments: true
author: zxy
---

## 如何理解是 TCP 面向字节流协议？

### 如何理解字节流？

之所以会说 TCP 是面向字节流的协议，UDP 是面向报文的协议，是因为操作系统对 TCP 和 UDP 协议的**发送方的机制不同**，也就是问题原因在发送方。

> 为什么 UDP 是面向报文的协议？

当用户消息通过 UDP 协议传输时，**操作系统不会对消息进行拆分**，在组装好 UDP 头部后就交给网络层来处理，所以发出去的 UDP 报文中的数据部分就是完整的用户消息，也就是**每个 UDP 报文就是一个用户消息的边界**，这样接收方在接收到 UDP 报文后，读一个 UDP 报文就能读取到完整的用户消息。

你可能会问，如果收到了两个 UDP 报文，操作系统是怎么区分开的？

操作系统在收到 UDP 报文后，会将其插入到队列里，**队列里的每一个元素就是一个 UDP 报文**，这样当用户调用 recvfrom() 系统调用读数据的时候，就会从队列里取出一个数据，然后从内核里拷贝给用户缓冲区。

![图片](https://cdn.xiaolincoding.com//mysql/other/a9116c5b375d356048df033dcb53582e.png)

> 为什么 TCP 是面向字节流的协议？

当用户消息通过 TCP 协议传输时，**消息可能会被操作系统分组成多个的 TCP 报文**，也就是一个完整的用户消息被拆分成多个 TCP 报文进行传输。

这时，接收方的程序如果不知道发送方发送的消息的长度，也就是不知道消息的边界时，是无法读出一个有效的用户消息的，因为用户消息被拆分成多个 TCP 报文后，并不能像 UDP 那样，一个 UDP 报文就能代表一个完整的用户消息。

举个实际的例子来说明。

发送方准备发送 「Hi.」和「I am Xiaolin」这两个消息。

在发送端，当我们调用 send 函数完成数据“发送”以后，数据并没有被真正从网络上发送出去，只是从应用程序拷贝到了操作系统内核协议栈中。

至于什么时候真正被发送，**取决于发送窗口、拥塞窗口以及当前发送缓冲区的大小等条件**。也就是说，我们不能认为每次 send 调用发送的数据，都会作为一个整体完整地消息被发送出去。

如果我们考虑实际网络传输过程中的各种影响，假设发送端陆续调用 send 函数先后发送 「Hi.」和「I am Xiaolin」 报文，那么实际的发送很有可能是这几种情况。

第一种情况，这两个消息被分到同一个 TCP 报文，像这样：

![图片](https://cdn.xiaolincoding.com//mysql/other/02dce678f870c8c70482b6e37dbb5574.png)

第二种情况，「I am Xiaolin」的部分随 「Hi」 在一个 TCP 报文中发送出去，像这样：

![图片](https://cdn.xiaolincoding.com//mysql/other/f58b70cde860188b8f95a433e2f5293b.png)

第三种情况，「Hi.」 的一部分随 TCP 报文被发送出去，另一部分和 「I am Xiaolin」 一起随另一个 TCP 报文发送出去，像这样。

![图片](https://cdn.xiaolincoding.com//mysql/other/68080e783d7acc842fa254e4f9ec5630.png)

类似的情况还能举例很多种，这里主要是想说明，我们不知道 「Hi.」和 「I am Xiaolin」 这两个用户消息是如何进行 TCP 分组传输的。

因此，**我们不能认为一个用户消息对应一个 TCP 报文，正因为这样，所以 TCP 是面向字节流的协议**。

当两个消息的某个部分内容被分到同一个 TCP 报文时，就是我们常说的 TCP 粘包问题，这时接收方不知道消息的边界的话，是无法读出有效的消息。

要解决这个问题，要交给**应用程序**。

### 如何解决粘包？

粘包的问题出现是因为不知道一个用户消息的边界在哪，如果知道了边界在哪，接收方就可以通过边界来划分出有效的用户消息。

一般有三种方式分包的方式：

- 固定长度的消息；
- 特殊字符作为边界；
- 自定义消息结构。

#### 固定长度的消息

这种是最简单方法，即每个用户消息都是固定长度的，比如规定一个消息的长度是 64 个字节，当接收方接满 64 个字节，就认为这个内容是一个完整且有效的消息。

但是这种方式灵活性不高，实际中很少用。

#### 特殊字符作为边界

我们可以在两个用户消息之间插入一个特殊的字符串，这样接收方在接收数据时，读到了这个特殊字符，就把认为已经读完一个完整的消息。

HTTP 是一个非常好的例子。

![图片](https://cdn.xiaolincoding.com//mysql/other/a49a6bb8cd38ae1738d9c00aec68b444.png)

HTTP 通过设置回车符、换行符作为 HTTP 报文协议的边界。

有一点要注意，这个作为边界点的特殊字符，如果刚好消息内容里有这个特殊字符，我们要对这个字符转义，避免被接收方当作消息的边界点而解析到无效的数据。

#### 自定义消息结构

我们可以自定义一个消息结构，由包头和数据组成，其中包头包是固定大小的，而且包头里有一个字段来说明紧随其后的数据有多大。

比如这个消息结构体，首先 4 个字节大小的变量来表示数据长度，真正的数据则在后面。

```c
struct {
    u_int32_t message_length;
    char message_data[];
} message;
```

当接收方接收到包头的大小（比如 4 个字节）后，就解析包头的内容，于是就可以知道数据的长度，然后接下来就继续读取数据，直到读满数据的长度，就可以组装成一个完整到用户消息来处理了。

## 为什么 TCP 每次建立连接时，初始化序列号都要不一样呢？

主要原因是为了防止历史报文被下一个相同四元组的连接接收。

> TCP 四次挥手中的 TIME_WAIT 状态不是会持续 2 MSL 时长，历史报文不是早就在网络中消失了吗？

是的，如果能正常四次挥手，由于 TIME_WAIT 状态会持续 2 MSL 时长，历史报文会在下一个连接之前就会自然消失。

但是来了，我们并不能保证每次连接都能通过四次挥手来正常关闭连接。

假设每次建立连接，客户端和服务端的初始化序列号都是从 0 开始：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/network/tcp/isn%E7%9B%B8%E5%90%8C.png)

过程如下：

- 客户端和服务端建立一个 TCP 连接，在客户端发送数据包被网络阻塞了，然后超时重传了这个数据包，而此时服务端设备断电重启了，之前与客户端建立的连接就消失了，于是在收到客户端的数据包的时候就会发送 RST 报文。
- 紧接着，客户端又与服务端建立了与上一个连接相同四元组的连接；
- 在新连接建立完成后，上一个连接中被网络阻塞的数据包正好抵达了服务端，刚好该数据包的序列号正好是在服务端的接收窗口内，所以该数据包会被服务端正常接收，就会造成数据错乱。

可以看到，如果每次建立连接，客户端和服务端的初始化序列号都是一样的话，很容易出现历史报文被下一个相同四元组的连接接收的问题。

> 客户端和服务端的初始化序列号不一样不是也会发生这样的事情吗？

是的，即使客户端和服务端的初始化序列号不一样，也会存在收到历史报文的可能。

但是我们要清楚一点，历史报文能否被对方接收，还要看该历史报文的序列号是否正好在对方接收窗口内，如果不在就会丢弃，如果在才会接收。

如果每次建立连接客户端和服务端的初始化序列号都「不一样」，就有大概率因为历史报文的序列号「不在」对方接收窗口，从而很大程度上避免了历史报文，比如下图：

![isn不同](https://cdn.xiaolincoding.com//picgo/isn%E4%B8%8D%E5%90%8C.png)

相反，如果每次建立连接客户端和服务端的初始化序列号都「一样」，就有大概率遇到历史报文的序列号刚「好在」对方的接收窗口内，从而导致历史报文被新连接成功接收。

所以，每次初始化序列号不一样能够很大程度上避免历史报文被下一个相同四元组的连接接收，注意是很大程度上，并不是完全避免了。

> 那客户端和服务端的初始化序列号都是随机的，那还是有可能随机成一样的呀？

RFC793 提到初始化序列号 ISN 随机生成算法：ISN = M + F(localhost, localport, remotehost, remoteport)。

- M 是一个计时器，这个计时器每隔 4 微秒加 1。
- F 是一个 Hash 算法，根据源 IP、目的 IP、源端口、目的端口生成一个随机数值，要保证 hash 算法不能被外部轻易推算得出。

可以看到，随机数是会基于时钟计时器递增的，基本不可能会随机成一样的初始化序列号。

> 客户端和服务端初始化序列号都是随机生成的话，就能避免连接接收历史报文了吗？

是的，但是也不是完全避免了。

为了能更好的理解这个原因，我们先来了解序列号（SEQ）和初始序列号（ISN）。

- **序列号**，是 TCP 一个头部字段，标识了 TCP 发送端到 TCP 接收端的数据流的一个字节，因为 TCP 是面向字节流的可靠协议，为了保证消息的顺序性和可靠性，TCP 为每个传输方向上的每个字节都赋予了一个编号，以便于传输成功后确认、丢失后重传以及在接收端保证不会乱序。**序列号是一个 32 位的无符号数，因此在到达 4G 之后再循环回到 0**。
- **初始序列号**，在 TCP 建立连接的时候，客户端和服务端都会各自生成一个初始序列号，它是基于时钟生成的一个随机数，来保证每个连接都拥有不同的初始序列号。**初始化序列号可被视为一个 32 位的计数器，该计数器的数值每 4 微秒加 1，循环一次需要 4.55 小时**。

给大家抓了一个包，下图中的 Seq 就是序列号，其中红色框住的分别是客户端和服务端各自生成的初始序列号。

![img](https://cdn.xiaolincoding.com//mysql/other/ed84bb4aa742a33f50d8035da2867ca2.png)

通过前面我们知道，**序列号和初始化序列号并不是无限递增的，会发生回绕为初始值的情况，这意味着无法根据序列号来判断新老数据**。

不要以为序列号的上限值是 4GB，就以为很大，很难发生回绕。在一个速度足够快的网络中传输大量数据时，序列号的回绕时间就会变短。如果序列号回绕的时间极短，我们就会再次面临之前延迟的报文抵达后序列号依然有效的问题。

为了解决这个问题，就需要有 TCP 时间戳。tcp_timestamps 参数是默认开启的，开启了 tcp_timestamps 参数，TCP 头部就会使用时间戳选项，它有两个好处，**一个是便于精确计算 RTT ，另一个是能防止序列号回绕（PAWS）**。

试看下面的示例，假设 TCP 的发送窗口是 1 GB，并且使用了时间戳选项，发送方会为每个 TCP 报文分配时间戳数值，我们假设每个报文时间加 1，然后使用这个连接传输一个 6GB 大小的数据流。

![图片](https://cdn.xiaolincoding.com//mysql/other/1d497c38621ebc44ee3d8763fd03da67.png)

32 位的序列号在时刻 D 和 E 之间回绕。假设在时刻 B 有一个报文丢失并被重传，又假设这个报文段在网络上绕了远路并在时刻 F 重新出现。如果 TCP 无法识别这个绕回的报文，那么数据完整性就会遭到破坏。

使用时间戳选项能够有效的防止上述问题，如果丢失的报文会在时刻 F 重新出现，由于它的时间戳为 2，小于最近的有效时间戳（5 或 6），因此防回绕序列号算法（PAWS）会将其丢弃。

防回绕序列号算法要求连接双方维护最近一次收到的数据包的时间戳（Recent TSval），每收到一个新数据包都会读取数据包中的时间戳值跟 Recent TSval 值做比较，**如果发现收到的数据包中时间戳不是递增的，则表示该数据包是过期的，就会直接丢弃这个数据包**。

> 懂了，客户端和服务端的初始化序列号都是随机生成，能很大程度上避免历史报文被下一个相同四元组的连接接收，然后又引入时间戳的机制，从而完全避免了历史报文被接收的问题。

## SYN 报文什么时候情况下会被丢弃？

两种情况下会被丢弃：

- 开启 tcp_tw_recycle 参数，并且在 NAT 环境下，造成 SYN 报文被丢弃
- TCP 两个队列满了（半连接队列和全连接队列），造成 SYN 报文被丢弃

### 坑爹的 tcp_tw_recycle

TCP 四次挥手过程中，主动断开连接方会有一个 TIME_WAIT 的状态，这个状态会持续 2 MSL 后才会转变为 CLOSED 状态。

![img](https://cdn.xiaolincoding.com//mysql/other/bee0c8e8d84047e7434803fb340f9e5d.png)

在 Linux 操作系统下，TIME_WAIT 状态的持续时间是 60 秒，这意味着这 60 秒内，客户端一直会占用着这个端口。要知道，端口资源也是有限的，一般可以开启的端口为 32768~61000 ，也可以通过如下参数设置指定范围：

```text
 net.ipv4.ip_local_port_range
```

**如果客户端（发起连接方）的 TIME_WAIT 状态过多**，占满了所有端口资源，那么就无法对「目的 IP+ 目的 PORT」都一样的服务器发起连接了，但是被使用的端口，还是可以继续对另外一个服务器发起连接的。

因此，客户端（发起连接方）都是和「目的 IP+ 目的 PORT 」都一样的服务器建立连接的话，当客户端的 TIME_WAIT 状态连接过多的话，就会受端口资源限制，如果占满了所有端口资源，那么就无法再跟「目的 IP+ 目的 PORT」都一样的服务器建立连接了。

不过，即使是在这种场景下，只要连接的是不同的服务器，端口是可以重复使用的，所以客户端还是可以向其他服务器发起连接的，这是因为内核在定位一个连接的时候，是通过四元组（源 IP、源端口、目的 IP、目的端口）信息来定位的，并不会因为客户端的端口一样，而导致连接冲突。

但是 TIME_WAIT 状态也不是摆设作用，它的作用有两个：

- 防止具有相同四元组的旧数据包被收到，也就是防止历史连接中的数据，被后面的连接接受，否则就会导致后面的连接收到一个无效的数据，
- 保证「被动关闭连接」的一方能被正确的关闭，即保证最后的 ACK 能让被动关闭方接收，从而帮助其正常关闭;

不过，Linux 操作系统提供了两个可以系统参数来快速回收处于 TIME_WAIT 状态的连接，这两个参数都是默认关闭的：

- net.ipv4.tcp_tw_reuse，如果开启该选项的话，客户端（连接发起方） 在调用 connect() 函数时，**如果内核选择到的端口，已经被相同四元组的连接占用的时候，就会判断该连接是否处于 TIME_WAIT 状态，如果该连接处于 TIME_WAIT 状态并且 TIME_WAIT 状态持续的时间超过了 1 秒，那么就会重用这个连接，然后就可以正常使用该端口了。**所以该选项只适用于连接发起方。
- net.ipv4.tcp_tw_recycle，如果开启该选项的话，允许处于 TIME_WAIT 状态的连接被快速回收；

要使得这两个选项生效，有一个前提条件，就是要打开 TCP 时间戳，即 net.ipv4.tcp_timestamps=1（默认即为 1)）。

**tcp_tw_recycle 在使用了 NAT 的网络下是不安全的！**

对于服务器来说，如果同时开启了 recycle 和 timestamps 选项，则会开启一种称之为「 per-host 的 PAWS 机制」。

> 什么是 PAWS 机制？

tcp_timestamps 选项开启之后， PAWS 机制会自动开启，它的作用是防止 TCP 包中的序列号发生绕回。

正常来说每个 TCP 包都会有自己唯一的 SEQ，出现 TCP 数据包重传的时候会复用 SEQ 号，这样接收方能通过 SEQ 号来判断数据包的唯一性，也能在重复收到某个数据包的时候判断数据是不是重传的。**但是 TCP 这个 SEQ 号是有限的，一共 32 bit，SEQ 开始是递增，溢出之后从 0 开始再次依次递增**。

所以当 SEQ 号出现溢出后单纯通过 SEQ 号无法标识数据包的唯一性，某个数据包延迟或因重发而延迟时可能导致连接传递的数据被破坏，比如：

![img](https://cdn.xiaolincoding.com//mysql/other/f5fbe947240026cc2f076267cb698496.png)

上图 A 数据包出现了重传，并在 SEQ 号耗尽再次从 A 递增时，第一次发的 A 数据包延迟到达了 Server，这种情况下如果没有别的机制来保证，Server 会认为延迟到达的 A 数据包是正确的而接收，反而是将正常的第三次发的 SEQ 为 A 的数据包丢弃，造成数据传输错误。

PAWS 就是为了避免这个问题而产生的，在开启 tcp_timestamps 选项情况下，一台机器发的所有 TCP 包都会带上发送时的时间戳，PAWS 要求连接双方维护最近一次收到的数据包的时间戳（Recent TSval），每收到一个新数据包都会读取数据包中的时间戳值跟 Recent TSval 值做比较，**如果发现收到的数据包中时间戳不是递增的，则表示该数据包是过期的，就会直接丢弃这个数据包**。

对于上面图中的例子有了 PAWS 机制就能做到在收到 Delay 到达的 A 号数据包时，识别出它是个过期的数据包而将其丢掉。

> 什么是 per-host 的 PAWS 机制呢？

**per-host 是对「对端 IP 做 PAWS 检查」**，而非对「IP + 端口」四元组做 PAWS 检查。

但是如果客户端网络环境是用了 NAT 网关，那么客户端环境的每一台机器通过 NAT 网关后，都会是相同的 IP 地址，在服务端看来，就好像只是在跟一个客户端打交道一样，无法区分出来。

Per-host PAWS 机制利用 TCP option 里的 timestamp 字段的增长来判断串扰数据，而 timestamp 是根据客户端各自的 CPU tick 得出的值。

当客户端 A 通过 NAT 网关和服务器建立 TCP 连接，然后服务器主动关闭并且快速回收 TIME-WAIT 状态的连接后，**客户端 B 也通过 NAT 网关和服务器建立 TCP 连接，注意客户端 A 和 客户端 B 因为经过相同的 NAT 网关，所以是用相同的 IP 地址与服务端建立 TCP 连接，如果客户端 B 的 timestamp 比 客户端 A 的 timestamp 小，那么由于服务端的 per-host 的 PAWS 机制的作用，服务端就会丢弃客户端主机 B 发来的 SYN 包**。

因此，tcp_tw_recycle 在使用了 NAT 的网络下是存在问题的，如果它是对 TCP 四元组做 PAWS 检查，而不是对「相同的 IP 做 PAWS 检查」，那么就不会存在这个问题了。

tcp_tw_recycle 在 Linux 4.12 版本后，直接取消了这一参数。

### accpet 队列满了

在 TCP 三次握手的时候，Linux 内核会维护两个队列，分别是：

- 半连接队列，也称 SYN 队列；
- 全连接队列，也称 accepet 队列；

服务端收到客户端发起的 SYN 请求后，**内核会把该连接存储到半连接队列**，并向客户端响应 SYN+ACK，接着客户端会返回 ACK，服务端收到第三次握手的 ACK 后，**内核会把连接从半连接队列移除，然后创建新的完全的连接，并将其添加到 accept 队列，等待进程调用 accept 函数时把连接取出来。**

![img](https://cdn.xiaolincoding.com//mysql/other/c9959166180b0e239bb48234ff7c2f5b.png)

#### 半连接队列满了

当服务器造成 syn 攻击，就有可能导致 **TCP 半连接队列满了，这时后面来的 syn 包都会被丢弃**。

但是，**如果开启了 syncookies 功能，即使半连接队列满了，也不会丢弃 syn 包**。

syncookies 是这么做的：服务器根据当前状态计算出一个值，放在己方发出的 SYN+ACK 报文中发出，当客户端返回 ACK 报文时，取出该值验证，如果合法，就认为连接建立成功。

#### 全连接队列满了

**在服务端并发处理大量请求时，如果 TCP accpet 队列过小，或者应用程序调用 accept() 不及时，就会造成 accpet 队列满了 ，这时后续的连接就会被丢弃，这样就会出现服务端请求数量上不去的现象。**

![img](https://cdn.xiaolincoding.com//mysql/other/d1538f8d3b50da26039bc6b171a13ad1.png)

## 已建立连接的 TCP，收到 SYN 会发生什么？

> 问题：一个已经建立的 TCP 连接，客户端中途宕机了，而服务端此时也没有数据要发送，一直处于 Established 状态，客户端恢复后，向服务端建立连接，此时服务端会怎么处理？

这个场景中，客户端的 IP、服务端 IP、目的端口并没有变化，所以这个问题关键要看客户端发送的 SYN 报文中的源端口是否和上一次连接的源端口相同。

**1.客户端的 SYN 报文里的端口号与历史连接不相同**

如果客户端恢复后发送的 SYN 报文中的源端口号跟上一次连接的源端口号不一样，此时服务端会认为是新的连接要建立，于是就会通过三次握手来建立新的连接。

那旧连接里处于 Established 状态的服务端最后会怎么样呢？

如果服务端发送了数据包给客户端，由于客户端的连接已经被关闭了，此时客户的内核就会回 RST 报文，服务端收到后就会释放连接。

如果服务端一直没有发送数据包给客户端，在超过一段时间后，TCP 保活机制就会启动，检测到客户端没有存活后，接着服务端就会释放掉该连接。

**2.客户端的 SYN 报文里的端口号与历史连接相同**

如果客户端恢复后，发送的 SYN 报文中的源端口号跟上一次连接的源端口号一样，也就是处于 Established 状态的服务端收到了这个 SYN 报文。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/est_syn.png)

**处于 Established 状态的服务端，如果收到了客户端的 SYN 报文（注意此时的 SYN 报文其实是乱序的，因为 SYN 报文的初始化序列号其实是一个随机数），会回复一个携带了正确序列号和确认号的 ACK 报文，这个 ACK 被称之为 Challenge ACK。**

**接着，客户端收到这个 Challenge ACK，发现确认号（ack num）并不是自己期望收到的，于是就会回 RST 报文，服务端收到后，就会释放掉该连接。**

### 源码分析

处于 Established 状态的服务端如果收到了客户端的 SYN 报文时，内核会调用这些函数：

```csharp
tcp_v4_rcv
  -> tcp_v4_do_rcv
    -> tcp_rcv_Establisheded
      -> tcp_validate_incoming
        -> tcp_send_ack
```

我们只关注 tcp_validate_incoming 函数是怎么处理 SYN 报文的，精简后的代码如下：

![img](https://cdn.xiaolincoding.com//mysql/other/780bc02c8fa940c0a320a5916b216c21.png)

从上面的代码实现可以看到，处于 Established 状态的服务端，在收到报文后，首先会判断序列号是否在窗口内，如果不在，则看看 RST 标记有没有被设置，如果有就会丢掉。然后如果没有 RST 标志，就会判断是否有 SYN 标记，如果有 SYN 标记就会跳转到 syn_challenge 标签，然后执行 tcp_send_challenge_ack 函数。

tcp_send_challenge_ack 函数里就会调用 tcp_send_ack 函数来回复一个携带了正确序列号和确认号的 ACK 报文。

## 如何关闭一个 TCP 连接？

> 直接杀死进程可行吗？

是的，这个是最粗暴的方式，杀掉客户端进程和服务端进程影响的范围会有所不同：

- 在客户端杀掉进程的话，就会发送 FIN 报文，来断开这个客户端进程与服务端建立的所有 TCP 连接，这种方式影响范围只有这个客户端进程所建立的连接，而其他客户端或进程不会受影响。
- 而在服务端杀掉进程影响就大了，此时所有的 TCP 连接都会被关闭，服务端无法继续提供访问服务。

所以，关闭进程的方式并不可取，最好的方式要精细到关闭某一条 TCP 连接。

有的小伙伴可能会说，伪造一个四元组相同的 RST 报文不就行了？

这个思路很好，但是不要忘了还有个序列号的问题，你伪造的 RST 报文的序列号一定能被对方接受吗？

如果 RST 报文的序列号不是对方期望收到的序列号，这个 RST 报文会被对方丢弃的，就达不到关闭的连接的效果。

举个例子，下面这个场景，客户端发送了一个长度为 100 的 TCP 数据报文，服务端收到后响应了 ACK 报文，表示收到了这个 TCP 数据报文。**服务端响应的这个 ACK 报文中的确认号（ack = x + 100）就是表明服务端下一次期望收到的序列号是 x + 100**。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/rst%E5%90%88%E6%B3%95.png)

所以，**要伪造一个能关闭 TCP 连接的 RST 报文，必须同时满足「四元组相同」和「序列号是对方期望的」这两个条件。**

直接伪造符合预期的序列号是比较困难，因为如果一个正在传输数据的 TCP 连接，序列号都是时刻都在变化，因此很难刚好伪造一个正确序列号的 RST 报文。

### killcx 的工具

办法还是有的，**我们可以伪造一个四元组相同的 SYN 报文，来拿到“合法”的序列号！**

正如我们最开始学到的，如果处于 Established 状态的服务端，收到四元组相同的 SYN 报文后，**会回复一个 Challenge ACK，这个 ACK 报文里的「确认号」，正好是服务端下一次想要接收的序列号，说白了，就是可以通过这一步拿到服务端下一次预期接收的序列号。**

**然后用这个确认号作为 RST 报文的序列号，发送给服务端，此时服务端会认为这个 RST 报文里的序列号是合法的，于是就会释放连接！**

在 Linux 上有个叫 killcx 的工具，就是基于上面这样的方式实现的，它会主动发送 SYN 包获取 SEQ/ACK 号，然后利用 SEQ/ACK 号伪造两个 RST 报文分别发给客户端和服务端，这样双方的 TCP 连接都会被释放，这种方式活跃和非活跃的 TCP 连接都可以杀掉。

killcx 的工具使用方式也很简单，如果在服务端执行 killcx 工具，只需指明客户端的 IP 和端口号，如果在客户端执行 killcx 工具，则就指明服务端的 IP 和端口号。

```csharp
./killcx <IP地址>:<端口号>
```

killcx 工具的工作原理，如下图，下图是在客户端执行 killcx 工具。

![img](https://cdn.xiaolincoding.com//mysql/other/95592346a9a747819cd27741a660213c.png)

它伪造客户端发送 SYN 报文，服务端收到后就会回复一个携带了正确「序列号和确认号」的 ACK 报文（Challenge ACK），然后就可以利用这个 ACK 报文里面的信息，伪造两个 RST 报文：

- 用 Challenge ACK 里的确认号伪造 RST 报文发送给服务端，服务端收到 RST 报文后就会释放连接。
- 用 Challenge ACK 里的序列号伪造 RST 报文发送给客户端，客户端收到 RST 也会释放连接。

正是通过这样的方式，成功将一个 TCP 连接关闭了！

### tcpkill 的工具

除了 killcx 工具能关闭 TCP 连接，还有 tcpkill 工具也可以做到。

这两个工具都是通过伪造 RST 报文来关闭指定的 TCP 连接，但是它们拿到正确的序列号的实现方式是不同的。

- tcpkill 工具是在双方进行 TCP 通信时，拿到对方下一次期望收到的序列号，然后将序列号填充到伪造的 RST 报文，并将其发送给对方，达到关闭 TCP 连接的效果。
- killcx 工具是主动发送一个 SYN 报文，对方收到后会回复一个携带了正确序列号和确认号的 ACK 报文，这个 ACK 被称之为 Challenge ACK，这时就可以拿到对方下一次期望收到的序列号，然后将序列号填充到伪造的 RST 报文，并将其发送给对方，达到关闭 TCP 连接的效果。

可以看到， 这两个工具在获取对方下一次期望收到的序列号的方式是不同的。

tcpkill 工具属于被动获取，就是在双方进行 TCP 通信的时候，才能获取到正确的序列号，很显然**这种方式无法关闭非活跃的 TCP 连接**，只能用于关闭活跃的 TCP 连接。因为如果这条 TCP 连接一直没有任何数据传输，则就永远获取不到正确的序列号。

killcx 工具则是属于主动获取，它是主动发送一个 SYN 报文，通过对方回复的 Challenge ACK 来获取正确的序列号，所以这种方式**无论 TCP 连接是否活跃，都可以关闭**。

> tcpkill 实验

用 nc 工具来模拟一个 TCP 服务端，监听 8888 端口。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/tcpkill/tcpkill1.png)

接着，在客户端机子上，用 nc 工具模拟一个 TCP 客户端，连接我们刚才启动的服务端，并且指定了客户端的端口为 11111。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/tcpkill/tcpkill2.png)

这时候， 服务端就可以看到这条 TCP 连接了。

![图片](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/tcpkill/tcpkill3.png)

注意，我这台服务端的公网 IP 地址是 121.43.173.240，私网 IP 地址是 172.19.11.21，在服务端通过 netstat 命令查看 TCP 连接的时候，则会将服务端的地址显示成私网 IP 地址 。至此，我们前期工作就做好了。

接下来，我们在服务端执行 tcpkill 工具，来关闭这条 TCP 连接，看看会发生什么？

在这里，我指定了要关闭的客户端 IP 为 114.132.166.90 和端口为 11111 的 TCP 连接。

![图片](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/tcpkill/tcpkill4.png)

可以看到，tcpkill 工具阻塞中，没有任何输出，而且此时的 TCP 连接还是存在的，并没有被干掉。

![图片](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/tcpkill/tcpkill5.png)

为什么 TCP 连接没用被干掉？

因为在执行 tcpkill 工具后，这条 TCP 连接并没有传输任何数据，而 tcpkill 工具是需要拦截双方的 TCP 通信，才能获取到正确的序列号，从而才能伪装出正确的序列号的 RST 报文。

所以，从这里也说明了，**tcpkill 工具不适合关闭非活跃的 TCP 连接**。

接下来，我们尝试在客户端发送一个数据。

![图片](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/tcpkill/tcpkill8.png)

可以看到，在发送了「hi」数据后，客户端就断开了，并且错误提示连接被对方关闭了。

此时，服务端已经查看不到刚才那条 TCP 连接了。

![图片](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/tcpkill/tcpkill7.png)

然后，我们在服务端看看 tcpkill 工具输出的信息。

![图片](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/tcpkill/tcpkill8.png)

可以看到， **tcpkill 工具给服务端和客户端都发送了伪造的 RST 报文，从而达到关闭一条 TCP 连接的效果**。

到这里我们知道了， 运行 tcpkill 工具后，只有目标连接有新 TCP 包发送/接收的时候，才能关闭一条 TCP 连接。因此，**tcpkill 只适合关闭活跃的 TCP 连接，不适合用来关闭非活跃的 TCP 连接**。

上面的实验过程，抓了数据包，流程如下：

![图片](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/tcpkill/tcpkill9.png)

最后一个 RST 报文就是 tcpkill 工具伪造的 RST 报文。

## 四次挥手中收到乱序的 FIN 包会如何处理？

> **在 FIN_WAIT_2 状态下，是如何处理收到的乱序到 FIN 报文，然后 TCP 连接又是什么时候才进入到 TIME_WAIT 状态?**

**在 FIN_WAIT_2 状态时，如果收到乱序的 FIN 报文，那么就被会加入到「乱序队列」，并不会进入到 TIME_WAIT 状态。**

**等再次收到前面被网络延迟的数据包时，会判断乱序队列有没有数据，然后会检测乱序队列中是否有可用的数据，如果能在乱序队列中找到与当前报文的序列号保持的顺序的报文，就会看该报文是否有 FIN 标志，如果发现有 FIN 标志，这时才会进入 TIME_WAIT 状态。**

![img](https://cdn.xiaolincoding.com//mysql/other/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5p6XY29kaW5n,size_20,color_FFFFFF,t_70,g_se,x_16-20230309230147654.png)

### TCP 源码分析

在 Linux 内核里，当 IP 层处理完消息后，会通过回调 tcp_v4_rcv 函数将消息转给 TCP 层，所以这个函数就是 TCP 层收到消息的入口。

![img](https://cdn.xiaolincoding.com//mysql/other/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5p6XY29kaW5n,size_20,color_FFFFFF,t_70,g_se,x_16.jpeg)

处于 FIN_WAIT_2 状态下的客户端，在收到服务端的报文后，最终会调用 tcp_v4_do_rcv 函数。

![img](https://cdn.xiaolincoding.com//mysql/other/c5ca5b3fea0e4ad6baa2ab370358f03e.jpg)

接下来，tcp_v4_do_rcv 方法会调用 tcp_rcv_state_process，在这里会根据 TCP 状态做对应的处理，这里我们只关注 FIN_WAIT_2 状态。

![img](https://cdn.xiaolincoding.com//mysql/other/f76b7e2167544fec859700f55138e95f.jpg)

在上面这个代码里，可以看到如果 shutdown 关闭了读方向，那么在收到对方发来的数据包，则会回复 RST 报文。

而我们这次的题目里， shutdown 只关闭了写方向，所以会继续往下调用 tcp_data_queue 函数（因为 case TCP_FIN_WAIT2 代码块里并没有 break 语句，所以会走到该函数）。

![img](https://cdn.xiaolincoding.com//mysql/other/4ff161a34408447fa38b120b014b29f4.jpg)

在上面的 tcp_data_queue 函数里，如果收到的报文的序列号是我们预期的，也就是有序的话：

- 会判断该报文有没有 FIN 标志，如果有的话就会调用 tcp_fin 函数，这个函数负责将 FIN_WAIT_2 状态转换为 TIME_WAIT。
- 接着还会看乱序队列有没有数据，如果有的话会调用 tcp_ofo_queue 函数，这个函数负责检查乱序队列中是否有数据包可用，即能不能在乱序队列找到与当前数据包保持序列号连续的数据包。

而当收到的报文的序列号不是我们预期的，也就是乱序的话，则调用 tcp_data_queue_ofo 函数，将报文加入到乱序队列，这个队列的数据结构是红黑树。

我们的题目里，客户端收到的 FIN 报文实际上是一个乱序的报文，因此此时并不会调用 tcp_fin 函数进行状态转换，而是将报文通过 tcp_data_queue_ofo 函数加入到乱序队列。

然后当客户端收到被网络延迟的数据包后，此时因为该数据包的序列号是期望的，然后又因为上一次收到的乱序 FIN 报文被加入到了乱序队列，表明乱序队列是有数据的，于是就会调用 tcp_ofo_queue 函数。

我们来看看 tcp_ofo_queue 函数。

![img](https://cdn.xiaolincoding.com//mysql/other/dd51b407245d45549eeae64d24634133.jpg)

在上面的 tcp_ofo_queue 函数里，在乱序队列中找到能与当前报文的序列号保持的顺序的报文后，会看该报文是否有 FIN 标志，如果有的话，就会调用 tcp_fin() 函数。

最后，我们来看看 tcp_fin 函数的处理。

![img](https://cdn.xiaolincoding.com//mysql/other/67b33007fcd04d2fa98e79d19823fc95.jpg)

可以看到，如果当前的 TCP 状态为 TCP_FIN_WAIT2，就会发送第四次挥手 ack，然后调用 tcp_time_wait 函数，这个函数里会将 TCP 状态变更为 TIME_WAIT，并启动 TIME_WAIT 的定时器。

## 在 TIME_WAIT 状态的 TCP 连接，收到 相同四元组的 SYN 后会发生什么？

问题现象如下图，左边是服务端，右边是客户端：

![图片](https://cdn.xiaolincoding.com//mysql/other/74b53919396dcda634cfd5b5795cbf16.png)

### 先说结论

针对这个问题，**关键是要看 SYN 的「序列号和时间戳」是否合法**，因为处于 TIME_WAIT 状态的连接收到 SYN 后，会判断 SYN 的「序列号和时间戳」是否合法，然后根据判断结果的不同做不同的处理。

先跟大家说明下， 什么是「合法」的 SYN？

- **合法 SYN**：客户端的 SYN 的「序列号」比服务端「期望下一个收到的序列号」要**大**，**并且** SYN 的「时间戳」比服务端「最后收到的报文的时间戳」要**大**。
- **非法 SYN**：客户端的 SYN 的「序列号」比服务端「期望下一个收到的序列号」要**小**，**或者** SYN 的「时间戳」比服务端「最后收到的报文的时间戳」要**小**。

上面 SYN 合法判断是基于双方都开启了 TCP 时间戳机制的场景，如果双方都没有开启 TCP 时间戳机制，则 SYN 合法判断如下：

- **合法 SYN**：客户端的 SYN 的「序列号」比服务端「期望下一个收到的序列号」要**大**。
- **非法 SYN**：客户端的 SYN 的「序列号」比服务端「期望下一个收到的序列号」要**小**。

#### 收到合法 SYN

如果处于 TIME_WAIT 状态的连接收到「合法的 SYN 」后，**就会重用此四元组连接，跳过 2MSL 而转变为 SYN_RECV 状态，接着就能进行建立连接过程**。

用下图作为例子，双方都启用了 TCP 时间戳机制，TSval 是发送报文时的时间戳：

![图片](https://cdn.xiaolincoding.com//mysql/other/39d0d04adf72fe3d37623acff9ae2507.png)

上图中，在收到第三次挥手的 FIN 报文时，会记录该报文的 TSval （21），用 ts_recent 变量保存。然后会计算下一次期望收到的序列号，本次例子下一次期望收到的序列号就是 301，用 rcv_nxt 变量保存。

处于 TIME_WAIT 状态的连接收到 SYN 后，**因为 SYN 的 seq（400） 大于 rcv_nxt（301），并且 SYN 的 TSval（30） 大于 ts_recent（21），所以是一个「合法的 SYN」，于是就会重用此四元组连接，跳过 2MSL 而转变为 SYN_RECV 状态，接着就能进行建立连接过程。**

#### 收到非法的 SYN

如果处于 TIME_WAIT 状态的连接收到「非法的 SYN 」后，就会**再回复一个第四次挥手的 ACK 报文，客户端收到后，发现并不是自己期望收到确认号（ack num），就回 RST 报文给服务端**。

用下图作为例子，双方都启用了 TCP 时间戳机制，TSval 是发送报文时的时间戳：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/network/tcp/tw%E6%94%B6%E5%88%B0%E4%B8%8D%E5%90%88%E6%B3%95.png)

上图中，在收到第三次挥手的 FIN 报文时，会记录该报文的 TSval （21），用 ts_recent 变量保存。然后会计算下一次期望收到的序列号，本次例子下一次期望收到的序列号就是 301，用 rcv_nxt 变量保存。

处于 TIME_WAIT 状态的连接收到 SYN 后，**因为 SYN 的 seq（200） 小于 rcv_nxt（301），所以是一个「非法的 SYN」，就会再回复一个与第四次挥手一样的 ACK 报文，客户端收到后，发现并不是自己期望收到确认号，就回 RST 报文给服务端**。

### 源码分析

下面源码分析是基于 Linux 4.2 版本的内核代码。

Linux 内核在收到 TCP 报文后，会执行 `tcp_v4_rcv` 函数，在该函数和 TIME_WAIT 状态相关的主要代码如下：

```c
int tcp_v4_rcv(struct sk_buff *skb)
{
  struct sock *sk;
 ...
  //收到报文后，会调用此函数，查找对应的 sock
 sk = __inet_lookup_skb(&tcp_hashinfo, skb, __tcp_hdrlen(th), th->source,
          th->dest, sdif, &refcounted);
 if (!sk)
  goto no_tcp_socket;

process:
  //如果连接的状态为 time_wait，会跳转到 do_time_wait
 if (sk->sk_state == TCP_TIME_WAIT)
  goto do_time_wait;

...

do_time_wait:
  ...
  //由tcp_timewait_state_process函数处理在 time_wait 状态收到的报文
 switch (tcp_timewait_state_process(inet_twsk(sk), skb, th)) {
    // 如果是TCP_TW_SYN，那么允许此 SYN 重建连接
    // 即允许TIM_WAIT状态跃迁到SYN_RECV
    case TCP_TW_SYN: {
      struct sock *sk2 = inet_lookup_listener(....);
      if (sk2) {
          ....
          goto process;
      }
    }
    // 如果是TCP_TW_ACK，那么，返回记忆中的ACK
    case TCP_TW_ACK:
      tcp_v4_timewait_ack(sk, skb);
      break;
    // 如果是TCP_TW_RST直接发送RESET包
    case TCP_TW_RST:
      tcp_v4_send_reset(sk, skb);
      inet_twsk_deschedule_put(inet_twsk(sk));
      goto discard_it;
     // 如果是TCP_TW_SUCCESS则直接丢弃此包，不做任何响应
    case TCP_TW_SUCCESS:;
 }
 goto discard_it;
}
```

该代码的过程：

1. 接收到报文后，会调用 `__inet_lookup_skb()` 函数查找对应的 sock 结构；
2. 如果连接的状态是 `TIME_WAIT`，会跳转到 do_time_wait 处理；
3. 由 `tcp_timewait_state_process()` 函数来处理收到的报文，处理后根据返回值来做相应的处理。

如果收到的 SYN 是合法的，`tcp_timewait_state_process()` 函数就会返回 `TCP_TW_SYN`，然后重用此连接。如果收到的 SYN 是非法的，`tcp_timewait_state_process()` 函数就会返回 `TCP_TW_ACK`，然后会回上次发过的 ACK。

接下来，看 `tcp_timewait_state_process()` 函数是如何判断 SYN 包的。

```c
enum tcp_tw_status
tcp_timewait_state_process(struct inet_timewait_sock *tw, struct sk_buff *skb,
      const struct tcphdr *th)
{
 ...
  //paws_reject 为 false，表示没有发生时间戳回绕
  //paws_reject 为 true，表示发生了时间戳回绕
 bool paws_reject = false;

 tmp_opt.saw_tstamp = 0;
  //TCP头中有选项且旧连接开启了时间戳选项
 if (th->doff > (sizeof(*th) >> 2) && tcptw->tw_ts_recent_stamp) {
  //解析选项
    tcp_parse_options(twsk_net(tw), skb, &tmp_opt, 0, NULL);

  if (tmp_opt.saw_tstamp) {
   ...
      //检查收到的报文的时间戳是否发生了时间戳回绕
   paws_reject = tcp_paws_reject(&tmp_opt, th->rst);
  }
 }

....

  //是SYN包、没有RST、没有ACK、时间戳没有回绕，并且序列号也没有回绕，
 if (th->syn && !th->rst && !th->ack && !paws_reject &&
     (after(TCP_SKB_CB(skb)->seq, tcptw->tw_rcv_nxt) ||
      (tmp_opt.saw_tstamp && //新连接开启了时间戳
       (s32)(tcptw->tw_ts_recent - tmp_opt.rcv_tsval) < 0))) { //时间戳没有回绕
    // 初始化序列号
    u32 isn = tcptw->tw_snd_nxt + 65535 + 2;
    if (isn == 0)
      isn++;
    TCP_SKB_CB(skb)->tcp_tw_isn = isn;
    return TCP_TW_SYN; //允许重用TIME_WAIT四元组重新建立连接
 }


 if (!th->rst) {
    // 如果时间戳回绕，或者报文里包含ack，则将 TIMEWAIT 状态的持续时间重新延长
  if (paws_reject || th->ack)
    inet_twsk_schedule(tw, &tcp_death_row, TCP_TIMEWAIT_LEN,
        TCP_TIMEWAIT_LEN);

     // 返回TCP_TW_ACK, 发送上一次的 ACK
    return TCP_TW_ACK;
 }
 inet_twsk_put(tw);
 return TCP_TW_SUCCESS;
}
```

如果双方启用了 TCP 时间戳机制，就会通过 `tcp_paws_reject()` 函数来判断时间戳是否发生了回绕，也就是「当前收到的报文的时间戳」是否大于「上一次收到的报文的时间戳」：

- 如果大于，就说明没有发生时间戳绕回，函数返回 false。
- 如果小于，就说明发生了时间戳回绕，函数返回 true。

从源码可以看到，当收到 SYN 包后，如果该 SYN 包的时间戳没有发生回绕，也就是时间戳是递增的，并且 SYN 包的序列号也没有发生回绕，也就是 SYN 的序列号「大于」下一次期望收到的序列号。就会初始化一个序列号，然后返回 TCP_TW_SYN，接着就重用该连接，也就跳过 2MSL 而转变为 SYN_RECV 状态，接着就能进行建立连接过程。

如果双方都没有启用 TCP 时间戳机制，就只需要判断 SYN 包的序列号有没有发生回绕，如果 SYN 的序列号大于下一次期望收到的序列号，就可以跳过 2MSL，重用该连接。

如果 SYN 包是非法的，就会返回 TCP_TW_ACK，接着就会发送与上一次一样的 ACK 给对方。

### 在 TIME_WAIT 状态，收到 RST 会断开连接吗？

会不会断开，关键看 `net.ipv4.tcp_rfc1337` 这个内核参数（默认情况是为 0）：

- 如果这个参数设置为 0， 收到 RST 报文会提前结束 TIME_WAIT 状态，释放连接。
- 如果这个参数设置为 1， 就会丢掉 RST 报文。

源码处理如下：

```c
enum tcp_tw_status
tcp_timewait_state_process(struct inet_timewait_sock *tw, struct sk_buff *skb,
      const struct tcphdr *th)
{
....
  //rst报文的时间戳没有发生回绕
 if (!paws_reject &&
     (TCP_SKB_CB(skb)->seq == tcptw->tw_rcv_nxt &&
      (TCP_SKB_CB(skb)->seq == TCP_SKB_CB(skb)->end_seq || th->rst))) {

      //处理rst报文
      if (th->rst) {
        //不开启这个选项，当收到 RST 时会立即回收tw，但这样做是有风险的
        if (twsk_net(tw)->ipv4.sysctl_tcp_rfc1337 == 0) {
          kill:
          //删除tw定时器，并释放tw
          inet_twsk_deschedule_put(tw);
          return TCP_TW_SUCCESS;
        }
      } else {
        //将 TIMEWAIT 状态的持续时间重新延长
        inet_twsk_reschedule(tw, TCP_TIMEWAIT_LEN);
      }

      ...
      return TCP_TW_SUCCESS;
    }
}
```

TIME_WAIT 状态收到 RST 报文而释放连接，这样等于跳过 2MSL 时间，这么做还是有风险。

sysctl_tcp_rfc1337 这个参数是在 rfc 1337 文档提出来的，目的是避免因为 TIME_WAIT 状态收到 RST 报文而跳过 2MSL 的时间，文档里也给出跳过 2MSL 时间会有什么潜在问题。（收到历史连接的数据 + 不能保证被动关闭的一方正确关闭）

### 总结

在 TCP 正常挥手过程中，处于 TIME_WAIT 状态的连接，收到相同四元组的 SYN 后会发生什么？

如果双方开启了时间戳机制：

- 如果客户端的 SYN 的「序列号」比服务端「期望下一个收到的序列号」要**大**，**并且**SYN 的「时间戳」比服务端「最后收到的报文的时间戳」要**大**。那么就会重用该四元组连接，跳过 2MSL 而转变为 SYN_RECV 状态，接着就能进行建立连接过程。
- 如果客户端的 SYN 的「序列号」比服务端「期望下一个收到的序列号」要**小**，**或者**SYN 的「时间戳」比服务端「最后收到的报文的时间戳」要**小**。那么就会**再回复一个第四次挥手的 ACK 报文，客户端收到后，发现并不是自己期望收到确认号，就回 RST 报文给服务端**。

在 TIME_WAIT 状态，收到 RST 会断开连接吗？

- 如果 `net.ipv4.tcp_rfc1337` 参数为 0，则提前结束 TIME_WAIT 状态，释放连接。
- 如果 `net.ipv4.tcp_rfc1337` 参数为 1，则会丢掉该 RST 报文。
