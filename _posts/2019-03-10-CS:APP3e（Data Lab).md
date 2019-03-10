---
published: true
layout: post
title: 'CS:APP3e（Data Lab)'
tags:
  - 计算机系统
description: 计算机系统实验解析
toc: true
share: true
comments: true
---
## Data Lab

### 与运算

用取反运算和或运算描述与运算

根据德摩根定律：

	非（ p 且 q ）等价于（ 非 p ）或（ 非 q ）
	非（ p 或 q ）等价于（ 非 p ）且（ 非 q ）

可得答案

```cpp
/* 
 * bitAnd - x&y using only ~ and | 
 *   Example: bitAnd(6, 5) = 4
 *   Legal ops: ~ |
 *   Max ops: 8
 *   Rating: 1
 */
int bitAnd(int x, int y) {
  return ~((~x)|(~y));
}
```

