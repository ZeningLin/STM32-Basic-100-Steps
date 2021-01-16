# GPIO

## 寄存器
### 1.1 GPIOx_CRH配置寄存器高位 / GPIOx_CRH配置寄存器低位
- 两个32位
- 每一个IO口需要4位，总共需要64位，因此需要两个32位
- 4位配置分为`CNFx[1:0]`和`MODEx[1:0]`两部分，`x`为IO口编号
  - `MODEx`配置输入或输出模式
    - 00:输入模式
    - 01:输出，10MHz
    - 10:输出，2MHz
    - 11:输出，50MHz
  - `CNFx`配置具体模式
    - 输入时
      - 00:模拟
      - 01:浮空
      - 10:上下拉(具体you由ODR寄存器控制)
      - 11:无
    - 输出时
      - 00:推挽
      - 01:开漏
      - 10:复用推挽
      - 11:复用开漏
### 1.2 GPIOx_IDR端口输入数据寄存器
- 32位，高16位保留，仅有16位用于存放输入数据，只能16位同时读出

### 1.3 GPIOx_ODR端口输出数据寄存器
- 32位，高16位保留，仅有16位用于存放输入数据，只能16位同时写入
- 在输入模式下最低位控制上下拉，0为下拉，1为上拉
### 1.4 GPIOx_BSRR端口位设置/清除寄存器
- [31:16]用于按位清除ODR，1为清除，0不操作，16位同时操作
- [15:0]用于按位写入ORD，1为置1，0不操作，16位同时操作

### 1.5 GPIOx_BRR端口位清除寄存器
- 功能与BSRR高16位相同



## 初始化
- 所有GPIO端口使用前都要进行初始化，内容包括：
  - GPIO端口结构体实例化
    ```C
    typedef struct{
        uint16_t GPIO_Pin;
        GPIOSpeed_TypeDef GPIO_Speed;
        GPIOMode_TypeDef GPIO_Mode;
    }GPIO_InitTypeDef;
    ```
  - 启动APB2总线时钟`RCC_APBxPeriphClockCmd()`
  - 结构体元素赋值
    - 设置GPIO端口名称
    - 设置端口速率
    - 设置端口模式
  - 将上述初始化内容写入内部寄存器

```C
void Init(void){
    GPIO_InitTypeDef GPIO_InitStructure; //声明结构体
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA|RCC_APB2Periph_GPIOB|RCC_APB2Periph_GPIOC,Enable); //启动APB2总线,第二个参数可为Enable和Disable对应使能和关闭
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_x|GPIO_Pin_x|GPIO_Pin_x; //指定端口号，x为0~15之间的值，对应每一组内的引脚号，可以有多个值，用‘|’隔开
    GPIO_InitSturcture.GPIO_Speed = FPIO_Speed_50MHz; //设置端口速率，可选2，10，50MHz，端口为输入时不需要设置
    GPIO_InitSturcture.GPIO_Mode = GPIO_Mode_xx_xx; //设置端口模式
    GPIO_Init(GPIOx, &GPIO_InitSturcture); //上述内容写入寄存器，GPIOx为端口组名称，x可为ABCD，视引脚而定
    
    /*
    端口模式：
    GPIO_Mode_AIN 模拟输入
    GPIO_Mode_IN_FLOATING 浮空输入
    GPIO_Mode_IPD 下拉输入
    GPIO_Mode_IPU 上拉输入
    GPIO_Mode_Out_PP 推挽输出
    GPIO_Mode_Out_OD 开漏输出
    GPIO_Mode_AF_PP 复用推挽输出
    GPIO_Mode_AF_OD 复用开漏输出
    */ 
}
```

## 控制GPIO口电平
### GPIO_WriteBit
```C
GPIO_WriteBit(GPIOx,GPIO_Pin_x,(BitAction)(x));
//GPIOx为端口组，GPIO_Pin_x为端口号
//(BitAction)(x)中x为1输出高电平，0输出低电平
```
### GPIO_Write
- 与`GPIO_WriteBit`不同，这个是对整组IO口进行操作
```C
GPIO_Write(GPIOx,Val);
//GPIOx为端口组
//Val为16进制数，0x开头，控制所有端口
```

### GPIO_SetBits
- 端口置位
```C
GPIO_SetBit(GPIOx,GPIO_Pin_x);
//GPIOx为端口组，GPIO_Pin_x为端口号
```
### GPIO_ResetBits
- 端口复位
```C
GPIO_ResetBit(GPIOx,GPIO_Pin_x);
//GPIOx为端口组，GPIO_Pin_x为端口号
```

## 示例：按键消抖
```C
if(!GPIO_ReadInputDataBit(GPIOx,GPIO_Pin_x)){//先判断是否为干扰
  delay_ms(20); //延时消抖
  if(!GPIO_ReadInputDataBit(GPIOx,GPIO_Pin_x)){//真正的读取按键状态
    xxx;
    while(!GPIO_ReadOutputDataBit(GPIOx,GPIO_Pin_x)); //等待按键松开
  }
}
```