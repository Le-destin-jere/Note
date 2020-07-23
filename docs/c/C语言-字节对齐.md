#  C语言学习笔记

>字节对齐：计算机并非逐字节大小读写内存，而是以2,4,或8的 倍数的字节块来读写内存，如此一来就会对基本数据类型的合法地址作出一些限制，即它的地址必须是2，4或8的倍数。那么就要求各种`数据类型按照一定的规则在空间上排列，这就是对齐。`
>
>1.为了`加快CPU访问内存的读取效率`，CPU读取数据往往是从数据对齐地址开始，并且一次载入固定位宽的数据。
>
>2.`节省内存大小`

[toc]

## 查看默认字节对齐方式

c语言在给不同类型变量分配地址空间时，并`不是总是紧邻着`上一个变量的地址空间分配的，而是它所在的地址空间，必须被它的默认对齐字节数整除。例如，int类型占4字节，其默认对齐字节数为4，那么它所在的地址的低4位必须为0x0、0x4、0x8...,因为这些地址才能被4整除；又如short类型，其本身占2字节，默认对齐字节数为2，则其地址的低4位必须为0x0, 0x2, 0x4, 0x6；那么char型就不用说了，他可以分配在任意地址。

```c
#include "stdio.h"

int main(int argc,char * argv )
{
    int  a;  // 4byte
    char b;  // 1byte
    short c; // 2byte
    
    printf("sizeof char [%ld]\r\n",sizeof(char));
    printf("sizeof int [%ld]\r\n",sizeof(int));
    printf("sizeof short [%ld]\r\n",sizeof(short));
    printf("int&.a:[%p]\r\n",&a);
    printf("char&.b:[%p]\r\n",&b);
    printf("short&.c:[%p]\r\n",&c);
    
    return 0;
}
```

结果：gcc version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04) 

```
sizeof char [1]
sizeof int [4]
sizeof short [2]
int&.a:[0x7fff8e0b6054]
char&.b:[0x7fff8e0b6051]
short&.c:[0x7fff8e0b6052]
```

结果看出，定义的三个变量分配的地址并不是根据变量先后定义顺序来的，可能编译器有优化。

存储结构：

| 0x..51 | [b]  |  [c  |  c]  |      |
| :----: | :--: | :--: | :--: | :--: |
| 0x..54 |  [a  |  a   |  a   |  a]  |

## 复合类型对齐方式 

除了基本类型之外，复合类型如结构体和联合等都能改变其默认对齐方式，gcc通常默认的对齐方式为4字节。

1.`#pragma pack(n) `
// 编译时，使对齐方式为n。n必须是2的幂指数，且n只有在小于结构体成员的最大默认对齐字节数才生效。就是只能用来`减小对齐方式`。
`#pragma pack()`  // 恢复默认对齐方式

2.`#pragma pack(push, n) ` //保存以前设置的对齐方式，并使当前对齐方式变成n
`#pragma pack(pop)`   //恢复到之前的对齐方式

3.`__attribute__((__packed__))` // 取消字节优化对齐，即都是1字节对齐
`__attribute__((packed))`// 同上 

4.`__attribute__((aligned(n))) ` //该参数并不会改变每个成员的默认对齐方式，而是将变量占用的空间大小指定为n的整数倍，空字节用于填充，且变量的首地址也必须被n整除。**只有当n大于结构体成员的最大对齐字节**数时才有效，n为2的幂指数。当然，该参数对于基本变量也有效。
`__attribute__((aligned))  `   // 自动选择最有益的方式对齐，可以提高拷贝操作的效率。
`__attribute__((__aligned__)) ` //同上

来看第三第四种情况的运行结果：

```c
#include "stdio.h"

typedef struct 
{
    char a;
    int b;
    short c;
}  __attribute__((__aligned__(4))) MT;
typedef struct 
{
    char a;
    int b;
    short c;
}  __attribute__((__aligned__)) MT2;

typedef struct 
{
    char a;
    int b;
    short c;
}  __attribute__((__packed__)) MT3;


MT mt1 = { 0 };
MT2 mt2 = { 0 };
MT3 mt3 = { 0 };

int main(int argc,char * argv )
{
    printf("sizeof char [%ld]\r\n",sizeof(char));
    printf("sizeof int [%ld]\r\n",sizeof(int));
    printf("sizeof short [%ld]\r\n",sizeof(short));
    
    printf("\r\nsize of MT sturct is %ld byte.\r\nsize of MT2 sturct is %ld byte.\r\nsize of MT3 sturct is %ld byte.\r\n",sizeof(mt1),sizeof(mt2),sizeof(mt3));
    printf("&mt1.a:[%p]\r\n",&mt1.a);
    printf("&mt1.b:[%p]\r\n",&mt1.b);
    printf("&mt1.c:[%p]\r\n",&mt1.c);
    printf("&mt2.a:[%p]\r\n",&mt2.a);
    printf("&mt2.b:[%p]\r\n",&mt2.b);
    printf("&mt2.c:[%p]\r\n",&mt2.c);
    printf("&mt3.a:[%p]\r\n",&mt3.a);
    printf("&mt3.b:[%p]\r\n",&mt3.b);
    printf("&mt3.c:[%p]\r\n",&mt3.c);

    return 0;
}
```

结果：gcc version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04) 

```
size of MT sturct is 12 byte.
size of MT2 sturct is 16 byte.
size of MT3 sturct is 7 byte.
&mt1.a:[0x5605295cd020]
&mt1.b:[0x5605295cd024]
&mt1.c:[0x5605295cd028]
&mt2.a:[0x5605295cd030]
&mt2.b:[0x5605295cd034]
&mt2.c:[0x5605295cd038]
&mt3.a:[0x5605295cd040]
&mt3.b:[0x5605295cd041]
&mt3.c:[0x5605295cd045]
```

存储结构：

对于mt1结构体进行四字节对齐`__attribute__((__aligned__(4)))`，结构体大小为12字节，从结果上看存储结构如下:

| 0x..20 | [a]  |      |      |      |
| :----: | :--: | :--: | :--: | :--: |
| 0x..24 |  [b  |  b   |  b   |  b]  |
| 0x..28 |  [c  |  c]  |      |      |

对于mt2使用自动对齐优化，`__attribute__((__aligned__))`,结构体大小为16字节，但从结果上看 分配的内存地址与4字节对齐的mt1一样，为什多出4字节？

对于mt3使用1字节对齐，`__attribute__((__packed__))`，结构体字节大小为7字节，就是结构体内部类型的大小总和。结构如下：

| 0x..40 | [a]  |  [b  |  b   | b    |
| :----: | :--: | :--: | :--: | ---- |
| 0x..45 |  b]  |  [c  |  c]  |      |





>
>
>参考资料：
>
>[1]: http://blog.chinaunix.net/uid-31343710-id-5756197.html
>
>

