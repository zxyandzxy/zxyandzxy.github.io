---
layout: post
title: "硬件结构"
date: 2024-5-19
tags: [operating system]
comments: true
author: zxy
---

## CPU 是如何执行程序的

### 冯诺依曼模型

五个基本部分：运算器、控制器、存储器、输入设备、输出设备

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/Von_Neumann_architecture.svg)

对应现在的计算机即为：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E7%A8%8B%E5%BA%8F%E6%89%A7%E8%A1%8C/%E5%86%AF%E8%AF%BA%E4%BE%9D%E6%9B%BC%E6%A8%A1%E5%9E%8B.png)

- 运算器对应逻辑，算术运算单元
- 控制器对应控制单元
- 存储器对应内存
- 输入/输出设备对应计算机外接，键盘、显示器等
- 总线是通信的桥梁

#### 内存

- 程序和数据都是存储在内存中的，存储区域是线性的。可以按照数组理解
- 存储数据的基本单位是字节(byte)，1byte = 8bit，一个字节对应一个内存地址
- 内存地址从 0 开始编号，最大地址为总字节数 - 1。**访问任意内存地址的速度相同**

#### CPU

我们常说 32 位和 64 位系统，这个指的是 CPU 的位宽，也可以理解为 CPU 一次可以运算的数字大小。32 位 CPU 一次能运算的数字最大为 2^32 = 4294967295。

从这个角度理解，64 位对比 32 位 CPU，能计算的数字范围就更大了。即可以计算 2^33 \* 2^33 这种。

CPU 内部的组件有**寄存器、控制单元、逻辑运算单元**，控制单元控制 CPU 工作，逻辑运算单元负责计算，寄存器主要用于存储计算时的数据。

> 寄存器种类

- 通用寄存器：存放要进行运算的数据，比如要相加的两个数字
- 程序计数器：存储 CPU 要执行的下一条指令的地址
- 指令寄存器：存放当前正在执行的指令

#### 总线

总线负责 CPU、内存、其他设备之间的通信，分为：

- 地址总线：指定 CPU 要操作的内存地址
- 数据总线：读写内存的数据
- 控制总线：发送和接收信号，如中断，设备复位等

CPU 读写内存数据时：

- 通过地址总线指定内存的地址
- 通过控制总线决定读/写
- 通过数据总线传输数据

### 线路位宽与 CPU 位宽

> 线路位宽

在实际的物理机中，各种设备用电线来连接；在线路上传输数据实际上是在传输电压。0 表示低电压，1 表示高电压。有两种传输方式：

- 串行传输：比如我要将 5 从内存传到 CPU，那么就要 101，三个 bit 一个一个传，一条电线
- 并行传输：101 三个 bit 我同时传输，这需要三条电线

那么我想要操作 4G 内存，地址大小是 2^32，那么就需要 32 条总线来并行传输 32 位地址

> CPU 位宽

- CPU 位宽其实就是上面说的，CPU 一次能计算的最大数字大小。
- CPU 位宽和线路位宽最好匹配，这样对于访存和数字计算的工作都不复杂
- 32 位 CPU 是可以实现两个 64 位数字相加的操作的，首先要将 64 位数字分拆为高 32 位和低 32 位；然后分别相加。
- 64 位 CPU 不一定比 32 位 CPU 性能高很多，因为大部分计算不会超过 2^32 这个数字
- 32 位 CPU 只能操作 2^32 = 4G 内存，因为地址在 CPU 看来也是数字，只有 32 位数字能处理

### 程序执行的基本过程

程序经过编译/解释后载入内存中，变成了一条条指令，程序的执行过程就是 CPU 执行一条条指令的过程。

- 第一步，CPU 读取「程序计数器」的值，这个值是指令的内存地址，然后 CPU 的「控制单元」操作「地址总线」指定需要访问的内存地址，接着通知内存设备准备数据，数据准备好后通过「数据总线」将指令数据传给 CPU，CPU 收到内存传来的数据后，将这个指令数据存入到「指令寄存器」。
- 第二步，「程序计数器」的值自增，表示指向下一条指令。这个自增的大小，由 CPU 的位宽决定，比如 32 位的 CPU，指令是 4 个字节，需要 4 个内存地址存放，因此「程序计数器」的值会自增 4；
- 第三步，CPU 分析「指令寄存器」中的指令，确定指令的类型和参数，如果是计算类型的指令，就把指令交给「逻辑运算单元」运算；如果是存储类型的指令，则交由「控制单元」执行；
- 就这样不停地取指令，分析指令和执行指令。**一个循环称为 CPU 指令周期**

### a = 1 + 2 执行具体过程（32 位 CPU）

a = 1 + 2 这是高级语言的书写，经过编译后会变成机器码。这条语句被编译成 4 条指令

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E7%A8%8B%E5%BA%8F%E6%89%A7%E8%A1%8C/%E6%95%B0%E6%8D%AE%E6%AE%B5%E4%B8%8E%E6%AD%A3%E6%96%87%E6%AE%B5.png)

- 0x100 的内容是 `load` 指令将 0x200 地址中的数据 1 装入到寄存器 `R0`；
- 0x104 的内容是 `load` 指令将 0x204 地址中的数据 2 装入到寄存器 `R1`；
- 0x108 的内容是 `add` 指令将寄存器 `R0` 和 `R1` 的数据相加，并把结果存放到寄存器 `R2`；
- 0x10c 的内容是 `store` 指令将寄存器 `R2` 中的数据存回数据段中的 0x208 地址中，这个地址也就是变量 `a` 内存中的地址；

在编译完成后，程序计数器设置为 0x100，开始执行 4 条指令

#### 指令

指令在 CPU 看来其实是机器码，不是我们看到的高级代码/汇编语句

不同的 CPU 有不同的指令集**（生产 CPU 的产商会把指令集设置好，这是硬件问题）**，也就是对应着不同的汇编语言和不同的机器码，接下来选用最简单的 MIPS 指集，来看看机器码是如何生成的，这样也能明白二进制的机器码的具体含义。

MIPS 的指令是一个 32 位的整数，高 6 位代表着操作码，表示这条指令是一条什么样的指令，剩下的 26 位不同指令类型所表示的内容也就不相同，主要有三种类型 R、I 和 J。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E7%A8%8B%E5%BA%8F%E6%89%A7%E8%A1%8C/MIPS%E6%8C%87%E4%BB%A4%E9%9B%86.png)

- _R 指令_，用在算术和逻辑操作，里面有读取和写入数据的寄存器地址。如果是逻辑位移操作，后面还有位移操作的「位移量」，而最后的「功能码」则是再前面的操作码不够的时候，扩展操作码来表示对应的具体指令的；
- _I 指令_，用在数据传输、条件分支等。这个类型的指令，就没有了位移量和功能码，也没有了第三个寄存器，而是把这三部分直接合并成了一个地址值或一个常数；
- _J 指令_，用在跳转，高 6 位之外的 26 位都是一个跳转后的地址；

栗子：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E7%A8%8B%E5%BA%8F%E6%89%A7%E8%A1%8C/add%E7%9A%84MIPS%E6%8C%87%E4%BB%A4.png)

加和运算 add 指令是属于 R 指令类型：

