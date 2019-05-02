---
layout: post
title: CS97SI
date: 2019-04-30
categories: 算法训练
tag: 教程
---

## [A+B Problem](http://poj.org/problem?id=1000)

过

## [Financial Management](http://poj.org/problem?id=1004)

此题加和求平均即可。

- 注意：

  printf的%f说明符的确既可以输出float型又可以输出double型。 根据"默认参数提升"规则（在printf这样的函数的可变参数列表中 ，不论作用域内有没有原型，都适用这一规则）float型会被提升为double型。因此printf()只会看到双精度数。

  对于scanf，情况就完全不同了，它接受指针，这里没有类似的类型提升。（通过指针）向float存储和向double存储大不一样，因此，scanf区别%f和%lf。

```cpp
#include <cstdio>
int main()
{
    int n=12;
    double sum=0,a;
    for(int i=0;i<n;i++)
    {
        scanf("%lf",&a);
        sum+=a;
    }
    printf("$%.2f\n",sum/n);
}
```

## [Hangover](http://poj.org/problem?id=1003)

按照题目所给的规律进行模拟即可，鉴于存在double类型变量的大小比较，建议设置eps消除误差。

```cpp
#include <cstdio>
const double eps = 1e-3;
int main()
{
    double n;
    while(scanf("%lf",&n)&&n>eps)
    {
        double goal=0;
        for(int i=2;;i++)
        {
            goal+=1.0/i;
            if((goal-n)>eps)
            {
                printf("%d card(s)\n",i-1);
                break;
            }
        }
    } 
}
```

## [DNA Sorting](http://poj.org/problem?id=1007)

