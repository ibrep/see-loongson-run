# 时钟设置
本节以龙芯2K1000为例，介绍CPU时钟架构和PMON中的时钟初始化代码。本节中的PMON代码
片段出自 `Targets/LS2K/ls2k/loongson3_clksetting.S`文件，为了提高可读性，对代码
进行了部分修改。

## 本节缩略词表

缩略词 | 全称 | 译名
-------|------|------
DFT | Design For Test | 面向实验的设计
PLL | Phase Locked Loop | 锁相环


## 2K1000 CPU的时钟相关引脚
2K1000 CPU有三个引脚是与系统时钟相关的：

  * SYS_SYSCLK 该引脚用来外接100MHz参考时钟，其引脚号为F6。在龙芯派中，该引
    脚外接了一颗100MHz晶振。
    
  * SYS_CLKSEL[1:0] 这两个引脚用于配置PLL时钟输入，其引脚号为B2和C3。这两个引脚
    的组合决定了时钟模式：00=低频模式,01=高频模式,10=软件模式(DFT),11=bypass 模
    式。在龙芯派中，SYS_CLKSEL1上拉到3V3，SYS_CLKSEL0接地，因此龙芯派的时钟配置
    模式为10=软件模式（DFT）。当 SYS_CLKSEL[1:0] 设置为 2’b10 时表示 PLL 频率通
    过软件配置。这种配置下，默认对应的时钟频率为外部参考时钟频率，即所有 PLL 输
    出都是 SYS_SYSCLK，需要在处理器启动过程中对时钟进行软件配置。
    
## CPU核工作频率设置

CPU核的时钟的产生结构图如下：
  
<img src="images/ls2k-clock-core.jpg" width="500px">

输出时钟频率的按下式计算：

`node_clock=refclk/L1_div_ref*L1_loopc/L2_divout`

其中：

  * `node_clock`是我们设定的CPU核的工作频率，比如`node_clock=1GHz`
  * `refclk = 100Mhz`，这是外部晶振输入的参考时钟频率
  * 可配分频器的输出等于 `refclk/L1_div_ref` 需要被限定在20~40MHz范围内，因此
  `L1_dev_ref`的取值范围在2.5~5之间，比如，可以取`L1_dev_ref=4` 
  * PLL 倍频值 `refclk/L1_div_ref*L1_loopc` 需要在 1.2GHz~3.2GHz范围内，因此
    `L1_loopc=(1.2G~3.2G)/refclk*L1_dev_ref=(12~32)*L1_dev_ref`，若
    `L1_dev_ref=4` ，则`L1_loopc=48~128`，比如可以取`L1_loopc=80`，此时PLL倍频值
    为2GHz，取`L2_divout=2`，可以获得`node_clock=1GHz`
    
综上所述，当CPU核工作频率设定为1GHz时，各参数的取值分别为：
  * `refclk=100MHz`
  * `L1_dev_ref=4`
  * `L1_loopc=80`
  * `L2_divout=2`
  * `node_clock=100Mhz/4*80/2=1000Mhz=1GHz`
  
如果要把CPU核工作频率设定为1.2GHz，各参数可取值为：
  * `refclk=100MHz`
  * `L1_dev_ref=4`
  * `L1_loopc=96`
  * `L2_divout=2`
  * `node_clock=100Mhz/4*96/2=1200Mhz=1.2GHz`
  
通过编程设定时钟频率时，需要注意以下几点：
  
  * 图中FREQ_SCALE可以做进一步的设置，其启动时的默认值为 1，PMON中未改设置
  * NODE PLL高64位配置寄存器，地址0x1fe10488，其[5:0]=L2_divout（手册中标注为
    L2_div_out_node）
  * NODE PLL 低 64 位配置寄存器，地址0x1fe10480，其[41:32]=L1_loopc（手册中标注
    为L1_div_loopc），[31:26]=L1_div_ref
    
## PMON中设置核心频率的代码



