![[Pasted image 20250731193828.png]]makeBaseApp.pl -t ioc basic 
makeBaseApp.pl -i -t ioc basic
- **makeBaseApp.pl**：EPICS 自带的 Perl 脚本，用于生成应用开发的基本目录结构和模板文件。
- **-t ioc**：指定要创建的是 “ioc”（Input Output Controller）类型的应用模板。
- **basic**：你要创建的应用名，最后会生成一个叫 `basic` 的目录（或相关文件）。这条命令会**创建一个名为 `basic` 的 IOC 应用的基础框架**，生成 Makefile、src 目录、以及一些模板文件，方便你后续开发。
- - **-i**：表示_只生成应用实例（instance）相关的目录和文件_。  
    具体来说，它会在你的当前目录下，基于之前创建的 “basic” 应用，再生成一层实例化目录和文件。  
    你可以把它理解为“为 basic 这个 IOC 应用创建一个新实例的配置环境”。
- 其它部分和上面一样。

#### 总结一句话：

> 这条命令**只会为 `basic` 应用生成应用实例相关的目录结构和配置文件**（比如 IOC 配置、启动脚本等），而不会再生成应用代码的框架（那些文件已经由第一条命令生成了）。
![[Pasted image 20250731200842.png]]- **make**  
    是 Linux 下的自动化构建工具，会读取当前目录下的 `Makefile` 文件，按里面的规则执行任务。
    
- **clean**  
    是 `Makefile` 里的一个目标，意思是**删除所有由编译过程生成的临时文件、目标文件（.o）、可执行文件等**，让目录恢复到“未编译”状态。
    
- **uninstall**  
    也是 `Makefile` 里的目标，意思是**把之前通过 `make install` 安装到系统或指定目录的程序或库全部删除**。
make clean
make uninstall
# 1. **IOC 和客户端之间怎么通信？**

## **通信机制：Channel Access（CA）协议**

- 在 EPICS 体系里，**IOC 和客户端之间通信，主要靠 EPICS 自己的一套网络协议，叫做 Channel Access（CA）**。
- **IOC 作为服务器**，**客户端（如 `caget`、`caput`、OPI 软件）作为客户端**，通过网络来交换数据。

---

### **通信流程示意**

1. **IOC 启动后，监听一个或多个端口（通常是 5064/UDP 和 TCP）。**
    
2. **客户端（比如你在 shell 输入 `caget NUMBER`）通过网络广播或单播，去“寻找”名字叫 `NUMBER` 的 PV。**
    
3. **如果有 IOC 发布了这个 PV，客户端就可以直接跟它“对话”：**
    
    - 读（`caget`）就是请求当前值
    - 写（`caput`）就是发送新值
4. **实际通信的底层用的是标准的网络套接字（socket），数据在网络中以 EPICS CA 协议格式传递。**
    

---

### **数据交互举例**

- **`caget NUMBER` 的过程：**
    
    1. 客户端向网络发送“谁有 NUMBER 这个 PV？”
    2. IOC 收到后回应“我有！”
    3. 然后客户端与 IOC 直接建立 TCP 连接，读取变量值。
- **`caput NUMBER 123` 的过程：**
    
    1. 客户端找到发布 `NUMBER` 的 IOC
    2. 建立连接后，发送写入命令，IOC 修改其内部变量并反馈

---

# 2. **“发布”PV 是什么意思？**

## **发布的意思：**

- 当你**写好 .db 文件、st.cmd 脚本，启动 IOC 后，IOC 会把所有加载的 PV “注册/公告” 到 EPICS 网络中**。
- 这相当于告诉网络里所有 EPICS 客户端：“我这里有这些变量，谁需要可以来找我！”

## **技术细节**

- IOC 在启动时会维护一个“PV 名字—内存地址”列表
- 当客户端请求某个 PV 时，IOC 查找自己是否有这个 PV，有就响应，没有就忽略
- PV 的所有“读、写”操作都必须通过 IOC 来完成

---

# 3. **IOC 与客户端之间的关系**

- IOC 就像“广播站”，不断对外发布自己有的“频道（PV）”
- 客户端就像“收音机”，你调到某个频道（PV 名），如果广播站（IOC）在，你就能收到
- 只要 IOC 还在运行，这些 PV 就一直“在线”对外提供服务

### asyn具体流程：

