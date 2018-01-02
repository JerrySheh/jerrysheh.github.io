---
title: ARM笔记（八）AD转换
categories: ARM基础
tags: ARM
abbrlink: f61a54c1
date: 2017-11-21 20:35:30
---

AD转换

<!-- more -->

# 寄存器

## 控制寄存器 ADCR

ADCR：A/D控制寄存器。A/D转换开始前，必须设置ADCR寄存器来选择工作模式。27位。

### 7:0 SEL

SEL：从AIN3～AIN0(LPC2114/2124)或AIN7～AIN0(LPC2212/2214)中选择采样和转换输入引脚。Bit0控制AIN0，bit1控制AIN1，依此类推。
1:对应输入端被选中；    0:对应输入端未选中；

注意：软件模式下只能置位其中一位，硬件模式下可以是任意组合


### 15:8 CLKDIV

CLKDIV：将VPB时钟（PCLK）进行分频，得到AD转换时钟。分频后的时钟必须小于或等于4.5MHz。通常将CLKDIV编程为允许的最小值，以获得4.5MHz或稍低于4.5MHz的时钟。

A/D转换器时钟 ＝ PCLK / ( CLKDIV + 1)

### 16 BURST

BURST：BURST/软件方式控制。当该位为0时，选择软件方式启动AD转换，需要11个时钟才能完成。当该位为1时，选择BURST(突发)模式启动AD转换，所需时钟数由CLK字段控制。

BURST模式下，对所有在SEL字段中置1的位对应的输入端进行转换，首先转换的是最低有效位。然后是更高的位。如此周而复始，直至该位清零。

### 19:17 CLKS

CLKS：控制BURST模式下每次转换需要使用的时钟数和所得ADDR转换结果的LS位中可确保精度的位的数目，CLKS可在11个时钟（10位）～4个时钟（3位）之间选择：000=11个时钟/10位，001=10个时钟/9位，…111=4个时钟/3位。

### 21 PDN

PDN：控制AD部件是否工作。

1:A/D转换器处于正常工作模式；	0:A/D转换器处于掉电模式

### 23:22 TEST1:0

TEST1:0：器件测试控制位。

	00:正常模式；		01:数字测试模式；
	10:DAC测试模式；	11:一次转换测试模式；

### 26:24

START：该字段用于控制AD转换的启动方式，该字段只有在BURST为0时有效。

置数|功能
---|---
000|不启动；
001|立即启动转换；
010|P0.16引脚
011|P0.22引脚
100|MAT0.1引脚
101|MAT0.3引脚
110|MAT1.0引脚
111|MAT1.1引脚

010的意思是，P0.16引脚出现预置的电平时，启动AD转换。011-111类比。

### 27

EDGE：当START字段的值为010～111时，该位的设置有效。


	0:在所选CAP/MAT信号的下降沿启动转换
	1:在所选CAP/MAT信号的上升沿启动转换


---

## 数据寄存器

ADDR：A/D数据寄存器。该寄存器包含ADC的结束标志位和10位的转换结果（当结束标志位为1时，转换结果才是有效的）。 32位

### 5:0

0：这些位读出时为0。用于未来扩展功能更强大的AD转换器。


### 15:6 V/VddA

V/VddA：当DONE位为1时，该字段包含对SEL字段选中的Ain脚的转换结果，为一个二进制数。

转换结果为0时，表示Ain引脚电平小于、等于或接近于VSSA。为0x3FF时，表示Ain引脚电平等于、大于或接近于VddA。

输入电压计算公式为：Vin = 结果×(VSSA / 0x400)


### 23:16

0：这些位读出时为0。它们允许连续A/D值的累加，而不需要屏蔽处理，使得至少有256个值不会溢出到CHN字段。

### 26:24 CHN

CHN：该字段包含的是LS位的转换通道

### 29:27

0：这些位读出为0。用于未来CHN字段的扩展，使之兼容更多通道的转换值。

### 30 OVERUN

OVERUN：在BURST模式下，如果在转换产生最低位之前，以前转换的结果丢失或被覆盖，该位将置位。读ADDR寄存器时，该位清零。

### 31 DONE

DONE：AD转换完成标志位。当AD转换结束时该位置位。在读取ADDR或ADCR被写入时，该位清零。如果在转换过程中，设置了ADCR，那么该位将置位，并启动一次新的转换。


---

# 使用AD转换器的注意要点

1. AD转换器的时钟不能大于4.5MHz；
2. 使用MAT引脚触发AD转换启动时，相应的MAT信号不必输出到引脚。使用MAT引脚触发的方法，可以实现AD转换定时启动；
3. BURST模式下，每次转换结束后立即开始下一路的转换，所以BURST模式具有最高的效率；
4. 软件模式下，SEL字段中只能有一位置位，如果多位置位，将使用最低有效位。

---

# A/D转换器操作示例

操作流程： 计算ADC部件时钟 -> 设置引脚连接模块 -> 设置AD工作模式 -> 启动AD转换 -> 等待转换结束 -> 读取转换结果

```c

#define ADCLK   4500000     // 定义AD部件时钟频率，单位：Hz
#define ADBIT   10          // 定义BURST模式下的转换精度
#define ADBIT2  (10 - ADBIT)

...
PINSEL1 = (PINSEL1 & 0xFC3FFFFF) | 0x00400000;  //设置引脚连接模块
ADCR =  (0x01 << 27)  |                // EDGE  硬件触发边沿设置
        (0x05 << 24)  |                // START AD启动设置
        (0x00 << 22)  |                // TEST1:0  测试模式设置
        (0x01 << 21)  |                // PDN    AD部件上电测试
        (ADBIT2 << 17)|                // CLKS   BURST模式精度
        (0x00 << 16)  |                // BUREST BURST模式使能
        ((Fpclk/ADCLK + 1) <<  8) |    // CLKDIV  ADC部件时钟
        (0x01 << 0);                   // SEL   转换通道选择
ADCR |= (1 << 24);                     // 启动AD转换
While((ADDR & 0x80000000) != 0);       // 等待转换结束
ADCData = (ADDR >> 6) & 0x3FF;         // 读取转换结果
...


```