- add 对应的 MIPS 指令里操作码是 `000000`，以及最末尾的功能码是 `100000`，这些数值都是固定的，查一下 MIPS 指令集的手册就能知道的；
- rs 代表第一个寄存器 R0 的编号，即 `00000`；
- rt 代表第二个寄存器 R1 的编号，即 `00001`；
- rd 代表目标的临时寄存器 R2 的编号，即 `00010`；
- 因为不是位移操作，所以位移量是 `00000`

把上面这些数字拼在一起就是一条 32 位的 MIPS 加法指令了，那么用 16 进制表示的机器码则是 `0x00011020`。

编译器在编译程序的时候，会构造指令，这个过程叫做指令的编码。CPU 执行程序的时候，就会解析指令，这个过程叫作指令的解码。

现代大多数 CPU 都使用来流水线的方式来执行指令，所谓的流水线就是把一个任务拆分成多个小任务，于是一条指令通常分为 4 个阶段，称为 4 级流水线，如下图：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E7%A8%8B%E5%BA%8F%E6%89%A7%E8%A1%8C/CPU%E6%8C%87%E4%BB%A4%E5%91%A8%E6%9C%9F.png)

1. CPU 通过程序计数器读取对应内存地址的指令，这个部分称为 **Fetch（取得指令）**；
2. CPU 对指令进行解码，这个部分称为 **Decode（指令译码）**；
3. CPU 执行指令，这个部分称为 **Execution（执行指令）**；
4. CPU 将计算结果存回寄存器或者将寄存器的值存入内存，这个部分称为 **Store（数据回写）**；

上面这 4 个阶段，我们称为**指令周期（Instrution Cycle）**，CPU 的工作就是一个周期接着一个周期，周而复始。

事实上，不同的阶段其实是由计算机中的不同组件完成的：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E7%A8%8B%E5%BA%8F%E6%89%A7%E8%A1%8C/%E6%8C%87%E4%BB%A4%E5%91%A8%E6%9C%9F%E5%B7%A5%E4%BD%9C%E7%BB%84%E4%BB%B6.png)

- 取指令的阶段，我们的指令是存放在**存储器**里的，实际上，通过程序计数器和指令寄存器取出指令的过程，是由**控制器**操作的；
- 指令的译码过程，也是由**控制器**进行的；
- 指令执行的过程，无论是进行算术操作、逻辑操作，还是进行数据传输、条件分支操作，都是由**算术逻辑单元**操作的，也就是由**运算器**处理的。但是如果是一个简单的无条件地址跳转，则是直接在**控制器**里面完成的，不需要用到运算器。

#### 指令的类型

- _数据传输类型的指令_，比如 `store/load` 是寄存器与内存间数据传输的指令，`mov` 是将一个内存地址的数据移动到另一个内存地址的指令；
- _运算类型的指令_，比如加减乘除、位运算、比较大小等等，它们最多只能处理两个寄存器中的数据；
- _跳转类型的指令_，通过修改程序计数器的值来达到跳转执行指令的过程，比如编程中常见的 `if-else`、`switch-case`、函数调用等。
- _信号类型的指令_，比如发生中断的指令 `trap`；
- _闲置类型的指令_，比如指令 `nop`，执行后 CPU 会空转一个周期；

#### 指令的执行速度

CPU 的硬件参数都会有 `GHz` 这个参数，比如一个 1 GHz 的 CPU，指的是时钟频率是 1 G，代表着 1 秒会产生 1G 次数的脉冲信号，每一次脉冲信号高低电平的转换就是一个周期，称为时钟周期。

对于 CPU 来说，在一个时钟周期内，CPU 仅能完成一个最基本的动作，时钟频率越高，时钟周期就越短，工作速度也就越快。

一个时钟周期一定能执行完一条指令吗？答案是不一定的，大多数指令不能在一个时钟周期完成，通常需要若干个时钟周期。不同的指令需要的时钟周期是不同的，加法和乘法都对应着一条 CPU 指令，但是乘法需要的时钟周期就要比加法多。

> 如何让程序跑的更快？

程序执行的时候，耗费的 CPU 时间少就说明程序是快的，对于程序的 CPU 执行时间，我们可以拆解成 **CPU 时钟周期数（\*CPU Cycles\*）和时钟周期时间（\*Clock Cycle Time\*）的乘积**。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E7%A8%8B%E5%BA%8F%E6%89%A7%E8%A1%8C/%E7%A8%8B%E5%BA%8F%E7%9A%84CPU%E6%89%A7%E8%A1%8C%E6%97%B6%E9%97%B4%E5%85%AC%E5%BC%8F1.png)

时钟周期时间就是我们前面提及的 CPU 主频，主频越高说明 CPU 的工作速度就越快，比如我手头上的电脑的 CPU 是 2.4 GHz 四核 Intel Core i5，这里的 2.4 GHz 就是电脑的主频，时钟周期时间就是 1/2.4G。

CPU 主频是硬件决定的，跟软件工程师没什么关系，我们只能在时钟周期数上下手

对于 CPU 时钟周期数我们可以进一步拆解成：「**指令数 x 每条指令的平均时钟周期数（\*Cycles Per Instruction\*，简称 `CPI`）**」

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E7%A8%8B%E5%BA%8F%E6%89%A7%E8%A1%8C/%E7%A8%8B%E5%BA%8F%E7%9A%84CPU%E6%89%A7%E8%A1%8C%E6%97%B6%E9%97%B4%E5%85%AC%E5%BC%8F2.png)

- _指令数_，表示执行程序所需要多少条指令，以及哪些指令。这个层面是基本靠编译器来优化，毕竟同样的代码，在不同的编译器，编译出来的计算机指令会有各种不同的表示方式。
- _每条指令的平均时钟周期数 CPI_，表示一条指令需要多少个时钟周期数，现代大多数 CPU 通过流水线技术（Pipeline），让一条指令需要的 CPU 时钟周期数尽可能的少；
- _时钟周期时间_，表示计算机主频，取决于计算机硬件。有的 CPU 支持超频技术，打开了超频意味着把 CPU 内部的时钟给调快了，于是 CPU 工作速度就变快了，但是也是有代价的，CPU 跑的越快，散热的压力就会越大，CPU 会很容易奔溃。

## 磁盘比内存慢几万倍

### 存储器的层次结构

按照价格、读取速度递减，存储容量递增的规则排序

- 寄存器，访问速度一般在半个 CPU 时钟周期
- L1 cache
- L2 cache
- L3 cache
- 内存
- SSD / HDD 硬盘

#### CPU Cache

CPU Cache 用的是一种叫 **SRAM（\*Static Random-Access\* Memory，静态随机存储器）** 的芯片。

SRAM 之所以叫「静态」存储器，是因为只要有电，数据就可以保持存在，而一旦断电，数据就会丢失了。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E5%AD%98%E5%82%A8%E7%BB%93%E6%9E%84/CPU-Cache.png)

> L1 高速缓存

L1 高速缓存的访问速度几乎和寄存器一样快，通常只需要 `2~4` 个时钟周期，而大小在几十 KB 到几百 KB 不等。

每个 CPU 核心都有一块属于自己的 L1 高速缓存，指令和数据在 L1 是分开存放的，所以 L1 高速缓存通常分成**指令缓存**和**数据缓存**。

