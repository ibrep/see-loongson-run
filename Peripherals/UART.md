# UART
## 本节缩略词表

缩略词 | 全称 | 译名
-------|------|------
UART | Universal Asynchronous Receiver／Transmitter | 通用异步收发器
RBR | Receiver Buffer Register | 接收器缓冲寄存器
THR | Transmitter Holding Register | 发送器保持寄存器
IER | Interrupt Enable Register | 中断使能寄存器
IIR | Interrupt Identification Register | 中断识别寄存器
FCR | FIFO Control Register | FIFO控制寄存器
LCR | Line Control Register | 线路控制寄存器
MCR | Modem Control Register | Modem控制寄存器
LSR | Line Status Register | 线路状态寄存器
MSR | Modem Status Register | Modem状态寄存器
SCR | Scratch Register | 暂存寄存器
DLL | Divisor Latch (Least Significant Byte) Register | 分频锁存寄存器（最低有效字节）
DLM | Divisor Latch (Most Significant Byte) Register | 分频锁存寄存器（最高有效字节）
DLAB | Divisor Latch Access Bit | 分频因子锁存访问位


## 龙芯 2K1000 的 UART
龙芯2K1000的用户手册中说，2K1000片内集成了12个UART控制器，其中3个是全功能UART。
2K1000的UART在寄存器与功能上兼容NS16550A。

既然手册中提到了NS16550A，我们就了聊聊它的前世今生。话说1980年代，一代神机IBM PC
上提供了用来连接外部设备的RS232串口，是用一颗叫做8250的芯片实现的。那个时代，串
口可以连接很多设备，比如鼠标、Modem、串行打印机、收银机等，还可以通过RS232串口连
接远程终端，串口是当时最重要的接口之一。然而8250的性能比较弱，于是有很多厂家开始
蹭热度，提供兼容8250但是功能更强的串口扩展卡。其中有家公司做的串口扩展芯片最好，
这家公司就是大名鼎鼎的美国国家半导体公司(National Semiconductor)，它提供的扩展芯
片NS16550A大获成功，各类IBM PC兼容机的串口都采用NS16550A实现，不再使用弱鸡8250，
以后的PC串口都在功能和寄存器上兼容NS16550A。

## NS16550A原理与设置
NS16550A的功能框图如下所示。

<img src="images/ns16550-blocks.jpg" width="500px">

NS16550A的主要功能是把CPU提供的并行数据转换成串行数据发送出去，把接收到的串行数
据重新整合成并行数据再交给CPU。为了完成这两个任务，芯片内集成了两个FIFO，也就是
先入先出队列，从CPU传过来的并行数据，先送入发送端FIFO，再串行发送出去；从外面接
收到的串行数据，先暂存在接收端FIFO中，再转换成并行数据交给CPU。

发送接收串行数据时，需要设置波特率以及串行数据的格式（也叫帧格式），比如数据占几
位？有几个停止位？有没有奇偶校验位？如果有，是奇校验还是偶校验等等，因此NS16550A
芯片中还提供了几个设置寄存器，以便用户可以进行相关的设置。下面我们了解一下这些寄
存器。

### 波特率的设置

NS16550A内部集成了一个波特率发生器，通过分频锁存寄存器(Divisor Latch)可以设置波
特率的值。分频锁存寄存器是两个8位寄存器，一个叫做DLM，用于存放分频系数的高8位；
另一个叫做DLL，用于存放分频系数的低8位。

分频系数可以用下面的公式计算：

`divisor = (frequency input) / (baud rate * 16)`

### 线路控制寄存器（LCR）

线路控制寄存器（LCR）用来说明异步通信时数据的传输格式，比如字符的位数、停止位的个数、奇
偶校验设置等。LCR寄存器几个位域的作用：
  * Bit[1:0] 这两个位规定了每一帧数据中字符的位数：
  
  Bit[1:0] | 字符长度
  ---------|---------
  00 | 5 Bits
  01 | 6 Bits
  10 | 7 Bits
  11 | 8 Bits
  
  * Bit[2] 若此位为0，则停止位为1个；若此位为1，则字符位数为5时，有1.5个停止位，
    字符位数为其它值时，有2个停止位。
    
  * Bit[3] 奇偶校验使能位，此位为0，则不采用奇偶校验；此位为1，则采用奇偶校验。
  
  * Bit[4] 奇偶校验选择位，此位为0，则采用奇校验；此位为1，则采用偶校验。
  
  * Bit[7] 这一位是分频锁存访问位（DLAB），此位为1时可以设置或读取波特率分频系数；
    此位为0时，才可以访问接收缓冲寄存器（RBR）、发送保持寄存器（THR）或中断使能
    寄存器（IER）。

### 线路状态寄存器（LSR）

通过这个寄存器可以了解数据传送过程中的状态信息，比如传输过程中是否出错，发送FIFO
是否为空，接收FIFO中是否有数据等

  * Bit[0] 这个位说明接收器中是否有准备好的数据(Data Ready)。此位为1，说明接收器
    缓冲寄存器（RBR）或接收FIFO中收到了一个完整的字符。数据被读出后，此位为0。
    
  * Bit[5] 此位为1，说明发送FIFO为空，也就是数据以及发送出去了；此位为0，说明发
    送FIFO中有数据，也就是数据尚未发送完成。
    
### 数据寄存器（DAT）

NS16550A中，接收缓冲寄存器（RBR）和发送保持寄存器（THR）的地址相同，偏移量均为0。
在龙芯2K1000手册中，把它们统一标注为数据寄存器（DAT）。

发送数据时，只需将字符写入DAT寄存器；接收数据时，只需要读入DAT寄存器。


