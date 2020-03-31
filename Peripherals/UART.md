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
片就是NS16550A大获成功，各类IBM PC兼容机的串口都采用NS16550A实现，不再使用弱鸡
8250，以后的PC串口都在功能和寄存器上兼容NS16550A。

## NS16550A原理与设置
NS16550A的功能框图如下所示。

<img src="images/ns16550-blocks.jpg" width="500px">
