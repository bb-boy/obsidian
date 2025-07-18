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

![[image-1.png]]![[image-2.png]]上拉模式，下拉模式
斯密特触发器
![[image-3.png]]

开漏，关闭模式