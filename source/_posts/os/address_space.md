---
title: 地址空间
tags:
  - os
  - 内存虚拟化
categories: os
date: 2021-02-22 11:02:46
---

## 地址空间(address space)
> 物理内存抽象

## 地址转换(address translation)
> 将虚拟地址转换为物理地址.
### 基址界限(base and bound)
cpu 需要两个寄存器: 基址寄存器和界限寄存器. 进程使用的地址都是虚拟地址, 为了得到物理内存地址, 需要加上base寄存器内容:
$$ physical \ address = virtual \ address + base $$

如果内存超过bound界限则抛出异常. 有两种方式检查(逻辑上是等价的):
1. 在虚拟地址和基址寄存器内容求和之前检查.
2. 虚拟地址和基址寄存器求和之后检查.

由于地址重定位发生于运行时, 也称为动态重定位.

base和bound寄存器是芯片中的硬件结构, 每个cpu中存在一对. 也将cpu中负责内存地址转换的部分称为Memory management unit(MMU, 内存管理单元).   