在 Linux 系统，我们可以通过这条命令，查看 CPU 里的 L1 Cache 「数据」缓存的容量大小：

```bash
$ cat /sys/devices/system/cpu/cpu0/cache/index0/size
32K
```

而查看 L1 Cache 「指令」缓存的容量大小，则是：

```bash
$ cat /sys/devices/system/cpu/cpu0/cache/index1/size
32K
```

> L2 高速缓存

L2 高速缓存同样每个 CPU 核心都有，但是 L2 高速缓存位置比 L1 高速缓存距离 CPU 核心 更远，它大小比 L1 高速缓存更大，CPU 型号不同大小也就不同，通常大小在几百 KB 到几 MB 不等，访问速度则更慢，速度在 `10~20` 个时钟周期。

在 Linux 系统，我们可以通过这条命令，查看 CPU 里的 L2 Cache 的容量大小：

```bash
$ cat /sys/devices/system/cpu/cpu0/cache/index2/size
256K
```

> L3 高速缓存

L3 高速缓存通常是多个 CPU 核心共用的，位置比 L2 高速缓存距离 CPU 核心 更远，大小也会更大些，通常大小在几 MB 到几十 MB 不等，具体值根据 CPU 型号而定。访问速度相对也比较慢一些，访问速度在 `20~60`个时钟周期。

在 Linux 系统，我们可以通过这条命令，查看 CPU 里的 L3 Cache 的容量大小：

```bash
$ cat /sys/devices/system/cpu/cpu0/cache/index3/size
3072K
```

#### 内存

内存使用的是一种叫作 **DRAM （\*Dynamic Random Access Memory\*，动态随机存取存储器）** 的芯片。相比 SRAM，DRAM 的密度更高，功耗更低，有更大的容量，而且造价比 SRAM 芯片便宜很多。

DRAM 存储一个 bit 数据，只需要一个晶体管和一个电容就能存储，但是因为数据会被存储在电容里，电容会不断漏电，所以需要「定时刷新」电容，才能保证数据不会被丢失，这就是 DRAM 之所以被称为「动态」存储器的原因，只有不断刷新，数据才能被存储起来。

DRAM 的数据访问电路和刷新电路都比 SRAM 更复杂，所以访问的速度会更慢，内存速度大概在 `200~300` 个 时钟周期之间。

#### SSD/HDD 硬盘

SSD（_Solid-state disk_） 就是我们常说的固体硬盘，结构和内存类似，但是它相比内存的优点是断电后数据还是存在的，而内存、寄存器、高速缓存断电后数据都会丢失。内存的读写速度比 SSD 大概快 `10~1000` 倍。

当然，还有一款传统的硬盘，也就是机械硬盘（_Hard Disk Drive, HDD_），它是通过物理读写的方式来访问数据的，因此它访问速度是非常慢的，它的速度比内存慢 `10W` 倍左右。

由于 SSD 的价格快接近机械硬盘了，因此机械硬盘已经逐渐被 SSD 替代了。

### 存储器的层次关系

每个存储器只和相邻的一层存储器设备打交道，并且存储设备为了追求更快的速度，所需的材料成本必然也是更高，也正因为成本太高，所以 CPU 内部的寄存器、L1\L2\L3 Cache 只好用较小的容量，相反内存、硬盘则可用更大的容量，这就我们今天所说的存储器层次结构。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E5%AD%98%E5%82%A8%E7%BB%93%E6%9E%84/%E7%BC%93%E5%AD%98%E4%BD%93%E7%B3%BB1.png)

这就是缓存体系

## 如何写出让 CPU 跑得更快的代码

### CPU Cache 的数据结构和读取过程是什么样的？

CPU Cache 是由很多个 Cache Line 组成的，Cache Line 是 CPU 从内存读取数据的基本单位，而 Cache Line 是由各种标志（Tag）+ 数据块（Data Block）组成

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98%E4%B8%80%E8%87%B4%E6%80%A7/Cache%E7%9A%84%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.png)

CPU Cache 的数据是从内存中读取过来的，它是以一小块一小块读取数据的，而不是按照单个数组元素来读取数据的，在 CPU Cache 中的，这样一小块一小块的数据，称为 **Cache Line（缓存块）**。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98/%E6%9F%A5%E7%9C%8BCPULine%E5%A4%A7%E5%B0%8F.png)

比如，有一个 `int array[100]` 的数组，当载入 `array[0]` 时，由于这个数组元素的大小在内存只占 4 字节，不足 64 字节，CPU 就会**顺序加载**数组元素到 `array[15]`，意味着 `array[0]~array[15]` 数组元素都会被缓存在 CPU Cache 中了，因此当下次访问这些数组元素时，会直接从 CPU Cache 读取，而不用再从内存中读取，大大提高了 CPU 读取数据的性能。

> 那怎么判断数据在不在缓存中呢 ？

#### 直接映射

内存块的地址始终「映射」在一个 CPU Cache Line（缓存块） 的地址。

至于映射关系实现方式，则是使用「取模运算」，取模运算的结果就是内存块地址对应的 CPU Cache Line（缓存块） 的地址。

使用取模方式映射的话，就会出现多个内存块对应同一个 CPU Cache Line

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98/%E6%B1%82%E6%A8%A1%E6%98%A0%E5%B0%84%E7%AD%96%E7%95%A5.png)

为了区别不同的内存块，在对应的 CPU Cache Line 中我们还会存储一个**组标记（Tag）**。这个组标记会记录当前 CPU Cache Line 中存储的数据对应的内存块，我们可以用这个组标记来区分不同的内存块。

除了组标记信息外，CPU Cache Line 还有两个信息：

- 一个是，从内存加载过来的实际存放**数据（\*Data\*）**。
- 另一个是，**有效位（\*Valid bit\*）**，它是用来标记对应的 CPU Cache Line 中的数据是否是有效的，如果有效位是 0，无论 CPU Cache Line 中是否有数据，CPU 都会直接访问内存，重新加载数据。

CPU 在从 CPU Cache 读取数据的时候，并不是读取 CPU Cache Line 中的整个数据块，而是读取 CPU 所需要的一个数据片段，这样的数据统称为一个**字（\*Word\*）**。那怎么在对应的 CPU Cache Line 中数据块中找到所需的字呢？答案是，需要一个**偏移量（Offset）**。

因此，一个内存的访问地址，包括**组标记、CPU Cache Line 索引、偏移量**这三种信息，于是 CPU 就能通过这些信息，在 CPU Cache 中找到缓存的数据。而对于 CPU Cache 里的数据结构，则是由**索引 + 有效位 + 组标记 + 数据块**组成。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98/%E7%9B%B4%E6%8E%A5Cache%E6%98%A0%E5%B0%84.png)

如果内存中的数据已经在 CPU Cache 中了，那 CPU 访问一个内存地址的时候，会经历这 4 个步骤：

1. 根据内存地址中索引信息，计算在 CPU Cache 中的索引，也就是找出对应的 CPU Cache Line 的地址；
2. 找到对应 CPU Cache Line 后，判断 CPU Cache Line 中的有效位，确认 CPU Cache Line 中数据是否是有效的，如果是无效的，CPU 就会直接访问内存，并重新加载数据，如果数据有效，则往下执行；
3. 对比内存地址中组标记和 CPU Cache Line 中的组标记，确认 CPU Cache Line 中的数据是我们要访问的内存数据，如果不是的话，CPU 就会直接访问内存，并重新加载数据，如果是的话，则往下执行；
4. 根据内存地址中偏移量信息，从 CPU Cache Line 的数据块中，读取对应的字。

