---
title: HID PART 2
date: 2019-11-19 08:20:24
tags: DRIVER
---



# HID -> HID 体系结构

HID驱动栈的体系结构被构建在`hidclass.sys`类驱动上. 客户和传输迷你驱动(transport minidriver)从用户模式或内核模式访问该驱动



## HID类驱动

系统提供的有: WDM功能驱动和HID设置启动类的bus驱动. HID类驱动的可执行组件是`hidclass.sys`.  它是HID客户端和不同传输协议之间的粘合剂. 这可以把HID客户端写在独立于传输(transport)里. 抽象的等级,在一个新的,第三方的传输协议被引用时,让客户继续工作.

下面是体系结构表示图

![](\asset\2019-11-19-hid-intro-simple.png)

包含:

- HID Client

  标识Windows和第三方的client和他们的interface.

- HID Class driver

  `hidclass.sys`可执行文件

- HID Transport Minidriver

  标识Windows和第三方的transport和他们的interface

这里是普通的HID client和transport的设备栈示例图

![](\asset\2019-11-19-hid-device-stacks-generic.png)

这里是USB版的HID键盘和鼠标collection的设备栈示例图

![](\asset\2019-11-19-hid-device-stacks.png)



## HID Client 客户端

它是驱动,windows服务或者程序, 交流与`HIDClass.sys`,并经常代表设备的某类型(如: 传感器,键盘,鼠标). 它通过硬件ID(hardware ID)和特定的HID集(HID Collection)标识着设备, 并通过以下方式与HID Collection交流.

用户模式的驱动和程序,内核模式驱动,  可根据以下来操作HID集

- 用户模式驱动和程序使用HIDClass支持的功能(HidD_Xxx) 来获取关于HID集的信息
- 内核模式驱动,用户模式驱动和程序,则使用`HID parsing`提供的功能(HidP_Xxx), 内核模式驱动使用HID类驱动`IOCTL`来处理HID报告

|          | 驱动                      | 程序     |
| -------- | ------------------------- | -------- |
| 用户模式 | HidD_Xxx                  | HidP_Xxx |
| 内核模式 | HidD_Xxx OR IOCTL_HID_xxx | N/A      |

## HID传输驱动

它用来使用HID微驱动来访问硬件读设备(hardware input device). 微驱动抽象设备的设备等级的操作. 微驱动绑定其操作到HID类驱动, 通过注册HID类驱动来实现. HID类驱动通过调用微驱动的routine来与微驱动进行交流. HID 微驱动, 轮流地,从驱动栈到底层bus/port驱动器发送交流信息.



# 打开HID集

该章节描述HID客户端如何交流与HID类驱动来操作设备的HID集.

HID客户端可以操作在以下模式:

- 用户模式的程序/驱动
- 内核模式的驱动

下面章节指明了HID客户端如何使用介绍在上述的模式来交流`HIDClass`

该章节描述用户模式程序和内核驱动如何操作HID集

常规上,用户模式程序做以下事情:

- 调用设备安装函数(SetupDiXxx函数)来寻找和标识HID集
- 调用`CreateFile`打开在HID集上的文件
- 调用`HidD_Xxx`来获取HID集的预解析数据和关于该HID集的信息
- 调用`ReadFile`来去读报告和`WriteFile`来发送报告
- 调用`HidP_Xxx`来中断HID报告

常规上,内核模式驱动做:

- 寻找并标识一个HID集

  如果该驱动是一个function/filter驱动, 那么它已经附加到HID集的设备栈. 不过, 如果该驱动没有附件上去,该驱动可以使用即插即用通知

- 使用`IRP_MJ_CREATE`请求来打开一个HID集

- 使用IOCTL_HID_Xxx请求来获取HID集的预解析数据和集的信息.

- 使用`IRP_MJ_READ`请求来读取报告并且使用`IRP_MJ_WRITE`请求发送报告

- 调用`HidP_Xxx` 来中断HID报告



# 寻找和打开HID集

## 用户模式程序

Windows提供设备安装功能来寻找和标识`HIDClass`设备, 提供其他win32函数来初始化和连接HID集

在用户模式程序加载后, 它做了下面连续地操作:

