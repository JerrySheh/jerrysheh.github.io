---
title: ARM笔记（二）中断
date: 2017-10-01 10:26:57
tags: ARM
---

今天是国庆节耶，早上起来整个朋友圈都是塞车。

没有选择回家。在宿舍继续挖坑。

笔记（二）涉及到的知识：
1. 向量中断控制器（VIC）
2. 三种中断类型（FIQ、非向量IRQ、向量IRQ）
3. 中断相关寄存器
4. 外部中断相关寄存器
5. 几个实例

---

<!-- more -->

# 向量中断控制器（VIC）

ARM 内核本身只有<font color=#FF0000 > 快速中断（FIQ）</font>和<font color=#FF0000 > 普通中断（IRQ）</font>两条中断输入信号线，只能接收两个中断。但是我们有很多个中断事件要处理怎么办？<font color=#FF0000 >  向量中断控制器（Vectored Interrupt Controller, VIC）</font> 介于内核和外设中断之间，就是用来解决这个问题的。

向量中断控制器（VIC）的作用就是允许哪些中断源可以产生中断、可以产生哪类中断、产生中断后执行哪段服务程序。


![interrupt](../../../../images/LPC2000_interrupt.jpg)



---

## 1. FIQ中断

## 2. 向量IRQ中断

VIC最多支持16个向量IRQ中断，这些中断被分为16个优先级，并且为每个优先级指定一个服务程序入口地址。在发生向量IRQ中断后，相应优先级的服务程序入口地址被装入向量地址寄存器VICVectAddr中，通过一条ARM指令即可跳转到相应的服务程序入口处，所以向量IRQ中断具有较快的中断响应。

## 3. 非向量IRQ中断

任何中断源都可以设置为非向量IRQ中断。它与向量IRQ中断的区别在于前者不能为每个非向量IRQ中断源设置服务程序地址，而是所有的非向量IRQ中断都共用一个相同的服务程序入口地址。
当有多个中断源被设置为非向量IRQ中断时，需要在用户程序中识别中断源，并分别作出处理。所以非向量IRQ中断响应延时相对较长。


---
# 中断源和中断相关寄存器

VIC可以管理32路中断请求，但在 LPC2000 系列中，并没有用完32个中断请求通道。没有使用的作为保留。

以下是中断源表

![interrupt](../../../../images/中断源.png)




## 中断相关寄存器

### 1. 中断使能寄存器(VICIntEnable)

功能：向某位写入1时，允许对应的中断源产生中断。

例子：中断源第17位写1，使能外部中断3

```c
VICIntEnable = 1<<17  
```


### 2. 中断使能清零寄存器(VICIntEnClr)

功能：向某位写入1时，禁止对应的中断源产生中断。


### 3. 中断选择寄存器(VICIntSelect)

功能：向某位写入1时，对应中断源产生的中断为FIQ中断，否则为IRQ中断。

例： 所有中断源设置为IRQ中断
```c
VICIntSelect = 0x00
```

例2：外部中断2为FIQ中断，其余为IRQ中断
```c
VICIntSelect = 0x00010000

// 0x00010000 = 0b 0000 0000 0000 0001 0000 0000 0000 0000
```

---

## 向量IRQ中断相关寄存器

### 1. 向量控制寄存器(VICVectCntl0～15)

功能：

VICVectCntlx[4:0]：分配给此优先级向量IRQ中断的中断源序号；

VICVectCntlx[5]：该位为1，使能当前优先级的向量IRQ中断。否则为禁止。

### 2. 向量地址寄存器(VICVectAddr0～15)

功能：存放对应优先级向量IRQ中断服务程序的入口地址。

---

## 非向量IRQ中断相关寄存器

### 1. 向量地址寄存器(VICDefVectAddr)

功能：存放非向量中断服务程序的入口地址，当发生非向量中断时该寄存器中保存的地址存入VICVectAddr寄存器。

---

## 中断状态寄存器

### 1. 所有中断状态寄存器(VICRawIntr)

功能：32位，当某位为1时表示对应位的中断源产生中断请求。

