---
title: ARM笔记（三）UART
categories: ARM基础
tags: ARM
abbrlink: cb46c933
date: 2017-10-08 10:54:31
---

什么是UART ？

通用异步收发传输器（Universal Asynchronous Receiver/Transmitter，通常称作UART，读音/ˈjuːart/）是一种异步收发传输器，是电脑硬件的一部分，将资料由串行通信与并行通信间作传输转换。UART通常用在与其他通讯界面（如EIA RS-232）的连结上。 ————摘自维基百科

我的理解是，UART是一种串口通讯方式，简单地说就是让 ARM 芯片跟电脑进行通信。电脑输入命令可以指挥芯片的操作。

笔记（三）涉及到的知识：
1. UART引脚
2. UART相关寄存器
3. 使用示例


<!-- more -->

---
# UART引脚

引脚号 | 引脚名称 | 类型 | 描述
---|---
p0.0|RxD0|输入|串行输入，接收数据
p0.1|TxD0|输出|串行输出，发送数据
p0.8|RxD1|输入|串行输入，接收数据
p0.9|TxD1|输出|串行输出，发送数据

设置方法：PINSEL0第  0:3 位 设为 0101

```
PINSEL0 = 0x00000005;
```

---
# UART0相关寄存器

## 1. U0RBR 接收缓存寄存器

功能：8位只读，包含了接收FIFO中最早接收到的字节

接收的数据不足8位时，高位用0填充。

---

## 2. U0THR 发送器保持寄存器

功能：8位只写，写入该寄存器的值保存到发送FIFO中，当该字节到达FIFO底部时，它将被送入发送移位寄存器（U0TSR）进行发送。

写入该寄存器的值将是发送FIFO中的最高字节。访问该寄存器时，U0LCR的除数锁存访问位（DLAB）必须为0。

---

## 3. U0DLL / U0DLM 除数锁存寄存器

U0DLL[7:0] 低字节
U0DLM[7:0] 高字节

功能：两个寄存器一起构成一个16位的除数，决定UART0的波特率。

波特率计算公式：U0DLM、U0DLL = FPCLK/(16×baud)；

访问它们时，U0LCR的除数锁存访问位（DLAB）必须为1。

---

## 4. U0IER 中断使能寄存器

功能：8位，每一位的功能如下图

![LARM](../../../../images/UART0IER.png)

---

## 5. U0FCR FIFO控制寄存器

功能：8位，如下图

位|[7:6]|[5:3]|2|1|0
---|---
功能|Rx触发点设置|保留|复位TxFIFO|复位RxFIFO|使能FIFO


使能FIFO：

值|功能
---|---
1|使能UART0的接收和发送FIFO，并且允许访问U0FCR[7:1]。
0|禁止接收FIFO，此时接收缓存只有1个字节。而发送FIFO不会被关闭。


Rx触发点设置：通过设置这两位可以调整接收FIFO中触发RDA中断的有效字节数量。

值|触发点
---|---
00|触发点0（1字节）
01|触发点1（4字节）
10|触发点2（8字节）
11|触发点3（14字节）

---

## 6. U0LCR 线状态控制寄存器

8位，每位的功能如下图

位|7|6|[5:4]|3|2|[1:0]
---|---
功能|除数锁存|间隔|奇偶选择|奇偶设置|停止位|字长

字长：

值|功能
---|---
00|5位字符长度
01|6位字符长度
10|7位字符长度
11|8位字符长度

停止位：

值|功能
---|---
0|1个停止位
1|2个停止位

奇偶使能

值|功能
---|---
0|禁止奇偶产生和校验
1|使能奇偶产生和校验

奇偶选择

值|功能
---|---
00|奇数(数据位+校验位＝奇数)
01|偶数(数据位+校验位＝偶数)
10|校验位强制为1
11|校验位强制为0  

除数锁存

值|功能
---|---
0|禁止访问除数锁存寄存器（说人话：不能设置波特率）
1|使能访问除数锁存寄存器（说人话：可以设置波特率）

---

## 7. U0LSR 线状态寄存器

位|7|6|5|4|3|2|1|0
---|---
功能|EXFE|TEMT|THRE|BI|FE|PE|OE|RDR

---

# UART0 应用示例


## 流程

