---
title: Mach-O
date: 2025-03-07 13:42:47
tags: [iOS]
categories: 
  - [iOS]
---

# Mach-O

Mach-O是Mach object缩写，是mac/ios上用于存储程序、库的标准格式

https://developer.apple.com/library/archive/navigation/

##### 常见类型

- MH_OBJECT
  - 目标文件(.o) 
  - 静态库(.a) 静态库其实就是多个.o合并在一起
- MH_EXECUTE
  - 可执行文件
- MH_DYLIB
  - 动态库文件(.dylib .framework)
- MH_DYLINKER
  - 动态链接编辑器 dyld
- MH_DSYM
  - 存储着二进制文件符号信息的文件

##### Mach-O 组成

https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html

A Mach-O file has the following regions of data (the complete format is described in OS X ABI Mach-O File Format Reference):

- Header: Specifies the target architecture of the file, such as PPC, PPC64, IA-32, or x86-64.
  文件类型、目标架构类型等
- Load commands: Specify the logical structure of the file and the layout of the file in virtual memory.
  描述文件在虚拟内存中的逻辑结构、布局
- Raw segment data: Contains raw data for the segments defined in the load commands.
  在load commands.中定义的Segment的原始数据

##### dyld

dyld用于加载以下类型的Mach-O文件

- MH_EXECUTE
- MH_DYLINKER
- MH_BUNDLE

> App的可执行文件、动态库都是由dyld负责加载的