### 2. FIQ状态寄存器(VICFIQStatus)

功能：32位，当某位为1时表示对应位的中断源产生FIQ中断请求。

### 3. IRQ状态寄存器(VICIRQStatus)

功能：32位，当某位为1时表示对应位的中断源产生IRQ中断请求。


---

## 软件中断寄存器(VICSoftInt)

### 1. 软件中断寄存器(VICSoftInt)

功能：32位，当某位为1时，将产生与该位相对应的中断请求。


### 2. 软件中断清零寄存器(VICSoftIntClear)

功能：32位，当某位为1时，将清零VICSoftInt寄存器中对应位

</br>

---

# 外部中断

外部中断是通过引脚输入符合要求的信号而触发的中断。LPC2114/2124/2212/2214含有4个外部中断输入（作为可选引脚功能，通过PINSEL0/1寄存器设置相应管脚为外部中断功能）。外部中断输入可用于将处理器从掉电模式唤醒。

## 外部中断相关寄存器

![interrupt](../../../../images/外部中断相关寄存器.png)

### 1. 外部中断极性控制寄存器(EXTPOLAR)

功能：控制外部中断输入信号的极性，其中低四位（EXTPOLAR[3:0]）分别对应外部中断3～0。

![interrupt](../../../../images/EXTPOLAR.png)

当EXTPOLARx设置为 1 时EINTx引脚输入信号高电平或上升沿有效。

当EXTPOLARx设置为 0 时EINTx引脚输入信号低电平或下降沿有效。


```c
EXTPOLAR2 = 1  // 中断2（EINT2）高电平或上升沿有效
```

### 2. 外部中断方式控制寄存器(EXTMODE)

功能：控制外部中断输入信号的有效触发方式，其中低四位（EXTMODE [3:0]）分别对应外部中断3～0。

当EXTMODEx设置为0时输入信号为电平触发有效。

当EXTMODEx设置为1时输入信号为边沿触发有效。

```c
EXTMODE1 = 0  // 中断1（EINT1）设置成电平触发有效
```

EXTPOLARx | EXTMODEx | 触发信号
---|---
0| 0 | 低电平触发
0| 1 | 下降沿触发
1| 0 | 高电平触发
1| 1 | 上升沿触发

### 3. 外部中断唤醒寄存器(EXTWAKE)

功能：设置该寄存器允许相应的外部中断将处理器从掉电模式唤醒。实现掉电唤醒不需要在向量中断控制器（VIC）中使能相应的中断。该寄存器的低四位（EXTWAKE[3:0]）分别对应外部中断3～0。

当EXTWAKEx设置为1时对应的外部中断将处理器从掉电模式唤醒。

```c
EXTMODE1 = 1  // 中断1（EINT1）将处理器从掉电模式唤醒
```


### 4. 外部中断标志寄存器(EXTINT)

功能：若引脚上出现了符合要求的信号，EXTINT寄存器中对应的中断标志将被置位。向该寄存器的EINT0～EINT3位写入1，可将其清零。

注意：在电平触发方式下，清除中断标志只有在引脚处于无效状态时才可实现。比如设置为低电平中断，则只有在中断引脚恢复为高电平后才能清除中断标志。


## 一个外部中断应用示例


## 需求

设置EINT0为低电平触发中断

## 步骤

1. 设置引脚连接模块，将P0.16设置为外部中断功能；
2. 设置中断方式寄存器，将外部中断0设置为电平触发；
3. 设置中断极性寄存器，将外部中断0设置为低电平触发；

```c
PINSEL1   = (PINSEL1 & 0xFFFFFFFC) | 0x01;
EXTMODE  &= 0x0E;
EXTPOLAR &= 0x0E;

```

---

# 一个IRQ中断的设计实例


## 需求

设置外部中断0产生向量IRQ中断后执行中断服务程序“IRQ_Eint0( )”

## 步骤