### 使用缓存加速程序执行

我们知道 CPU 访问内存的速度，比访问 CPU Cache 的速度慢了 100 多倍，所以如果 CPU 所要操作的数据在 CPU Cache 中的话，这样将会带来很大的性能提升。访问的数据在 CPU Cache 中的话，意味着**缓存命中**，缓存命中率越高的话，代码的性能就会越好，CPU 也就跑的越快。

于是，「如何写出让 CPU 跑得更快的代码？」这个问题，可以改成「如何写出 CPU 缓存命中率高的代码？」。

#### 如何提升数据缓存的命中率？

假设要遍历二维数组，有以下两种形式

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98/%E9%81%8D%E5%8E%86%E6%95%B0%E7%BB%84.png)

经过测试，形式一 `array[i][j]` 执行时间比形式二 `array[j][i]` 快好几倍。

之所以有这么大的差距，是因为二维数组 `array` 所占用的内存是连续的，比如长度 `N` 的值是 `2` 的话，那么内存中的数组元素的布局顺序是这样的：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98/%E6%95%B0%E7%BB%84%E5%86%85%E5%AD%98%E5%B8%83%E5%B1%80%E9%A1%BA%E5%BA%8F.png)

形式一用 `array[i][j]` 访问数组元素的顺序，正是和内存中数组元素存放的顺序一致。当 CPU 访问 `array[0][0]` 时，由于该数据不在 Cache 中，于是会「顺序」把跟随其后的 3 个元素从内存中加载到 CPU Cache，这样当 CPU 访问后面的 3 个数组元素时，就能在 CPU Cache 中成功地找到数据，这意味着缓存命中率很高，缓存命中的数据不需要访问内存，这便大大提高了代码的性能。

而如果用形式二的 `array[j][i]` 来访问，则访问的顺序就是：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98/%E5%BD%A2%E5%BC%8F%E4%BA%8C%E8%AE%BF%E9%97%AE%E9%A1%BA%E5%BA%8F.png)

你可以看到，访问的方式跳跃式的，而不是顺序的，那么如果 N 的数值很大，那么操作 `array[j][i]` 时，是没办法把 `array[j+1][i]` 也读入到 CPU Cache 中的，既然 `array[j+1][i]` 没有读取到 CPU Cache，那么就需要从内存读取该数据元素了。很明显，这种不连续性、跳跃式访问数据元素的方式，可能不能充分利用到了 CPU Cache 的特性，从而代码的性能不高。

当 CPU 访问内存数据时，如果数据不在 CPU Cache 中，则会一次性会连续加载 64 字节大小的数据到 CPU Cache，那么当访问 `array[0][0]` 时，由于该元素不足 64 字节，于是就会往后**顺序**读取 `array[0][0]~array[0][15]` 到 CPU Cache 中。顺序访问的 `array[i][j]` 因为利用了这一特点，所以就会比跳跃式访问的 `array[j][i]` 要快。

**因此，遇到这种遍历数组的情况时，按照内存布局顺序访问，将可以有效的利用 CPU Cache 带来的好处，这样我们代码的性能就会得到很大的提升**

#### 如何提升指令缓存的命中率？

提升数据的缓存命中率的方式，是按照内存布局顺序访问，那针对指令的缓存该如何提升呢？

我们以一个例子来看看，有一个元素为 0 到 100 之间随机数字组成的一维数组：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98/%E9%9A%8F%E6%9C%BA%E6%95%B0%E6%95%B0%E7%BB%84.png)

接下来，对这个数组做两个操作：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98/%E6%8E%92%E5%BA%8Fand%E6%95%B0%E7%BB%84%E9%81%8D%E5%8E%86.png)

- 第一个操作，循环遍历数组，把小于 50 的数组元素置为 0；
- 第二个操作，将数组排序；

那么问题来了，你觉得先遍历再排序速度快，还是先排序再遍历速度快呢？

在回答这个问题之前，我们先了解 CPU 的**分支预测器**。对于 if 条件语句，意味着此时至少可以选择跳转到两段不同的指令执行，也就是 if 还是 else 中的指令。那么，**如果分支预测可以预测到接下来要执行 if 里的指令，还是 else 指令的话，就可以「提前」把这些指令放在指令缓存中，这样 CPU 可以直接从 Cache 读取到指令，于是执行速度就会很快**。

当数组中的元素是随机的，分支预测就无法有效工作，而当数组元素都是是顺序的，分支预测器会动态地根据历史命中数据对未来进行预测，这样命中率就会很高。

因此，**先排序再遍历速度会更快**，这是因为排序之后，数字是从小到大的，那么前几次循环命中 `if < 50` 的次数会比较多，于是分支预测就会缓存 `if` 里的 `array[i] = 0` 指令到 Cache 中，后续 CPU 执行该指令就只需要从 Cache 读取就好了。

如果你肯定代码中的 `if` 中的表达式判断为 `true` 的概率比较高，我们可以使用显示分支预测工具，比如在 C/C++ 语言中编译器提供了 `likely` 和 `unlikely` 这两种宏，如果 `if` 条件为 `ture` 的概率大，则可以用 `likely` 宏把 `if` 里的表达式包裹起来，反之用 `unlikely` 宏。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98/likely.png)

实际上，CPU 自身的动态分支预测已经是比较准的了，所以只有当非常确信 CPU 预测的不准，且能够知道实际的概率情况时，才建议使用这两种宏。

#### 如何提升多核 CPU 的缓存命中率？

现代 CPU 都是多核心的，线程可能在不同 CPU 核心来回切换执行，这对 CPU Cache 不是有利的，虽然 L3 Cache 是多核心之间共享的，但是 L1 和 L2 Cache 都是每个核心独有的，**如果一个线程在不同核心来回切换，各个核心的缓存命中率就会受到影响**，相反如果线程都在同一个核心上执行，那么其数据的 L1 和 L2 Cache 的缓存命中率可以得到有效提高，缓存命中率高就意味着 CPU 可以减少访问 内存的频率。

当有多个同时执行「计算密集型」的线程，为了防止因为切换到不同的核心，而导致缓存命中率下降的问题，我们可以把**线程绑定在某一个 CPU 核心上**，这样性能可以得到非常可观的提升。

在 Linux 上提供了 `sched_setaffinity` 方法，来实现将线程绑定到某个 CPU 核心这一功能。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98/sched_setaffinity.png)

## CPU 缓存一致性

### CPU Cache 的数据写入

在上文讲过，硬件只会跟它相邻的硬件交互，比如 CPU 只会和 L1 缓存交互，内存只会和 L3 缓存交互。

对于 CPU 来说，不仅有读取数据，还会写入数据，那么 CPU 是写入缓存的，找到实际写回内存才能保证数据的同步。

有两种方法：写直达 + 写回

#### 写直达

