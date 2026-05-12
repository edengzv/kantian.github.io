---
title: "浅谈大端模式和小端模式"
date: 2015-03-05 00:00:00
categories: ["Computer Science"]
---

## 简介

计算机内存中，数据是按照字节进行存储，对应于内存中的每一个字节都有一个地址。如果我们内存想象成一个无比庞大的数组，那么这个数组包含若干个元素，我们通过地址来访问数组中的每一个元素。每一个数组元素占一个字节大小并且存放一个字接大小的内容。

## 数据在内存中的存放

以32位为例，通常的int型和float型数据都占用32位（bits），也就是4字节（bytes）。每一个内存地址指向的内存单元只能存放1字节(8-bits)，**因此需要把一个占4字节的一个int型数据拆分，存放到4个连续的内存单元中**。比方说,有一个占32位的int型数据(十六进制表示）： k = 0x12345678，拆分后每一个字节对应的部分数据为12,34,56,78。由于[历史的原因](http://en.wikipedia.org/wiki/Endianness#History)，存在两种不同的字节顺序(endianness)来存放这4个字节——大端字节序（Big-endian）和小端字节序（Little-endian），其在内存中的存放顺序大致分别如下图所示：

#### 大端字节序(Big-endian)

地址
数据  

P
12

P+1
34

P+2
56

P+3
78

#### 小端字节序(Little-endian)

地址
数据

P
78

P+1
56

P+2
34

P+3
12

**字节序**是一个处理器架构特性，用来指示超过单个字节大小的数据类型（如：int,float,double，etc…)在内存中存储的字节顺序。
以上两种字节序的存放顺序是恰好相反的。注意，为了方便记忆，可以理解为：

> 
大端字节序：  *低地址存放数字**高**位*
小端字节序：  *低地址存放数字**低**位*

#### 常见几个系统的字节序

操作系统
处理器架构
字节序

FreeBSD 5.2.1
Intel Pentium
Little-endian

Linux 2.4.22
Intel Pentium
Little-endian

Mac Ox X 10.3
PowerPC
Big-endian

Solaris 9
Sun SPARC
Big-endian

## 字节序与编程

通常来说，在同一台计算机上的进程进行通信时，不必考虑字节序。然而在不同计算机之间进行数据传输和存储的时候就必然要考虑不同的计算机的字节序差异了，否则会带来意想不到的结果。
不妨看看如下代码：

```c
#include <stdio.h>
#include <string.h>

int main (int argc, char* argv[]) {
    FILE* fp;

    /* Our example data structure */
    struct {
        char one[4];
        int  two;
        char three[4];
    } data;

    /* Fill our structure with data */
    strcpy (data.one, "foo");
    data.two = 0x01234567;
    strcpy (data.three, "bar");

    /* Write it to a file */
    fp = fopen ("output", "wb");
    if (fp) {
        fwrite (&data, sizeof (data), 1, fp);
        fclose (fp);
    }
}
```

这段代码把一个结构体data写入到文件output当中。这个结构体包含一个char类型数组（one），一个int型数（two），以及另一个char类型数组（three）。
不论在大端字节序还是在小端字节序的机子上都能正常编译和运行，然而当我们通过hexdump工具查看程序输出文件内容的时候，差别就显现出来了。

在大端字节序（Big-endian）机子下：

```dns
00000000  66 6f 6f 00 12 34 56 78  62 61 72 00     |foo..4Vxbar.|
0000000c
```

在小端字节序（Little-endian）机子下：

```dns
00000000  66 6f 6f 00 78 56 34 12  62 61 72 00     |foo.xV4.bar.|
0000000c
```

可见，在不同的平台上面，对结构体中int类型的数two的存储顺序是不同的。如果处理器只按照各自默然的方式来读取文件的话，文件的内容就并非我们所想象的那样了。

#### 判断机器字节序

那么,如何通过程序来判断某台机子的字节序呢？且看如下代码：

```c
#include <stdio.h>
#include <stdlib.h>
#include <inttypes.h>
int main(){
    uint32_t i;
    unsigned char *ch;
    i = 0x04050607;
    ch = (unsigned char *)&i;
    if(*ch == 4){
        printf("big endian\n");
    }else if(*ch == 7){
        printf("little endian\n");
    }else{
        printf("unknow\n");
    }
    exit(0);
}
```

根据前文的描述不难理解：只要判断其指针所指的第一个字节的内容即可判定是大端字节序还是小端字节序。
1）如果第一个字节（低地址）存放的是整数的低位数字，则为小端字节序（Little-endian)
2)如果第一个字节（低地址）存放的是整数的高位数字，则为大端字节序（Big-endian）

