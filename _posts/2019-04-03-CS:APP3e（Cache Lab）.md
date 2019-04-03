

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

处理指令函数

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
主函数，参照课件读取文件
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

### 矩阵一

题目要求的cache大小是32 × 32 = 1024个字节，矩阵大小也是32 x 32的，可以存下256个4位int数字。每一行32个数字，所以可以存下8行。为了减少不命中的次数，我们可以创建中间变量，并且变量个数为8的倍数作为cache的扩增。

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


