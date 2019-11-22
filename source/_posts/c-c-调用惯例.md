---
title: c/c++调用惯例
date: 2019-11-20 11:10:17
tags: c++
---

[参考](https://scc.ustc.edu.cn/zlsc/sugon/intel/compiler_c/main_cls/bldaps_cls/common/bldaps_calling_conv.htm)

# C/C++调用惯例

这里有一些调用惯例, 它建立了`在参数如何传递给函数`和`值如何从函数里返回`的规则

## 在 Windows* 系统的调用惯例

| 调用惯例   | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| __cdecl    | c/c++程序默认的调用惯例.支持可变参数的传递                   |
| __thiscall | c++成员函数的默认调用惯例,不能使用可变参数                   |
| __clrcall  | 该函数只能从managed code里被调用                             |
| __stdcall  | win32 API函数的标准调用惯例                                  |
| __fastcall | 快速调用惯例, 参数传递给寄存器而不是栈                       |
| __regcall  | intel编译器调用惯例, 参数尽可能传递给寄存器;同样地,尽可能使用寄存器返回值.如果该函数带有可变参数,该调用惯例自动被忽略 |
|            |                                                              |
|            |                                                              |
|            |                                                              |

