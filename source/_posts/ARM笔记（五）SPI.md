---
title: ARM笔记（五）SPI
categories: ARM基础
tags: ARM
abbrlink: b10da3ff
date: 2017-10-23 12:02:44
---

最近有点累，更新都放缓了。

SPI是一种全双工的同步串行接口，一个SPI总线可以连接多个主机和多个从机。在同一时刻只允许一个主机操作总线，并且同时只能和一个从机通信。串行时钟由主机产生，当主机发送一字节数据（通过MOSI）的同时，从机返回一字节数据（通过MISO）。

笔记（五）涉及到的知识：
1. SPI引脚
2. SPI寄存器
3. 简单例子

<!-- more -->

# 一. 引脚

![SPI](../../../../images/SPI.png)

使用示例：
```c
PINSEL0 = 0x5500;  // 0b 0101 0101 0000 0000  //SCK0 MISO0
```

---

# 二. 相关寄存器

![SPII](../../../../images/SPII.png)


## 1. SPI控制寄存器 SPCR

位|7|6|5|4|3|2:0
---|---
功能|SPIE|LSBF|MSTR|CPOL|CPHA|保留

### CPHA:时钟相位控制
   该位决定SPI传输时数据和时钟的关系，并控制从机传输的起始和结束。当该位为：
1:时钟前沿数据输出，后沿数据采样；
0:时钟前沿数据采样，后沿数据输出；


### CPOL:时钟极性控制
1:SCK为低有效；
0:SCK为高有效；

### CPOL:主模式控制
1:SPI处于主模式；
0:SPI处于从模式；

### LSBF:字节移动方向控制。

1:每字节数据从低位(LSB)开始传输；
0:每字节数据从高位(MSB)开始传输；

### SPIE:SPI中断使能。

1:每次SPIF或MODF置位时都会产生硬件中断；
0:SPI中断被禁止；

---

## 2. SPI状态寄存器 SPSR

![SPISTAT](../../../../images/SPISTAT.png)


---

## 3. SPI数据寄存器 SPDR

8位，SPDR寄存器为SPI提供数据的发送和接收。

处于主模式时，向该寄存器写入数据，将启动SPI数据传输。从数据传输开始到SPIF状态位置位并且没有读取状态寄存器的这段时间内不能对该寄存器执行写操作。

---

## 4. SPI时钟计数寄存器 SPCCR

作为主机时，SPCCR寄存器控制SCK的频率。寄存器的值为一位SCK时钟所占用的PCLK周期数。该寄存器的值必须为偶数，并且必须不小于8。如果寄存器的值不符合以上条件，可能会导致产生不可预测的动作。
SPI速率 = Fpclk / SPCCR

---

## 5. SPI中断寄存器 SPINT

只有第0位需要设置，写入1清零。第1-7位无需设置。

注：当SPIE位置一，并且SPIF和WCOL位种至少有一位为1时，该位置位。但是只有当SPI中断位置位并且SPI中断在VIC种被使能，SPI中断才能有中断处理软件处理。

---

# 三. 一个简单的例子

## 作为主机

```c
#define  MSTR	(1 << 5)
#define  CPOL	(1 << 4)
#define  CPHA  (1 << 3)
#define  LSBF  (1 << 6)
#define SPI_MODE (MSTR | CPOL)

void MSpiIni(uint8 fdiv)
{
    if(fdiv < 8)          //过滤分频值，如果小于8为非法
        fdiv = 8;
    S0PCCR = fdiv & 0xFE; //设置SPI时钟速率
    S0PCR  = SPI_MODE;    //设置为SPI主机

}

uint8 MSendData(uint8 data)
{
    IO0CLR = HC595_CS;   //选择从机
    S0PDR = data;        //发送一字节数据,启动SPI数据传输
    while(0 == (S0PSR & 0x80)); //等待数据发送结束

    IO0SET = HC595_CS;
    return(S0PDR);  //读出从机发送的数据或释放从机
}


```

## 作为从机

```c
void SSpiIni(uint8 fdiv)
{
    S0PCR  = (1 << 4);  //设置为SPI从机

}

//收发一字节数据
uint8 SSwapData(uint8 data)
{
    S0PDR = data;               //将要发送的数据放入SPDR
    while(0 == (S0PSR & 0x80)); //等待数据发送结束
    return(S0PDR);              //从SPDR中读出接收的数据
}

```
