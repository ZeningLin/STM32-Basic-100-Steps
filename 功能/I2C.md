# I2C通信协议
## 综述
- 两线通信，半双工
- 所有设备都可以作为主机/从机，但一个时刻只能有一个主机


## 通信格式
- 传送数据时，仅当`SCL`低电平时`SDA`才能变化
- 当`SCL`处于高电平时，若`SDA`拉低则表示起始位，`SDA`拉高则表示终止位

### 主机向从机写入数据
主机发送起始位后，发送一个字节的接收端地址位，地址位高7位表示地址，第0位表示读(1)或者写(0)，此处为0，从机接收后发送一位低电平应答位；之后主机发送数据位，主机发送一个字节的数据紧跟从机回复的一个字节的应答位，以此类推；发送结束后发送端发送停止位，通信结束。

### 主机读取从机数据
主机发送起始位后紧跟从机地址位，从机回复低电平应答位；之后从机发送一个字节数据，主机回复低电平应答位；重复数据发送过程；主机接收最后一个字节后回复高电平非应答位，从机停止数据发送；主机发送停止位释放总线。

## 在STM32F1上的实际应用
### 引脚
- STM32F103C8有两组I2C接口，分别占用42/43(PB6/PB7)和21/22(PB10/PB11)引脚

### 线路配置
- 两引脚配置复用开漏模式
- 接上拉电阻(理论2k，实际1~10k)
- 在STM32F1系列中，I2C接入APB1总线，初始化时记得打开总线时钟
- 总线上的器件拥有唯一地址，最多127个器件(新版规范增加10位地址模式，最多达1023个器件)
----
### STM32F1事件响应
#### 主发送模式
1. 产生起始信号后，触发事件`EV5`，该事件对`SR1`寄存器的`SB`位置1，表示信号已发送，可使用`I2C_CheckEvent(I2Cx,I2C_EVENT_MASTER_MODE_SELECT)`来等待查询
2. 发送地址后，若有从机应答，则产生事件`EV6`和`EV8`，分别操作`ADDR`=1(地址已发送)和`TxE`=1(数据寄存器清空)，可使用`I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED)`查询
3. 清空`TxE`，发送数据，发送后，产生`EV8`，`TxE`=1，通过`I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_BYTE_TRANSMITTED)`查询
4. 发送数据完成后，产生停止信号，触发事件`EV8_2`，`TxE` `BTF`均置1，通讯结束

- 使能中断时，上述事件发生都会触发中断，进入同一个中断服务函数

#### 主接收模式
1. 产生起始信号后，触发事件`EV5`，该事件对`SR1`寄存器的`SB`位置1，表示信号已发送，可使用`I2C_CheckEvent(I2Cx,I2C_EVENT_MASTER_MODE_SELECT)`来等待查询
2. 发送地址后，若有从机应答，则产生事件`EV6`，`TxE`=1(数据寄存器清空)，可使用`I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED)`查询
3. 清空`TxE`，发送数据，发送后，产生`EV8`，`TxE`=1，通过`I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_BYTE_TRANSMITTED)`查询
4. 发送数据完成后，产生停止信号，触发事件`EV8_2`，`TxE` `BTF`均置1，通讯结束
----
### 寄存器综述
#### 时钟配置寄存器I2C_CCR
- 控制`SCLK`信号

|位置|名称|功能|
|--|--|--|
|15|F/S模式选择|0为标准模式，1为快速模式|
|14|DUTY|快速模式下选择时钟占空比，0为1/3，1为9/25|
|11:0|CCR|分频系数|

其中分频系数规则如下：
- 标准模式
  - $T_{high} = CCR \times T_{PCLK1}$
  - $T_{low} = CCR \times T_{PCLK1}$
- 快速模式
  - DUTY=0
    - $T_{high}=CCR \times T_{PCLK1}$
    - $T_{low}=2 \times CCR \times T_{PCLK1}$
  - DUTY=1
    - $T_{high}=9 \times CCR \times T_{PCLK1}$
    - $T_{high}=16 \times CCR \times T_{PCLK1}$

#### 控制寄存器`I2C_CRx`
#### 自身地址寄存器`I2C_OARx`
- STM32F1支持两个设备地址的模式，第一个地址支持10位模式，第二个只支持7位
- 最高位选择地址模式，0为7位，1为10位
- 9:0位用于10位地址
- 7:1位用于7位地址
#### 数据寄存器`I2C_DR`
- 占用7:0一字节，存放接收到的数据或者要发送的数据
#### 状态寄存器`I2C_SRx`
----
### 标准库开发流程
#### 1. 初始化结构体
```C
typedef struct{
    uint32_t I2C_ClockSpeed; /*!< 设置 SCL 时钟频率，此值要低于 400000*/
    uint16_t I2C_Mode; /*!< 指定工作模式，可选 I2C 模式及 SMBUS 模式 */
    uint16_t I2C_DutyCycle; /*指定时钟占空比，可选I2C_DutyCycle_2和I2C_DutyCycle_16_9*/
    uint16_t I2C_OwnAddress1; /*!< 指定自身的 I2C 设备地址1 */
    uint16_t I2C_Ack; /*!< 使能或关闭响应(一般都要使能，I2C_Ack_Enable) */
    uint16_t I2C_AcknowledgedAddress; /*!< 指定地址的长度，可为 7 位及 10 位 */
}I2C_InitTypeDef;
```
#### 2.配置I2C使用的GPIO
- 开漏模式
- 使能总线时钟APB1和APB2

#### 3.发送数据
```C
void I2C_SEND_BYTE(u8 SlaveAddr,u8 writeAddr,u8 pBuffer){ //I2C发送一个字节（从地址，内部地址，内容）
	I2C_GenerateSTART(I2C1,ENABLE); //发送开始信号
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_MODE_SELECT)); //等待完成	
	I2C_Send7bitAddress(I2C1,SlaveAddr, I2C_Direction_Transmitter); //发送从器件地址及状态（写入）
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED)); //等待完成	
	I2C_SendData(I2C1,writeAddr); //发送从器件内部寄存器地址
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_BYTE_TRANSMITTED)); //等待完成	
	I2C_SendData(I2C1,pBuffer); //发送要写入的内容
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_BYTE_TRANSMITTED)); //等待完成	
	I2C_GenerateSTOP(I2C1,ENABLE); //发送结束信号
}
```

#### 4.接收数据
```C
u8 I2C_READ_BYTE(u8 SlaveAddr,u8 readAddr){ //I2C读取一个字节
	u8 a;
	while(I2C_GetFlagStatus(I2C1,I2C_FLAG_BUSY));
	I2C_GenerateSTART(I2C1,ENABLE);
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_MODE_SELECT));
	I2C_Send7bitAddress(I2C1,SlaveAddr, I2C_Direction_Transmitter); 
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED));
	I2C_Cmd(I2C1,ENABLE);
	I2C_SendData(I2C1,readAddr);
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_BYTE_TRANSMITTED));
	I2C_GenerateSTART(I2C1,ENABLE);
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_MODE_SELECT));
	I2C_Send7bitAddress(I2C1,SlaveAddr, I2C_Direction_Receiver);
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED));
	I2C_AcknowledgeConfig(I2C1,DISABLE); //最后有一个数据时关闭应答位
	I2C_GenerateSTOP(I2C1,ENABLE);	//最后一个数据时使能停止位
	a = I2C_ReceiveData(I2C1);
	return a;
}

```