1. PV `ROOM:TEMP` 定时扫描到，需要温度 → 发请求给 asyn → 马上返回
2. PV `ROOM:HUM` 需要湿度 → 也发请求给 asyn → 马上返回
3. PV `ROOM:PRES` 需要压力 → 也发请求给 asyn → 马上返回
4. asyn 线程里排队（或并发）去串口轮流访问温度计、湿度计、压力计
5. 谁先拿到数据，就“推送”回来，PV 自动刷新
`RELEASE` 文件在 EPICS 构建系统中扮演着“模块寻路器”的角色。它的主要目的是**定义 EPICS 模块、外部依赖库以及其他 EPICS 核心组件的路径**。构建系统会读取这些路径，以便在编译你的 IOC 或其他 EPICS 应用程序时知道去哪里找到所需的头文件、库文件和源文件。


初步使用Asyn模块
外部写 → record → asyn模块转发 → 你的驱动 → 设备”
外部读 → record → asyn模块要数据 → 你的驱动 → 设备，最后回传到客户端”


Ioc线程包含哪些
## **（1）主线程（Main Thread/Program Thread）**

- 作用：负责 IOC 程序的初始化、启动、管理其它线程，处理命令行（epics shell）输入，做一些管理调度。
- 举例：你启动 `basic st.cmd`，看到 `epics>` 提示符，这就是主线程的交互界面。

---

## **（2）定时扫描线程（Scan Tasks）**

- 作用：负责**周期性地扫描所有 PV（record）**，看有没有需要处理的输入/输出，比如“每秒读取一次温度”。
- EPICS 会根据不同的扫描周期（1秒、2秒、0.1秒等）创建多个 scan 线程，分别服务于不同优先级和周期的 PV。
- 举例：你有5个 PV 设置 `field(SCAN, "1 second")`，这些 PV 就由“1秒扫描线程”每隔1秒扫描一次。

---

## **（3）I/O Interrupt 线程（I/O Interrupt Threads）**

- 作用：负责处理**被硬件/驱动主动通知的“数据变更”**（比如外部硬件触发中断或者asyn驱动发来回调）。
- 举例：温度传感器主动报告温度变化，而不是 IOC 定期去“轮询”，这个“主动推送”事件就由 I/O Interrupt 线程响应。

---

## **（4）驱动/外部库线程（Driver/Asyn Threads）**

- 作用：当你用 asyn、streamDevice 或某些设备库时，会有**专门的线程负责和底层硬件通讯**（如串口、Modbus、TCP/IP等）。
- 这些线程维护自己的队列、定时器、状态机，把慢操作“异步”托管，不阻塞主线程。
- 举例：asyn 创建的串口通讯线程，负责和实际的串口设备对话，IOC 只管发请求，不等待结果。

---

## **（5）事件/消息处理线程（Event Tasks）**

- 作用：响应外部或内部的“事件”——比如软件触发的 PV 更新、报警、联动等。
- 举例：当某个 PV 值越限时，发出报警消息，event 线程负责通知相关系统。

---

## **（6）数据库维护线程（Database Maintenance Thread）**

- 作用：管理 record 的状态变更、数据同步、后台清理等。
- 举例：数据库垃圾回收、定时刷新某些缓存值。

---

## **（7）网络通信线程（Channel Access Threads）**

- 作用：负责通过网络接收客户端请求（如 caget、caput），处理 CA 协议包，返回数据。
- 举例：有客户端用 `caget` 读 PV，CA 线程负责接收请求、查找 PV、打包发送返回结果。


假设你有一个温度传感器（串口），PV叫 `ROOM:TEMP`，客户端不断 caget，偶尔还要 caput 设置参数：

1. **定时扫描线程**每秒钟触发一次，要求 asyn去读温度值
2. **asyn驱动线程**负责真正与传感器通讯，把最新温度反馈回来
3. **网络通信线程**响应客户端的 `caget ROOM:TEMP` 请求，查询 PV，返回最新值
4. 有硬件主动推送（如温度报警），**I/O interrupt线程**直接处理这个“推送事件”，不等下一次扫描

# 1. **定时扫描线程是什么？**

- EPICS IOC 为每种 SCAN 周期（比如 1s、2s、0.1s）自动创建“定时扫描线程”。
- 这些线程会定时“唤醒”挂在自己名下的 record（PV），让它们进行处理（通常是读/写操作）。

---

# 2. **asyn模块的角色**

- asyn 负责和底层的慢设备（如串口、网口、PLC等）做数据交互，提供统一、异步、线程安全的接口。
- 你的 record 只要声明 `DTYP` 为 asyn 支持的类型，INP/OUT 连接 asyn 的 port，**所有IO操作都由 asyn驱动实现**。

