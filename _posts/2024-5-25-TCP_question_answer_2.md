---
layout: post
title: "TCP问题回答2"
date: 2024-5-25
tags: [network]
comments: true
author: zxy
---

## TCP 连接，一端断电和进程崩溃有什么区别？

> 问题详细描述：一个 TCP 连接，没有打开 keepalive 选项，没有数据交互，现在一端断电 和 一端进程崩溃 。这两种情况有什么区别？

这个问题有几个关键词：

- 没有开启 keepalive；
- 一直没有数据交互；
- 进程崩溃；
- 主机崩溃；

> keepalive 是指 TCP 保活机制

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODI3Njc0,size_16,color_FFFFFF,t_70-20230309225825871.png)

如果两端的 TCP 连接一直没有数据交互，达到了触发 TCP 保活机制的条件，那么内核里的 TCP 协议栈就会发送探测报文。

- 如果对端程序是正常工作的。当 TCP 保活的探测报文发送给对端, 对端会正常响应，这样 **TCP 保活时间会被重置**，等待下一个 TCP 保活时间的到来。
- 如果对端主机崩溃，或对端由于其他原因导致报文不可达。当 TCP 保活的探测报文发送给对端后，石沉大海，没有响应，连续几次，达到保活探测次数后，**TCP 会报告该 TCP 连接已经死亡**。

所以，TCP 保活机制可以在双方没有数据交互的情况，通过探测报文，来确定对方的 TCP 连接是否存活。

注意，应用程序若想使用 TCP 保活机制需要通过 socket 接口设置 `SO_KEEPALIVE` 选项才能够生效，如果没有设置，那么就无法使用 TCP 保活机制。

### 主机崩溃

> 在没有开启 TCP keepalive，且双方一直没有数据交互的情况下，如果客户端的「主机崩溃」了，会发生什么。

客户端主机崩溃了，服务端是**无法感知到的**，在加上服务端没有开启 TCP keepalive，又没有数据交互的情况下，**服务端的 TCP 连接将会一直处于 ESTABLISHED 连接状态**，直到服务端重启进程。

所以，我们可以得知一个点，在没有使用 TCP 保活机制且双方不传输数据的情况下，一方的 TCP 连接处在 ESTABLISHED 状态，并不代表另一方的连接还一定正常。

### 进程崩溃

> 在没有开启 TCP keepalive，且双方一直没有数据交互的情况下，如果服务端的「进程崩溃」了，会发生什么。

TCP 的连接信息是由内核维护的，所以当服务端的进程崩溃后，内核需要回收该进程的所有 TCP 连接资源，于是内核会发送第一次挥手 FIN 报文，后续的挥手过程也都是在内核完成，并不需要进程的参与，所以即使服务端的进程退出了，还是能与客户端完成 TCP 四次挥手的过程。

我自己做了实验，使用 kill -9 来模拟进程崩溃的情况，发现**在 kill 掉进程后，服务端会发送 FIN 报文，与客户端进行四次挥手**。

所以，即使没有开启 TCP keepalive，且双方也没有数据交互的情况下，如果其中一方的进程发生了崩溃，这个过程操作系统是可以感知的到的，于是就会发送 FIN 报文给对方，然后与对方进行 TCP 四次挥手。

### 有数据传输的场景

在「**有数据传输**」的场景下的一些异常情况：

- 第一种，客户端主机宕机，又迅速重启，会发生什么？
- 第二种，客户端主机宕机，一直没有重启，会发生什么？

#### 客户端主机宕机，又迅速重启

在客户端主机宕机后，服务端向客户端发送的报文会得不到任何的响应，在一定时长后，服务端就会触发**超时重传**机制，重传未得到响应的报文。

服务端重传报文的过程中，客户端主机重启完成后，客户端的内核就会接收重传的报文，然后根据报文的信息传递给对应的进程：

- 如果客户端主机上**没有**进程绑定该 TCP 报文的目标端口号，那么客户端内核就会**回复 RST 报文，重置该 TCP 连接**；
- 如果客户端主机上**有**进程绑定该 TCP 报文的目标端口号，由于客户端主机重启后，之前的 TCP 连接的数据结构已经丢失了，客户端内核里协议栈会发现找不到该 TCP 连接的 socket 结构体，于是就会**回复 RST 报文，重置该 TCP 连接**。

所以，**只要有一方重启完成后，收到之前 TCP 连接的报文，都会回复 RST 报文，以断开连接**。

#### 客户端主机宕机，一直没有重启

这种情况，服务端超时重传报文的次数达到一定阈值后，内核就会判定出该 TCP 有问题，然后通过 Socket 接口告诉应用程序该 TCP 连接出问题了，于是服务端的 TCP 连接就会断开。

> 那 TCP 的数据报文具体重传几次呢？

在 Linux 系统中，提供一个叫 tcp_retries2 配置项，默认值是 15，如下图：

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/20210615134059647.png)

这个内核参数是控制，在 TCP 连接建立的情况下，超时重传的最大次数。

不过 tcp_retries2 设置了 15 次，并不代表 TCP 超时重传了 15 次才会通知应用程序终止该 TCP 连接，**内核会根据 tcp_retries2 设置的值，计算出一个 timeout**（_如果 tcp_retries2 =15，那么计算得到的 timeout = 924600 ms_），**如果重传间隔超过这个 timeout，则认为超过了阈值，就会停止重传，然后就会断开 TCP 连接**。

在发生超时重传的过程中，每一轮的超时时间（RTO）都是**倍数增长**的，比如如果第一轮 RTO 是 200 毫秒，那么第二轮 RTO 是 400 毫秒，第三轮 RTO 是 800 毫秒，以此类推。

而 RTO 是基于 RTT（一个包的往返时间） 来计算的，如果 RTT 较大，那么计算出来的 RTO 就越大，那么经过几轮重传后，很快就达到了上面的 timeout 值了。

举个例子，如果 tcp_retries2 =15，那么计算得到的 timeout = 924600 ms，如果重传总间隔时长达到了 timeout 就会停止重传，然后就会断开 TCP 连接：

- 如果 RTT 比较小，那么 RTO 初始值就约等于下限 200ms，也就是第一轮的超时时间是 200 毫秒，由于 timeout 总时长是 924600 ms，表现出来的现象刚好就是重传了 15 次，超过了 timeout 值，从而断开 TCP 连接
- 如果 RTT 比较大，假设 RTO 初始值计算得到的是 1000 ms，也就是第一轮的超时时间是 1 秒，那么根本不需要重传 15 次，重传总间隔就会超过 924600 ms。

最小 RTO 和最大 RTO 是在 Linux 内核中定义好了：

```c
#define TCP_RTO_MAX ((unsigned)(120*HZ))
#define TCP_RTO_MIN ((unsigned)(HZ/5))
```

Linux 2.6+ 使用 1000 毫秒的 HZ，因此`TCP_RTO_MIN`约为 200 毫秒，`TCP_RTO_MAX`约为 120 秒。

如果`tcp_retries`设置为`15`，且 RTT 比较小，那么 RTO 初始值就约等于下限 200ms，这意味着**它需要 924.6 秒**才能将断开的 TCP 连接通知给上层（即应用程序），每一轮的 RTO 增长关系如下表格：

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODI3Njc0,size_16,color_FFFFFF,t_70-20230309225847765.png)

## 拔掉网线后， 原本的 TCP 连接还存在吗？

TCP 连接在 Linux 内核中是一个名为 `struct socket` 的结构体，该结构体的内容包含 TCP 连接的状态等信息。当拔掉网线的时候，操作系统并不会变更该结构体的任何内容，所以 TCP 连接的状态也不会发生改变。

> 要看拔掉网线后，双方做了什么动作?

### 拔掉网线后，有数据传输

在客户端拔掉网线后，服务端向客户端发送的数据报文会得不到任何的响应，在等待一定时长后，服务端就会触发**超时重传**机制，重传未得到响应的数据报文。