- 调用`HidD_GetHidGuid`来获取HIDClass设备的系统定义的GUID.
- 调用`SetupDiGetClassDevs`来获取设备信息集的Handle, 设备信息集描述了当前安装在系统的所有HID集的设备接口. 该程序需指定`DIGCF_PRESENT`和`DIGCF_DEVICEINTERFACE`Flag参数, 传递给`SetupDiGetClassDevs`
- 重复地调用`SetupDiEnumDeviceInterfaces`,来获取所有有效的接口信息
- 调用`SetupDiGetDeviceInterfaceDetail`来格式每个HID集的接口信息为` SP_INTERFACE_DEVICE_DETAIL_DATA`格式.该结构的`DevicePath`成员包含用户模式名称, 可用来使用在`CreateFile`函数上来获取HID集的文件Handle.
- 调用`CreateFile`来获取一个HID集的一个文件handle 



## 内核模式驱动

如果该驱动是一个function/filter驱动, 那么它已经附件一个设备对象到HID集设备栈了. 该设备仅仅使用创建请求来打开该设备.

如果该驱动不是function/filter驱动,那么它典型的使用即插即用通知来寻找一个HID集. 在找到集时, 该驱动可以使用创建请求来打开该驱动.



# 强制一个HID集的安全读

该章节描述如何用户模式程序和内核模式驱动能强制一个顶层HID集的安全读.

如果集的安全读为启动, 只有"已信任"的客户端(SeTcbPrivilege权限)能从集的已打开文件里获取数据. 内核模式驱动默认拥有该权限, 但用户模式程序没有. 那么如何获取其权限呢?在SDK文档的authorization(授权)里.

"已信任"的客户端可以使用`IOCTL_HID_ENABLE_SECURE_READ`和`IOCTL_HID_DISABLE_SECURE_READ`请求来启动/禁用安全读. 如果没有权限的客户端使用了该请求,那么该驱动会返回`STATUS_PRIVILEGE_NOT_HELD`的状态值.

启动/禁用中的安全读在下面方式里工作:

- HID类驱动为每个HID集的已打开的文件handle维持着一个文件方式的安全读计数. HID类驱动也维持着集的安全读计数, 是文件式安全读计数的总数. 集的安全读计数在集创建时初始化为0, 文件的安全读计数在文件创建时初始化为0
- 在HID类驱动接受到文件的启动请求时, 文件的安全读计数加1(并且集的安全读计数加1)
- 在HID类驱动接收到文件的禁用请求时:
  - 如果文件的安全读计数大于0, 则减1
  - 如果等于0, 则不改变安全读
- 如果集的安全读计数大于0, 那么HID类驱动强制集的安全读. 否则,不强制.
- 客户端可使用禁用请求来取消对应的启动请求. 然而,如果客户端不做,在它在集的文件里处理`IRP_MJ_CLOSE`请求时, 该HID类驱动适当的减少安全读计数. 当驱动处理关闭请求时,它的安全读计数减1.



# 获取预解析数据

该章节秒速用户模式程序和内核模式驱动如何获取HID集的预解析数据, 它是一个不透明的结构描述着集的HID报告

## 用户模式程序

在调用任何HIDClass功能之前, 用户模式程序必须获取集的预解析数据. 程序可以在打开设备上文件期间一样长的时间段里,保持集的预解析数据的访问

在打开集的文件之后, 程序调用`HidD_GetPreparsedData`在routine-分配的buffer里返回集的预解析数据.

程序需调用`HidD_FreePreparsedData`, 在程序不在请求对集的访问时.

## 内核模式驱动

在内核模式驱动打开集之后, 驱动在以下方式获取集的预解析数据:

- 获取集的预解析数据的长度
- 获取集的预解析数据

为了决定预解析的长度, 驱动是有一个`IOCTL_HID_GET_COLLECTION_INFORMATION`的请求. 该请求返回`HID_COLLECTION_INFORMATION`结构. 该结构的DescriptorSize成员指明其集的预解析数据的字节大小. 该驱动必须从能保存预解析数据的大小的nonpaged池中分配buffer.

在分配之后, 驱动使用`IOCTL_HID_GET_COLLECTION_DESCRIPTOR` 请求来获取预解析数据.

在获取预解析数据后, 该驱动可以用它来使用HidP_Xxx来获取HID集的能力的信息, 或从HID报告里提取控制数据



# 获取集的信息

该章节寻找获取用户模式程序和内核模式驱动操作HID集的信息

在程序或驱动连接一个HID集之后, 他能获取以下信息:

- 集的能力
- 按钮能力组和值能力组, 描述集支持的按钮和值的能力
- 链接集组(Link colletion array), 貌似link集的内部组织行为

这些信息包含了集的HID用法, 集的所有control. 如果程序和驱动没有使用这些control, 他会立即关闭集的连接.

在获取信息之后, 程序和驱动拥有了需要访问在HID报告里的control数据的信息



## 集能力