---

# 3. **沟通流程举例**

以“读取温度”为例：

1. 你的 PV（比如 `ROOM:TEMP`）写在 .db 文件里，设定：
    
    scss
    
    复制代码
    
    `field(SCAN, "1 second") field(DTYP, "asynFloat64") field(INP,  "@asyn(port1 0)")`
    
2. **定时扫描线程**每隔1秒会“唤醒”一次 `ROOM:TEMP` 这个 PV，要求它执行一次“读取操作”。
3. 由于 DTYP/INP 用了 asyn，record 的 process 方法就会通过 asyn 接口，去找“port1”对应的 asyn驱动。
4. **asyn驱动线程**和底层硬件通信，拿到最新温度数据，再把数据返回给 PV。
5. PV 更新自己的 VAL 字段，供客户端读取。
### 类定义部分

C++

```
class MyDriver:public asynPortDriver{
```

- **解释：** `class` 是 C++ 中定义“类”的关键字。类就像一个蓝图或模板，用于创建对象。
    
- **`MyDriver`**：这是我们定义的这个类的名字，就像你给一个蓝图起的名字。
    
- **`:public asynPortDriver`**：这表示 `MyDriver` 这个类**继承**自 `asynPortDriver` 这个类。继承意味着 `MyDriver` 自动拥有了 `asynPortDriver` 的所有功能和特性，我们只需要添加或修改我们自己的部分即可。这是一种非常重要的面向对象编程思想。
    

#### 公有成员（`public`）

C++

```
public:
MyDriver(const char *portName);
virtual asynStatus writeInt32(asynUser *pasynUser, epicsInt32 value);
virtual asynStatus readInt32(asynUser *pasynUser, epicsInt32 *value);
```

- **`public:`**：这表示下面的成员（函数和变量）可以在类的外部被访问。
    
- **`MyDriver(const char *portName);`**：这是这个类的**构造函数**。它的名字和类名一模一样，当一个新的 `MyDriver` 对象被创建时，它就会被自动调用。`const char *portName` 是它的一个参数，用于给这个驱动起一个名字。
    
- **`virtual asynStatus writeInt32(...)`**：这是一个**虚函数**。`virtual` 关键字表示我们正在重写父类 `asynPortDriver` 中同名的函数。这个函数负责处理对我们驱动的 32 位整数（`Int32`）的**写入**请求。
    
    - `asynStatus` 是函数返回值的类型，它表示操作是成功还是失败。
        
- **`virtual asynStatus readInt32(...)`**：和上面类似，这个虚函数负责处理对 32 位整数的**读取**请求。
    

#### 私有成员（`private`）

C++

```
private:
int myNumber;
int numberIndex_;
```

- **`private:`**：这表示下面的成员只能在类的内部被访问，外部无法直接访问。
    
- **`int myNumber;`**：这是一个普通的整数变量，用于在类内部存储一个 32 位整数值。
    
- **`int numberIndex_;`**：这也是一个整数变量。它将用于存储一个索引，这个索引是用来在我们的驱动中唯一地标识一个参数。
    

### 3. 构造函数的实现

C++

```
MyDriver::MyDriver(const char *portName)
:asynPortDriver(portName,
1,
asynInt32Mask | asynDrvUserMask,
asynInt32Mask,
0,1,0,0)
{
    createParam("NUMBER", asynParamInt32, &numberIndex_);
    setIntegerParam(numberIndex_,0);
    callParamCallbacks();
}
```

- **`MyDriver::MyDriver(...)`**：`::` 符号表示我们正在实现 `MyDriver` 类中的 `MyDriver` 函数。
    
- **`:asynPortDriver(...)`**：冒号后面的部分是初始化列表。它调用了父类 `asynPortDriver` 的构造函数，并传递了一些参数来配置我们的驱动。
    
    - `portName`：给驱动起的名字。
        
    - `1`：表示这个驱动只有一个地址（可以想象成一个设备）。
        
    - `asynInt32Mask | asynDrvUserMask`：这是一种位掩码，表示我们的驱动支持两种接口：32位整数 (`asynInt32`) 和用户驱动接口 (`asynDrvUser`)。
        
- **`createParam("NUMBER", asynParamInt32, &numberIndex_);`**：
    
    - **`createParam`** 是父类 `asynPortDriver` 提供的一个方法。
        
    - 我们用它来**创建一个新的参数**。这个参数的名字是 `"NUMBER"`。
        
    - `asynParamInt32` 表示它的数据类型是 32 位整数。
        
    - `&numberIndex_` 表示将这个新创建参数的索引**存入**我们之前定义的 `numberIndex_` 变量中，方便以后使用。
        
