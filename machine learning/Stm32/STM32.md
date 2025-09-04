## 一、点灯实验
**GPIO**是**General Purpose Input/Output**的缩写，中文意思是**通用输入输出端口**。
**GPIO**是一种用途很广泛的引脚，可以被配置为输入或输出，用来与外部设备进行数据交换。
- **输入模式**：作为输入端口时，可以用来读取外部设备（如按键、传感器等）的电平信号。
- **输出模式**：作为输出端口时，可以用来控制外部设备（如LED灯、继电器、电机等）的开关。

**GND**是整个电路的电压参考点。所有的电压都是相对于GND来测量的。
- **电流回路通道**：电流从电源正极流向负载后，最终都要回到电源负极（即GND）。
- **保证电路正常工作**：没有GND或者GND没有接好，电路可能无法正常工作、信号不稳定、甚至损坏器件。
- 
**点灯电路图示：**
 VCC（3.3V）── LED灯 ── 电阻 ── GPIO引脚
 
**GPIO初始化**：
需要在代码中配置LED对应的引脚为**推挽输出**模式。  
一般采用如下步骤：
1. 使能GPIO端口时钟。
2. 配置GPIO为输出模式。
3. 设置输出速度（如50MHz）。

Push-Pull Output**推挽输出（Push-Pull）模式**是一种**输出方式**，最常用于控制LED、蜂鸣器等负载。
- **高电平时**，引脚被内部电路“推”到VCC电压（比如3.3V），可以向外提供电流。
- **低电平时**，引脚被“拉”到GND（0V），可以吸收电流

**点灯代码


``` 
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC,ENABLE);   //开启GPIOC时钟
	GPIO_InitTypeDef GPIO_InitStructure;					//定义GPIO初始化结构体
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;	//设置GPIO端口的模式为推挽模式
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;				//选择操作第13个引脚
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;		//选择电平变化频率
	GPIO_Init(GPIOC,&GPIO_InitStructure);
	GPIO_SetBits(GPIOC,GPIO_Pin_13);						//设置端口为高电平
	//GPIO_ResetBits(GPIOC,GPIO_Pin_13);         	//设置端口低电平

```
**RCC_APB2PeriphClockCmd命令解析：**
- **RCC**：时钟控制模块
- **APB2**：外设总线
- **PeriphClockCmd**：外设时钟命令
- **RCC_APB2Periph_GPIOC**：指定C组GPIO
- **ENABLE**：使能（打开）



## 二、GPIO通用输入输出接口

**IO端口**，全称**输入/输出端口**（Input/Output Port），就是**单片机（或微处理器、MCU）用来与外部设备（比如灯、开关、传感器等）通信和交互的“引脚”或“接口”**。

**`&=` 和 `|=`**
`|=` 叫做“**按位或赋值**”，作用是把某一位**变成1**（置位）
`&=` 叫做“**按位与赋值**”，作用是把某一位**变成0**（清零）
例子：
```
	uint32_t reg = 0b0000 0000 0000 0000 0000 0000 0000 0000; // 全部为0
	reg |= (1 << 3);  // (1<<3)表示二进制的0000...1000，即只第3位是1
	0b0000 0000 0000 0000 0000 0000 0000 1000    //操作后

```

```
	reg = 0b0000 0000 0000 0000 0000 0000 0000 1100;  // 二进制，3、2位都是1
	reg &= ~(1 << 3);  // 先左移后按位取反，这里（1<<3）的结果是0x00000008,全部取反得到
	 0xfffffff7,在进行&操作
	输出的结果是0b0000 0000 0000 0000 0000 0000 0000 0100

```

### GPIO功能简介
**多模式配置**：可配置为8种不同的输入输出模式。
**电压标准**：引脚电平通常为 0V 至 3.3V，但特别指出 **部分引脚可以容忍5V** 的电压。
**输出功能**：在输出模式下，可以控制端口输出高电平或低电平，用来驱动LED灯、控制蜂鸣器，或模拟通信协议的时序信号。
**输入功能**：在输入模式下，可以读取端口的高低电平或电压值，用于检测按键输入、接收外接模块的信号、进行ADC电压采集，或通过模拟通信协议接收数据。
![[GPIO基本结构.png]]


### GPIO位结构
![[GPIO位结构.png]]

**斯密特触发器**
- 有两个不同的电压阈值：**上阈值（VT+）和下阈值（VT−）**。
- 当输入电压上升超过上阈值时，输出才从低变高。
- 当输入电压下降低于下阈值时，输出才从高变低。
- 中间的“区间”无论输入如何小幅变化，输出都不变。

### GPIO端口模式

![[GPIO端口模式.png]]

