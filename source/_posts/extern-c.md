---
title: extern(c++)
date: 2019-11-22 16:44:01
tags: C C++
---

# extern(c++)的一些简单说明

**extern**关键词用于全局变量,函数,模板的声明指明那名称拥有external linkage.

**extern**有四种意思, 依赖于上下文:

1. 在non-const 全局变量声明里, **extern**指明那些变量或函数定义在其他翻译单元里. **extern**必须应用在所有文件,除了定义它的文件
2. 在const 变量声明里, 它指明变量拥有external linkage. **extern**必须应用在所有文件里的所有声明
3. **extern "C"** 指明函数被定义在其他地方, 并且使用C语言调用惯例. 该修饰符也可以应用于在块里的多个函数声明
4. ...



## 关于non-const全局的外部linkage

```c++
//fileA.cpp
int i = 42; // declaration and definition

//fileB.cpp
extern int i;  // declaration only. same as i in FileA

//fileC.cpp
extern int i;  // declaration only. same as i in FileA

//fileD.cpp
int i = 43; // LNK2005! 'i' already has a definition.
extern int i = 43; // same error (extern is ignored on definitions)
```



## 关于const全局的extern linkage

```c++
//fileA.cpp
extern const int i = 42; // extern const definition

//fileB.cpp
extern const int i;  // declaration only. same as i in FileA
```

## extern "C" 和 extern "C++" 的函数声明

```c++
// Declare printf with C linkage.
extern "C" int printf(const char *fmt, ...);

//  Cause everything in the specified
//  header files to have C linkage.
extern "C" {
    // add your #include statements here
#include <stdio.h>
}

//  Declare the two functions ShowChar
//  and GetChar with C linkage.
extern "C" {
    char ShowChar(char ch);
    char GetChar(void);
}

//  Define the two functions
//  ShowChar and GetChar with C linkage.
extern "C" char ShowChar(char ch) {
    putchar(ch);
    return ch;
}

extern "C" char GetChar(void) {
    char ch;
    ch = getchar();
    return ch;
}

// Declare a global variable, errno, with C linkage.
extern "C" int errno;
```

