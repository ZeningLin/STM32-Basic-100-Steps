# GPIO

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
  - 启动APB2总线
  - 结构体元素赋值
    - 设置GPIO端口名称
    - 设置端口速率
    - 设置端口模式
  - 将上述初始化内容写入内部寄存器

```C
void Init(void){
    GPIO_InitTypeDef GPIO_InitStructure; //声明结构体
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA|RCC_APB2Periph_GPIOB|RCC_APB2Periph_GPIOC,Enable);//启动APB2总线
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_x; //指定端口号，x为0~15之间的值，对应每一组内的引脚号
    GPIO_InitSturcture.GPIO_Speed = FPIO_Speed_50MHz; //设置端口速率，可选2，10，50MHz
    GPIO_InitSturcture.GPIO_Mode = GPIO_Mode_xx_xx; //设置端口模式
    GPIO_Init(LEDPORT, &GPIO_InitSturcture);上述内容写入寄存器
    
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