***更进一步，通过宏定义来实现：***

```c
const int i = 1;
#define is_big_endian() ((*(unsigned char *)&i)==0)
```

#### 字节序转换

如何把一个int型数转换为大端字节序呢？

```c
/*System Env:Linux version 3.13.0-43-generic (buildd@akateko) (gcc version 4.6.3 (Ubuntu/Linaro 4.6.3-1ubuntu5) )
  Author: kantian
  Date: 2015-01-06 17:01
  Description:Litter-endian int ---> Big-endian int 
*/
#include <stdio.h>
#include <stdlib.h>
const int i = 1;
#define is_big_endian() ((*(unsigned char*)&i) == 0)
//方法一:
int big_endian_int(int i){
    int a,b,c,d;
    //alread is big-endian,just return i
    if(is_big_endian()){
        return i;
    }
    //get the 1st,2nd,3rd and 4th byte
    a = i & 255;
    b = (i >> 8) & 255;
    c = (i >> 16) & 255;
    d = (i >> 24) & 255;
    //combine them in new order
    return (int)((int)d + (int)(c << 8) + (int)(b <<16) + (int)(a << 24));
}
//方法二:
int big_endian_int_2(char *i){
    int a;    
    char *b = (char *)&a;
    //for each byte
    if(is_big_endian()){
       b[0] = i[0];
       b[1] = i[1];
       b[2] = i[2];
       b[3] = i[3];
    }else{
       b[0] = i[3];
       b[1] = i[2];
       b[2] = i[1];
       b[3] = i[0];
    }
    return a;
}

int main(){
    int k = 0x01020304;    
    int b = big_endian_int(k);
    int c = big_endian_int_2((char *)&k);
    printf("%x\n",b);//04030201
    printf("%x\n",c);//04030201
    return 0;
}
```

#### 网络和主机间字节序交换

如前文所述，不同的计算机进程之间通过网络进行通信的时候，不同的字节序会给其带来麻烦。那么如何来消除不同计算机之间字节序的不同所造成的影响呢？事实上网络协议通常规定了字节序，因此，异构计算机系统能够交换协议信息而不会混淆字节序，如TCP/IP协议就是大端字节序（Big-endian)。对于TCP/IP应用程序，提供了四个通用的程序用来进行本地字节序跟网络字节序之间的转换，如下所示：

```c
#include <arpa/inet.h>
//将主机字节序转换为32位网络字节序
uint32_t htonl(uint32_t hostint32);

//将主机字节序转换微16位网络字节序
uint16_t htons(uint16_t hostint16);

//将网络字节序转换为32位主机字节序
uint32_t ntohl(uint32_t hostint32);

//将网络字节序转换微16位主机字节序
uint16_t ntohs(uint16_t hostint16);
```

> 
注意，便于记忆：
n:  network 网络
h： host 主机
l:  long 整型（4 bytes) 相对于short
s： short 短整型（2 bytes) 

## 问题来了

下面程序在大端字节序（Big-endian)的机子上和小端字节序(Little-endian)的机子上分别输出什么？

```c
#include <stdlib.h>
#include <stdio.h>
int main(){
    unsigned char endian[2] = {1,0};
    short x;
    x = *(short *) endian;
    printf("%d\n",x);
    return 0;
}
```

答案是：

> 
Little-endian:  1
Big-endian:  256

##  参考文献

- [1] [http://en.wikipedia.org/w/index.php?title=Endianness](http://en.wikipedia.org/w/index.php?title=Endianness)

- [2] [http://www.ibm.com/developerworks/aix/library/au-endianc/index.html](http://www.ibm.com/developerworks/aix/library/au-endianc/index.html)

- [3] [http://www.apuebook.com/apue2e.html](http://www.apuebook.com/apue2e.html)