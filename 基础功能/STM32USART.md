# USART

**未完善，涉及中断嵌套等知识**

## 寄存器标志位
- FLAG的状态有两种，`SET`和`RESET`
- `FlagStatus USART_GetFlagStatus(USART_TypeDef* USARTx, u16 USART_FLAG)`查询标志位，常用标志位如下

|FLAG|描述|
|--|--|
|USART_FLAG_TC|发送完成标志位，每发送一个符号最好判断一下才能继续|
|USART_FLAG_TXE|发送寄存器空标志位|
|USART_FLAG_RXNE|接收寄存器非空标志位|
|USART_FLAG_PE|奇偶校验错标志位|

## USART中断标志位
- `ITStatus USART_GetITStatus(USART_TypeDef* USARTx, u16 USART_IT)`查询中断标志位

|FLAG|描述|
|--|--|
|USART_IT_PE|奇偶错中断|
|USART_IT_TXE|发送中断|
|USART_IT_TC|发送完中断|
|USART_IT_RXNE|接收中断|

- 串口1中断服务程序为`void USART1_IRQHandler()`

## 数据发送
- `void USART_SendData(USART_TypeDef* USARTx, u8 Data)`用于发送单个数据，`USARTx`为串口号，x可为123，`Data`为待传送数据
- `printf()`用于发送字符串，在单片机中`printf`默认为USART1输出，可通过宏定义`#define USART_n USARTx`修改使用的串口


## 数据接收
- 两种方式，查询和中断
  - **查询**通过调用查询标志位函数查询`USART_FLAG_RXNE`标志位判断是否有内容，之后调用`USART_ReceiveData()`读取数据
  - **中断**设置中断标志位，接受结束后进入中断服务函数，在函数中调用`USART_ReceiveData()`读取数据
- `u8 USART_ReceiveData(USART_TypeDef* USARTx)`用于接收单个数据，返回值为接收的数据值

### 查询方式
```C
if(USART_GetFlagStatus(USART1,USART_FLAG_RXNE) != RESET){
			receive =USART_ReceiveData(USART1);
            //Action
		}
USART_RX_STA|=0x8000;//Reset Flag
```