**如果在服务端重传报文的过程中，客户端刚好把网线插回去了**，由于拔掉网线并不会改变客户端的 TCP 连接状态，并且还是处于 ESTABLISHED 状态，所以这时客户端是可以正常接收服务端发来的数据报文的，然后客户端就会回 ACK 响应报文。

此时，客户端和服务端的 TCP 连接依然存在的，就感觉什么事情都没有发生。

但是，**如果如果在服务端重传报文的过程中，客户端一直没有将网线插回去**，服务端超时重传报文的次数达到一定阈值后，内核就会判定出该 TCP 有问题，然后通过 Socket 接口告诉应用程序该 TCP 连接出问题了，于是服务端的 TCP 连接就会断开。

而等客户端插回网线后，如果客户端向服务端发送了数据，由于服务端已经没有与客户端相同四元祖的 TCP 连接了，因此服务端内核就会回复 RST 报文，客户端收到后就会释放该 TCP 连接。

此时，客户端和服务端的 TCP 连接都已经断开了。

### 拔掉网线后，没有数据传输

针对拔掉网线后，没有数据传输的场景，还得看是否开启了 TCP keepalive 机制 （TCP 保活机制）。

如果**没有开启** TCP keepalive 机制，在客户端拔掉网线后，并且双方都没有进行数据传输，那么客户端和服务端的 TCP 连接将会一直保持存在。

而如果**开启**了 TCP keepalive 机制，在客户端拔掉网线后，即使双方都没有进行数据传输，在持续一段时间后，TCP 就会发送探测报文：

- 如果**对端是正常工作**的。当 TCP 保活的探测报文发送给对端, 对端会正常响应，这样 **TCP 保活时间会被重置**，等待下一个 TCP 保活时间的到来。
- 如果**对端主机宕机**（_注意不是进程崩溃，进程崩溃后操作系统在回收进程资源的时候，会发送 FIN 报文，而主机宕机则是无法感知的，所以需要 TCP 保活机制来探测对方是不是发生了主机宕机_），或对端由于其他原因导致报文不可达。当 TCP 保活的探测报文发送给对端后，石沉大海，没有响应，连续几次，达到保活探测次数后，**TCP 会报告该 TCP 连接已经死亡**。

所以，TCP 保活机制可以在双方没有数据交互的情况，通过探测报文，来确定对方的 TCP 连接是否存活。

## tcp_tw_reuse 为什么默认是关闭的？

> 问题：**既然打开 net.ipv4.tcp_tw_reuse 参数可以快速复用处于 TIME_WAIT 状态的 TCP 连接，那为什么 Linux 默认是关闭状态呢？**

其实这题在变相问「**如果 TIME_WAIT 状态持续时间过短或者没有，会有什么问题？**」

因为开启 tcp_tw_reuse 参数可以快速复用处于 TIME_WAIT 状态的 TCP 连接时，相当于缩短了 TIME_WAIT 状态的持续时间。

> 使用 tcp_tw_reuse 快速复用处于 TIME_WAIT 状态的 TCP 连接时，是需要保证 net.ipv4.tcp_timestamps 参数是开启的（默认是开启的），而 tcp_timestamps 参数可以避免旧连接的延迟报文，这不是解决了没有 TIME_WAIT 状态时的问题了吗？
>
> 是解决部分问题，但是不能完全解决.

### 为什么 tcp_tw_reuse 默认是关闭的？

#### 第一个问题

我们知道开启 tcp_tw_reuse 的同时，也需要开启 tcp_timestamps，意味着可以用时间戳的方式有效的判断回绕序列号的历史报文。

但是，在看我看了防回绕序列号函数的源码后，发现对于 **RST 报文的时间戳即使过期了，只要 RST 报文的序列号在对方的接收窗口内，也是能被接受的**。

下面 tcp_validate_incoming 函数就是验证接收到的 TCP 报文是否合格的函数，其中第一步就会进行 PAWS 检查，由 tcp_paws_discard 函数负责。

```c
static bool tcp_validate_incoming(struct sock *sk, struct sk_buff *skb, const struct tcphdr *th, int syn_inerr)
{
    struct tcp_sock *tp = tcp_sk(sk);

    /* RFC1323: H1. Apply PAWS check first. */
    if (tcp_fast_parse_options(sock_net(sk), skb, th, tp) &&
        tp->rx_opt.saw_tstamp &&
        tcp_paws_discard(sk, skb)) {
        if (!th->rst) {
            ....
            goto discard;
        }
        /* Reset is accepted even if it did not pass PAWS. */
    }
```

当 tcp_paws_discard 返回 true，就代表报文是一个历史报文，于是就要丢弃这个报文。但是在丢掉这个报文的时候，会先判断是不是 RST 报文，如果不是 RST 报文，才会将报文丢掉。也就是说，即使 RST 报文是一个历史报文，并不会被丢弃。

假设有这样的场景，如下图：

