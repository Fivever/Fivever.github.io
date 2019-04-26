---
layout: post
title: CS:APP3e（Cache Lab）
date: 2019-04-03
categories: 计算机系统
tag: 教程
---

* content
{:toc}

## Part A

这一部分需要模拟缓存的运行。也就是要仿造出一个csim-ref的程序。

首先定义好各种结构体。主要是参数存储Arguments，缓存行CacheLine，缓存组CacheSet。缓存组就是一个装有缓存行的数组。下面根据官网上的[教学课件](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/recitations/rec07.pdf)提示构建结构体。

```cpp
#include "cachelab.h"
#include <stdio.h>
#include <stdlib.h>
#include <getopt.h>
#include <string.h>

typedef struct arguments
{
    int h;
    int v;
    int s;
    int E;
    int b;
    char *t;
}Arguments;

typedef struct cacheLine
{
    int valid;
    int tag;
    int counter;
} CacheLine;    //每个高速缓存行有1个有效位，t个标记位和2^b个字节

typedef CacheLine *CacheSet;   //高速缓存组，每组有E个高速缓存行
typedef CacheSet *Cache;   //高速缓存存储器，有2^s个高速缓存组

int miss=0, hit=0, eviction=0;  //全局变量
```

从命令行中读取参数读取，借鉴课件中的方法。

```cpp
void getArgs(Arguments *args, int argc, char **argv)
{
    int opt;
    while(-1!=(opt = getopt(argc, argv, "hvs:E:b:t:"))) //hv无参数故不加':'
    {
        switch(opt)
        {
            case 'h':
                args->h = 1;
                break;
            case 'v':
                args->v = 1;
                break;
            case 's':
                args->s = atoi(optarg);
                break;
            case 'E':
                args->E  = atoi(optarg);
                break;
            case 'b':
                args->b = atoi(optarg);
                break;
            case 't':
                args->t = (char*)malloc(sizeof(char)*strlen(optarg));
                strcpy(args->t, optarg);    //将文件地址传给t
                break;
            default:
                printf("wrong argument\n");
                return;
        }
    }
}
```
如果参数不正常，或者输入-h帮助，则另外输出帮助信息。

```cpp
void helpInfo(Arguments args)
{
    if(args.h == 1 || args.s == -1 || args.E  == -1 || args.b == -1 || args.t == NULL)
    {
        if(args.h != 1)
            printf("./csim: Missing required command line argument\n");
        printf("Usage: ./csim [-hv] -s <num> -E <num> -b <num> -t <file>\n");
        printf("Options:\n");
        printf("  -h         Print this help message.\n");
        printf("  -v         Optional verbose flag.\n");
        printf("  -s <num>   Number of set index bits.\n");
        printf("  -E <num>   Number of lines per set.\n");
        printf("  -b <num>   Number of block offset bits.\n");
        printf("  -t <file>  Trace file.\n\n");
        printf("Examples:\n");
        printf("  linux>  ./csim -s 4 -E 1 -b 4 -t traces/yi.trace\n");
        printf("  linux>  ./csim -v -s 8 -E 2 -b 4 -t traces/yi.trace\n");
        return;
    }
}
```

程序刚开始进行初始化操作。

```cpp
void initArgs(Arguments *args)
{//参数初始化
    args->h = 0;
    args->v = 0;
    args->s = -1;
    args->E = -1;
    args->b = -1;
    args->t = NULL;
}

void initCache(Arguments *args, Cache *cache)
{//高速缓存器初始化
    int set_size = 1 << args->s;    //2^s个的高速缓存组
    int num_lines = args->E;    //一组有E行高速缓存行
    *cache = (CacheSet *)malloc(sizeof(CacheSet)*set_size);
    for(int i =0; i<set_size; ++i)
    {
        (*cache)[i] = (CacheLine*)malloc(sizeof(CacheLine)*num_lines);
        for(int j = 0; j<num_lines; ++j)
        {
            (*cache)[i][j].valid = 0;
            (*cache)[i][j].tag = 0;
            (*cache)[i][j].counter = 0;
        }
    }
}
```

