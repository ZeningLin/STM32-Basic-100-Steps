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