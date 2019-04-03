

## bitAnd

问题：用取反运算和或运算描述与运算

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

## getByte

问题：抽取x中第n个字节

一个字节是8位二进制数，所以第0个字节所对应的二进制数中的0~7位，第n个字节对应二进制数中的第8(n-1)~8n位，只需要将原二进制数右移8(n-1)位，所需要的字节就在二进制数的低八位中，再和0xff进行与运算，只保留低八位

```cpp
/* 
 * getByte - Extract byte n from word x
 *   Bytes numbered from 0 (LSB) to 3 (MSB)
 *   Examples: getByte(0x12345678,1) = 0x56
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 2
 */
int getByte(int x, int n) {
  return (x>>(n<<3))&0xff;
}
```

## logicalShift

问题：将x逻辑右移n位

无符号数的右移运算是逻辑右移，高位都用0填充，有符号数的右移运算是算术右移,高位用符号位填充。所以答案应该是在原二进制数右移n位的基础上，只保留低(32-n)位，也就是与(1<<(32-n)+~(1<<(32-n))-1)得到的，低(32-n)均为1，高位均为0的二进制数进行与运算即可

```cpp
/* 
 * logicalShift - shift x to the right by n, using a logical shift
 *   Can assume that 0 <= n <= 31
 *   Examples: logicalShift(0x87654321,4) = 0x08765432
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 3 
 */
int logicalShift(int x, int n) {
  int m = 32 + (~n);
  return (x>>n)&((1<<m)+(~0)+(1<<m));
}
```

## bitCount

问题：求出二进制数中1的个数

首先将二进数按位拆开，比如0101001101011001拆开得到0 1 0 1 0 0 1 1 0 1 0 1 1 0 0 1，将相邻数相加并以二进制数运算得到01 01 00 10 01 01 01 01，继续将相邻数相加得到0010 0010 0010 0010，继续上述操作得到0100 0100，继续得到1000，即有8个1。第一步通过0x55实现，即01010101，第二步用0x33，即00110011，第三步用0xf，即00001111。

```cpp
/*
 * bitCount - returns count of number of 1's in word
 *   Examples: bitCount(5) = 2, bitCount(7) = 3
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 40
 *   Rating: 4
 */
int bitCount(int x) {
  int n, ans = x;
  n = 0x55 + (0x55 << 8) + (0x55 << 16) + (0x55 << 24);
  ans = (ans&n) + ((ans>>1)&n);
  n = 0x33 + (0x33 << 8) + (0x33 << 16) + (0x33 << 24);
  ans = (ans&n) + ((ans>>2)&n);
  n = 0xF + (0xF << 8) + (0xF << 16) + (0xF << 24);
  ans = (ans&n) + ((ans>>4)&n);
  return (ans+(ans>>8)+(ans>>16)+(ans>>24))&0xff;
}
```

## bang

问题：不用非运算符实现非运算符

补码中0的二进制数为全0，所以只需比较x和-x中是否存在符号位为1，若存在符号位为1则补码不为0，返回0，否则返回1

```cpp
/* 
 * bang - Compute !x without using !
 *   Examples: bang(3) = 0, bang(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int bang(int x) {
  return ((~(x|(~x+1)))>>31)&1;
}
```

## tmin

问题：求32位补码的最小值

32位补码的范围是-2^31~2^31-1，即最小值为符号位为1，其余均为0的二进制数

```cpp
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
  return 1<<31;
}
```

## fitsBits

问题：判断x是否能被n位补码表示

首先分析，如果是正数，在n位范围内，从0（00...00)开始，1（00...01）到2^{n-1}（00...11），只有低n-1位里有1, 如果是负数，从0（00...00)开始，-1（11...11）到-2^{n-1}（11...000），只有低n-1位里有0，所以将x右移n-1位后应该全是0或者全是1，所以在右移n-1位后+1再右移1位就得到全0的二进制数

```cpp
/* 
 * fitsBits - return 1 if x can be represented as an 
 *  n-bit, two's complement integer.
 *   1 <= n <= 32
 *   Examples: fitsBits(5,3) = 0, fitsBits(-4,3) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int fitsBits(int x, int n) {
  return !(((x>>(n+(~0)))+1)>>1);
}
```

## divpwr2

问题：求x除以2^n

这道题可以用移位解决，但对于符号数而言，正数除法是向下取整，负数除法是向上取整，而移位运算都是向下取整，所以对于正数可以直接移位，而对于负数则要给x添加偏移量以便向上舍入，也就是加上(1<<n)-1，用(x>>31)&1来判断是不是负数，并且如果是正数则没有偏移量

```cpp
/* 
 * divpwr2 - Compute x/(2^n), for 0 <= n <= 30
 *  Round toward zero
 *   Examples: divpwr2(15,1) = 7, divpwr2(-33,4) = -2
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int divpwr2(int x, int n) {
    return (x+(((x>>31)&1)<<n)+(!((x>>31)&1)+(~0)))>>n;
}
```

## negate

问题：求x的相反数

对于补码而言，x的相反数就是x逐位取反再+1