- **`setIntegerParam(numberIndex_,0);`**：
    
    - 这是父类提供的另一个方法，用于**设置**一个整数参数的值。
        
    - 我们把 `numberIndex_` 对应的参数（也就是 "NUMBER"）的初始值设置为 `0`。
        
- **`callParamCallbacks();`**：
    
    - 这个方法会通知所有监听这个驱动的 EPICS 记录（Record），说我们已经设置了参数的初始值。
        

### 4. 读写函数的实现

C++

```
asynStatus MyDriver::writeInt32(...)
{
    int function = pasynUser->reason;
    if(function == numberIndex_){
        printf("write data \n");
        return asynSuccess;
    }
    return asynPortDriver::writeInt32(pasynUser, value);
}

asynStatus MyDriver::readInt32(...)
{
    int function = pasynUser->reason;
    if(function == numberIndex_){
        printf("read data \n");
        return asynSuccess;
    }
    return asynPortDriver::readInt32(pasynUser,value);
}
```

- **`int function = pasynUser->reason;`**：`pasynUser` 是一个指向 `asynUser` 结构体的指针，它包含了这次读写请求的各种信息。`reason` 就是一个整数，告诉我们这次请求是针对哪个参数的。
    
- **`if(function == numberIndex_)`**：
    
    - 这里我们进行判断。如果请求的参数索引（`function`）和我们创建的 "NUMBER" 参数的索引（`numberIndex_`）相同，就执行 `if` 语句里的代码。
        
    - **`printf(...)`**：目前代码只是简单地打印一条信息，告诉你“写入数据”或“读取数据”的操作正在进行。
        
    - **`return asynSuccess;`**：返回一个 `asynSuccess` 值，告诉 EPICS 这个请求已经成功处理了。
        
- **`return asynPortDriver::...`**：
    
    - 如果 `if` 的条件不成立（请求的不是我们的 "NUMBER" 参数），我们就调用父类 `asynPortDriver` 的同名函数来处理它。这是一种很好的编程习惯。
        

### 5. C接口部分

C++

```
extern "C"{
// ...
epicsExportRegistrar(MyDriverRegister);
}
```

- **`extern "C"`**：这是一种特殊的语法，用于告诉 C++ 编译器，这部分代码应该以 C 语言的规则来编译和链接。这是因为 EPICS 的核心部分是用 C 语言写的，这样可以确保 C++ 和 C 之间的代码能够顺利地互相调用。
    
- **`int MyDriverConfigure(...)`**：这是一个函数，它的作用是创建一个新的 `MyDriver` 对象。在 EPICS 的配置文件（`st.cmd`）中，你会使用 `MyDriverConfigure("端口名")` 来调用它。
    
- **`static const iocshArg...`**：这几行代码是 EPICS 的标准“样板代码”，用于**将 `MyDriverConfigure` 函数注册到 EPICS 的命令行工具 `iocsh` 中**。这样，当 IOC 启动后，你就可以在命令行里直接输入 `MyDriverConfigure "端口名"` 来调用它。
    
- **`epicsExportRegistrar(MyDriverRegister);`**：这是EPICS的另一个标准宏。它确保 `MyDriverRegister` 这个注册函数在IOC启动时能够被自动找到并执行，从而完成整个注册过程。















```
record(ai, "PRESSURE:1") {
    field(DTYP, "asynFloat64")
    field(INP,  "@asyn($(PORT))CH0_DATA")
    field(SCAN, "1 second")
}
```

1. **record(ai, "PRESSURE:1")**
    
    - 定义一个类型为`ai`（模拟量输入）的记录，名字叫`PRESSURE:1`。
    - 你可以通过 EPICS 的 `caget PRESSURE:1` 来读这个PV的值。
2. **field(DTYP, "asynFloat64")**
    
    - 指定这个记录使用 asyn 驱动（asynFloat64 类型，表示64位浮点数）来做底层接口。
3. **field(INP, "@asyn($(PORT))CH0_DATA")**
    
    - 指定底层输入接口。这里的 `@asyn($(PORT))CH0_DATA` 是 EPICS asyn 驱动的特殊语法，意思是：
        - 使用 asyn 驱动，通过端口名（比如 `PORT=TEST`）和参数 `CH0_DATA`，去底层采集数据。
        - `CH0_DATA` 就是在你的 C++ 驱动里用 `createParam("CH0_DATA", ...)` 注册的参数。