集的能力被它的用法,报告,link集, control来定义. 为了获取集能力的综合性描述, 程序和驱动调用`HidP_GetCaps`来获取`HIDP_CAPS`结构. 该结构包含着关于集的link集, 按钮能力组和值能力组的信息:

- 集的用法页和用法ID
- 报告的字节大小
- 在集的link集组里`HIDP_LINK_COLLECTION_NODE`结构的数量.
- 对于每种报告类型, 由`HidP_GetButtonCaps`返回的按钮能力组里的`HIDP_BUTTON_CAPS`的数量
- 对于每种报告类型,由`HidP_GetValueCaps`返回的值能力组里的`HIDP_VALUE_CAPS`结构的数量
- 对于每种报告类型, 集支持的按钮和值的数量, 被`NumberXxxDataIndices`成员所指明



## 按钮能力组

它包含关于一个HID报告的特定类型的一个顶级集所支持的按钮用法的信息. 关于集的能力的信息包含在`HIDP_CAPS`结构里.

程序和驱动使用下列HIDClass支持的函数之一来获取按钮能力的信息:

- `HidP_GetButtonCaps`  返回一个按钮能力组, 描述了所有包含在一个特定报告类型的按钮用法
- `HidP_GetSpecificButtonCaps`过滤按钮能力信息, 它被一个调用者指明的用法页,用法ID和link集返回.

一个按钮能力包含着`HIDP_BUTTON_CAPS`结构, 包含下列的有关HID用法和用法范围信息:

- 关于用法和用法范围的用法页
- 包含按钮数据的报告的报告ID
- 用法ID或用法范围
- 一个flag,表明用法是否是一个别名用法
- 字符串描述符和赋值了用法和用法范围的标志符
- 数据索引, HID解析器分配用法和用法组

常规里, 下列条件情况持有被一个按钮能力组描述的所有用法:

- 每个按钮能力结构代表着一个单一的用法或者用法范围

- 别名用法用于a variable main item. 分配到an array main item里的用法不能是别名用法.用法范围不能被别名的

- HID 解析器只使用最小必须数的用法来分配一个用法给每种按钮. 该解析器以在报告描述符里指明的顺序来分配用法. 在报告里的用法不是必须的, 是被抛弃的. 按钮能力组不包含任何关于已抛弃的用法的信息 

  > 一个按钮一个用法

- 如果在a variable main item里的用法的数量小于在a variable main item里按钮的数量, 那么能力组只包含一个描述一个按钮用法的能力结构. 

- HID解析器分配一个唯一数据索引给每个描述在能力组里的用法

## 在Variable Main Item里的按钮用法

应该参考[HIDP_BUTTON_CAPS 结构](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/hidpi/ns-hidpi-_hidp_button_caps)

被一个报告描述符所说明每种用法/用法范围.

能力结构的`IsAlias`成员被用于指明一集n个别名用法,如下:

- 在首个第n-1个能力结构里的`IsAlias`设置为TRUE. 在第n个能力结构里的`IsAlias`被设为FALSE.那么该首选是在该顺序下的最后一个别名用法.

程序和驱动能决定哪一个按钮用法在扫描这样的序列里被别名化了

下列表格简单描述了三种别名用法的案例

| 在报告描述符里的别名用法序号 | 在能力组里的用法序号 | IsAlias Member Value |
| ---------------------------- | -------------------- | -------------------- |
| usage1                       | usage3               | true                 |
| usage2                       | usage2               | true                 |
| usage3                       | usage1               | false                |



## 在Array Main Item里的按钮用法

在一个报告描述符里的按钮array main item的每种用法/用法范围是被被一个按钮能力组里的自己的能力结构描述的. 能力结构被添加到能力组的顺序和被main item指定的用法的顺序是相反.

HID解析器用在报告描述符里的用法的顺序来分配一个数据索引给array item里每个用法. 例如, 下列表格在一集合用法之间显示对应关系, 如在报告描述符,用法和数据索引里指明,如在能力数组里指明.(在这表里, n是首个数据索引,解析器分配给array item里的首个用法)



# 值能力组

值能力组包含关于被HID集里的每种报告类型支持的值用法的信息.

程序和驱动可以用以下HIDClass支持的函数来获取按钮能力信息:

- `HidP_GetValueCaps`返回一个值能力组
- `HidP_GetSpecificValueCaps`过滤值能力信息,通过用法页,用法,link集和报告类型

值能力结构描述了:

- 用法/用法范围的用法页
- 包含值的报告的报告ID
- 用法ID和用法范围
- 表示用法是否为别名用法
- 关于link集的信息
- 值的字节大小和报告数量
- 值的属性, 包含: 是否为null值,它的单位和指数,它的逻辑和物理范围
- 数据索引的信息, HID解析器把它分配用法/用法范围