处理指令函数，这里不需要真正模拟cache，所以没有存储数据的操作，也和size无关，只需要实现索引之后的命中、不命中和替换即可。

```cpp
void process(Cache *cache, Arguments *args, int address)
{
    address >>= args->b;    //得到标记位和组索引
    int set_index = 0;
    int tag, i;
    long mask = 0xffffffffffffffff >> (64 - args->s);   //由handout知，使用64位存储地址
    set_index = mask & address; //组索引
    address >>= args->s;    //得到标记位
    tag = address;  //标记位
    CacheSet set = (*cache)[set_index];
    for (i = 0; i < args->E; ++i)   //记录每行未使用的次数
        if (set[i].valid == 1)
            set[i].counter++;
    for (i = 0; i < args->E; ++i)   //判断是否命中
        if (set[i].valid == 1 && set[i].tag == tag)
        {
            if(args->v) printf(" hit");
            set[i].counter = 0; //命中则清0
            ++hit;
            return;
        }
    ++miss;
    if(args->v) printf(" miss");
    for (i = 0; i < args->E; ++i)   //如果有效位为0，将其替换
        if (set[i].valid == 0)
        {
            set[i].tag = tag;
            set[i].valid = 1;
            set[i].counter = 0;
            return;
        }
    ++eviction;
    if(args->v) printf(" eviction");
    int max_index = 0, max_time = set[0].counter;   //由LRU替换策略，替换最少使用的行
    for (i = 0; i < args->E; ++i)
        if (set[i].counter > max_time)
        {
            max_index = i;
            max_time = set[i].counter;
        }
    set[max_index].tag = tag;
    set[max_index].counter = 0;
}
```
主函数，参照课件读取文件。
```cpp
int main(int argc, char **argv)
{
    Arguments args;
    Cache cache;
    initArgs(&args);
    getArgs(&args, argc, argv);
    initCache(&args, &cache);
    helpInfo(args);

    FILE *fp = fopen(args.t, "r");
    if(fp == NULL)
    {
        printf("%s: No such file or directory\n", args.t);
        return 0;
    }
    char op;
    unsigned address = 0;
    int size = -1;

    while(fscanf(fp, " %c %x,%d", &op, &address, &size) > 0)
    {
        if(size==-1) continue;
        switch(op)
        {
            case 'I':   //instruction load不需要处理
                break;
            case 'M':   //data modify，修改数据，先读后写两次操作
                if(args.v) printf("M, %x, %d", address, size);
                process(&cache, &args, address);
                process(&cache, &args, address);
                if(args.v) printf("\n");
                break;
            case 'L':   //data load，读取数据，进行一次操作
                if(args.v) printf("L, %x, %d", address, size);
                process(&cache, &args, address);
                if(args.v) printf("\n");
                break;
            case 'S':   //data store，存储数据，进行一次操作
                if(args.v) printf("S, %x, %d", address, size);
                process(&cache, &args, address);
                if(args.v) printf("\n");
                break;
        }
    }
    fclose(fp);

    printSummary(hit, miss, eviction);
    for(int i=0; i<(1<<args.s); ++i)
        free(cache[i]);
    free(cache);
    return 0;
}
```

## Part B

test-trans程序需要依赖valgrind。

```
sudo apt install valgrind
make
./test-trans -M 32 -N 32
```

### 矩阵一

满分的要求是300个misses以内，misses超过600则0分。

题目要求的cache大小是32 × 32 = 1024个字节，可以存下256个4位int数字，矩阵大小也是32 x 32的。每一行32个数字，所以可以存下8行。为了减少不命中的次数，我们可以创建中间变量，并且变量个数为8的倍数作为cache的辅助变量。