保持内存与 Cache 一致性最简单的方式是，**把数据同时写入内存和 Cache 中**，这种方法称为**写直达（\*Write Through\*）**。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98%E4%B8%80%E8%87%B4%E6%80%A7/%E5%86%99%E7%9B%B4%E8%BE%BE.png)

问题很明显啊，数据在不在缓存中，CPU 都要执行将数据写入内存的操作，这很耗时。

#### 写回

写回机制中，**当发生写操作时，新的数据仅仅被写入 Cache Block 里，只有当修改过的 Cache Block「被替换」时才需要写到内存中**，减少了数据写回内存的频率，这样便可以提高系统的性能。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98%E4%B8%80%E8%87%B4%E6%80%A7/%E5%86%99%E5%9B%9E1.png)

- 如果当发生写操作时，数据已经在 CPU Cache 里的话，则把数据更新到 CPU Cache 里，同时标记 CPU Cache 里的这个 Cache Block 为脏（Dirty）的，这个脏的标记代表这个时候，我们 CPU Cache 里面的这个 Cache Block 的数据和内存是不一致的，这种情况是不用把数据写到内存里的；
- 如果当发生写操作时，数据所对应的 Cache Block 里存放的是「别的内存地址的数据」的话，就要检查这个 Cache Block 里的数据有没有被标记为脏的：
  - 如果是脏的话，我们就要把这个 Cache Block 里的数据写回到内存，然后再把当前要写入的数据，先从内存读入到 Cache Block 里（**不这样做，那么相当于别的内存地址的数据被修改了，但没写写回内存，相当于白做了那条指令**），然后再把当前要写入的数据写入到 Cache Block，最后也把它标记为脏的；
  - 如果不是脏的话，把当前要写入的数据先从内存读入到 Cache Block 里，接着将数据写入到这个 Cache Block 里，然后再把这个 Cache Block 标记为脏的就好了。

**关于写回机制，读写流程如下**

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E5%85%AB%E8%82%A1%E6%96%87/writeback.png)

### 缓存一致性问题

#### 问题产生原因

现代 CPU 是多核的，L1/L2 级缓存是多个 CPU 核心各自独有的，如果多个 CPU 核心读取同一块内存的数据，在实际上它们是读取各自缓存中的数据，这就会产生缓存一致性问题。

栗子：

这时如果 A 号核心执行了 `i++` 语句的时候，为了考虑性能，使用了我们前面所说的写回策略，先把值为 `1` 的执行结果写入到 L1/L2 Cache 中，然后把 L1/L2 Cache 中对应的 Block 标记为脏的，这个时候数据其实没有被同步到内存中的，因为写回策略，只有在 A 号核心中的这个 Cache Block 要被替换的时候，数据才会写入到内存里。

如果这时旁边的 B 号核心尝试从内存读取 i 变量的值，则读到的将会是错误的值，因为刚才 A 号核心更新 i 值还没写入到内存中，内存中的值还依然是 0。**这个就是所谓的缓存一致性问题，A 号核心和 B 号核心的缓存，在这个时候是不一致，从而会导致执行结果的错误。**

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98%E4%B8%80%E8%87%B4%E6%80%A7/%E7%BC%93%E5%AD%98%E4%B8%80%E8%87%B4%E6%80%A7%E9%97%AE%E9%A2%98%E4%BE%8B%E5%AD%902.png)

解决缓存一致性问题的两个关键点：

- 第一点，某个 CPU 核心里的 Cache 数据更新时，必须要传播到其他核心的 Cache，这个称为**写传播（\*Write Propagation\*）**；
- 第二点，多个 CPU 核心里对数据的操作顺序，必须在其他核心看起来顺序是一样的，这个称为**事务的串行化（\*Transaction Serialization\*）**。

> 理解事务串行化

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98%E4%B8%80%E8%87%B4%E6%80%A7/%E4%BA%8B%E4%BB%B6%E9%A1%BA%E5%BA%8F%E9%97%AE%E9%A2%98.png)

假设我们有一个含有 4 个核心的 CPU，这 4 个核心都操作共同的变量 i（初始值为 0 ）。A 号核心先把 i 值变为 100，而此时同一时间，B 号核心先把 i 值变为 200，这里两个修改，都会「传播」到 C 和 D 号核心。

如果 C,D 两个核心收到 A,B 两个核心更改数据的顺序不一致就会产生各个核心中 cache 的数据值不同。

所以，我们要保证 C 号核心和 D 号核心都能看到**相同顺序的数据变化**，比如变量 i 都是先变成 100，再变成 200，这样的过程就是事务的串行化。

要实现事务串行化，要做到 2 点：

- CPU 核心对于 Cache 中数据的操作，需要同步给其他 CPU 核心；
- 要引入「锁」的概念，如果两个 CPU 核心里有相同数据的 Cache，那么对于这个 Cache 数据的更新，只有拿到了「锁」，才能进行对应的数据更新。

### 总线嗅探：解决写传播问题

以前面的 i 变量例子来说明总线嗅探的工作机制，当 A 号 CPU 核心修改了 L1 Cache 中 i 变量的值，通过**总线把这个事件广播通知给其他所有的核心**，然后**每个 CPU 核心都会监听总线上的广播事件**，并检查是否有相同的数据在自己的 L1 Cache 里面，如果 B 号 CPU 核心的 L1 Cache 中有该数据，那么也需要把该数据更新到自己的 L1 Cache。

总线嗅探的问题很明显：一个是没办法保证事务串行化，另一个是 CPU 需要时刻监听总线活动，加重总线负担。

### MESI 协议：缓存一致性解决方案

- _Modified_，已修改
- _Exclusive_，独占
- _Shared_，共享
- _Invalidated_，已失效

「已修改」状态就是我们前面提到的脏标记，代表该 Cache Block 上的数据已经被更新过，但是还没有写到内存里。

「已失效」状态，表示的是这个 Cache Block 里的数据已经失效了，不可以读取该状态的数据。

「独占」和「共享」状态都代表 Cache Block 里的数据是干净的，也就是说，这个时候 Cache Block 里的数据和内存里面的数据是一致性的。

独占状态的时候，数据只存储在一个 CPU 核心的 Cache 里，而其他 CPU 核心的 Cache 没有该数据。这个时候，如果要向独占的 Cache 写数据，就可以直接自由地写入，而不需要通知其他 CPU 核心，因为只有你这有这个数据，就不存在缓存一致性的问题了，于是就可以随便操作该数据。

在「独占」状态下的数据，如果有其他核心从内存读取了相同的数据到各自的 Cache ，那么这个时候，独占状态下的数据就会变成共享状态。

「共享」状态代表着相同的数据在多个 CPU 核心的 Cache 里都有，所以当我们要更新 Cache 里面的数据的时候，不能直接修改，而是要先向所有的其他 CPU 核心广播一个请求，要求先把其他核心的 Cache 中对应的 Cache Line 标记为「无效」状态，然后再更新当前 Cache 里面的数据。

状态转换图：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98%E4%B8%80%E8%87%B4%E6%80%A7/MESI%E5%8D%8F%E8%AE%AE.png)

MESI 协议的四种状态之间的流转过程，每个状态转换的原因：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98%E4%B8%80%E8%87%B4%E6%80%A7/%20MESI%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2%E8%A1%A8%E6%A0%BC.png)

## CPU 是如何执行任务的

