# RTC与后备寄存器

## RTC
### 0. 概述
- 32位寄存器计数，分为`RTC_CNTL`和`RTC_CNTH`两个16位
- 三种时钟源
  - HSE的128分频，依赖VDD
  - LSI，依赖VDD
  - LSE，**常用**，因为VDD掉电还能通过VBAT工作
- 输入的时钟源要经过预分频模块，由20位寄存器控制，一般经过软件配置将LSE/32768得到1Hz的时钟信号，控制`RTC_CNT`每秒增加一次
- 有接口与APB1总线相连，但是两者使用完全不同的时钟源，要保证RTC的时钟源频率小于APB1时钟源频率的1/4
- 产生三种中断
  - 溢出中断，32位计数器溢出时触发
  - 秒中断，每秒中断一次
  - 闹钟中断，`RTC_CNT`与`RTC_ALR`值一致时触发，可用于将单片机从待机模式唤醒

### 1. 寄存器组成：
寄存器实际上为32位，但是高16位保留，所以看起来就是16位寄存器
#### 1.1 RTC_CRH 控制寄存器高位
- 专门控制中断
- 2，1，0位分别控制溢出，闹钟，秒中断
- 1为允许中断，0为屏蔽中断

#### 1.2 RTC_CRL 控制寄存器低位
- `RTOFF`：1说明上一次的配置操作已完成，0说明未完成，对RTC的配置一定要等到其置1才能进行
- `CNF`：配置开关，1进入配置模式，0则退出，仅当1时才能向`RTC_CNT` `RTC_ALR` `RTC_PRL`写入数据
- `RSF`：指示与APB1总线的同步，1时说明已同步，0时未同步，需要等待其为1时才能读取数据
- `OWF`：1时指示计数器溢出标志
- `ALRF`：1时指示闹钟到时间
- `SECF`：秒标志，1说明预分频器溢出，`RTC_CNT`加一位

#### 1.3 RTC_PRLH/RTC_PRLL 预分频装载寄存器
- 高位寄存器低四位和低位寄存器16位组成20位寄存器存放分频值
- 计算公式：$f_{TR\_CLK}=f_{TCLK}/(PRL[19:0]+1)$
- 每一个TR_CLK周期结束后会将该值送入预分频计数器

#### 1.4 RTC_DIC 预分频余数寄存器
- 存放预分频计数器的当前值
- 20位，高位低四位与低位16位组成

#### 1.5 其他的寄存器(原理没什么好说的就放在这了)
- `RTC_CNT`
- `RTC_ALR`

### 2. RTC寄存器读取
- 通过APB1总线接口读取寄存器`RTC_CNT`,`RTC_ALR`,`RTC_PRL`,`RTC_DIV`的内容 
- RTC内部与APB1总线完全独立，若APB1总线被关闭过，则会出现不同步问题，需要等待硬件将`RTC_CRL`中的`RSF`复位为1才可读取

### 3. RTC寄存器写入
- 必须设置`RTC_CRL`寄存器中的`CNF`位，使RTC进入配置模式后，才能写入`RTC_PRL`、`RTC_CNT`、`RTC_ALR`寄存器。
- 对RTC任何寄存器的写操作，都必须在前一次写操作结束后进行。可以通过查询`RTC_CR`寄存器中的`RTOFF`状态位，判断RTC寄存器是否处于更新中。仅当`RTOFF`状态位是’1’时，才可以写入RTC寄存器。

配置过程：
1. 查询`RTOFF`位，直到`RTOFF`的值变为’1’
2. 置`CNF`值为1，进入配置模式
3. 对一个或多个RTC寄存器进行写操作
4. 清除`CNF`标志位，退出配置模式
5. 查询`RTOFF`，直至`RTOFF`位变为’1’以确认写操作已经完成。
- 仅当`CNF`标志位被清除时，写操作才能进行，这个过程至少需要**3个RTCCLK周期**。

### 总配置流程
```C
void RTC_First_Config(void){ 
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR | RCC_APB1Periph_BKP, ENABLE);
    PWR_BackupAccessCmd(ENABLE);
    BKP_DeInit();
    RCC_LSEConfig(RCC_LSE_ON);
    while (RCC_GetFlagStatus(RCC_FLAG_LSERDY) == RESET);   
    RCC_RTCCLKConfig(RCC_RTCCLKSource_LSE);
    RCC_RTCCLKCmd(ENABLE);
    RTC_WaitForSynchro();
    RTC_WaitForLastTask();
    RTC_SetPrescaler(32767); 
    RTC_WaitForLastTask();
    
    RTC_ITConfig(RTC_IT_SEC, ENABLE);   
    RTC_WaitForLastTask();
}

void RTC_Config(void){ 
    if (BKP_ReadBackupRegister(BKP_DR1) != 0xA5A5){ 
        RTC_First_Config();
        BKP_WriteBackupRegister(BKP_DR1, 0xA5A5);
    }else{
        if (RCC_GetFlagStatus(RCC_FLAG_PORRST) != RESET){

        }
        else if (RCC_GetFlagStatus(RCC_FLAG_PINRST) != RESET){
            
        }       
        RCC_ClearFlag();

      
        RCC_RTCCLKCmd(ENABLE);       
        RTC_WaitForSynchro();

        
        RTC_ITConfig(RTC_IT_SEC, ENABLE);  
        RTC_WaitForLastTask();
    }
	#ifdef RTCClockOutput_Enable   
	    RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR | RCC_APB1Periph_BKP, ENABLE);
	    PWR_BackupAccessCmd(ENABLE);   
	    BKP_TamperPinCmd(DISABLE);   
	    BKP_RTCOutputConfig(BKP_RTCOutputSource_CalibClock);
	#endif
}
```


---
## 后备寄存器
- 10个16位寄存器组成
- VDD断电后可通过VBAT供电
- VBAT断电后数据丢失，可通过写入一个特殊的数据，每次运行时读取来判断VBAT是否掉电