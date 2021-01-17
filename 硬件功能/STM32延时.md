# 嘀嗒定时器与延时
在没有操作系统的情况下使用嘀嗒定时器完成精准延时
## 嘀嗒定时器寄存器
|符号|寄存器|
|--|--|
|SysTick->CTRL|控制和状态寄存器|
|SysTick->LOAD|重装载值寄存器|
|SysTick->VAL|当前值寄存器|
|SysTick->CALIB|校准值寄存器|

### **Systick->CTRL**
|位|名称|描述|
|--|--|--|
|16|COUNTFLAG|倒计时结束后置1，指示计时结束|
|2|CLKSOURCE|选择时钟源，0为外部参考时钟，1为内核时钟|
|1|TICKINT|1=计时结束产生异常，0=不产生异常|
|0|ENABLE|计时器使能|

### **Systick->LOAD**
|位|名称|描述|
|--|--|--|
|23..0|RELOAD|重装载值|

### **Systick->VAL**
|位|名称|描述|
|--|--|--|
|23..0|CURRENT|当前值|

## 延时程序
```C
void delay_us(u32 us){
    SysTick->LOAD = AHB_INPUT * us; //AHB_INPUT 对应主频率，RCC设置，需要宏定义声明
    SysTick->CTRL = 0x00000005; //使用内核时钟，不产生异常，使能计时器
    while(!(SysTick->CTRL&0x00010000)); //等待计时结束控制寄存器16位置1
    SysTick->CTRL=0x00000004; //使能关闭，停止计时
}
```