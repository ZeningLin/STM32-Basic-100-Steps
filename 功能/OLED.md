# OLED(4Pin I2C, 128*64)

- 使用SH1106芯片主控

## 区块划分
- 英文字母大小为16行8列
- 汉字大小为16行16列

- 屏幕以8\*8个像素点为一个区块，重新划分后屏幕大小为8\*16个区块，每一行称为一个页，共8页。
- 一页的宽度为8bit即1字节，写入数据控制屏幕时按照低位bit在上，高位bit在下的顺序写入
- 数据可通过字模软件获取

## 初始化与屏幕开关
```C
#define OLED0561_ADD	0x78  // OLED的I2C地址（禁止修改）
#define COM				0x00  // OLED 指令（禁止修改）
#define DAT 			0x40  // OLED 数据（禁止修改）


void OLED0561_Init (void){//OLED屏开显示初始化
	OLED_DISPLAY_OFF(); //OLED关显示
	OLED_DISPLAY_CLEAR(); //清空屏幕内容
	OLED_DISPLAY_ON(); //OLED屏初始值设置并开显示
}

void OLED_DISPLAY_ON (void){//OLED屏初始值设置并开显示
	u8 buf[28]={
	0xae,//0xae:关显示，0xaf:开显示
    0x00,0x10,//开始地址（双字节）       
	0xd5,0x80,//显示时钟频率？
	0xa8,0x3f,//复用率？
	0xd3,0x00,//显示偏移？
	0XB0,//写入页位置（0xB0~7）
	0x40,//显示开始线
	0x8d,0x14,//VCC电源
	0xa1,//设置段重新映射？
	0xc8,//COM输出方式？
	0xda,0x12,//COM输出方式？
	0x81,0xff,//对比度，指令：0x81，数据：0~255（255最高）
	0xd9,0xf1,//充电周期？
	0xdb,0x30,//VCC电压输出
	0x20,0x00,//水平寻址设置
	0xa4,//0xa4:正常显示，0xa5:整体点亮
	0xa6,//0xa6:正常显示，0xa7:反色显示
	0xaf//0xae:关显示，0xaf:开显示
	}; //
	I2C_SEND_BUFFER(OLED0561_ADD,COM,buf,28);
}

void OLED_DISPLAY_OFF (void){//OLED屏关显示
	u8 buf[3]={
		0xae,//0xae:关显示，0xaf:开显示
		0x8d,0x10,//VCC电源
	}; //
	I2C_SEND_BUFFER(OLED0561_ADD,COM,buf,3);
}

void OLED_DISPLAY_CLEAR(){
	for(t=0xB0;t<0xB7;t++){
		I2C_SEND_BYTE(OLED0561_ADD,COM,t); //页地址（从0xB0到0xB7）
		I2C_SEND_BYTE(OLED0561_ADD,COM,0x10); //起始列地址的高4位
		I2C_SEND_BYTE(OLED0561_ADD,COM,0x00); //起始列地址的低4位
		for(j=0;j<128;j++){
 			I2C_SEND_BYTE(OLED0561_ADD,DAT,0x00);
		}
	}
}
```


## 数据写入
OLED屏地址固定为`0x78`，主控芯片内操作指令的地址为`0x00`，数据存放地址为`0x40`

1. 向片内操作地址`0x00`写入页地址，范围为`0xB0`~`0xB7`
2. 向片内操作地址`0x00`写入起始列地址
   1. 注意芯片将`0x02`作为最左边的列地址，注意偏移量的转换
   2. 起始地址拆分为高低四位`H` `L`分别输入，第一次输入为`0x1H`,第二次输入为`0x1L`
3. 向数据地址`0x40`写入数据内容
   1. 经取模软件生成数组，选择纵向8点下高位模式
   2. 生成数组中，一个英文字母由16个1字节数据组成；汉字则为32个。英文数据每8个字节一组代表一页写入的内容，共2页；汉字共4页。通过循环依次将数据送入。

### 示例程序
```C
#define OLED0561_ADD	0x78  // OLED的I2C地址（禁止修改）
#define COM				0x00  // OLED 指令（禁止修改）
#define DAT 			0x40  // OLED 数据（禁止修改）

void OLED_DISPLAY_8x16(u8 x, //显示字母的页坐标（从0到7）（此处不可修改）
						u8 y, //显示字母的列坐标（从0到128）
						u16 w){ //要显示字母的编号
	u8 j,t,c=0;
	y=y+2; //因OLED屏的内置驱动芯片是从0x02列作为屏上最左一列，所以要加上偏移量
	for(t=0;t<2;t++){
		I2C_SEND_BYTE(OLED0561_ADD,COM,0xb0+x); //页地址（从0xB0到0xB7）
		I2C_SEND_BYTE(OLED0561_ADD,COM,y/16+0x10); //起始列地址的高4位
		I2C_SEND_BYTE(OLED0561_ADD,COM,y%16);	//起始列地址的低4位
		for(j=0;j<8;j++){ //整页内容填充
 			I2C_SEND_BYTE(OLED0561_ADD,DAT,ASCII_8x16[(w*16)+c-512]);//为了和ASII表对应要减512
			c++;
		}
		x++; //页地址加1
	}
}
```