### CPU 读写数据的伪共享问题

#### 分析伪共享的问题

伪共享问题是伴随着 MESI 协议产生的，因为多个线程同时读写同一个 Cache Line 的不同变量时，而导致 CPU Cache 失效的现象称为**伪共享（\*False Sharing\*）**。

栗子：

①. 最开始变量 A 和 B 都还不在 Cache 里面，假设 1 号核心绑定了线程 A，2 号核心绑定了线程 B，线程 A 只会读写变量 A，线程 B 只会读写变量 B。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/%E5%88%86%E6%9E%90%E4%BC%AA%E5%85%B1%E4%BA%AB1.png)

②. 1 号核心读取变量 A，由于 CPU 从内存读取数据到 Cache 的单位是 Cache Line，也正好变量 A 和 变量 B 的数据归属于同一个 Cache Line，所以 A 和 B 的数据都会被加载到 Cache，并将此 Cache Line 标记为「独占」状态。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/%E5%88%86%E6%9E%90%E4%BC%AA%E5%85%B1%E4%BA%AB2.png)

③. 接着，2 号核心开始从内存里读取变量 B，同样的也是读取 Cache Line 大小的数据到 Cache 中，此 Cache Line 中的数据也包含了变量 A 和 变量 B，此时 1 号和 2 号核心的 Cache Line 状态变为「共享」状态。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/%E5%88%86%E6%9E%90%E4%BC%AA%E5%85%B1%E4%BA%AB3.png)

④. 1 号核心需要修改变量 A，发现此 Cache Line 的状态是「共享」状态，所以先需要通过总线发送消息给 2 号核心，通知 2 号核心把 Cache 中对应的 Cache Line 标记为「已失效」状态，然后 1 号核心对应的 Cache Line 状态变成「已修改」状态，并且修改变量 A。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/%E5%88%86%E6%9E%90%E4%BC%AA%E5%85%B1%E4%BA%AB4.png)

⑤. 之后，2 号核心需要修改变量 B，此时 2 号核心的 Cache 中对应的 Cache Line 是已失效状态，另外由于 1 号核心的 Cache 也有此相同的数据，且状态为「已修改」状态，所以要先把 1 号核心的 Cache 对应的 Cache Line 写回到内存，然后 2 号核心再从内存读取 Cache Line 大小的数据到 Cache 中，最后把变量 B 修改到 2 号核心的 Cache 中，并将状态标记为「已修改」状态。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/%E5%88%86%E6%9E%90%E4%BC%AA%E5%85%B1%E4%BA%AB5.png)

所以，可以发现如果 1 号和 2 号 CPU 核心这样持续交替的分别修改变量 A 和 B，就会重复 ④ 和 ⑤ 这两个步骤，Cache 并没有起到缓存的效果，虽然变量 A 和 B 之间其实并没有任何的关系，但是因为同时归属于一个 Cache Line ，这个 Cache Line 中的任意数据被修改后，都会相互影响，从而出现 ④ 和 ⑤ 这两个步骤。

#### 避免伪共享的方法

思路：对于多个线程共享的热点数据，即经常会修改的数据，应该避免这些数据刚好在同一个 Cache Line 中，否则就会出现为伪共享的问题。

> Linux 内核宏定义

在 Linux 内核中存在 `__cacheline_aligned_in_smp` 宏定义，是用于解决伪共享的问题。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/__cacheline_aligned.png)

从上面的宏定义，我们可以看到：

- 如果在多核（MP）系统里，该宏定义是 `__cacheline_aligned`，也就是 Cache Line 的大小；
- 而如果在单核系统里，该宏定义是空的；

因此，针对在同一个 Cache Line 中的共享的数据，如果在多核之间竞争比较严重，为了防止伪共享现象的发生，可以采用上面的宏定义使得变量在 Cache Line 里是对齐的。

举个例子，有下面这个结构体：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/struct_test.png)

结构体里的两个成员变量 a 和 b 在物理内存地址上是连续的，于是它们可能会位于同一个 Cache Line 中，如下图：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/struct_ab.png)

所以，为了防止前面提到的 Cache 伪共享问题，我们可以使用上面介绍的宏定义，将 b 的地址设置为 Cache Line 对齐地址，如下：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/struct_test1.png)

这样 a 和 b 变量就不会在同一个 Cache Line 中了，如下图：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/struct_ab1.png)

所以，避免 Cache 伪共享实际上是用空间换时间的思想，浪费一部分 Cache 空间，从而换来性能的提升。

### CPU 如何选择线程的？

在 Linux 内核中，进程和线程都是用 `task_struct` 结构体表示的，区别在于线程的 task_struct 结构体里部分资源是共享了进程已创建的资源，比如内存地址空间、代码段、文件描述符等，所以 Linux 中的线程也被称为轻量级进程，因为线程的 task_struct 相比进程的 task_struct 承载的 资源比较少，因此以「轻」得名。

一般来说，没有创建线程的进程，是只有单个执行流，它被称为是主线程。如果想让进程处理更多的事情，可以创建多个线程分别去处理，但不管怎么样，它们对应到内核里都是 `task_struct`。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/%E4%BB%BB%E5%8A%A1.png)

所以，Linux 内核里的调度器，调度的对象就是 `task_struct`，接下来我们就把这个数据结构统称为**任务**。

在 Linux 系统中，根据任务的优先级以及响应要求，主要分为两种，其中优先级的数值越小，优先级越高：

- 实时任务，对系统的响应时间要求很高，也就是要尽可能快的执行实时任务，优先级在 `0~99` 范围内的就算实时任务；
- 普通任务，响应时间没有很高的要求，优先级在 `100~139` 范围内都是普通任务级别；

#### 调度类

由于任务有优先级之分，Linux 系统为了保障高优先级的任务能够尽可能早的被执行，于是分为了这几种调度类，如下图：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/%E8%B0%83%E5%BA%A6%E7%B1%BB.png)

Deadline 和 Realtime 这两个调度类，都是应用于实时任务的，这两个调度类的调度策略合起来共有这三种，它们的作用如下：

- _SCHED_DEADLINE_：是按照 deadline 进行调度的，距离当前时间点最近的 deadline 的任务会被优先调度；
- _SCHED_FIFO_：对于相同优先级的任务，按先来先服务的原则，但是优先级更高的任务，可以抢占低优先级的任务，也就是优先级高的可以「插队」；
- _SCHED_RR_：对于相同优先级的任务，轮流着运行，每个任务都有一定的时间片，当用完时间片的任务会被放到队列尾部，以保证相同优先级任务的公平性，但是高优先级的任务依然可以抢占低优先级的任务；

而 Fair 调度类是应用于普通任务，都是由 CFS 调度器管理的，分为两种调度策略：

- _SCHED_NORMAL_：普通任务使用的调度策略；
- _SCHED_BATCH_：后台任务的调度策略，不和终端进行交互，因此在不影响其他需要交互的任务，可以适当降低它的优先级。

#### 完全公平调度

对于普通任务来说，公平性最重要，在 Linux 里面，实现了一个基于 CFS 的调度算法，也就是**完全公平调度（\*Completely Fair Scheduling\*）**。