---

# AD转换控制 —— 按需采样

```c

//ADC模块初始化设置
PINSEL1 = 0x01400000;         // 设置P0.27、P0.28连接到AIN0、AIN1
ADCR = (1 << 0)                    |   // SEL = 1 ，选择通道0
      ((Fpclk / 1000000 - 1) << 8) |  // CLKDIV = Fpclk / 1000000 - 1 ，转换时钟1M
          (0 << 16)               |   // BURST = 0 ，软件控制转换操作
          (0 << 17)               |   // CLKS = 0 ，使用11clock转换
          (1 << 21)               |   // PDN = 1 ， 正常工作模式(非掉电转换模式)
          (0 << 22)               |   // TEST1:0 = 00 ，正常工作模式(非测试模式)
          (1 << 24)               |    // START = 1 ，直接启动ADC转换
          (0 << 27);		          // EDGE = 0 (CAP/MAT引脚下降沿触发ADC转换)

    DelayNS(10);							
    ADC_Data = ADDR;	// 读取ADC结果，并清除DONE标志位


//主函数不断启动AD转换
while(1)    {  
		ADCR = (ADCR&0x00FFFF00)|0x01|(1 << 24); //设置通道1,第1次转换
		while( (ADDR&0x80000000)==0 );                     // 等待转换结束
	  ADCR = ADCR | (1 << 24);	                               // 再次启运转换
		while( (ADDR&0x80000000)==0 );                     // 等待转换结束            
	  ADC_Data = ADDR;                                             // 读取ADC结果
		ADC_Data = (ADC_Data>>6) & 0x3FF;            // 提取AD转换值
		ADC_Data = ADC_Data * 3300;                          // 数值转换
		ADC_Data = ADC_Data / 1024;
		sprintf(str, "%4dmV at VIN1", ADC_Data);
	  ISendStr(60, 23, 0x30, str);                  
		ADCR = (ADCR&0x00FFFF00)|0x02|(1 << 24);// 设置通道2

     while( (ADDR&0x80000000)==0 );	           // 等待转换结束
	   ADCR = ADCR | (1 << 24);		           // 再次启运转换
     while( (ADDR&0x80000000)==0 );                // 等待转换结束
     ADC_Data = ADDR;		                      // 读取ADC结果
	   ADC_Data = (ADC_Data>>6) & 0x3FF;      // 提取AD转换值
	   ADC_Data = ADC_Data * 3300;                    // 数值转换
     ADC_Data = ADC_Data / 1024;
	   sprintf(str, "%4dmV at VIN2", ADC_Data);
     UART0SendStr(str);
     DelayNS(10);        
}


```

---


## AD转换控制 —— 周期采样

1. 以周期2s的时间对通道0电压ADC转换；
2. 把结果转换成电压值，然后发送到串口

```c

//ADC中断服务程序
void  __irq  IRQ_ADC(void){
        uint32  ADC_Data;
        char    str[20];
         ADC_Data = ADDR;	                                 // 读取ADC结果
         ADC_Data = (ADC_Data>>6) & 0x3FF;   // 提取AD转换值
         ADC_Data = ADC_Data * 3300;                  // 数值转换
         ADC_Data = ADC_Data / 1024;
         sprintf(str, "%4dmV at VIN2", ADC_Data);
         UART0SendStr(str);
        VICVectAddr=0;
}

//ADC工作模式初始化
void ADC_Init(void){
         uint32  ADC_Data;
         PINSEL1 = 1<<22;      // 设置P0.27 连接到AIN0
         ADCR = (1 << 0)         |// SEL = 1 ，选择通道0
((Fpclk / 1000000 - 1) << 8) | // CLKDIV = Fpclk / 1000000 - 1  转换时钟:1MHz
           (0 << 16)         | // BURST = 0 ，软件控制转换操作
           (0 << 17)         | // CLKS = 0 ，使用11clock转换
           (1 << 21)         | // PDN = 1 ， 正常工作模式(非掉电转换模式)
           (0 << 22)         | // TEST1:0 = 00 ，正常工作模式(非测试模式)
           (4 << 24)         | // START = 100 ，MAT0.1出现时启动转换
           (1 << 27);		       // EDGE = 0 (CAP/MAT引脚下降沿触发ADC转换)
          DelayNS(10);							
         ADC_Data = ADDR;	// 读取ADC结果，并清除DONE标志位
}


//ADC中断初始化
void ADC_Int( void)
{
    VICIntSelect = 0x00000000;		    VICVectCntl0 = 0x20|18;		
    VICVectAddr0 = (int)IRQ_ADC;
    VICIntEnable = 1<<18;           		
}


//定时器初始化
void  Time0Init(void){
     T0PR = 99;			    		
     T0MCR = 0x18;		   				
     T0EMR =0xc0;    //MAT0.1匹配翻转
     T0MR1 = 110591;	    				
     T0TCR = 0x03;		   			
     T0TCR = 0x01;
}


int  main(void){  
       IRQEnable();
       Time0Init();					    
       ADC_Init();
       ADC_Int();
       UART0Init(9600);
       while(1);
}



```
