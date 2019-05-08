---
layout: post
title: Mathematics
date: 2019-05-08
categories: 算法训练
tag: 教程
---

## [Yeehaa!](http://poj.org/problem?id=1799)

初中几何，设两个小圆圆心和大圆圆心所成夹角为<img src="https://latex.codecogs.com/png.latex?\theta">，则<img src="https://latex.codecogs.com/png.latex?(R-r)sin\frac{\theta}2=r\Rightarrow r=\frac{Rsin\frac{\theta}2}{1+sin\frac{\theta}2}">

```cpp
#include <cstdio>
#include <cmath>

const double PI = acos(-1.0);

int main()
{
    int m;
    scanf("%d",&m);
    for(int i=0;i<m;i++)
    {
        int n;
        double R;
        scanf("%lf%d",&R,&n);
        printf("Scenario #%d:\n%.3f\n\n",i+1,sin(PI/n)*R/(1+sin(PI/n)));
    }
}
```

## [Factorial](http://poj.org/problem?id=1401)

计算末尾0的个数，因为因数中2的个数足够多，所以可以简化为算因数中5的个数。另外，线性时间会超时。

- 注意

  int的范围为-2^31 ~ 2^31-1，即正负21亿。

```cpp
#include <cstdio>

int main()
{
    int n;
    scanf("%d",&n);
    while(n--)
    {
        int a,sum=0;
        scanf("%d",&a);
        for(int i=5;i<=a;i*=5)
            sum+=a/i;
        printf("%d\n",sum);
    }
}