1. 设置引脚连接模块，将P0.16设置为外部中断功能；
2. 设置所有中断为IRQ中断；
3. 将外部中断0（在中断源列表中序号14）设置到优先级0中，并使能IRQ中断；
4. 将外部中断0的中断服务程序写入对应优先级的地址寄存器中；
5. 清除外部中断0的标志后使能外部中断0


```c
PINSEL1 = (PINSEL1&0xFFFFFFFC)|0x01;    // p0.16正好是 PINSEL1 的第0和1位，写入01时为中断功能
VICIntSelect = 0x00000000;              //写1表示FIQ，否则为IRQ      
VICVectCntl0 = (0x20 | 14);             // 0x20表示第[5]位写1，使能。 中断源序号14设置为优先级0
VICVectAddr0 = (int)IRQ_Eint0;          // 优先级为0的IRQ中断服务程序的入口地址
EXTINT = 0x01;                          //清除外部中断0的标志
VICIntEnable = (1 << 14);               //使能外部中断0

```

在退出中断服务程序时要清零相应外设的中断标志，以及VICVectAddr寄存器，为响应下次中断作好准备。

```c
EXTINT = 0x01;
VICVectAddr = 0;
```

---

# 一个蜂鸣器例子

## 使用向量IRQ方法

```c
void   __irq IRQ_Eint3(void){
    dir= (dir==0)?1:0;
    if(dir==0) {  IO0CLR = BEEPCON;  }
    else       {  IO0SET = BEEPCON;  }
     EXTINT = 1<<3;
     VICVectAddr = 0;
}

void main(void){  
    IRQEnable();                           // 开IRQ中断
    PINSEL1 = 3<<8;	                       // P0.20设置为EINT3
    IO0DIR = BEEPCON;	                     // 设置B1控制口为输出
    VICIntSelect = 0x00000000;	           // 所有中断分配为IRQ中断
    VICVectCntl0 =0x20|17;                 // EINT3为向量IRQ中断
    VICVectAddr0 = (int)IRQ_Eint3;         // 中断服务程序地址
    EXTMODE = 0x00;	                       // EINT3中断为电平触发模式
    EXTINT = 1<<3;		                     // 清除EINT3中断标志
    VICIntEnable = 1<<17;	                 // 使能EINT3中断
    while(1);		                           // 等待中断
}

```

## 使用非向量IRQ方法

```c

void   __irq IRQ_Eint3(void){
    dir= (dir==0)?1:0;
    if(dir==0) {  IO0CLR = BEEPCON;  }
    else       {  IO0SET = BEEPCON;  }
     EXTINT = 1<<3;
     VICVectAddr = 0;
}

void  main(void)
{  
    IRQEnable();                           // 开IRQ中断
    PINSEL1 = 3<<8;	                       // P0.20设置为EINT3
    IO0DIR = BEEPCON;	                     // 设置B1控制口为输出
    VICIntSelect = 0x00000000;	           // 所有中断分配为IRQ中断
    VICDefVectAddr = (int)IRQ_Eint3;       // 中断服务程序地址
    EXTMODE = 0x00;	                       // EINT3中断为电平触发模式
    EXTINT = 1<<3;		                     // 清除EINT3中断标志
    VICIntEnable = 1<<17;	                 // 使能EINT3中断
    while(1);		                           // 等待中断
}
```


## 使用FIQ方法

```c
void  FIQ_Exception (void){
    dir= (dir==0)?1:0;
    if(dir==0) {  IO0CLR = BEEPCON;  }
    else       {  IO0SET = BEEPCON;  }
    EXTINT = 1<<3;
    VICVectAddr = 0;
}


int  main(void)
{  
    FIQEnable();                    // 开FIQ中断
    PINSEL1 = 3<<8;	                // P0.20设置为EINT3
    IO0DIR = BEEPCON；              // 设置B1控制口为输出
    VICIntSelect = 1<<17;	          // 第17位写1，选择中断源17为FIQ中断
    EXTMODE = 0x00;                 // EINT3中断为电平触发模式
    EXTINT = 1<<3;                  // 清除EINT3中断标志
    VICIntEnable = 1<<17;           // 使能EINT3中断
    while(1);                       // 等待中断
    return(0);
}


```
