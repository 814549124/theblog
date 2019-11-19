# HID 驱动

## What's New in HID



## HID 概念的介绍



### HID的经历

HID的出现开始于覆盖在USB之上的设备类. 那时的目标是定义PS/2的代替物, 并创建覆盖在USB之上的接口, 可创造类似于**键盘**,**鼠标**,**游戏控制器**的HID设备的驱动. 在HID之前, 设备不得不严格地遵循鼠标键盘已定义的协议. 硬件改革迫使对已存在协议上的数据的使用或者需要自己驱动的非标准硬件的生产不堪负重.  HID的看法开始于寻找一种能为这些设备提供基础支持的方法, 并且允许硬件卖家提供可扩展,标准化,易于编程的接口.

今天, HID设备包含: **字符显示器**,**二维码阅读器**,**话筒/听筒**,**辅助显示器**,**传感器**等. 还有, 许多硬件卖方为其专有设备而使用HID.

HID虽开始于USB,但它被设计在`a bus agnostic fashion`里. 它初始为低延迟, 低带宽但高柔韧性的设备而设计, 并且传输速率被底层传输所指定. 该在USB之上的HID的说明书在1990末被批准, 并且在之后支持额外的传输方式. 今天HID有多种传输的标准协议, 并且以下协议支持在win8里的HID:

- USB
- Bluetooth
- Bluetooth LE
- I²C

卖方特殊的传输方式也可以通过卖方特殊的传输驱动所支持. 更多细节提供在以下章节



### HID 概念

包括: **Report Descriptor** 报告描述符, *Report* 报告.

报告, 是实际的数据块, 可在设备和软件客户端交换. 报告描述符描述每个数据块的格式和含义.

#### Reports

这里有三种报告: **Input Reports**,**Output Report**,**Feature Report**特征报告.

- Input Report 可读报告

  典型地在control的状态改变时,从HID设备发向程序的数据块

- Output Report 可写报告

  从程序发现HID设备的数据块, 例如键盘上的LED灯

- Feature Report

  可读/写的数据块, 典型地关联与配置信息

定义在报告描述符里的每种顶级Collection能包含0或多个报告.

#### Usage Tables 用法表

HID工作组发布了一集文档, 组建HID用法表. 这高效地描述的HID设备可以做什么. 这HID用法表包含了一列用法的描述. 一个Usage用法提供信息给程序开发者关于在报告描述符里的Item的用法和含义. 例如, 有个Usage用法定义了鼠标的左按钮. 这个描述符能定义了在一个report报表里程序能找到鼠标左键状态的地方. 用法表可以分开到几个命名空间, 称为**Usage Page**用法页. 每个用法页描述一集相关用法来帮助组织文档. 用法页和用法的组成部分定义了**Usage ID**, 唯一地表示用法表里的用法



### HID程序编程接口API

HID API的三类接口: 设备发现和设置, 数据转移, 报告的创建和解释



#### 设备发现和设置

以下列表标识着程序能使用的HID API: 标识HID设备的属性, 与该设备建立连. 额外的, 程序可以使用这些API来标识顶级集.

- HidD_GetAttributes
- HidD_GetHidGuid
- HidD_GetIndexedString
- HidD_GetManufactureString
- HidD_GetPhysicalDescriptor
- HidD_GetPreparsedData
- HidD_GetProductString
- HidD_GetSerialNumberString
- HidD_GetNumInputBuffers
- HidD_SetNumInputBuffers

#### 数据转移

以下列表标识着程序在app和已选择的设备之间能使用它向前向后地移动数据的HID API

- HidD_GetInputReport
- HidD_SetFeature
- HidD_SetOutputReport
- ReadFile
- WriteFile



#### 报告的创建和解释

如果你在为你的硬件写HID程序, 那么你需要知道你设备发布的每种报告的格式和大小. 在这种情况下, 你的程序要转换读/写报告缓存为struct然后消费数据

然而,如果你写交流所有揭露通用功能(如, 播放APP在播放按钮按下时需要检测)的设备的HID程序, 你可能不知道HID报告的大小和格式. 程序的种类理解已知的顶级集合已知的用法.

为了解释接收至设备的报告,或者创建要发送的报告, 程序需要调节报告描述符,为了:

特定的用法是否被定位在报告里

特定用法被定位在报告里的哪里 

该报告里的值的单位. 

这里是HID解析是必须的地方. 为了驱动和程序能使用这些,Windows提供了HID解析器. 这里解析器揭露一集API,能用来发现被设备支持的用法的类型, 决定在报告里用法的状况, 或者构建报告来改变在设备里用法的状况.

以下列表标识HID解析器API

- HidP_GetButtonCaps
- HidP_GetButtons
- HidP_GetButtonsEx
- HidP_GetCaps
- HidP_GetData
- HidP_GetExtendedAttributes
- HidP_GetLinkCollectionNodes
- HidP_GetScaledUsageValue
- HidP_GetSpecificButtonCaps
- HidP_GetSpecificValueCaps
- HidP_GetUsage
- HidP_GetUsageEx
- HidP_GetUsageValue
- HidP_GetUsageValueArray
- HidP_GetValueCaps
- HidP_InitializeReportForID
- HidP_IsSameUsageAndPage
- HidP_MaxDataListLength
- HidP_MaxUsageListLength
- HidP_SetButtons
- HidP_SetData
- HidP_SetScaledUsageValue
- HidP_SetUsages
- HidP_SetUsageValue
- HidP_SetUsageValueArray
- HidP_UnsetButtons
- HidP_UnsetUsages
- HidP_UsageAndPageListDifference
- HidP_UsageListDifference