常规上, 下列情况持有被值能力组描述的所有用法:

- 每种值能力结果代表一个用法,一个用法范围或者一个用法值组. 他们被分配在a variable main item. an array main item则不支持.
- 别名用法可以被使用.用法范围不能被别名.别名值是链接在一个值能力组里的, 相同地, 别名按钮也被链接在一个按钮能力组里.
- HID解析器只使用最小必须用法数来分配一个用法给一个值. 该解析器用指明在报告描述符里的用法的顺序分配用法. 在报告描述符里的用法是被抛弃的. 值能力组不包含任何有关废弃用法的信息.
- 在能力组里描述的每一个用法,HID解析器分配一个唯一的数据索引

## 用法值组

一个用法值组是一个连续集合的值, 所有这些被分配到相同的用法里.这只发生在一个用法的报告计数大于1时(ReportCount成员)

下列图显示了一个包含了五个六位的数据item的用法值组的示例

![](\asset\2019-11-19-repcount.png)

在上面例子里, 有用法值组的值能力结构将有`IsRange`被设置为FALSE, 它的`NotRange.Usage`设置为17,`ReportCount`设置为5,它的`BitSize`设置为6.

如果用法报告计数是1, 使用`Hid_GetUsageValue`来提取用法值. 如果用法的报告计数是1, `HidP_GetUsageValue`只返回用法值组的首个Item.为了提取用法值组的所有item,使用`HidP_GetUsageValueArray`



# Link集

它是顶级HID集的嵌套性的子HID集.一个顶级集包含0或多个link集

`HidP_GetLinkCollectionNodes`返回一个顶级集的link集组.

## Link集组

link集组描述了一个顶级集的所有link集. 每个link集,由`HIDP_LINK_COLLECTION_NODE`所表示. 这数组的link节点用一种方式连接在一起, 它标识在顶级集里连续和分层的序号. link集组的首个元素代表一个顶级集,接下来的成员代表顶级集的link集.

通过link连接组里遍历节点, 程序和驱动能确定顶级集的所有link集的组织方式和用法. 额外的, 程序和驱动通过link集来组织control. 这是可能的, 因为顶级集的按钮能力组和值能力组标识了包含了由该能力组描述的每种HID用法的link集.

下面图显示了包含4个link集的顶级集的示例

![](\asset\2019-11-19-linkcol.png)

如图, link集以上到下,左到右的顺序连接在一起. 下面的表格表明, 在该示例的每个link集, 顶级集和link集之间的连接

| Link节点 | 父     | 子   | 首子节点 | 下一个兄弟节点 |
| -------- | ------ | ---- | -------- | -------------- |
| A        | 顶级集 | B,C  | B        | None           |
| B        | A      | D    | D        | C              |
| C        | A      | None | None     | None           |
| D        | B      | None | None     | None           |

在一个link集组里, 下列的定义持有:

**父节点**

link集的父节点是集们的上到下层次里直接在它之上的集. link集有一个父节点. 节点的*Parent*成员里在link集组里指明父节点的索引.

**子节点**

父节点的一个子节点是一个link集. 一个父节点有0或多个子节点. 节点*NumberOfChildren*成员指明子节点的数量

**兄弟节点**

父节点的子节点都是兄弟节点

**下一个兄弟节点**

兄弟节点是按左到右排序的.节点的下一个兄弟节点是立即地它右边的兄弟节点. 节点`NextSibling`成员用link集组来指明下一个兄弟节点的索引. 如果一个link集节点没有兄弟节点, 则设置为0

**首个子节点**



## 别名集

划界符item用在报告描述符里来划分一列别名集. 每个别名集由别名link集节点所表示. 一个完整和唯一的n(n >= 2)的集合, 别名节点用以下方式连接在一起:

- 别名节点的顺序是在link集组里连续地表示
- 首个n-1个节点的`IsAlias`成员设置为TRUE.按顺序下来的第n个节点的`IsAlias`设置为FALSE. 这些节点终止了别名节点的连续.该节点的用法是首选用法

一个程序和驱动可以通过重复性地增加link集组的数组索引来决定哪个集是别名的并寻找这样的连续.

按钮能力组和值能力组为他们描述的每个用法标识着包含该用法的link集. 如果该link集是别名的, 那么该能力组指明了首选的用法



# 笔记

[集能力结构](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/hidpi/ns-hidpi-_hidp_caps)

