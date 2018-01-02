---
title: ARM笔记（六）定时器
categories: ARM基础
tags: ARM
abbrlink: 9713d312
date: 2017-11-21 20:03:27
---

决定换一种笔记方式，直接上代码。然后从代码中去理解。


<!-- more -->

# 寄存器介绍

名称|描述
---|---
TCR|定时器控制寄存器。控制定时器计数器功能（禁止或复位）。
TC|定时器计数器。为32位计数器，计数频率为PCLK经过预分频计数器后频率值。
PR|预分频控制寄存器。用于设定预分频值，为32位寄存器。该寄存器指定了预分频计数器的最大值。
PC|预分频计数器。32位计数器，计数频率为PCLK，当计数值等于预分频计数器的值时，TC计数器加一。
IR|中断标志寄存器。读该寄存器识别中断源，写该寄存器清除中断标志。



名称|描述
---|---
MCR|控制在匹配时是否产生中断或复位TC
MR0|通过MCR寄存器可以设置匹配发生时的动作
MR1|通过MCR寄存器可以设置匹配发生时的动作
MR2|通过MCR寄存器可以设置匹配发生时的动作
MR3|通过MCR寄存器可以设置匹配发生时的动作
EMR|EMR控制外部匹配管脚MATx.0～MATx.3

---

# 一、定时器定时（查询方式）

## 1. 功能需求

1. 使用定时器0实现 2秒 定时，控制蜂鸣器蜂鸣。采用查询方式查询T0IR的相应标志位等待定时时间到达。

2. 在T0MR中设置定时常数，在T0MCR中设置定时器0匹配后复位TC并产生中断标志，接下来程序查询等待中断标志置位。若定时时间到，先清除Timer0中断标志，然后将蜂鸣器控制输出信号取反。


```c
//初始化定时器0，定时时间为2S。
void  Time0Init(void) {
    T0TC = 0;           //定时器计数器设置为0
    T0PR = 0;           // 设置定时器0不分频
    T0MCR = 0x03;  // 设置Timer0匹配后产生中断并复位T0TC
    T0MR0 = Fpclk*2-1;		//2秒定时值
    T0TCR = 0x01;   // 启动Timer0
}

int  main(void)
{  
    PINSEL0 = 0x00000000;	   // 设置管脚连接GPIO   
    IO0DIR = BEEPCON; 	   // 设置I/O为输出
    Time0Init();			   // 初始化定时器0   		
    while(1)   {  
        while( (T0IR&0x01) == 0 );     // 等待定时时间到
        T0IR = 0x01;		  // 清除中断标志
        if( (IO0SET&BEEPCON) == 0 ) {
          IO0SET = BEEPCON;
        }
        else        IO0CLR = BEEPCON;				
   }
   return(0);
}

```


# 二、定时器定时（中断方式）

1. 使用定时器0实现2秒定时，控制蜂鸣器蜂鸣。采用中断方式实现定时控制。

2. 在T0MR中设置定时常数，在T0MCR中设置定时器0匹配后复位TC并产生中断，还需设置向量中断控制器(VIC)，使能并设置Timer0中断，最后等待中断。中断服务程序将蜂鸣器控制输出信号取反，并清除Timer0中断标志，最后通知VIC中断处理结束并返回。


```c

void  Time0Init(void) {
	T0PR = 0;                // 设置定时器0不分频
	T0MCR = 0x03;       //设置Timer0匹配产生中断并复位T0TC
	T0MR0 = Fpclk*2-1;  // 2秒定时)
	T0TCR = 0x03;        // 启动并复位T0TC
	T0TCR = 0x01; 	

	// 设置定时器0中断IRQ
	VICIntSelect = 0x00000000;     // 所有中断通道设置为IRQ中断
	VICVectCntl0 = 0x20 | 0x04;	// 定时器0中断通道分配最高优先级
	VICVectAddr0 = (uint32)IRQ_Time0; // 设置中断服务程序地址向量
	VICIntEnable = 0x00000010;		// 使能定时器0中断
}


// 定时器0中断服务程序，取反BEEPCON控制口。
void __irq  IRQ_Time0(void)
{  
    if( (IO0SET&BEEPCON) == 0 ) {
        IO0SET = BEEPCON;
    }
    else IO0CLR = BEEPCON;         
    T0IR = 0x01;                // 清除中断标志
    VICVectAddr = 0x00000000;   // 通知VIC中断处理结束
}


int  main(void)
{  
     IRQEnable();
    PINSEL0 = 0x00000000;			// 设置管脚连接GPIO   
    IO0DIR = BEEPCON; 			  // 设置I/O为输出
    Time0Init();	 			// 初始化定时器0及使能中断
    while(1);		        // 等待定时器0中断或定时器1匹配输出

    return(0);
}


```