```cpp
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return ~x+1;
}
```

## isPositive

问题：判断x是否是正数

一个正数的符号位为0，并且0的最高位也为零，所以如果x是正数，x-1的的最高位必定为0，但是因为-2^31-1溢出，即100...00-1得到的01...11，符号位变为0，所以还需先判断x的最高位为0，即x和x-1的最高位同时为0时，x才是正数

```cpp
/* 
 * isPositive - return 1 if x > 0, return 0 otherwise 
 *   Example: isPositive(-1) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 8
 *   Rating: 3
 */
int isPositive(int x) {
  return !(((x>>31)&1)+(((x+(~0))>>31)&1));
}
```

## isLessOrEqual

问题：判断x<=y

原本可以x-y之后判断符号位即可，但考虑到x、y符号不同的时候相减可能溢出，所以先判断符号，如果符号位不同则可以直接得出结论，如果符号位相同则计算x-y的值，因为可能x=y，所以计算x-y-1的值再判断符号位

```cpp
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
  int tx = (x>>31)&1, ty = (y>>31)&1;
  int ok1 = tx&(!ty), ok2 = (((x+(~y))>>31)&1)&(!(tx^ty));
  return ok1|ok2;
}
```

## ilog2

问题：求出log2 x的值

log2 x的值对于二进制而言就是求出x的最高位的1在哪一位，这里可以采取二分搜索计算，先计算前16位是否有1，如果前16位有1，则下一次计算就在16位的基础上向右移8位，计算前8位是否有1，否则计算前24位是否有1，以此类推
```cpp
/*
 * ilog2 - return floor(log base 2 of x), where x > 0
 *   Example: ilog2(16) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 90
 *   Rating: 4
 */
int ilog2(int x) {
  int ans = (!!(x>>16))<<4;
  ans = ans + ((!!(x>>(8+ans)))<<3);
  ans = ans + ((!!(x>>(4+ans)))<<2);
  ans = ans + ((!!(x>>(2+ans)))<<1);
  ans = ans + (!!(x>>(1+ans)));
  return ans;
}
```

## float_neg

问题：求浮点数的相反数

对于32位浮点数而言，最高位（即第31位）表示符号，第23到第30位表示阶码，第0到第22位表示尾数，所以求浮点数的相反数只需将浮点数的最高位取反即可，要注意的是，题目强调了如果浮点数为NAN，也就是阶码全为1，尾数不等于0的时候，需要返回原浮点数

```cpp
/* 
 * float_neg - Return bit-level equivalent of expression -f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representations of
 *   single-precision floating point values.
 *   When argument is NaN, return argument.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 10
 *   Rating: 2
 */
unsigned float_neg(unsigned uf) {
  unsigned temp = uf&0x7fffffff;
  unsigned ans = uf^0x80000000;
  if(temp>0x7f800000) ans=uf;
  return ans; 
}
```

## float_i2f

问题：将有符号数x转化为浮点数的表示形式，并返回无符号二进制数

首先，如果x=0，则特殊处理直接返回0，如果x<0，则将x转换为-x，以便接下来统一对正数进行操作（要记录下符号位），对正数x而言，首先要找到x的最高位1在哪一位，以确定阶码和尾数，知道最高位1的位置后，也就确定了阶码，然后将x左移至只剩下尾数，因为32位浮点数的尾数存储只有23位，所以需要确定低9位的进位或舍入情况，所以取0x3ff和0x1ff，(tx&0x3ff)>=0x300说明第9位和要舍入的第8位均为1，此时因为使用的是偶数舍入法，偏向使最低有效位为0，所以需要进位，(tx&0x1ff)>0x100说明要舍入的第8位为1且低八位大于0，超过一半所以需要进位，最后将答案综合

```cpp
/* 
 * float_i2f - Return bit-level equivalent of expression (float) x
 *   Result is returned as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point values.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_i2f(int x) {
  unsigned ans;
  unsigned sign = (x>>31)&1;
  unsigned cnt = 0, temp = 0x80000000, c = 0, tx = x;
  if(x==0) ans = 0;
  else
  {
    if(sign) tx=(~x)+1;
    while(!(temp&tx))
    {
      cnt++;
      temp = temp>>1;
    }
    tx = tx<<(cnt+1);
    if(((tx&0x3ff)>=0x300)||((tx&0x1ff)>0x100)) c=1;
    ans = (sign<<31) + ((127+31-cnt)<<23) + (tx>>9) + c;
  }
  return ans;
}
```


## float_twice

问题：求浮点数乘以2

对于非规格化浮点数，乘以2只需将浮点数左移一位并保持符号不变，对于规格化浮点数，只需在阶码+1，如果浮点数为NAN，则直接返回原浮点数

```cpp
/* 
 * float_twice - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_twice(unsigned uf) {
  unsigned ans = uf;
  if((ans&0x7f800000)==0)
    ans = ((uf&0x007fffff)<<1)|(uf&0x80000000);
  else if((ans&0x7f800000)!=0x7f800000)
    ans = ans + 0x00800000;
  return ans;
}
```
