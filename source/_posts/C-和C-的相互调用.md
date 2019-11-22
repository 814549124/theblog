---
title: 'C#和C++的相互调用的相关案例'
date: 2019-11-19 16:33:47
tags: C#
---

# 相关知识点

1. 都包含在 [System.Runtime.InteropServices](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.interopservices) 命名空间里
2.  [C/C++ Calling Conventions](https://scc.ustc.edu.cn/zlsc/sugon/intel/compiler_c/main_cls/bldaps_cls/common/bldaps_calling_conv.htm)



# 问题集

1. 错误 "Attempted to read or write protected memory" 如何解决.

   [参考](https://stackoverflow.com/questions/10681887/calling-c-code-from-c-sharp-error-using-references-in-c-ref-in-c-sharp)

2. 错误 `A call to PInvoke function 'CSharpInteroperatorStudy1!CSharpInteroperatorStudy1.UnitTest1+DllExample::Num1' has unbalanced the stack. This is likely because the managed PInvoke signature does not match the unmanaged target signature. Check that the calling convention and parameters of the PInvoke signature match the target unmanaged signature.` 如何解决

   尝试加上`__stdcall`

# 案例

1. 如何简单的调用里面的DLL里面的函数

```csharp

```

 https://stackoverflow.com/questions/20752001/passing-strings-from-c-sharp-to-c-dll-and-back-minimal-example 