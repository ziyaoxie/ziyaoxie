# Linux C 位域、大小端和数据对齐

## Linux C 位域和大小端

### 大小端

当数据类型大于一个字节时，其所占用的字节在内存中的顺序存在两种模式：小端模式（little endian）和大端模式（big endian），其中`MSB`（Most Significant Bit）最高有效位，`LSB`（Least Significant Bit）最低有效位。

例如，`0x1234`使用两个字节储存，其中高位字节是`0x12`，低位字节是`0x34`。

```text
小端模式
MSB                             LSB
+-------------------------------+
|   1   |   2   |   3   |   4   | int 0x01020304
+-------------------------------+
  0x03    0x02    0x01    0x00   Address

大端模式
MSB                             LSB
+-------------------------------+
|   1   |   2   |   3   |   4   | int 0x01020304
+-------------------------------+
  0x00    0x01    0x02    0x03   Address
```

两种类型的字节序介绍如下：

- 大端字节序：高位字节在前，低位字节在后，也是人类读写数值的方法；
- 小端字节序：低位字节在前，高位字节在后，更适合计算机；

计算都是从低位开始的，那么计算机先处理低位字节时效率比较高，计算机的内部处理都是小端字节序；但是人类还是习惯读写大端字节序，所以除了计算机的内部处理，其他的场合几乎都是大端字节序，比如网络传输和文件储存。

```c
// 测试程序
#include <stdio.h>

void main(void)
{
	int test = 0x41424344;
	char *ptr = (char*)&test;

#ifdef DEBUG
	printf("int  Address:%x Value:%x\n", (unsigned int)&test, test);
	printf("\n------------------------------------\n");

	int j;
	for(j = 0; j <= 3; j++) {
		printf("char Address:%x Value:%c\n", (unsigned int)ptr, *ptr);
		ptr++;
	}
	printf("------------------------------------\n\n");
	pAddress = (char*)&test;
#endif
	if(*ptr == 0x44)
		printf("Little-Endian\n");
	else if(*ptr == 0x41)
		printf("Big-Endian\n");
	else
		printf("Something Error!\n");
}
```

假设，在保存一个二进制文件时采用的是大端字节序，那么读取一个`32bits`的数据方式如下：

```c
uint32_t length = (data[3]<<0) | (data[2]<<8) | (data[1]<<16) | (data[0]<<24);
```

对于小端字节序的读取方式如下：

```c
uint32_t length = (data[0]<<0) | (data[1]<<8) | (data[2]<<16) | (data[3]<<24);
```

### 位域

“位域”是把一个字节中的二进位划分为几个不同的区域，并标明每个区域的位数，每个域有一个域名，允许在程序中按域名进行操作。

```c
struct bitfield {
	int a:8;
	int b:2;
	int c:6;
};
```

注意事项：

- 其中类型必须为整形，也就是`int`、`unsigned int`、`signed int`三种之一，不能是浮点类型或者`char`类型；
- 二进制位数不能超过基本类型表示的最大位数，例如`x64`上的最多不能超过`64`；
- 可以使用空的位域，此时可以占用空间但是不能直接引用，下个位域从新存储单元开始存放；
- 不能对位域进行取地址操作。

## Linux C 数据对齐

自然对齐：对于基础类型来说，就是具体类型的长度，例如`short`为2字节，`int`为4字节等（与编译器相关）；而结构体则是最大的那个值。

指定对齐：通过伪指令`#pragma pack(n)`将编译器将按照`n`个字节对齐；结束时通过`#pragma pack()`取消自定义字节对齐方式。另外，对于结构体，还可以：`__attribute((aligned(n)))`结构成员对齐在`n`字节自然边界上，如果有成员长度大于`n`，则按照最大成员的长度来对齐；`__attribute__((packed))`取消编译过程中的优化对齐，按照实际占用字节数进行对齐。

当`#pragma pack`的`n`值大于等于所有数据成员长度的时候，这个`n`值的大小将不产生任何效果。

