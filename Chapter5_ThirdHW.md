#Third Homework of CSAPP


##### 7.6

此题目的解题分析为代码的注释

```c
extern int buf[]; //buf为引用的外部变量,在main中defined，所以为extern，又因为在main中已经被初始化，所以应该在.data Section中

int* bufp0 = &buf[0]; //global变量，且已经初始化，所以在.data section,并且在.symtab中
static int* bufp1; //课本中原文：Static C functions and global variables that are defined with the static attribute is a local symbols。且没有被初始化，因此在.bss中

static void incr() {//课本原文：Static C functions and global variables that are defined with the static attribute is a local symbols。此处为函数，所以在.text中
  static int count=0;//同bufp1，初始化为0，所以也在.bss中
  count++;
}

void swap() {//global c function,in .text
  int temp; //local variable in swap() function, 所以不会在symtab中出现。
  incr();
  bufp1 = &buf[1];
  temp = *bufp0;
  *bufp0 = *bufp1;
  *bufp1 = temp;
}
```

|       | **swap.o  .symtab** entry | Symbol type | Module | Section |
| ----- | ------------------------- | ----------- | ------ | ------- |
| buf   | Yes                       | external    | main.o | .data   |
| bufp0 | Yes                       | global      | swap.o | .data   |
| bufp1 | Yes                       | local       | swap.o | .bss    |
| swap  | Yes                       | global      | swap.o | .text   |
| temp  | No                        | ------      | -----  | ------  |
| incr  | Yes                       | local       | swap.o | .text   |
| count | Yes                       | local       | swap.o | .bss    |

##### 7.10

```c
A. p.o -> libx.a ->p.o
gcc p.o libx.a
//p.o中有依赖于libx.a的函数，所以先写p.o 再写libx.a。而libx.a中依赖于p.o的函数已经存放在E之中，所以不需要再写p.o

B. p.o -> libx.a ->liby.a and liby.a->libx.a
gcc p.o libx.a liby.a libx.a
//p.o libx.a liby.a的顺序由第一段库的依赖关系可以写出。但是由于liby.a中有依赖于libx.a的函数，而libx.a先于liby.a被linker解析，所以U集合非空，需要再将libx.a写入去reslove the functions.

C. p.o -> libx.a ->liby.a->libz.a and liby.a->libx.a->libz.a
gcc p.o libx.a liby.a libx.a libz.a
//由两段依赖关系可以推断出libz.a不依赖于之前出现的库而其他库对它有一定的依赖。所以libz.a写在最后且只出现一次。又因为libx.a与liby.a有相互依赖关系，所以libx.a出现两次。
```

##### 7.12

A.

ADDR(s) = ADDR(.text) = 0x4004e0

ADDR(r.symbol) = ADDR(swap) = 0x4004f8

refaddr = ADDR(s) + r.offset = 0x4004ea

*refptr = (unsigned) (ADDR(r.symbol) + r.addend - refaddr) = 0xa

so：0x 0a 00 00 00 00

B.

ADDR(s) = ADDR(.text) = 0x4004d0

ADDR(r.symbol) = ADDR(swap) = 0x400500

refaddr = ADDR(s) + r.offset = 0x4004da

*refptr = (unsigned) (ADDR(r.symbol) + r.addend - refaddr) = 0x22

so : 0x 22 00 00 00