```c++
typedef struct _HIDP_CAPS {
  USAGE  UsagePage;
  USAGE  Usage;
  USHORT InputReportByteLength;
  USHORT OutputReportByteLength;
  USHORT FeatureReportByteLength;
  USHORT Reserved[17];
  USHORT NumberLinkCollectionNodes;
  USHORT NumberInputButtonCaps;
  USHORT NumberInputValueCaps;
  USHORT NumberInputDataIndices;
  USHORT NumberOutputButtonCaps;
  USHORT NumberOutputValueCaps;
  USHORT NumberOutputDataIndices;
  USHORT NumberFeatureButtonCaps;
  USHORT NumberFeatureValueCaps;
  USHORT NumberFeatureDataIndices;
} HIDP_CAPS, *PHIDP_CAPS;
```

[按钮能力结构](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/hidpi/ns-hidpi-_hidp_button_caps)

```c++
typedef struct _HIDP_BUTTON_CAPS {
  USAGE   UsagePage;
  UCHAR   ReportID;
  BOOLEAN IsAlias;
  USHORT  BitField;
  USHORT  LinkCollection;
  USAGE   LinkUsage;
  USAGE   LinkUsagePage;
  BOOLEAN IsRange;
  BOOLEAN IsStringRange;
  BOOLEAN IsDesignatorRange;
  BOOLEAN IsAbsolute;
  ULONG   Reserved[10];
  union {
    struct {
      USAGE  UsageMin;
      USAGE  UsageMax;
      USHORT StringMin;
      USHORT StringMax;
      USHORT DesignatorMin;
      USHORT DesignatorMax;
      USHORT DataIndexMin;
      USHORT DataIndexMax;
    } Range;
    struct {
      USAGE  Reserved1;
      USAGE  Usage;
      USHORT StringIndex;
      USHORT Reserved2;
      USHORT DesignatorIndex;
      USHORT Reserved3;
      USHORT DataIndex;
      USHORT Reserved4;
    } NotRange;
  };
} HIDP_BUTTON_CAPS, *PHIDP_BUTTON_CAPS;
```

[值能力结构](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/hidpi/ns-hidpi-_hidp_value_caps)

```c++
typedef struct _HIDP_VALUE_CAPS {
  USAGE   UsagePage;
  UCHAR   ReportID;
  BOOLEAN IsAlias;
  USHORT  BitField;
  USHORT  LinkCollection;
  USAGE   LinkUsage;
  USAGE   LinkUsagePage;
  BOOLEAN IsRange;
  BOOLEAN IsStringRange;
  BOOLEAN IsDesignatorRange;
  BOOLEAN IsAbsolute;
  BOOLEAN HasNull;
  UCHAR   Reserved;
  USHORT  BitSize;
  USHORT  ReportCount;
  USHORT  Reserved2[5];
  ULONG   UnitsExp;
  ULONG   Units;
  LONG    LogicalMin;
  LONG    LogicalMax;
  LONG    PhysicalMin;
  LONG    PhysicalMax;
  union {
    struct {
      USAGE  UsageMin;
      USAGE  UsageMax;
      USHORT StringMin;
      USHORT StringMax;
      USHORT DesignatorMin;
      USHORT DesignatorMax;
      USHORT DataIndexMin;
      USHORT DataIndexMax;
    } Range;
    struct {
      USAGE  Reserved1;
      USAGE  Usage;
      USHORT StringIndex;
      USHORT Reserved2;
      USHORT DesignatorIndex;
      USHORT Reserved3;
      USHORT DataIndex;
      USHORT Reserved4;
    } NotRange;
  };
} HIDP_VALUE_CAPS, *PHIDP_VALUE_CAPS;
```

[link集结构](https://docs.microsoft.com/windows-hardware/drivers/ddi/hidpi/ns-hidpi-_hidp_link_collection_node)

```c++
typedef struct _HIDP_LINK_COLLECTION_NODE {
  USAGE  LinkUsage;
  USAGE  LinkUsagePage;
  USHORT Parent;
  USHORT NumberOfChildren;
  USHORT NextSibling;
  USHORT FirstChild;
  ULONG  CollectionType : 8;
  ULONG  IsAlias : 1;
  ULONG  Reserved : 23;
  PVOID  UserContext;
} HIDP_LINK_COLLECTION_NODE, *PHIDP_LINK_COLLECTION_NODE;
```



# 问题集

1. 什么是按钮能力组?按钮能力组对于一个HID集来说一共有多少组?

   按钮能力组根据报告类型分3组,分别为读报告组,写报告组,特征报告组.

2. 报告描述符是什么结构的?

3. 别名用法是什么?

4. a variable main item和an array main item是什么?