4. **field(SCAN, "1 second")**
    
    - 这个记录每1秒钟被EPICS自动扫描一次，自动触发一次采集。
    - 换句话说，每秒钟你的 `readFloat64` 函数就会被调用一次，采集一次硬件数据。
**总体来说**，这行代码告诉 EPICS：“当这个记录被处理时，通过名为 `$(PORT)` 的 asyn 端口，去读取 `CH0_DATA` 参数的值”`"PRESSURE:1"` 的记录，它会每隔一秒钟主动向其驱动（由 `INP` 字段指定）请求一次数据。当驱动收到请求时，它会从 NI DAQmx 硬件的通道 `ai0` 读取当前的电压值，并将这个值返回给 EPICS，最终更新到 `"PRESSURE:1"` 这个 PV 上 。

```
class DaqDriver : public asynPortDriver {
public:
    DaqDriver(const char *portName);
    ~DaqDriver();
    virtual asynStatus readFloat64(asynUser *pasynUser, epicsFloat64 *value);
private:
    double daqData_;       // 存储采集到的数据
    int daqDataIndex_;     // EPICS参数索引
    TaskHandle taskhandle_;// NI DAQmx任务句柄
};

```
**作用**：

- 定义了一个 EPICS 驱动类 `DaqDriver`，继承自 asynPortDriver。
- `daqData_` 用于保存当前采集值。
- `daqDataIndex_` 绑定到 asyn 参数。
- `taskhandle_` 用于操作DAQ硬件。


```
DaqDriver::DaqDriver(const char *portName)
: asynPortDriver(portName, 1, asynFloat64Mask | asynDrvUserMask, asynFloat64Mask, 0,1,0,0)
{
    createParam("CH0_DATA", asynParamFloat64, &daqDataIndex_);
    daqData_ = -1;
    printf("Init Daq Here:\n");
    DAQmxCreateTask("", &taskhandle_);
    DAQmxCreateAIVoltageChan(taskhandle_, "NIPCI6251/ai0", "", DAQmx_Val_Cfg_Default, -10.0, 10.0, DAQmx_Val_Volts, NULL);
}

DaqDriver::~DaqDriver() {
    printf("Stop DAQ Here\n");
    DAQmxStopTask(taskhandle_);
    DAQmxClearTask(taskhandle_);
}

```
**作用**：

- **构造函数**：
    - 初始化 asyn 驱动基类。
    - 注册一个参数 `"CH0_DATA"`，用于和 EPICS record 绑定。
    - 创建 NI DAQmx 任务并配置通道。
- **析构函数**：
    - 程序退出时释放DAQ硬件资源，停止并清除任务

```

asynStatus DaqDriver::readFloat64(asynUser *pasynUser, epicsFloat64 *value) {
    int function = pasynUser->reason;
    if (function == daqDataIndex_) {
        DAQmxReadAnalogScalarF64(taskhandle_, 10.0, &daqData_, 0);
        *value = daqData_;
        printf("read daqData: %f\n", *value);
        return asynSuccess;
    }
    return asynPortDriver::readFloat64(pasynUser, value);
}
```

**作用**：

- 每次 EPICS 记录被扫描时（如`SCAN="1 second"`），就会调用这个函数。
- 调用 NI DAQmx API 从硬件读取一个模拟量（电压等），赋值到 `daqData_`，并返回给 EPICS。
- 只对 `"CH0_DATA"` 这个参数响应。


```
extern "C" {
int DaqDriverConfigure(const char *portName) {
    new DaqDriver(portName);
    return(asynSuccess);
}
static const iocshArg initArg0 = {"portName", iocshArgString};
static const iocshArg *const initArgs[] = {&initArg0};
static const iocshFuncDef initFuncDef = {"DaqDriverConfigure", 1, initArgs};
static void initCallFunc(const iocshArgBuf *args) {
    DaqDriverConfigure(args[0].sval);
}
void daqDriverRegister(void) {
    iocshRegister(&initFuncDef, initCallFunc);
}
epicsExportRegistrar(daqDriverRegister);
}
```

- 定义并注册 IOC shell 命令 `DaqDriverConfigure`，可在 `st.cmd` 里调用。
- `epicsExportRegistrar(daqDriverRegister)` 将注册函数导出给 EPICS 系统，让 `.dbd` 文件可以 `registrar(daqDriverRegister)`。