这个算法的理念是想让分配给每个任务的 CPU 时间是一样，于是它为每个任务安排一个虚拟运行时间 vruntime，如果一个任务在运行，其运行的越久，该任务的 vruntime 自然就会越大，而没有被运行的任务，vruntime 是不会变化的。

那么，**在 CFS 算法调度的时候，会优先选择 vruntime 少的任务**，以保证每个任务的公平性。

虽然是普通任务，但是普通任务之间还是有优先级区分的，所以在计算虚拟运行时间 vruntime 还要考虑普通任务的**权重值**，注意权重值并不是优先级的值，内核中会有一个 nice 级别与权重值的转换表，nice 级别越低的权重值就越大。 于是就有了以下这个公式：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/vruntime.png)

NICE_0_LOAD 是一个常量，那么在「同样的实际运行时间」里，高权重任务的 vruntime 比低权重任务的 vruntime **少**。于是高权重值的任务下次就会被优先调度。

#### CPU 运行队列

一个系统通常都会运行着很多任务，多任务的数量基本都是远超 CPU 核心数量，因此这时候就需要**排队**。

事实上，每个 CPU 都有自己的**运行队列（\*Run Queue, rq\*）**，用于描述在此 CPU 上所运行的所有进程，其队列包含三个运行队列，Deadline 运行队列 dl_rq、实时任务运行队列 rt_rq 和 CFS 运行队列 cfs_rq，其中 cfs_rq 是用红黑树来描述的，按 vruntime 大小来排序的，最左侧的叶子节点，就是下次会被调度的任务。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/CPU%E9%98%9F%E5%88%97.png)

这几种调度类是有优先级的，优先级如下：Deadline > Realtime > Fair，这意味着 Linux 选择下一个任务执行的时候，会按照此优先级顺序进行选择，也就是说先从 `dl_rq` 里选择任务，然后从 `rt_rq` 里选择任务，最后从 `cfs_rq` 里选择任务。因此，**实时任务总是会比普通任务优先被执行**。

#### 调整优先级

如果我们启动任务的时候，没有特意去指定优先级的话，默认情况下都是普通任务，普通任务的调度类是 Fair，由 CFS 调度器来进行管理。CFS 调度器的目的是实现任务运行的公平性，也就是保障每个任务的运行的时间是差不多的。

如果你想让某个普通任务有更多的执行时间，可以调整任务的 `nice` 值，从而让优先级高一些的任务执行更多时间。nice 的值能设置的范围是 `-20～19`， 值越低，表明优先级越高，因此 -20 是最高优先级，19 则是最低优先级，默认优先级是 0。

是不是觉得 nice 值的范围很诡异？事实上，nice 值并不是表示优先级，而是表示优先级的修正数值，它与优先级（priority）的关系是这样的：priority(new) = priority(old) + nice。内核中，priority 的范围是 0~139，值越低，优先级越高，其中前面的 0~99 范围是提供给实时任务使用的，而 nice 值是映射到 100~139，这个范围是提供给普通任务用的，因此 nice 值调整的是普通任务的优先级。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/%E4%BC%98%E5%85%88%E7%BA%A7.png)

我们可以在启动任务的时候，可以指定 nice 的值，比如将 mysqld 以 -3 优先级：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/nice.png)

如果想修改已经运行中的任务的优先级，则可以使用 `renice` 来调整 nice 值：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/renice.png)

nice 调整的是普通任务的优先级，所以不管怎么缩小 nice 值，任务永远都是普通任务，如果某些任务要求实时性比较高，那么你可以考虑改变任务的优先级以及调度策略，使得它变成实时任务，比如：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/chrt.png)

## 什么是软中断

### 中断是什么？

在计算机中，中断是系统用来响应硬件设备请求的一种机制，操作系统收到硬件的中断请求，会打断正在执行的进程，然后调用内核中的中断处理程序来响应请求。所以**中断请求的响应程序，也就是中断处理程序，要尽可能快的执行完，这样可以减少对正常进程运行调度地影响。**

中断处理程序在响应中断时，可能还会「临时关闭中断」，这意味着，如果当前中断处理程序没有执行完之前，系统中其他的中断请求都无法被响应，也就说中断有可能会丢失，所以中断处理程序要短且快。

### 什么是软中断？

Linux 系统**为了解决中断处理程序执行过长和中断丢失的问题，将中断过程分成了两个阶段，分别是「上半部和下半部分」**。

- **上半部用来快速处理中断**，一般会暂时关闭中断请求，主要负责处理跟硬件紧密相关或者时间敏感的事情。
- **下半部用来延迟处理上半部未完成的工作**，一般以「内核线程」的方式运行。

- **上半部直接处理硬件请求，也就是硬中断**，主要是负责耗时短的工作，特点是快速执行；
- **下半部是由内核触发，也就说软中断**，主要是负责上半部未完成的工作，通常都是耗时比较长的事情，特点是延迟执行；

硬中断（上半部）是会打断 CPU 正在执行的任务，然后立即执行中断处理程序，而软中断（下半部）是以内核线程的方式执行，并且每一个 CPU 都对应一个软中断内核线程，名字通常为「ksoftirqd/CPU 编号」，比如 0 号 CPU 对应的软中断内核线程的名字是 `ksoftirqd/0`

软中断不只是包括硬件设备中断处理程序的下半部，一些内核自定义事件也属于软中断，比如内核调度等、RCU 锁（内核里常用的一种锁）等。

### 系统里有哪些软中断？

在 Linux 系统里，我们可以通过查看 `/proc/softirqs` 的 内容来知晓「软中断」的运行情况，以及 `/proc/interrupts` 的 内容来知晓「硬中断」的运行情况。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E8%BD%AF%E4%B8%AD%E6%96%AD/softirqs.png)

每一个 CPU 都有自己对应的不同类型软中断的**累计运行次数**，有 3 点需要注意下。

第一点，要注意第一列的内容，它是代表着软中断的类型，在我的系统里，软中断包括了 10 个类型，分别对应不同的工作类型，比如 `NET_RX` 表示网络接收中断，`NET_TX` 表示网络发送中断、`TIMER` 表示定时中断、`RCU` 表示 RCU 锁中断、`SCHED` 表示内核调度中断。

第二点，要注意同一种类型的软中断在不同 CPU 的分布情况，正常情况下，同一种中断在不同 CPU 上的累计次数相差不多，比如我的系统里，`NET_RX` 在 CPU0 、CPU1、CPU2、CPU3 上的中断次数基本是同一个数量级，相差不多。

第三点，这些数值是系统运行以来的累计中断次数，数值的大小没什么参考意义，但是系统的**中断次数的变化速率**才是我们要关注的，我们可以使用 `watch -d cat /proc/softirqs` 命令查看中断次数的变化速率。

### 如何定位软中断 CPU 使用率过高的问题？

要想知道当前的系统的软中断情况，我们可以使用 `top` 命令查看，下面是一台服务器上的 top 的数据：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E8%BD%AF%E4%B8%AD%E6%96%AD/top_si.png)

上图中的黄色部分 `si`，就是 CPU 在软中断上的使用率，而且可以发现，每个 CPU 使用率都不高，两个 CPU 的使用率虽然只有 3% 和 4% 左右，但是都是用在软中断上了。

另外，也可以看到 CPU 使用率最高的进程也是软中断 `ksoftirqd`，因此可以认为此时系统的开销主要来源于软中断。

