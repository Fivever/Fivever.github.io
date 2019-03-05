---
published: false
---
---

published: true

layout: post

title: 信息的表示与处理

tags:
  - 计算机组成
  
description: 读书笔记

toc: true

share: true

comments: true

---

## 信息的表示与处理

无符号数编码的定义

	对一个二进制数，第i位乘以2^{i-1}求和

无符号数编码的唯一性

	函数B2U是一个双射

补码编码的定义

	对于一个二进制数，符号位的权值为-2^{w-1}，其余位各乘以2^{i-1}求和

补码编码的唯一性

	函数B2T是一个双射

补码转换为无符号数：

	对于TMin<=x<=TMax，且x<0, T2U=x+2^w, x>=0, T2U=x
    
无符号数转换为补码

	对于0<=u<=UMax, u<=TMax, U2T=u, u>TMax, U2T=u-2^w
    
无符号数的零扩展

	在无符号数前添加0，B2U大小不变

补码数的符号扩展

	在补码前添加原补码符号位，B2T大小不变
    
截断无符号数

	若x为w位无符号数，x'为x抛弃前w-k位的k位无符号数，则B2U(x')=B2U(x)%2^k

截断补码数值

	若x为w位补码，x'为x抛弃前w-k位的k位补码，则B2T(x')=U2T(B2U(x)%2^k)




