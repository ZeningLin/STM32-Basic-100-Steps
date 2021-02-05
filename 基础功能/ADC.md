# ADC
## 概述
对于STM32F103系列单片机有
- 2个ADC
- 每个ADC分辨率为12位
- 2个ADC共用16个外部通道
  - `F103C8T6`中占用引脚10~19，共10个外部通道
- 支持DMA操作
- VDDA和VSSA可接独立稳定的电源为ADC供电提高精度和稳定性
- ADC挂载总线APB2
- 若开启DMA模式，还需要打开DMA挂载的AHB总线

## 在STM32F1中的使用
**初始化**
1. 开启总线时钟
    ```C
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA|RCC_APB2Periph_GPIOB, ENABLE);       
    RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);//使能DMA时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);//使能ADC1时钟
    ```
2. 初始化`DMA_InitTypeDef`结构体，调用`DMA_Init(DMA_Channel, *DMA_InitTypeDef DMA_InitStruct)`函数初始化参数，同时调用`DMA_Cmd(DMAx_Channely, ENABLE)`使能DMA通道
3. 初始化`GPIO_InitTypeDef`结构体，模式设置为**模拟输入模式**
4. 初始化`ADC_InitTypeDef`结构体，调用`void ADC_Init(ADC_TypeDef *ADCx, ADC_InitTypeDef *ADC_InitStruct)`将结构体送入
5. 调用`ADC_RegularChannelConfig(ADC_TypeDef* ADCx, uint8_t ADC_Channel, uint8_t Rank, uint8_t ADC_SampleTime)`设置ADC通道采样通道和采样周期
6. `ADC_DMACmd(ADC1, ENABLE)`开启ADC的DMA支持
7. `ADC_Cmd(ADC1, ENABLE)`使能ADC1
8. 校准
    ```C
    ADC_ResetCalibration(ADCx); //重置ADCx校准寄存器
    while(ADC_GetResetCalibrationStatus(ADCx));//等待ADCx校准重置完成
    ADC_StartCalibration(ADCx);//开始ADCx校准
    while(ADC_GetCalibrationStatus(ADCx));//等待ADCx校准完成
    ADC_SoftwareStartConvCmd(ADCx, ENABLE); //使能ADCx软件开始转换
    ```

**数据读取**
- 由于使用了DMA，因此数据直接被送入SRAM中，若使用ADC1，查映射表可知数据存放在0x4001244C单元中(0x40012400~0x400127FF为ADC1的地址空间，数据存放寄存器偏移量为0x4C，该寄存器低16位存放的即为ADC转换所得的数据；0x40012800~0x40012BFF为ADC2的地址空间)
- 在DMA初始化结构体中，将`DMA_PeripheralBaseAddr`项设置为0x4001244C，将`DMA_MemoryBaseAddr`项设置为一个u16数组的指针，即可通过该数组读取模拟量的值。数组长度等于需要读取的模拟量的个数。
- 使用多个ADC时，注意`ADC_InitStructure.ADC_NbrOfChannel`需要设置为使用的ADC个数，同时`ADC_RegularChannelConfig(ADC_TypeDef* ADCx, uint8_t ADC_Channel, uint8_t Rank, uint8_t ADC_SampleTime)`也需要调用相应的次数分别设置不同通道的采样顺序和采样周期

---

### ADC_InitTypeDef结构
```C
typedef struct{
    u32 ADC_Mode;
    FunctionalState ADC_ScanConvMode;
    FunctionalState ADC_ContinuousConvMode;
    u32 ADC_ExternalTrigConv;
    u32 ADC_DataAlign;
    u8 ADC_NbrOfChannel;
} ADC_InitTypeDef
```

#### **ADC_Mode**
ADC_Mode 设置 ADC 工作在独立或者双 ADC 模式。
|ADC_Mode| 描述|
|--|--|
|*ADC_Mode_Independent*| *ADC1 和 ADC2 工作在独立模式*|
|ADC_Mode_RegInjecSimult| ADC1 和 ADC2 工作在同步规则和同步注入模式|
|ADC_Mode_RegSimult_AlterTrig| ADC1 和 ADC2 工作在同步规则模式和交替触发模式|
|ADC_Mode_InjecSimult_FastInterl| ADC1 和 ADC2 工作在同步规则模式和快速交替模式|
|ADC_Mode_InjecSimult_SlowInterl| ADC1 和 ADC2 工作在同步注入模式和慢速交替模式|
|ADC_Mode_InjecSimult| ADC1 和 ADC2 工作在同步注入模式|
|ADC_Mode_RegSimult| ADC1 和 ADC2 工作在同步规则模式|
|ADC_Mode_FastInterl| ADC1 和 ADC2 工作在快速交替模式|
|ADC_Mode_SlowInterl| ADC1 和 ADC2 工作在慢速交替模式|
|ADC_Mode_AlterTrig| ADC1 和 ADC2 工作在交替触发模式|
#### **ADC_ScanConvMode**
ADC_ScanConvMode 规定了模数转换工作在扫描模式（多通道）还是单次（单通道）模式。可以设置这个
参数为 *ENABLE* 或者 DISABLE。
#### **ADC_ContinuousConvMode**
ADC_ContinuousConvMode 规定了模数转换工作在连续还是单次模式。可以设置这个参数为 *ENABLE* 或
者 DISABLE。
#### **ADC_ExternalTrigConv**
ADC_ExternalTrigConv 定义了使用外部触发来启动规则通道的模数转换.
|ADC_ExternalTrigConv| 描述|
|--|--|
|ADC_ExternalTrigConv_T1_CC1| 选择定时器 1 的捕获比较 1 作为转换外部触发|
|ADC_ExternalTrigConv_T1_CC2| 选择定时器 1 的捕获比较 2 作为转换外部触发|
|ADC_ExternalTrigConv_T1_CC3| 选择定时器 1 的捕获比较 3 作为转换外部触发|
|ADC_ExternalTrigConv_T2_CC2| 选择定时器 2 的捕获比较 2 作为转换外部触发|
|ADC_ExternalTrigConv_T3_TRGO| 选择定时器 3 的 TRGO 作为转换外部触发|
|ADC_ExternalTrigConv_T4_CC4| 选择定时器 4 的捕获比较 4 作为转换外部触发|
|ADC_ExternalTrigConv_Ext_IT11| 选择外部中断线 11 事件作为转换外部触发|
|*ADC_ExternalTrigConv_None*| *转换由软件而不是外部触发启动*|
#### **ADC_DataAlign**
ADC_DataAlign 规定了 ADC 数据向左边对齐还是向右边对齐。
|ADC_DataAlign| 描述|
|--|--|
|*ADC_DataAlign_Right*| *ADC 数据右对齐*|
|ADC_DataAlign_Left| ADC 数据左对齐|
#### **ADC_NbrOfChannel**
ADC_NbreOfChannel 规定了顺序进行规则转换的 ADC 通道的数目。这个数目的取值范围是 1 到 16。

---

