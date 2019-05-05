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

- 注意
  
  因为algorithm里的sort被定义在std命名空间里，运用sort需要同时有algorithm和std。

  struct中的对象存储为const DNA。所以当你试图用const对象调用<运算符时编译器会检测到一个问题，你主要是在const对象上调用一个非const成员函数，这是不允许的，因为非const成员函数不承诺不修改对象；所以编译器会做一个安全的假设该操作符可能会尝试修改对象，但同时，它也会注意到对象是const；因此，任何修改const对象的尝试都应该是错误的。因此编译器生成一个错误消息。


```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

struct DNA
{
    char str[51];
    int rank;
    bool operator<(DNA x) const
    {
        return rank<=x.rank;
    }
}dna[100];

int main()
{
    int len,n;
    scanf("%d%d",&len,&n);
    for(int i=0;i<n;i++)
    {
        scanf("%s",dna[i].str);
        dna[i].rank=0;
        for(int j=0;j<len;j++)
            for(int k=j+1;k<len;k++)
                dna[i].rank+=(dna[i].str[j]>dna[i].str[k]);
    }
    sort(dna,dna+n);
    for(int i=0;i<n;i++)
        printf("%s\n",dna[i].str);
}
```

## [Vertical Histogram](http://poj.org/problem?id=2136)

考察格式化输出。
- 注意
  memset在cstring头文件中。
  
  c字符串以'\0'结尾。
```cpp
#include <cstdio>

const int N=26;
int sum[N];
char out[2*N];

int main()
{
    char c;
    for(int i=0;i<N;i++)
        sum[i]=0;
    while((c=getchar())!=-1)
        if(c>='A'&&c<='Z')
            sum[c-'A']++;
    while(true)
    {
        int maxn=0;
        for(int i=0;i<N;i++)
            if(sum[i]>maxn)
                maxn=sum[i];
        if(!maxn) break;
        for(int i=0;i<N;i++)
        {
            if(sum[i]==maxn)
            {
                out[2*i]='*';
                sum[i]--;
            }
            else out[2*i]=' ';
            out[2*i+1]=' ';
        }
        for(int i=2*N-1;i>0;i--)
            if(out[i]=='*')
            {
                out[i+1]='\0';
                break;
            }
        printf("%s\n",out);
    }
    for(int i=0;i<N-1;i++)
        printf("%c ",'A'+i);
    printf("Z\n");
}
```

## [Herd Sums](http://poj.org/problem?id=2140)

a+1+a+2+...+a+k=n\Rightarrow (k+1)a+0.5*k*(k＋1)=n\Rightarrow (a+0.5*k)(k+1)=n

若k为奇数，k*x_{\frac{k+1}2}=n的整数解为答案的一部分。

若k为偶数，(2*x_{\frac k2}+1)*\frac k2=n的整数解为答案的另一部分。

因为k为奇数时，只需求符合条件的k，k为偶数时，只需求(2*x_{\frac k2}+1)，二者均为奇数，所以合起来就是n有多少个奇因数，就是本题的答案。


```cpp
#include <cstdio>

int main()
{
    int n,ans=0;
    scanf("%d",&n);
    for(int i=1;i<=n;i+=2)
        if(n%i==0)
            ans++;
    printf("%d\n",ans);
}
```

## [Adding Reversed Numbers](http://poj.org/problem?id=1504)

将两个数反转相加再反转，思路简单。

- 注意

字符串在输入后的后一位自动变为'\0'。

```cpp
#include <cstdio>
#include <cstring>

const int N=100;
char a[N],b[N];

int main()
{
    int n;
    scanf("%d",&n);
    while(n--)
    {
        memset(a,'0',sizeof(a));
        memset(b,'0',sizeof(b));
        scanf("%s %s",a,b);
        for(int i=0;i<N;i++)
        {
            if(a[i+1]=='\0') a[i+1]='0';
            if(b[i+1]=='\0') b[i+1]='0';
            a[i]+=b[i]-'0';
            while(a[i]>'9')
                a[i]-=10, a[i+1]++;
        }
        int s=0,e=N-1;
        while(a[s]=='0') s++;
        while(a[e]=='0') e--;
        for(int i=s;i<=e;i++)
            printf("%c",a[i]);
        printf("\n");
    }
}
```

