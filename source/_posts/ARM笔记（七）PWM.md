---
title: ARM笔记（七）PWM
date: 2017-11-21 20:21:13
tags: ARM
---

LPC2000的PWM基于标准的定时器模块（此定时器是PWM专用），具有定时器的所有特性，它是定时器功能中匹配事件的功能扩展。使用PWM功能，可以在指定引脚输出需要的波形。


---

<!-- more -->

#  PWM 引脚

管脚名称|管脚方向|管脚描述
---|---|---
p0.0/PWM1|输出|PWM输出通道1
p0.7/PWM2|输出|PWM输出通道2
p0.3/PWM3|输出|PWM输出通道3
p0.8/PWM4|输出|PWM输出通道4
p0.21/PWM5|输出|PWM输出通道5
p0.9/PWM6|输出|PWM输出通道6


---

# 基本控制寄存器

![PWM1](../../../../images/PWM1.png)

---

# 匹配功能寄存器

![PWM2](../../../../images/PWM2.png)

---

# 使用PWM的注意要点

1. 修改匹配寄存器之后，必须设置锁存使能寄存器中的相应位，否则匹配寄存器的值不能生效；
2. 修改匹配寄存器时，不需要停止PWM定时器，以免产生无完整的PWM波形；
3. 不使用PWM功能时，可将该部件作为一个标准的32位定时器使用；
4. PWMTC计数频率 = Fpclk / (PWMPR+1)


---

# PWM使用示例1 —— 单边沿输出

```c

Void PWM1Out(uint16 FREQ)  //FREQ为输出频率，单位Hz
{
    PINSEL0 &= 0xFFFFFFFC;    
    PINSEL0 |= 0x00000002;    //设置引脚连接模块连接PWM1输出
    PWMPCR   = 0x200;         //使能PWM1输出
    PWMMCR   = 0x02;          //MR0匹配后复位定时器
    PWMPR    = 0x00;          //设置预分频值为0
    PWMMR0   = Fpclk / FREQ-1;//设置PWM周期
    PWMMR1   = PWMMR0 / 2;    //设置占空比为50%
    PWMLER   = 0x03;          //使能PWM匹配锁存
    PWMTCR   = 0x09;          //使能PWM，启动定时器
}

```

# PWM使用示例2 —— 双边沿输出

```c

Void PWM2Out(uint16 FREQ)    //FREQ为输出频率，单位Hz
{
    PINSEL0 &= 0xFFFF3FFF;   
    PINSEL0 |= 0x00008000;   //设置引脚连接模块连接PWM2输出
    PWMPCR   = 0x404;        //设置PWM2双边沿输出
    PWMMCR   = 0x02;         //MR0匹配后复位定时器
    PWMPR    = 0x00;         //设置预分频值为0
    PWMMR0   = Fpclk / FREQ; //设置PWM周期
    PWMMR1   = PWMMR0 / 5;   //设置前沿在周期的1/5处
    PWMMR2   = PWMMR1 * 2;   //设置后沿在周期的2/5处
    PWMLER   = 0x07;         //使能PWM匹配锁存
    PWMTCR   = 0x09;         //使能PWM，启动定时器
}


```


---


# 使用PWM6输出占空比可调的PWM波形

```c

#define  CYCLE_DATA		2000
#define  DUTY_CYCLE_DATA	1000
int  main(void){  
    PINSEL0 = 0x00080000;        // 设置PWM6连接到P0.9管脚
    PWMPR = 0x00;		            // 不分频，计数频率为Fpclk
    PWMMCR = 0x02;		          // 设置PWMMR0匹配时复位PWMTC
    PWMMR0 = CYCLE_DATA;        // 设置PWM周期
    PWMMR6 = DUTY_CYCLE_DATA;   // 设置PWM占空比   
    PWMLER = 0x41;	            // PWMMR0、PWMMR6锁存
    PWMPCR = 0x4000;		        // 允许PWM6输出，单边PWM
    PWMTCR = 0x09;	            // 启动定时器，PWM使能   
    while(1);
    return(0);
}


```