1. 设置引脚连接模块将对应IO连接到UART0
2. 设置串口波特率
3. 设置串口工作模式
4. 发送或接收数据
5. 检查串口状态字或等待串口中断


## 代码

### 初始化

```c
#define  UART_BPS  115200  //定义波特率

Void UART0_Ini(void)
{
    uint16 Fdiv;
    PINSEL0 = 0x00000005;           //设置引脚连接到UART0
    U0LCR = 0x83;  //0b 1000 0011   //使能除数锁存，8位字符长度
    Fdiv = (Fpclk / 16) / UART_BPS; //根据波特率计算分频值
    U0DLM = Fdiv / 256;             //设置除数寄存器
    U0LLM = Fdiv % 256;             //设置除数寄存器
    U0LCR = 0x03;                   //禁止除数锁存，并设置工作模式
}
```

### 查询方式发送一字节数据

```c
void UART0_SendByte(uint8 data)
{
    U0THR = data;                //将要发送的一字节数据写入U0THR
    while((U0LSR & 0x40) == 0);  //等待数据发送完毕
}

```

### 查询方式接收一字节数据

```c
uint8 UART0_RcvByte(void)
{
    uint8 rcv_data;            
    while((U0LSR & 0x01) == 0);//等待数据到达
    rcv_data = U0RBR;          //从U0RBR中读出接收的数据
    return(rcv_data);          //返回接收的数据
}
```

---

## 两种方式实现一个完整的UART0通信例子


### 使用查询方式

```c
PINSEL0 = 0x00000005;

void  UART0_Init(void)  {
     uint16 Fdiv;
     U0LCR = 0x83;                    // DLAB = 1 允许设置波特率
     Fdiv = (Fpclk / 16) / UART_BPS;  //设置波特率
     U0DLM = Fdiv / 256;
     U0DLL = Fdiv % 256;
     U0LCR = 0x03;                    // DLAB = 0 不允许设置波特率
}		


uint8  UART0_GetByte(void) {
    uint8 rcv_dat;
    while((U0LSR & 0x01)==0) ;
     //等待U0LSR的接收数据就绪位置位
     rcv_dat = U0RBR;   //读取数据
     return (rcv_dat);
}


void  UART0_GetByte(uint8 dat) {
     U0THR = dat;                   //写入数据
    while((U0LSR & 0x40)==0);       //等待U0LSR的发送器空位置位，即发送数据完毕
}


```

### 使用中断方式

```c
uint8  rcv_buf[8];          // UART0 接收数据缓冲区
volatile uint8  rcv_new;    // 接收新数据标志

//UART0接收中断服务程序
void   __irq IRQ_UART0(void)  {  
    uint8  i;     
    if( (U0IIR&0x0F) == 0x04 )
         rcv_new = 1;   //设置接收到新的数据标志
    for(i=0; i<8; i++)    {
        rcv_buf[i] = U0RBR;   //读取FIFO数据，并清除中断
    }
    VICVectAddr = 0x00;     //中断处理结束
}

//向串口发送一个字节数据
void  UART0_SendByte(uint8 dat) {
     U0THR = dat;
    while (U0LSR & 0x20 == 0);
}

//向串口发送缓冲区8个字节数据
void UART0_SendBuf(void) {
    uint8 i;
     for(i=0; i<8; i++) UART0_SendByte(rcv_buf[i]);
}


int main(void)
{  
      uint16 Fdiv;
      PINSEL0 = 0x00000005;
      U0LCR=0x83;          //8位数据位 1位停止位，无奇偶校验
      Fdiv=(Fpclk/16)/9600;
      U0DLM=Fdiv /256;
      U0DLL=Fdiv %256;
      U0LCR=0x03;
      U0FCR = 0x81;        //使能FIFO，设置触发点为8字节
      U0IER = 0x01;       

      IRQEnable();
      VICIntSelect = 0x00000000;
      VICVectCntl0 = 0x20 | 0x06;       //0x20向量IRQ，0x06中断源第六位UART0
      VICVectAddr0 = (uint32)IRQ_UART0; //入口地址
      VICIntEnabel = 1<<0x06;
      while(1)
      {
          if(rcv_new ==1)
           {
              rcv_new = 0;
              UART0_SendBuf();
           }
      }

      return(0);
}
```
