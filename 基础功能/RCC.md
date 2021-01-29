# RCC

## 概述
- 5个时钟源
  - HSI
  - HSE
  - LSI:用于WDG
  - LSE:用于RTC
  - PLL:输入来自HSI/2,HSE,HSE/2;输出倍频可选2~16倍，最大不超过72MHz
- SYSCLK来自三个源：
  - PLL
  - HSE
  - HSI
- 时钟可输出到MCO(PA8)上，可为SYSCLK，PLL/2，HSI，HSE

## 寄存器

### 时钟控制寄存器(RCC_CR)
- 配置`HSE` `HSI`特性

### 时钟配置寄存器(RCC_CFGR)
- 选择时钟源，控制内部的倍频、通路、总线预分频等

### 时钟中断寄存器 (RCC_CIR) 

### APB2 外设复位寄存器 (RCC_APB2RSTR) 

### APB1 外设复位寄存器 (RCC_APB1RSTR) 

### AHB外设时钟使能寄存器 (RCC_AHBENR) 

### APB2 外设时钟使能寄存器(RCC_APB2ENR) 

### APB1 外设时钟使能寄存器(RCC_APB1ENR) 

### 备份域控制寄存器 (RCC_BDCR) 

### 控制/状态寄存器 (RCC_CSR) 

## 时钟初始化
1. 初始化RCC寄存器
2. 使能HSE并等待其启动完成
3. 设置PLL时钟源及倍频系数
4. 设置Flash
5. 启动PLL并等待其稳定
6. 选择PLL作为SYSCLK时钟源并等待稳定
7. 开启总线设备时钟

## 配置模板
```C
void RCC_Configuration(void){ //RCC时钟的设置  
	ErrorStatus HSEStartUpStatus;   
	RCC_DeInit();              /* RCC system reset(for debug purpose) RCC寄存器恢复初始化值*/   
	RCC_HSEConfig(RCC_HSE_ON); /* Enable HSE 使能外部高速晶振*/   
	HSEStartUpStatus = RCC_WaitForHSEStartUp(); /* Wait till HSE is ready 等待外部高速晶振使能完成*/   
	if(HSEStartUpStatus == SUCCESS){   
		/*设置PLL时钟源及倍频系数*/   
		RCC_PLLConfig(RCC_PLLSource_HSE_Div1, RCC_PLLMul_9); //RCC_PLLMul_x（枚举2~16）是倍频值。当HSE=8MHZ,RCC_PLLMul_9时PLLCLK=72MHZ   
		/*设置AHB时钟（HCLK）*/   
		RCC_HCLKConfig(RCC_SYSCLK_Div1); //RCC_SYSCLK_Div1——AHB时钟 = 系统时钟(SYSCLK) = 72MHZ（外部晶振8HMZ）   
		/*注意此处的设置，如果使用SYSTICK做延时程序，此时SYSTICK(Cortex System timer)=HCLK/8=9MHZ*/   
		RCC_PCLK1Config(RCC_HCLK_Div2); //设置低速AHB时钟（PCLK1）,RCC_HCLK_Div2——APB1时钟 = HCLK/2 = 36MHZ（外部晶振8HMZ）   
		RCC_PCLK2Config(RCC_HCLK_Div1); //设置高速AHB时钟（PCLK2）,RCC_HCLK_Div1——APB2时钟 = HCLK = 72MHZ（外部晶振8HMZ）   
		/*注：AHB主要负责外部存储器时钟。APB2负责AD，I/O，高级TIM，串口1。APB1负责DA，USB，SPI，I2C，CAN，串口2，3，4，5，普通TIM */  
		FLASH_SetLatency(FLASH_Latency_2); //设置FLASH存储器延时时钟周期数   
		/*FLASH时序延迟几个周期，等待总线同步操作。   
		推荐按照单片机系统运行频率：
		0—24MHz时，取Latency_0；   
		24—48MHz时，取Latency_1；   
		48~72MHz时，取Latency_2*/   
		FLASH_PrefetchBufferCmd(FLASH_PrefetchBuffer_Enable); //选择FLASH预取指缓存的模式，预取指缓存使能   
		RCC_PLLCmd(ENABLE);	//使能PLL
		while(RCC_GetFlagStatus(RCC_FLAG_PLLRDY) == RESET); //等待PLL输出稳定   
		RCC_SYSCLKConfig(RCC_SYSCLKSource_PLLCLK); //选择SYSCLK时钟源为PLL
		while(RCC_GetSYSCLKSource() != 0x08); //等待PLL成为SYSCLK时钟源   
	}  

/*开始使能程序中需要使用的外设时钟*/   
//  RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1 | RCC_APB2Periph_GPIOA | RCC_APB2Periph_GPIOB |   
//  RCC_APB2Periph_GPIOC | RCC_APB2Periph_GPIOD | RCC_APB2Periph_GPIOE, ENABLE); //APB2外设时钟使能      
//	RCC_APB1PeriphClockCmd(RCC_APB1Periph_USART2, ENABLE); //APB1外设时钟使能  
//	RCC_APB1PeriphClockCmd(RCC_APB1Periph_USART3, ENABLE);   
//	RCC_APB2PeriphClockCmd(RCC_APB2Periph_SPI1, ENABLE);   	 
//	RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);    
}  
```