、、、cpp
for(i = 0; i < N; i+=8)
    for(j = 0; j < M; j+=8)
        for(k=i; k< i+8; ++k)
        {
            a1 = A[k][j];
            a2 = A[k][j+1];
            a3 = A[k][j+2];
            a4 = A[k][j+3];
            a5 = A[k][j+4];
            a6 = A[k][j+5];
            a7 = A[k][j+6];
            a8 = A[k][j+7];

            B[j][k] = a1;
            B[j+1][k] = a2;
            B[j+2][k] = a3;
            B[j+3][k] = a4;
            B[j+4][k] = a5;
            B[j+5][k] = a6;
            B[j+6][k] = a7;
            B[j+7][k] = a8;
        }
```

### 矩阵二

满分要求是misses小于1300，当misses大于2000则零分。

这里如果继续采用8×8的分块运算会有问题，因为这个矩阵是64 x 64大小的，按照原有的cache大小，一行64个数字，只能存4行，其他的会冲突不命中。而四个变量则不能充分利用cache的大小。方法如下：

<img src="https://blog.codedragon.tech/2017/09/25/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%B3%BB%E7%BB%9FCacheLab-PartB%E5%AE%9E%E9%AA%8C%E6%8A%A5%E5%91%8A/matrix64-2.png">

1. 按行加载矩阵A，此时A的前四行存储在cache中，并且将其分块转置存入矩阵B。依次执行4次，直到整个分块的上半部分处理完毕。其中，每行的前4个元素被正确转置，后四个元素被暂存至矩阵B的右上分块。
2. 对于分块的下半部分的第一行，先将矩阵B的右上分块的一行4个元素载入至临时变量，然后从矩阵A中的左下分块读取第一列并转置进入矩阵右上分块的第一行，然后将之前存的4个临时变量存入矩阵B右下分块的第一行，因为在第一步中，B的右上分块矩阵已经转置过，所以将B的右上分块水平移到左下分块就是对整个矩阵转置的结果。最后再将矩阵A右下分块第一列转置送入矩阵B右下分块的第一行。  
3. 按照2的方式依次处理完下半部分的所有行。

```cpp
for(i=0; i<N; i+=8)
    for(j=0; j<M; j+=8)
    {
        for(k=j; k<j+4; ++k)
        {
            a1=A[k][i];
            a2=A[k][i+1];
            a3=A[k][i+2];
            a4=A[k][i+3];
            a5=A[k][i+4];
            a6=A[k][i+5];
            a7=A[k][i+6];
            a8=A[k][i+7];

            B[i][k]=a1;
            B[i+1][k]=a2;
            B[i+2][k]=a3;
            B[i+3][k]=a4;
            B[i][k+4]=a5;
            B[i+1][k+4]=a6;
            B[i+2][k+4]=a7;
            B[i+3][k+4]=a8;
        }
        for(k=i; k<i+4; ++k)
        {
            a1=B[k][j+4];
            a2=B[k][j+5];
            a3=B[k][j+6];
            a4=B[k][j+7];
            a5=A[j+4][k];
            a6=A[j+5][k];
            a7=A[j+6][k];
            a8=A[j+7][k];

            B[k+4][j]=a1;
            B[k+4][j+1]=a2;
            B[k+4][j+2]=a3;
            B[k+4][j+3]=a4;
            B[k][j+4]=a5;
            B[k][j+5]=a6;
            B[k][j+6]=a7;
            B[k][j+7]=a8;
        }
        for(k=i+4; k<i+8; ++k)
        {
            a1=A[j+4][k];
            a2=A[j+5][k];
            a3=A[j+6][k];
            a4=A[j+7][k];

            B[k][j+4]=a1;
            B[k][j+5]=a2;
            B[k][j+6]=a3;
            B[k][j+7]=a4;
        }
    }
```

### 矩阵三

由于61 × 67的矩阵不是方阵，限制放的比较宽松，满分misses小于2000，misses大于3000零分。16 × 16的分块已可以保证满分。

```cpp
for(i=0; i<N; i+=16)
    for(j=0; j<M; j+=16)
        for(k=i; k<i+16&&k<N; ++k)
            for(h=j; h<j+16&&h<M; ++h)
                B[h][k] = A[k][h];
```