![img](https://cdn.xiaolincoding.com//mysql/other/0df2003d41ec0ef23844975a85cfb722.png)

过程如下：

- 客户端向一个还没有被服务端监听的端口发起了 HTTP 请求，接着服务端就会回 RST 报文给对方，很可惜的是 **RST 报文被网络阻塞了**。
- 由于客户端迟迟没有收到 TCP 第二次握手，于是重发了 SYN 包，与此同时服务端已经开启了服务，监听了对应的端口。于是接下来，客户端和服务端就进行了 TCP 三次握手、数据传输（HTTP 应答-响应）、四次挥手。
- 因为**客户端开启了 tcp_tw_reuse，于是快速复用 TIME_WAIT 状态的端口，又与服务端建立了一个与刚才相同的四元组的连接**。
- 接着，**前面被网络延迟 RST 报文这时抵达了客户端，而且 RST 报文的序列号在客户端的接收窗口内，由于防回绕序列号算法不会防止过期的 RST，所以 RST 报文会被客户端接受了，于是客户端的连接就断开了**。

上面这个场景就是开启 tcp_tw_reuse 风险，**因为快速复用 TIME_WAIT 状态的端口，导致新连接可能被回绕序列号的 RST 报文断开了，而如果不跳过 TIME_WAIT 状态，而是停留 2MSL 时长，那么这个 RST 报文就不会出现下一个新的连接**。

> 那为什么 PAWS 检查要放过过期的 RST 报文呢？

RFC1323：_建议 RST 段不携带时间戳，并且无论其时间戳如何，RST 段都是可接受的。老的重复的 RST 段应该是极不可能的，并且它们的清除功能应优先于时间戳。_

> RST 报文经过了一个 HTTP 请求后还存活？

一个 HTTP 请求其实很快的，比如我下面这个抓包，只需要 0.2 秒就完成了，远小于 MSL，所以延迟的 RST 报文存活是有可能的。

![图片](https://cdn.xiaolincoding.com//mysql/other/2ac40ca1757888b7154a2baa8bbb9885.png)

#### 第二个问题

开启 tcp_tw_reuse 来快速复用 TIME_WAIT 状态的连接，如果第四次挥手的 ACK 报文丢失了，服务端会触发超时重传，重传第三次挥手报文，处于 syn_sent 状态的客户端收到服务端重传第三次挥手报文，则会回 RST 给服务端。如下图：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/network/tcp/tcp_tw_reuse%E7%AC%AC%E4%BA%8C%E4%B8%AA%E9%97%AE%E9%A2%98.drawio.png)

> 如果 TIME_WAIT 状态被快速复用后，刚好第四次挥手的 ACK 报文丢失了，那客户端复用 TIME_WAIT 状态后发送的 SYN 报文被处于 last_ack 状态的服务端收到了会发生什么呢？

处于 last_ack 状态的服务端收到了 SYN 报文后，会回复确认号与服务端上一次发送 ACK 报文一样的 ACK 报文，这个 ACK 报文称为 Challenge ACK ，并不是确认收到 SYN 报文。

处于 syn_sent 状态的客户端收到服务端的 Challenge ACKm 后，发现不是自己期望收到的确认号，于是就会回复 RST 报文，服务端收到后，就会断开连接。

## HTTPS 中 TLS 和 TCP 能同时握手吗？

> TCP 和 TLS 握手顺序？

_HTTPS 建立连接的过程，先进行 TCP 三次握手，再进行 TLS 四次握手_。因为 HTTPS 都是基于 TCP 传输协议实现的，得先建立完可靠的 TCP 连接才能做 TLS 握手的事情。

> TLS 的握手次数？

TLS 握手过程的次数得看版本。

TLSv1.2 握手过程基本都是需要四次，也就是需要经过 2-RTT 才能完成握手，然后才能发送请求，而 TLSv1.3 只需要 1-RTT 就能完成 TLS 握手，如下图。

![图片](https://cdn.xiaolincoding.com//mysql/other/0877fe78380bf34ad3b28768e59fb53a.png)

> 「HTTPS 中的 TLS 握手过程可以同时进行三次握手」

这个场景是可能发生的，但是需要在特定的条件下才可能发生。

需要下面这两个条件同时满足才可以：

- **客户端和服务端都开启了 TCP Fast Open 功能，且 TLS 版本是 1.3；**
- **客户端和服务端已经完成过一次通信。**

### TCP Fast Open

常规的情况下，如果要使用 TCP 传输协议进行通信，则客户端和服务端通信之前，先要经过 TCP 三次握手后，建立完可靠的 TCP 连接后，客户端才能将数据发送给服务端。

其中，TCP 的第一次和第二次握手是不能够携带数据的，而 TCP 的第三次握手是可以携带数据的，因为这时候客户端的 TCP 连接状态已经是 ESTABLISHED，表明客户端这一方已经完成了 TCP 连接建立。

![图片](https://cdn.xiaolincoding.com//mysql/other/35bc3541c237686aa36e0a88f80592d4.png)

就算客户端携带数据的第三次握手在网络中丢失了，客户端在一定时间内没有收到服务端对该数据的应答报文，就会触发超时重传机制，然后客户端重传该携带数据的第三次握手的报文，直到重传次数达到系统的阈值，客户端就会销毁该 TCP 连接。

说完常规的 TCP 连接后，我们再来看看 TCP Fast Open。

TCP Fast Open 是为了绕过 TCP 三次握手发送数据，在 Linux 3.7 内核版本之后，提供了 TCP Fast Open 功能，这个功能可以减少 TCP 连接建立的时延。

要使用 TCP Fast Open 功能，客户端和服务端都要同时支持才会生效。

不过，开启了 TCP Fast Open 功能，**想要绕过 TCP 三次握手发送数据，得建立第二次以后的通信过程。**

在客户端首次建立连接时的过程，如下图：

![图片](https://cdn.xiaolincoding.com//mysql/other/7cb0bd3cde30493fec9562cbdb549f83.png)

具体介绍：

- 客户端发送 SYN 报文，该报文包含 Fast Open 选项，且该选项的 Cookie 为空，这表明客户端请求 Fast Open Cookie；
- 支持 TCP Fast Open 的服务器生成 Cookie，并将其置于 SYN-ACK 报文中的 Fast Open 选项以发回客户端；
- 客户端收到 SYN-ACK 后，本地缓存 Fast Open 选项中的 Cookie。

所以，第一次客户端和服务端通信的时候，还是需要正常的三次握手流程。随后，客户端就有了 Cookie 这个东西，它可以用来向服务器 TCP 证明先前与客户端 IP 地址的三向握手已成功完成。

对于客户端与服务端的后续通信，客户端可以在第一次握手的时候携带应用数据，从而达到绕过三次握手发送数据的效果，整个过程如下图：

![图片](https://cdn.xiaolincoding.com//mysql/other/fc452688b9351e0cabf60212dde3f21e.png)

我详细介绍下这个过程：

- 客户端发送 SYN 报文，该报文可以携带「应用数据」以及此前记录的 Cookie；
- 支持 TCP Fast Open 的服务器会对收到 Cookie 进行校验：如果 Cookie 有效，服务器将在 SYN-ACK 报文中对 SYN 和「数据」进行确认，服务器随后将「应用数据」递送给对应的应用程序；如果 Cookie 无效，服务器将丢弃 SYN 报文中包含的「应用数据」，且其随后发出的 SYN-ACK 报文将只确认 SYN 的对应序列号；
- **如果服务器接受了 SYN 报文中的「应用数据」，服务器可在握手完成之前发送「响应数据」，这就减少了握手带来的 1 个 RTT 的时间消耗**；
- 客户端将发送 ACK 确认服务器发回的 SYN 以及「应用数据」，但如果客户端在初始的 SYN 报文中发送的「应用数据」没有被确认，则客户端将重新发送「应用数据」；
- 此后的 TCP 连接的数据传输过程和非 TCP Fast Open 的正常情况一致。

所以，如果客户端和服务端同时支持 TCP Fast Open 功能，那么在完成首次通信过程后，后续客户端与服务端 的通信则可以绕过三次握手发送数据，这就减少了握手带来的 1 个 RTT 的时间消耗。

### TLSv1.3

TLSv1.3 握手过程只需 1-RTT 的时间，

TCP 连接的第三次握手是可以携带数据的，如果客户端在第三次握手发送了 TLSv1.3 第一次握手数据，是不是就表示「_HTTPS 中的 TLS 握手过程可以同时进行三次握手_」？。

不是的，因为服务端只有在收到客户端的 TCP 的第三次握手后，才能和客户端进行后续 TLSv1.3 握手。

TLSv1.3 还有个更厉害到地方在于**会话恢复**机制，在**重连 TLvS1.3 只需要 0-RTT**，用“pre_shared_key”和“early_data”扩展，在 TCP 连接后立即就建立安全连接发送加密消息，过程如下图：

![图片](https://cdn.xiaolincoding.com//mysql/other/59539201f006d7dc0a06333617e5ea85.png)

### TCP Fast Open + TLSv1.3

在前面我们知道，客户端和服务端同时支持 TCP Fast Open 功能的情况下，**在第二次以后到通信过程中，客户端可以绕过三次握手直接发送数据，而且服务端也不需要等收到第三次握手后才发送数据。**（HTTP）

如果 HTTPS 的 TLS 版本是 1.3，那么 TLS 过程只需要 1-RTT。

**因此如果「TCP Fast Open + TLSv1.3」情况下，在第二次以后的通信过程中，TLS 和 TCP 的握手过程是可以同时进行的。**

**如果基于 TCP Fast Open 场景下的 TLSv1.3 0-RTT 会话恢复过程，不仅 TLS 和 TCP 的握手过程是可以同时进行的，而且 HTTP 请求也可以在这期间内一同完成。**

## TCP 协议有什么缺陷？

### 升级 TCP 的工作很困难

TCP 协议是在内核中实现的，应用程序只能使用不能修改，如果要想升级 TCP 协议，那么只能升级内核。

而升级内核这个工作是很麻烦的事情，麻烦的事情不是说升级内核这个操作很麻烦，而是由于内核升级涉及到底层软件和运行库的更新，我们的服务程序就需要回归测试是否兼容新的内核版本，所以服务器的内核升级也比较保守和缓慢。

很多 TCP 协议的新特性，都是需要客户端和服务端同时支持才能生效的，比如 TCP Fast Open 这个特性，虽然在 2013 年就被提出了，但是 Windows 很多系统版本依然不支持它，这是因为 PC 端的系统升级滞后很严重，W indows Xp 现在还有大量用户在使用，尽管它已经存在快 20 年。

所以，即使 TCP 有比较好的特性更新，也很难快速推广，用户往往要几年或者十年才能体验到。

### TCP 建立连接的延迟

基于 TCP 实现的应用协议，都是需要先建立三次握手才能进行数据传输，比如 HTTP 1.0/1.1、HTTP/2、HTTPS。

现在大多数网站都是使用 HTTPS 的，这意味着在 TCP 三次握手之后，还需要经过 TLS 四次握手后，才能进行 HTTP 数据的传输，这在一定程序上增加了数据传输的延迟。

TCP 三次握手的延迟被 TCP Fast Open （快速打开）这个特性解决了，这个特性可以在「第二次建立连接」时减少 TCP 连接建立的时延。

TCP Fast Open 这个特性是不错，但是它需要服务端和客户端的操作系统同时支持才能体验到，而 TCP Fast Open 是在 2013 年提出的，所以市面上依然有很多老式的操作系统不支持，而升级操作系统是很麻烦的事情，因此 TCP Fast Open 很难被普及开来。

还有一点，针对 HTTPS 来说，TLS 是在应用层实现的握手，而 TCP 是在内核实现的握手，这两个握手过程是无法结合在一起的，总是得先完成 TCP 握手，才能进行 TLS 握手。

也正是 TCP 是在内核实现的，所以 TLS 是无法对 TCP 头部加密的，这意味着 TCP 的序列号都是明文传输，所以就存安全的问题。

一个典型的例子就是攻击者伪造一个的 RST 报文强制关闭一条 TCP 连接，而攻击成功的关键则是 TCP 字段里的序列号位于接收方的滑动窗口内，该报文就是合法的。

为此 TCP 也不得不进行三次握手来同步各自的序列号，而且初始化序列号时是采用随机的方式（不完全随机，而是随着时间流逝而线性增长，到了 2^32 尽头再回滚）来提升攻击者猜测序列号的难度，以增加安全性。

但是这种方式只能避免攻击者预测出合法的 RST 报文，而无法避免攻击者截获客户端的报文，然后中途伪造出合法 RST 报文的攻击的方式。

![img](https://cdn.xiaolincoding.com//mysql/other/A*po6LQIBU7zIAAAAAAAAAAAAAARQnAQ.png)

大胆想一下，如果 TCP 的序列号也能被加密，或许真的不需要三次握手了，客户端和服务端的初始序列号都从 0 开始，也就不用做同步序列号的工作了，但是要实现这个要改造整个协议栈，太过于麻烦，即使实现出来了，很多老的网络设备未必能兼容。

### TCP 存在队头阻塞问题

TCP 是字节流协议，**TCP 层必须保证收到的字节数据是完整且有序的**，如果序列号较低的 TCP 段在网络传输中丢失了，即使序列号较高的 TCP 段已经被接收了，应用层也无法从内核中读取到这部分数据。

这就是 TCP 队头阻塞问题，但这也不能怪 TCP ，因为只有这样做才能保证数据的有序性。

HTTP/2 多个请求是跑在一个 TCP 连接中的，那么当 TCP 丢包时，整个 TCP 都要等待重传，那么就会阻塞该 TCP 连接中的所有请求，所以 HTTP/2 队头阻塞问题就是因为 TCP 协议导致的。

### 网络迁移需要重新建立 TCP 连接

基于 TCP 传输协议的 HTTP 协议，由于是通过四元组（源 IP、源端口、目的 IP、目的端口）确定一条 TCP 连接。

![TCP 四元组](https://cdn.xiaolincoding.com//mysql/other/format,png.png)

那么**当移动设备的网络从 4G 切换到 WIFI 时，意味着 IP 地址变化了，那么就必须要断开连接，然后重新建立 TCP 连接**。

而建立连接的过程包含 TCP 三次握手和 TLS 四次握手的时延，以及 TCP 慢启动的减速过程，给用户的感觉就是网络突然卡顿了一下，因此连接的迁移成本是很高的。

## 服务端没有 listen，客户端发起连接建立，会发生什么？

> 问题描述：服务端只 bind 了 IP 和端口，但是没有调用 listen 让 socket 监听连接，此时如果客户端朝服务器端 socket 发数据，会发生什么？

### 做个实验

```c
/*******服务器程序  TCPServer.c ************/
#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <string.h>
#include <netdb.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>

int main(int argc, char *argv[])
{
    int sockfd, ret;
    struct sockaddr_in server_addr;

    /* 服务器端创建 tcp socket 描述符 */
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if(sockfd < 0)
    {
        fprintf(stderr, "Socket error:%s\n\a", strerror(errno));
        exit(1);
    }

    /* 服务器端填充 sockaddr 结构 */
    bzero(&server_addr, sizeof(struct sockaddr_in));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    server_addr.sin_port = htons(8888);

  /* 绑定 ip + 端口 */
    ret = bind(sockfd, (struct sockaddr *)(&server_addr), sizeof(struct sockaddr));
    if(ret < 0)
    {
        fprintf(stderr, "Bind error:%s\n\a", strerror(errno));
        exit(1);
    }

  //没有调用 listen

    sleep(1000);
    close(sockfd);
    return 0;
}
```

然后，我用浏览器访问这个地址：http://121.43.173.240:8888/

![图片](https://cdn.xiaolincoding.com//mysql/other/5bdb5443db5b97ff724ab94e014af6a5.png)

报错连接服务器失败。

同时，我也用抓包工具，抓了这个过程。

![图片](https://cdn.xiaolincoding.com//mysql/other/a77921ffafbbff86d07983ca0db3e6e0.png)

可以看到，客户端对服务端发起 SYN 报文后，服务端回了 RST 报文。

所以，这个问题就有了答案，**服务端如果只 bind 了 IP 地址和端口，而没有调用 listen 的话，然后客户端对服务端发起了连接建立，服务端会回 RST 报文。**

### 源码分析

Linux 内核处理收到 TCP 报文的入口函数是 tcp_v4_rcv，在收到 TCP 报文后，会调用 \_\_inet_lookup_skb 函数找到 TCP 报文所属 socket 。

```text
int tcp_v4_rcv(struct sk_buff *skb)
{
 ...

 sk = __inet_lookup_skb(&tcp_hashinfo, skb, th->source, th->dest);
 if (!sk)
  goto no_tcp_socket;
 ...
}
```

**inet_lookup_skb 函数首先查找连接建立状态的 socket（**inet_lookup_established），在没有命中的情况下，才会查找监听套接口（\_\_inet_lookup_listener）。

![图片](https://cdn.xiaolincoding.com//mysql/other/88416aa95d255495e07fb3a002b2167b.png)

查找监听套接口（\_\_inet_lookup_listener）这个函数的实现是，根据目的地址和目的端口算出一个哈希值，然后在哈希表找到对应监听该端口的 socket。

本次的案例中，服务端是没有调用 listen 函数的，所以自然也是找不到监听该端口的 socket。

所以，\_\_inet_lookup_skb 函数最终找不到对应的 socket，于是跳转到 no_tcp_socket。

![图片](https://cdn.xiaolincoding.com//mysql/other/54ee363e149ee3dfba30efb1a542ef5c.png)

在这个错误处理中，只要收到的报文（skb）的「校验和」没问题的话，内核就会调用 tcp_v4_send_reset 发送 RST 中止这个连接。

至此，整个源码流程就解析完。

### 没有 listen，能建立 TCP 连接吗？

答案，**是可以的，客户端是可以自己连自己的形成连接（TCP 自连接），也可以两个客户端同时向对方发出请求建立连接（TCP 同时打开），这两个情况都有个共同点，就是没有服务端参与，也就是没有 listen，就能建立连接**。

> 那没有 listen，为什么还能建立连接？

我们知道执行 listen 方法时，会创建半连接队列和全连接队列。

三次握手的过程中会在这两个队列中暂存连接信息。

所以形成连接，前提是你得有个地方存放着，方便握手的时候能根据 IP + 端口等信息找到对应的 socket。

> 那么客户端会有半连接队列吗？

显然没有，因为客户端没有执行 listen，因为半连接队列和全连接队列都是在执行 listen 方法时，内核自动创建的。

但内核还有个全局 hash 表，可以用于存放 sock 连接的信息。

这个全局 hash 表其实还细分为 ehash，bhash 和 listen_hash 等，但因为过于细节，大家理解成有一个全局 hash 就够了，

**在 TCP 自连接的情况中，客户端在 connect 方法时，最后会将自己的连接信息放入到这个全局 hash 表中，然后将信息发出，消息在经过回环地址重新回到 TCP 传输层的时候，就会根据 IP + 端口信息，再一次从这个全局 hash 中取出信息。于是握手包一来一回，最后成功建立连接**。

TCP 同时打开的情况也类似，只不过从一个客户端变成了两个客户端而已。

> 做个实验

客户端自连接的代码，TCP socket 可以 connect 它本身 bind 的地址和端口：

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>

#define LOCAL_IP_ADDR		(0x7F000001) // IP 127.0.0.1
#define LOCAL_TCP_PORT		(34567) // 端口

int main(void)
{
	struct sockaddr_in local, peer;
	int ret;
	char buf[128];
	int sock = socket(AF_INET, SOCK_STREAM, 0);

	memset(&local, 0, sizeof(local));
	memset(&peer, 0, sizeof(peer));

	local.sin_family = AF_INET;
	local.sin_port = htons(LOCAL_TCP_PORT);
	local.sin_addr.s_addr = htonl(LOCAL_IP_ADDR);

	peer = local;

    int flag = 1;
    ret = setsockopt(sock, SOL_SOCKET, SO_REUSEADDR, &flag, sizeof(flag));
    if (ret == -1) {
        printf("Fail to setsocket SO_REUSEADDR: %s\n", strerror(errno));
        exit(1);
    }

	ret = bind(sock, (const struct sockaddr *)&local, sizeof(local));
	if (ret) {
		printf("Fail to bind: %s\n", strerror(errno));
		exit(1);
	}

	ret = connect(sock, (const struct sockaddr *)&peer, sizeof(peer));
	if (ret) {
		printf("Fail to connect myself: %s\n", strerror(errno));
		exit(1);
	}

	printf("Connect to myself successfully\n");

    //发送数据
	strcpy(buf, "Hello, myself~");
	send(sock, buf, strlen(buf), 0);

	memset(buf, 0, sizeof(buf));

	//接收数据
	recv(sock, buf, sizeof(buf), 0);
	printf("Recv the msg: %s\n", buf);

    sleep(1000);
	close(sock);
	return 0;
}
```

编译运行：

![img](https://cdn.xiaolincoding.com//mysql/other/9db974179b9e4a279f7edb0649752c27.png)

通过 netstat 命令命令客户端自连接的 TCP 连接：

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/e2b116e843c14e468eadf9d30e1b877c.png)

从截图中，可以看到 TCP socket 成功的“连接”了自己，并发送和接收了数据包，netstat 的输出更证明了 TCP 的两端地址和端口是完全相同的。

## 没有 accept，能建立 TCP 连接吗？

![握手建立连接流程](https://cdn.xiaolincoding.com//mysql/other/e0d405a55626eb8e4a52553a54680618.gif)

> **如果没有这个 accept 方法，TCP 连接还能建立起来吗？**

其实只要在执行`accept()` 之前执行一个 `sleep(20)`，然后立刻执行客户端相关的方法，同时抓个包，就能得出结论。

![不执行accept时抓包结果](https://cdn.xiaolincoding.com//mysql/other/2cfc1d028f3e37f10c2f81375ddb998a.png)

从抓包结果看来，**就算不执行 accept()方法，三次握手照常进行，并顺利建立连接。**

更骚气的是，**在服务端执行 accept()前，如果客户端发送消息给服务端，服务端是能够正常回复 ack 确认包的。**

并且，`sleep(20)`结束后，服务端正常执行`accept()`，客户端前面发送的消息，还是能正常收到的。

### 三次握手的细节分析

![半连接队列和全连接队列](https://cdn.xiaolincoding.com//mysql/other/36242c85809865fcd2da48594de15ebb.png)

建立连接的过程中根本不需要`accept()`参与， **执行 accept()只是为了从全连接队列里取出一条连接。**

**全连接队列（icsk_accept_queue）是个链表**，而**半连接队列（syn_table）是个哈希表**。

#### 为什么半连接队列要设计成哈希表

先对比下**全连接里队列**，他本质是个链表，因为也是线性结构，说它是个队列也没毛病。它里面放的都是已经建立完成的连接，这些连接正等待被取走。而服务端取走连接的过程中，并不关心具体是哪个连接，只要是个连接就行，所以直接从队列头取就行了。这个过程算法复杂度为`O(1)`。

而**半连接队列**却不太一样，因为队列里的都是不完整的连接，嗷嗷等待着第三次握手的到来。那么现在有一个第三次握手来了，则需要从队列里把相应 IP 端口的连接取出，**如果半连接队列还是个链表，那我们就需要依次遍历，才能拿到我们想要的那个连接，算法复杂度就是 O(n)。**

而如果将半连接队列设计成哈希表，那么查找半连接的算法复杂度就回到`O(1)`了。

因此出于效率考虑，全连接队列被设计成链表，而半连接队列被设计为哈希表。

#### 怎么观察两个队列的大小

**查看全连接队列**

```shell
# ss -lnt
State      Recv-Q Send-Q     Local Address:Port           Peer Address:Port
LISTEN     0      128        127.0.0.1:46269              *:*
```

通过`ss -lnt`命令，可以看到全连接队列的大小，其中`Send-Q`是指全连接队列的最大值，可以看到我这上面的最大值是`128`；`Recv-Q`是指当前的全连接队列的使用值，我这边用了`0`个，也就是全连接队列里为空，连接都被取出来了。

当上面`Send-Q`和`Recv-Q`数值很接近的时候，那么全连接队列可能已经满了。可以通过下面的命令查看是否发生过队列**溢出**。

```shell
# netstat -s | grep overflowed
    4343 times the listen queue of a socket overflowed
```

上面说明发生过`4343次`全连接队列溢出的情况。这个查看到的是**历史发生过的次数**。

如果配合使用`watch -d` 命令，可以自动每`2s`间隔执行相同命令，还能高亮显示变化的数字部分，如果溢出的数字不断变多，说明**正在发生**溢出的行为。

```shell
# watch -d 'netstat -s | grep overflowed'
Every 2.0s: netstat -s | grep overflowed
Fri Sep 17 09:00:45 2021

    4343 times the listen queue of a socket overflowed
```

**查看半连接队列**

半连接队列没有命令可以直接查看到，但因为半连接队列里，放的都是`SYN_RECV`状态的连接，那可以通过统计处于这个状态的连接的数量，间接获得半连接队列的长度。

```shell
# netstat -nt | grep -i '127.0.0.1:8080' | grep -i 'SYN_RECV' | wc -l
0
```

注意半连接队列和全连接队列都是挂在某个`Listen socket`上的，我这里用的是`127.0.0.1:8080`，大家可以替换成自己想要查看的**IP 端口**。

可以看到我的机器上的半连接队列长度为`0`，这个很正常，**正经连接谁会没事老待在半连接队列里。**

当队列里的半连接不断增多，最终也是会发生溢出，可以通过下面的命令查看。

```shell
# netstat -s | grep -i "SYNs to LISTEN sockets dropped"
    26395 SYNs to LISTEN sockets dropped
```

可以看到，我的机器上一共发生了`26395`次半连接队列溢出。同样建议配合`watch -d` 命令使用。

```shell
# watch -d 'netstat -s | grep -i "SYNs to LISTEN sockets dropped"'
Every 2.0s: netstat -s | grep -i "SYNs to LISTEN sockets dropped"
Fri Sep 17 08:36:38 2021

    26395 SYNs to LISTEN sockets dropped
```

#### 全连接队列满了会怎么样？

如果队列满了，服务端还收到客户端的第三次握手 ACK，默认当然会丢弃这个 ACK。

但除了丢弃之外，还有一些附带行为，这会受 `tcp_abort_on_overflow` 参数的影响。

```shell
# cat /proc/sys/net/ipv4/tcp_abort_on_overflow
0
```

- `tcp_abort_on_overflow`设置为 0，全连接队列满了之后，会丢弃这个第三次握手 ACK 包，并且开启定时器，重传第二次握手的 SYN+ACK，如果重传超过一定限制次数，还会把对应的**半连接队列里的连接**给删掉。

![tcp_abort_on_overflow为0](https://cdn.xiaolincoding.com//mysql/other/874f2fb7108020fd4dcfa021f377ec66.png)

- `tcp_abort_on_overflow`设置为 1，全连接队列满了之后，就直接发 RST 给客户端，效果上看就是连接断了。

这个现象是不是很熟悉，服务端**端口未监听**时，客户端尝试去连接，服务端也会回一个 RST。这两个情况长一样，所以客户端这时候收到 RST 之后，其实无法区分到底是**端口未监听**，还是**全连接队列满了**。

![tcp_abort_on_overflow为1](https://cdn.xiaolincoding.com//mysql/other/6a01c5df74748870a69921da89825d9c.png)

#### 半连接队列要是满了会怎么样

**一般是丢弃**，但这个行为可以通过 `tcp_syncookies` 参数去控制。但比起这个，更重要的是先了解下半连接队列为什么会被打满。

首先我们需要明白，一般情况下，半连接的"生存"时间其实很短，只有在第一次和第三次握手间，如果半连接都满了，说明服务端疯狂收到第一次握手请求，如果是线上游戏应用，能有这么多请求进来，那说明你可能要富了。但现实往往比较骨感，你可能遇到了**SYN Flood 攻击**。

所谓**SYN Flood 攻击**，可以简单理解为，攻击方模拟客户端疯狂发第一次握手请求过来，在服务端憨憨地回复第二次握手过去之后，客户端死活不发第三次握手过来，这样做，可以把服务端半连接队列打满，从而导致正常连接不能正常进来。

![syn攻击](https://cdn.xiaolincoding.com//mysql/other/d894de5374a12bd5d75d86d4a718d186.png)

那这种情况怎么处理？有没有一种方法可以**绕过半连接队列**？

有，上面提到的`tcp_syncookies`派上用场了。

> 会有一个 cookies 队列吗

生成是`cookies`，保存在哪呢？**是不是会有一个队列保存这些 cookies？**

我们可以反过来想一下，如果有`cookies`队列，那它会跟半连接队列一样，到头来，还是会被**SYN Flood 攻击**打满。

实际上`cookies`并不会有一个专门的队列保存，它是通过**通信双方的 IP 地址端口、时间戳、MSS**等信息进行**实时计算**的，保存在**TCP 报头**的`seq`里。

![tcp报头_seq的位置](https://cdn.xiaolincoding.com//mysql/other/6d280b0946a73ea6185653cbcfcc489f.png)

当服务端收到客户端发来的第三次握手包时，会通过 seq 还原出**通信双方的 IP 地址端口、时间戳、MSS**，验证通过则建立连接。

> cookies 方案为什么不直接取代半连接队列？

目前看下来`syn cookies`方案省下了半连接队列所需要的队列内存，还能解决 **SYN Flood 攻击**，那为什么不直接取代半连接队列？

凡事皆有利弊，`cookies`方案虽然能防 **SYN Flood 攻击**，但是也有一些问题。因为服务端并不会保存连接信息，所以如果传输过程中数据包丢了，也不会重发第二次握手的信息。

另外，编码解码`cookies`，都是比较**耗 CPU**的，利用这一点，如果此时攻击者构造大量的**第三次握手包（ACK 包）**，同时带上各种瞎编的`cookies`信息，服务端收到`ACK包`后**以为是正经 cookies**，憨憨地跑去解码（**耗 CPU**），最后发现不是正经数据包后才丢弃。

这种通过构造大量`ACK包`去消耗服务端资源的攻击，叫**ACK 攻击**，受到攻击的服务器可能会因为**CPU 资源耗尽**导致没能响应正经请求。

![ack攻击](https://cdn.xiaolincoding.com//mysql/other/15a0a5f7fe15ee2bc5e07492eda5a8ea.gif)

## TCP 四次挥手，可以变成三次吗？

### 为什么 TCP 挥手需要四次呢？

服务器收到客户端的 FIN 报文时，内核会马上回一个 ACK 应答报文，**但是服务端应用程序可能还有数据要发送，所以并不能马上发送 FIN 报文，而是将发送 FIN 报文的控制权交给服务端应用程序**：

- 如果服务端应用程序有数据要发送的话，就发完数据后，才调用关闭连接的函数；
- 如果服务端应用程序没有数据要发送的话，可以直接调用关闭连接的函数，

从上面过程可知，**是否要发送第三次挥手的控制权不在内核，而是在被动关闭方（上图的服务端）的应用程序，因为应用程序可能还有数据要发送，由应用程序决定什么时候调用关闭连接的函数，当调用了关闭连接的函数，内核就会发送 FIN 报文了，**所以服务端的 ACK 和 FIN 一般都会分开发送。

> FIN 报文一定得调用关闭连接的函数，才会发送吗？

不一定。

如果进程退出了，不管是不是正常退出，还是异常退出（如进程崩溃），内核都会发送 FIN 报文，与对方完成四次挥手。

### 粗暴关闭 vs 优雅关闭

关闭的连接的函数有两种函数：

- close 函数，同时 socket 关闭发送方向和读取方向，也就是 socket 不再有发送和接收数据的能力。如果有多进程/多线程共享同一个 socket，如果有一个进程调用了 close 关闭只是让 socket 引用计数 -1，并不会导致 socket 不可用，同时也不会发出 FIN 报文，其他进程还是可以正常读写该 socket，直到引用计数变为 0，才会发出 FIN 报文。
- shutdown 函数，可以指定 socket 只关闭发送方向而不关闭读取方向，也就是 socket 不再有发送数据的能力，但是还是具有接收数据的能力。如果有多进程/多线程共享同一个 socket，shutdown 则不管引用计数，直接使得该 socket 不可用，然后发出 FIN 报文，如果有别的进程企图使用该 socket，将会受到影响。

如果客户端是用 close 函数来关闭连接，那么在 TCP 四次挥手过程中，如果收到了服务端发送的数据，由于客户端已经不再具有发送和接收数据的能力，所以客户端的内核会回 RST 报文给服务端，然后内核会释放连接，这时就不会经历完成的 TCP 四次挥手，所以我们常说，调用 close 是粗暴的关闭。

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/3b5f1897d2d74028aaf4d552fbce1a74.png)

当服务端收到 RST 后，内核就会释放连接，当服务端应用程序再次发起读操作或者写操作时，就能感知到连接已经被释放了：

- 如果是读操作，则会返回 RST 的报错，也就是我们常见的 Connection reset by peer。
- 如果是写操作，那么程序会产生 SIGPIPE 信号，应用层代码可以捕获并处理信号，如果不处理，则默认情况下进程会终止，异常退出。

相对的，shutdown 函数因为可以指定只关闭发送方向而不关闭读取方向，所以即使在 TCP 四次挥手过程中，如果收到了服务端发送的数据，客户端也是可以正常读取到该数据的，然后就会经历完整的 TCP 四次挥手，所以我们常说，调用 shutdown 是优雅的关闭。

![优雅关闭.drawio.png](https://cdn.xiaolincoding.com//mysql/other/71f5646ec58849e5921adc08bb6789d4.png)

但是注意，shutdown 函数也可以指定「只关闭读取方向，而不关闭发送方向」，但是这时候内核是不会发送 FIN 报文的，因为发送 FIN 报文是意味着我方将不再发送任何数据，而 shutdown 如果指定「不关闭发送方向」，就意味着 socket 还有发送数据的能力，所以内核就不会发送 FIN。

### 什么情况会出现三次挥手？

当被动关闭方（上图的服务端）在 TCP 挥手过程中，「**没有数据要发送」并且「开启了 TCP 延迟确认机制」，那么第二和第三次挥手就会合并传输，这样就出现了三次挥手。**

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/d7b349efa4f94453943b433b704a4ca8.png)

然后因为 TCP 延迟确认机制是默认开启的，所以导致我们抓包时，看见三次挥手的次数比四次挥手还多。

> 什么是 TCP 延迟确认机制？

当发送没有携带数据的 ACK，它的网络效率也是很低的，因为它也有 40 个字节的 IP 头 和 TCP 头，但却没有携带数据报文。 为了解决 ACK 传输效率低问题，所以就衍生出了 **TCP 延迟确认**。 TCP 延迟确认的策略：

- 当有响应数据要发送时，ACK 会随着响应数据一起立刻发送给对方
- 当没有响应数据要发送时，ACK 将会延迟一段时间，以等待是否有响应数据可以一起发送
- 如果在延迟等待发送 ACK 期间，对方的第二个数据报文又到达了，这时就会立刻发送 ACK

![img](https://cdn.xiaolincoding.com//mysql/other/33f3d2d54a924b0a80f565038327e0e4.png)

### 实验验证

#### 实验一

> 当被动关闭方（上图的服务端）在 TCP 挥手过程中，「**没有数据要发送」并且「开启了 TCP 延迟确认机制」，那么第二和第三次挥手就会合并传输，这样就出现了三次挥手。**

服务端的代码如下，做的事情很简单，就读取数据，然后当 read 返回 0 的时候，就马上调用 close 关闭连接。因为 TCP 延迟确认机制是默认开启的，所以不需要特殊设置。

```cpp
#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <string.h>
#include <netdb.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <netinet/tcp.h>

#define MAXLINE 1024

int main(int argc, char *argv[])
{

    // 1. 创建一个监听 socket
    int listenfd = socket(AF_INET, SOCK_STREAM, 0);
    if(listenfd < 0)
    {
        fprintf(stderr, "socket error : %s\n", strerror(errno));
        return -1;
    }

    // 2. 初始化服务器地址和端口
    struct sockaddr_in server_addr;
    bzero(&server_addr, sizeof(struct sockaddr_in));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    server_addr.sin_port = htons(8888);

    // 3. 绑定地址+端口
    if(bind(listenfd, (struct sockaddr *)(&server_addr), sizeof(struct sockaddr)) < 0)
    {
        fprintf(stderr,"bind error:%s\n", strerror(errno));
        return -1;
    }

    printf("begin listen....\n");

    // 4. 开始监听
    if(listen(listenfd, 128))
    {
        fprintf(stderr, "listen error:%s\n\a", strerror(errno));
        exit(1);
    }


    // 5. 获取已连接的socket
    struct sockaddr_in client_addr;
    socklen_t client_addrlen = sizeof(client_addr);
    int clientfd = accept(listenfd, (struct sockaddr *)&client_addr, &client_addrlen);
    if(clientfd < 0) {
        fprintf(stderr, "accept error:%s\n\a", strerror(errno));
        exit(1);
    }

    printf("accept success\n");

    char message[MAXLINE] = {0};

    while(1) {
        //6. 读取客户端发送的数据
        int n = read(clientfd, message, MAXLINE);
        if(n < 0) { // 读取错误
            fprintf(stderr, "read error:%s\n\a", strerror(errno));
            break;
        } else if(n == 0) {  // 返回 0 ，代表读到 FIN 报文
            fprintf(stderr, "client closed \n");
            close(clientfd); // 没有数据要发送，立马关闭连接
            break;
        }

        message[n] = 0;
        printf("received %d bytes: %s\n", n, message);
    }

    close(listenfd);
    return 0;
}
```

客户端代码如下，做的事情也很简单，与服务端连接成功后，就发送数据给服务端，然后睡眠一秒后，就调用 close 关闭连接，所以客户端是主动关闭方：

```cpp
#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <string.h>
#include <netdb.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>

int main(int argc, char *argv[])
{

    // 1. 创建一个监听 socket
    int connectfd = socket(AF_INET, SOCK_STREAM, 0);
    if(connectfd < 0)
    {
        fprintf(stderr, "socket error : %s\n", strerror(errno));
        return -1;
    }

    // 2. 初始化服务器地址和端口
    struct sockaddr_in server_addr;
    bzero(&server_addr, sizeof(struct sockaddr_in));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    server_addr.sin_port = htons(8888);

    // 3. 连接服务器
    if(connect(connectfd, (struct sockaddr *)(&server_addr), sizeof(server_addr)) < 0)
    {
        fprintf(stderr,"connect error:%s\n", strerror(errno));
        return -1;
    }

    printf("connect success\n");


    char sendline[64] = "hello, i am xiaolin";

    //4. 发送数据
    int ret = send(connectfd, sendline, strlen(sendline), 0);
    if(ret != strlen(sendline)) {
        fprintf(stderr,"send data error:%s\n", strerror(errno));
        return -1;
    }

    printf("already send %d bytes\n", ret);

    sleep(1);

    //5. 关闭连接
    close(connectfd);
    return 0;
}
```

编译服务端和客户端的代码：

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/291c6bdf93fa4e04b1606eef57d76836.png)

先启用服务端：

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/a975aa542caf41b2a1f303563df697b7.png)

然后用 tcpdump 工具开始抓包，命令如下：

```bash
tcpdump -i lo tcp and port 8888 -s0 -w /home/tcp_close.pcap
```

然后启用客户端，可以看到，与服务端连接成功后，发完数据就退出了。

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/8ea9f527a68a4c0184edc8842aaf55d6.png)

此时，服务端的输出：

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/ff5f9ae91a3a4576b59e6e9c4716464d.png)

接下来，我们来看看抓包的结果。

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/b542a2777aca4419b47205484b52cc03.png)

可以看到，TCP 挥手次数是 3 次。

#### 实验二

我们再做一次实验，来看看**关闭 TCP 延迟确认机制，会出现四次挥手吗？**

客户端代码保持不变，服务端代码需要增加一点东西。

在上面服务端代码中，增加了打开了 TCP_QUICKACK （快速应答）机制的代码，如下：

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/fbbe19e6b1cc4a21b024588950b88eee.png)

编译好服务端代码后，就开始运行服务端和客户端的代码，同时用 tcpdump 进行抓包。

抓包的结果如下，可以看到是四次挥手。 ![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/b6327b1057d64f54997c0eb322b28a55.png)

所以，当被动关闭方（上图的服务端）在 TCP 挥手过程中，「**没有数据要发送」，同时「关闭了 TCP 延迟确认机制」，那么就会是四次挥手。**

> 设置 TCP_QUICKACK 的代码，为什么要放在 read 返回 0 之后？

多次实验发现，在 bind 之前设置 TCP_QUICKACK 是不生效的，只有在 read 返回 0 的时候，设置 TCP_QUICKACK 才会出现四次挥手。

网上查了下资料说，设置 TCP_QUICKACK 并不是永久的，所以每次读取数据的时候，如果想要立刻回 ACK，那就得在每次读取数据之后，重新设置 TCP_QUICKACK。

而我这里的实验，目的是为了当收到客户端的 FIN 报文（第一次挥手）后，立马回 ACK 报文。所以就在 read 返回 0 的时候，设置 TCP_QUICKACK。当然，实际应用中，没人会在这个位置设置 TCP_QUICKACK，因为操作系统都通过 TCP 延迟确认机制帮我们把四次挥手优化成了三次挥手了。

## TCP 序列号和确认号是如何变化的？

### 万能公式

**发送的 TCP 报文：**

- **公式一：序列号 = 上一次发送的序列号 + len（数据长度）。特殊情况，如果上一次发送的报文是 SYN 报文或者 FIN 报文，则改为 上一次发送的序列号 + 1。**
- **公式二：确认号 = 上一次收到的报文中的序列号 + len（数据长度）。特殊情况，如果收到的是 SYN 报文或者 FIN 报文，则改为上一次收到的报文中的序列号 + 1。**

### 三次握手阶段的变化

假设客户端的初始化序列号为 client_isn，服务端的初始化序列号为 server_isn，TCP 三次握手的流程如下：

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/06c4ed62087040438f86ba64e9e609e7.png)

在这里我们重点关注，下面这两个过程。

服务端收到客户端的 SYN 报文后，会将 SYN-ACK 报文（第二次握手报文）中序列号和确认号分别设置为：

- 序列号设置为服务端随机初始化的序列号 server_isn。
- 确认号设置为 client_isn + 1，服务端上一次收到的报文是客户端发来的 SYN 报文，该报文的 seq = client_isn，那么根据公式 2（_确认号 = 上一次收到的报文中的序列号 + len。特殊情况，如果收到的是 SYN 报文或者 FIN 报文，则改为 + 1_），可以得出当前确认号 = client_isn + 1。

客户端收到服务端的 SYN-ACK 报文后，会将 ACK 报文（第三次握手报文）中序列号和确认号分别设置为：

- 序列号设置为 client_isn + 1。客户端上一次发送报文是 SYN 报文，SYN 的序列号为 client_isn，根据公式 1（_序列号 = 上一次发送的序列号 + len。特殊情况，如果上一次发送的报文是 SYN 报文或者 FIN 报文，则改为 + 1_），所以当前的序列号为 client_isn + 1。
- 确认号设置为 server_isn + 1，客户端上一次收到的报文是服务端发来的 SYN-ACK 报文，该报文的 seq = server_isn，那么根据公式 2（_确认号 = 收到的报文中的序列号 + len。特殊情况，如果收到的是 SYN 报文或者 FIN 报文，则改为 + 1_），可以得出当前确认号 = server_isn + 1。

> 为什么第二次和第三次握手报文中的确认号是将对方的序列号 + 1 后作为确认号呢？

SYN 报文是特殊的 TCP 报文，用于建立连接时使用，虽然 SYN 报文不携带用户数据，但是 **TCP 将 SYN 报文视为 1 字节的数据**，当对方收到了 SYN 报文后，在回复 ACK 报文时，就需要将 ACK 报文中的确认号设置为 SYN 的序列号 + 1 ，这样做是有两个目的：

- **告诉对方，我方已经收到 SYN 报文。**
- **告诉对方，我方下一次「期望」收到的报文的序列号为此确认号，比如客户端与服务端完成三次握手之后，服务端接下来期望收到的是序列号为 client_isn + 1 的 TCP 数据报文。**

### 数据传输阶段的变化

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/dadf9a94328a4446b32ebabf1623c729.png)

客户端发送 10 字节的数据，通常 TCP 数据报文的控制位是 [PSH, ACK]，此时该 TCP 数据报文的序列号和确认号分别设置为：

- 序列号设置为 client_isn + 1。客户端上一次发送报文是 ACK 报文（第三次握手），该报文的 seq = client_isn + 1，由于是一个单纯的 ACK 报文，没有携带用户数据，所以 len = 0。根据公式 1（_序列号 = 上一次发送的序列号 + len_），可以得出当前的序列号为 client_isn + 1 + 0，即 client_isn + 1。
- 确认号设置为 server_isn + 1。没错，还是和第三次握手的 ACK 报文的确认号一样，这是因为客户端三次握手之后，发送 TCP 数据报文 之前，如果没有收到服务端的 TCP 数据报文，确认号还是延用上一次的，其实根据公式 2 你也能得到这个结论。

可以看到，**客户端与服务端完成 TCP 三次握手后，发送的第一个 「TCP 数据报文的序列号和确认号」都是和「第三次握手的 ACK 报文中序列号和确认号」一样的**。

接着，当服务端收到客户端 10 字节的 TCP 数据报文后，就需要回复一个 ACK 报文，此时该报文的序列号和确认号分别设置为：

- 序列号设置为 server_isn + 1。服务端上一次发送报文是 SYN-ACK 报文，序列号为 server_isn，根据公式 1（_序列号 = 上一次发送的序列号 + len。特殊情况，如果上一次发送的报文是 SYN 报文或者 FIN 报文，则改为 + 1_），所以当前的序列号为 server_isn + 1。
- 确认号设置为 client_isn + 11 。服务端上一次收到的报文是客户端发来的 10 字节 TCP 数据报文，该报文的 seq = client_isn + 1，len = 10。根据公式 2（_确认号 = 上一次收到的报文中的序列号 + len_），也就是将「收到的 TCP 数据报文中的序列号 client_isn + 1，再加上 10（len = 10） 」的值作为了确认号，表示自己收到了该 10 字节的数据报文。

> 如果客户端发送的第三次握手 ACK 报文丢失了，处于 SYN_RCVD 状态服务端收到了客户端第一个 TCP 数据报文会发生什么？

刚才前面我也说了，发送的第一个 「TCP 数据报文的序列号和确认号」都是和「第三次握手的 ACK 报文中序列号和确认号」一样的，并且该 TCP 数据报文也有将 ACK 标记位置为 1。如下图：

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/942c2a1e67224c8c8bd41b13d7c89a96.png)

所以，服务端收到这个数据报文，是可以正常完成连接的建立，然后就可以正常接收这个数据包了。

### 四次挥手阶段的变化

![在这里插入图片描述](https://cdn.xiaolincoding.com//mysql/other/ae18cbf6071c47b98014a68d05c37d16.png)

客户端发送的第一次挥手的序列号和确认号分别设置为：

- 序列号设置为 client_isn + 11。客户端上一次发送的报文是 [PSH, ACK] ，该报文的 seq = client_isn + 1, len = 10，根据公式 1（_序列号 = 上一次发送的序列号 + len_），可以得出当前的序列号为 client_isn + 11。
- 确认号设置为 server_isn + 1。客户端上一次收到的报文是服务端发来的 ACK 报文，该报文的 seq = server_isn + 1，是单纯的 ACK 报文，不携带用户数据，所以 len 为 0。那么根据公式 2（确认号 = 上一次收到的序列号 + len），可以得出当前的确认号为 server_isn + 1 + 0 （len = 0），也就是 server_isn + 1。

服务端发送的第二次挥手的序列号和确认号分别设置为：

- 序列号设置为 server_isn + 1。服务端上一次发送的报文是 ACK 报文，该报文的 seq = server_isn + 1，而该报文是单纯的 ACK 报文，不携带用户数据，所以 len 为 0，根据公式 1（_序列号 = 上一次发送的序列号 + len_），可以得出当前的序列号为 server_isn + 1 + 0 （len = 0），也就是 server_isn + 1。
- 确认号设置为 client*isn + 12。服务端上一次收到的报文是客户端发来的 FIN 报文，该报文的 seq = client_isn + 11，根据公式 2（*确认号= _上一次\_收到的序列号 + len，特殊情况，如果收到报文是 SYN 报文或者 FIN 报文，则改为 + 1_），可以得出当前的确认号为 client_isn + 11 + 1，也就是 client_isn + 12。

服务端发送的第三次挥手的序列号和确认号还是和第二次挥手中的序列号和确认号一样。

- 序列号设置为 server_isn + 1。
- 确认号设置为 client_isn + 12。

客户端发送的四次挥手的序列号和确认号分别设置为：

- 序列号设置为 client_isn + 12。客户端上一次发送的报文是 FIN 报文，该报文的 seq = client_isn + 11，根据公式 1（_序列号 = 上一次发送的序列号 + len。特殊情况，如果收到报文是 SYN 报文或者 FIN 报文，则改为 + 1_），可以得出当前的序列号为 client_isn + 11 + 1，也就是 client_isn + 12。
- 确认号设置为 server*isn + 2。客户端上一次收到的报文是服务端发来的 FIN 报文，该报文的 seq = server_isn + 1，根据公式 2（*确认号 = _上一次\_收到的序列号 + len，特殊情况，如果收到报文是 SYN 报文或者 FIN 报文，则改为 + 1_），可以得出当前的确认号为 server_isn + 1 + 1，也就是 server_isn + 2。