### 开漏/推挽输出模式
![[开漏_推挽输出.png]]推挽输出通常由**两个MOS管或晶体管**组成——一个接电源（VCC），一个接地（GND），两管互锁：
- 当要输出高电平时，上管导通，下管截止，IO口输出VCC；
- 当要输出低电平时，下管导通，上管截止，IO口输出GND。


## 三、新建项目流程

- 本地新建User Library Start文件夹
- 组管理删除默认组，添加三个新建文件夹并添加文件，Start文件夹中添加md.s、.c .h文件
- lib中添加所有文件，user中添加所有文件
- 魔术棒按钮 C/C++   include path中添加三个文件夹
- define中写USE_STDPERIPH_DRIVER
- debug中选择ST_link，settting中勾选复位并执行

## 四、LED闪烁＋蜂鸣器

```
#include "stm32f10x.h"                  // Device header
#include "Delay.h"                 //延时函数
int main(void){
	
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);  
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE); //开始GPIOA/B口时钟
	
	GPIO_InitTypeDef GPIOA_InitStructure;
	GPIO_InitTypeDef GPIOB_InitStructure;              //GPIO初始化结构体
	
	GPIOA_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;   //设置模式为推免输出
	GPIOA_InitStructure.GPIO_Pin = GPIO_Pin_All ;    //初始化所有引脚
	GPIOA_InitStructure.GPIO_Speed =  GPIO_Speed_50MHz;
	
	GPIOB_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP; //设置模式为开漏输出
	GPIOB_InitStructure.GPIO_Pin = GPIO_Pin_12 ;   //初始化第12个引脚
	GPIOB_InitStructure.GPIO_Speed =  GPIO_Speed_50MHz;
	
	GPIO_Init(GPIOA,&GPIOA_InitStructure);
	GPIO_Init(GPIOB,&GPIOB_InitStructure);//初始化
	
	
	while(1){
		GPIO_Write(GPIOA,~0x0001);
		Delay_ms(100);
		GPIO_Write(GPIOA,~0x0002);
		Delay_ms(100);
		GPIO_Write(GPIOA,~0x0004);
		Delay_ms(100);
		GPIO_Write(GPIOA,~0x0008);
		Delay_ms(100);
		GPIO_Write(GPIOA,~0x0010);
		Delay_ms(100);
		GPIO_Write(GPIOA,~0x0020);
		Delay_ms(100);
		GPIO_Write(GPIOA,~0x0040);
		Delay_ms(100);
		GPIO_Write(GPIOA,~0x0080);
		Delay_ms(100);
		
		GPIO_ResetBits(GPIOB,GPIO_Pin_12);
		Delay_ms(100);
		GPIO_SetBits(GPIOB,GPIO_Pin_12);
		Delay_ms(100);
		GPIO_ResetBits(GPIOB,GPIO_Pin_12);
		Delay_ms(100);
		GPIO_SetBits(GPIOB,GPIO_Pin_12);
		Delay_ms(700);
		
	}
}


```



**`GPIO_SetBits、GPIO_ResetBits`** 这个函数用于**将某个GPIO端口的指定引脚**拉低为逻辑“0”，也就是输出低电平。
**`GPIO_Write`** 这个函数用于**一次性设置整个GPIO端口的输出电平**，可以将所有8个/16个引脚同时写成你指定的状态（高或低）。
**GPIO_WriteBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin, BitAction BitVal)**
**参数解释：**

- `GPIOx`: 指定要操作的GPIO端口。在这里 `GPIOA` 指的是A端口。
    
- `GPIO_Pin`: 指定要操作的具体引脚。在这里 `GPIO_Pin_0` 指的是0号引脚。
    
- `BitVal`: 指定要写入的电平状态。这个参数可以是两个值：
    
    - `Bit_SET`: 设置引脚为高电平 (Logic 1)。
        
    - `Bit_RESET`: 设置引脚为低电平 (Logic 0)。

**推挽输出高低电平都有驱动能力和开漏输出只有低电平有驱动能力**
延时函数是怎样工作的


### GPIO输入
按键抖动
#### **什么是按键抖动？**

按键抖动（Key Bounce 或 Switch Bounce）是指在机械按键（如普通开关、键盘按键）按下或释放时，触点并不会一次性稳定地接通或断开电路，而是会在极短时间内多次断开和接通，导致电路输出一连串高低电平的“毛刺”信号。

#### 2. **为什么会发生抖动？**

机械按键内部通常有弹片结构。当你按下按键时，弹片会从一个位置跳到另一个位置，在这个过程中，弹片由于机械振动、弹性回复等原因，会在接触点之间快速跳动几次，导致信号多次变化。这种信号在实际电路中表现为**一系列短时间的高低电平变化**，而不是理想的单一跳变。
![[按键抖动.png]]
抖动的过滤


![[Stm32/pic/image-1.png]]滤波电容有什么用
放大器？

![[Stm32/pic/image 1.png]]![[Stm32/pic/image 2.png]]