如果要知道是哪种软中断类型导致的，我们可以使用 `watch -d cat /proc/softirqs` 命令查看每个软中断类型的中断次数的变化速率。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E8%BD%AF%E4%B8%AD%E6%96%AD/watch.png)

一般对于网络 I/O 比较高的 Web 服务器，`NET_RX` 网络接收中断的变化速率相比其他中断类型快很多。

如果发现 `NET_RX` 网络接收中断次数的变化速率过快，接下来就可以使用 `sar -n DEV` 查看网卡的网络包接收速率情况，然后分析是哪个网卡有大量的网络包进来。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E8%BD%AF%E4%B8%AD%E6%96%AD/sar_dev.png)

接着，在通过 `tcpdump` 抓包，分析这些包的来源，如果是非法的地址，可以考虑加防火墙，如果是正常流量，则要考虑硬件升级等。

## 为什么 0.1 + 0.2 != 0.3

### 为什么负数要用补码表示？

我们以 `int` 类型的数字作为例子，int 类型是 `32` 位的，其中**最高位是作为「符号标志位」**，正数的符号位是 `0`，负数的符号位是 `1`，**剩余的 31 位则表示二进制数据**。

负数在计算机中是以「补码」表示的，**所谓的补码就是把正数的二进制全部取反再加 1**

如果负数不是使用补码的方式表示，则在做基本对加减法运算的时候，**还需要多一步操作来判断是否为负数，如果为负数，还得把加法反转成减法，或者把减法反转成加法**，这就非常不好了，毕竟加减法运算在计算机里是很常使用的，所以为了性能考虑，应该要尽量简化这个运算过程。

**而用了补码的表示方式，对于负数的加减法操作，实际上是和正数加减法操作一样的**。

### 十进制小数与二进制的转换

十进制转换为二进制流程

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%B5%AE%E7%82%B9/%E5%8D%81%E8%BF%9B%E5%88%B6%E5%B0%8F%E6%95%B0%E8%BD%AC%E4%BA%8C%E8%BF%9B%E5%88%B6.png)

二进制转成十进制

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%B5%AE%E7%82%B9/%E5%B0%8F%E6%95%B0%E8%BD%AC%E4%BA%8C%E8%BF%9B%E5%88%B62.png)

### 计算机是怎么存小数的？

计算机存储小数的采用的是**浮点数**，名字里的「浮点」表示小数点是可以浮动的。

通常将 `1000.101` 这种二进制数，规格化表示成 `1.000101 x 2^3`，其中，最为关键的是 000101 和 3 这两个东西，它就可以包含了这个二进制小数的所有信息：

- `000101` 称为**尾数**，即小数点后面的数字；
- `3` 称为**指数**，指定了小数点在数据中的位置；

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%B5%AE%E7%82%B9/IEEE%E6%A0%87%E5%87%86.png)

这三个重要部分的意义如下：

- _符号位_：表示数字是正数还是负数，为 0 表示正数，为 1 表示负数；
- _指数位_：指定了小数点在数据中的位置，指数可以是负数，也可以是正数，**指数位的长度越长则数值的表达范围就越大**；
- _尾数位_：小数点右侧的数字，也就是小数部分，比如二进制 1.0011 x 2^(-2)，尾数部分就是 0011，而且**尾数的长度决定了这个数的精度**，因此如果要表示精度更高的小数，则就要提高尾数位的长度；

用 `32` 位来表示的浮点数，则称为**单精度浮点数**，也就是我们编程语言中的 `float` 变量，而用 `64` 位来表示的浮点数，称为**双精度浮点数**，也就是 `double` 变量，它们的结构如下：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%B5%AE%E7%82%B9/float.png)

可以看到：

- double 的尾数部分是 52 位，float 的尾数部分是 23 位，由于同时都带有一个固定隐含位（这个后面会说），所以 double 有 53 个二进制有效位，float 有 24 个二进制有效位，所以它们的精度在十进制中分别是 `log10(2^53)` 约等于 `15.95` 和 `log10(2^24)` 约等于 `7.22` 位，因此 double 的有效数字是 `15~16` 位，float 的有效数字是 `7~8` 位，这些有效位是包含整数部分和小数部分；
- double 的指数部分是 11 位，而 float 的指数位是 8 位，意味着 double 相比 float 能表示更大的数值范围；

我们就以 `10.625` 作为例子，看看这个数字在 float 里是如何存储的。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%B5%AE%E7%82%B9/float%E5%AD%98%E5%82%A8.png)

首先，我们计算出 10.625 的二进制小数为 1010.101。

然后**把小数点，移动到第一个有效数字后面**，即将 1010.101 右移 `3` 位成 `1.010101`，右移 3 位就代表 +3，左移 3 位就是 -3。

**float 中的「指数位」就跟这里移动的位数有关系，把移动的位数再加上「偏移量」，float 的话偏移量是 127，相加后就是指数位的值了**，即指数位这 8 位存的是 `10000010`（十进制 130），因此你可以认为「指数位」相当于指明了小数点在数据中的位置。

`1.010101` 这个数的**小数点右侧的数字就是 float 里的「尾数位」**，由于尾数位是 23 位，则后面要补充 0，所以最终尾数位存储的数字是 `01010100000000000000000`。

在算指数的时候，你可能会有疑问为什么要加上偏移量呢？

前面也提到，指数可能是正数，也可能是负数，即指数是有符号的整数，而有符号整数的计算是比无符号整数麻烦的，所以为了减少不必要的麻烦，在实际存储指数的时候，需要把指数转换成**无符号整数**。

float 的指数部分是 8 位，IEEE 标准规定单精度浮点的指数取值范围是 `-126 ~ +127`，于是为了把指数转换成无符号整数，就要加个**偏移量**，比如 float 的指数偏移量是 `127`，这样指数就不会出现负数了。

细心的朋友肯定发现，移动后的小数点左侧的有效位（即 1）消失了，它并没有存储到 float 里。

这是因为 IEEE 标准规定，二进制浮点数的小数点左侧只能有 1 位，并且还只能是 1，**既然这一位永远都是 1，那就可以不用存起来了**。

于是就让 23 位尾数只存储小数部分，然后在计算时会**自动把这个 1 加上，这样就可以节约 1 位的空间，尾数就能多存一位小数，相应的精度就更高了一点**。

那么，对于我们在从 float 的二进制浮点数转换成十进制时，要考虑到这个隐含的 1，转换公式如下：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%B5%AE%E7%82%B9/float%E5%85%AC%E5%BC%8F.png)

### 0.1 + 0.2 == 0.3 ?

并不是所有小数都可以用「完整」的二进制来表示的，比如十进制 0.1 在转换成二进制小数的时候，是一串无限循环的二进制数，计算机是无法表达无限循环的二进制数的，毕竟计算机的资源是有限。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%B5%AE%E7%82%B9/0.1%E6%B5%AE%E7%82%B9%E6%95%B0%E8%BD%AC%E4%BA%8C%E8%BF%9B%E5%88%B6%E5%B0%8F%E6%95%B0.png)

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%B5%AE%E7%82%B9/0.2%E6%B5%AE%E7%82%B9%E6%95%B0%E8%BD%AC%E6%8D%A2.png)
