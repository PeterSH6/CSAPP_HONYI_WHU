<center><font size="5">Second Homework of CSAPP</font></center>
<center><font size="4">2019 11.3 </font></center>
<center><font size="4">生广明</font></center>
<center><font size="4">2018300003086 </font></center>

##### 3.60

x: %r8

n: %ecx

result: %rax

mask: %edx

Test condition for mask :  mask != 0;

How does mask get updated: mask = mask << n

How does result get updated:  result |= (x & mask);

```c
long loop(long x, int n) {
    long result = 0;
    long mask;
    for (mask = 1; mask != 0; mask = mask << n) {
        result |= (x & mask);
    }
    return result;
}
```

##### 3.62

```c
long switch3(long *p1, long *p2, mode_t action) {
    long result = 0;
    switch(action) {
        case MODE_A:
            result = *p2;     
            *p2 = *p1;         //汇编代码中用了两个movq，代表memory->memory的mov操作需要两步
            break;            //汇编代码中的ret对应每个case中的break；
        case MODE_B:
            result = *p1 + *p2;
            *p1 = result;
            break;
        case MODE_C:
            *p1 = 59;
            result = *p2;
            break;
        case MODE_D:
            result = *p2;     //此处虽然看似也是memory->memory 但是用的中间寄存器是%rax，代表result，所以需要写成两步的c语言代码
            *p1 = result;
            result = 27;
            break;
        case MODE_E:
            result = 27;
            break;
        default:
            result = 12;
    }

    return result;
}
```

##### 3.65

```assembly
.L6:
movq (%rdx), %rcx       // %rdx 对应于 A[i][j]
movq (%rax), %rsi       // %rax 对应于 A[j][i]
movq %rsi, (%rdx)       // A[i][j] = A[j][i]
movq %rcx, (%rax)       // A[j][i] = A[i][j]
addq $8, %rdx           // %rdx += 8
addq $120, %rax         // %rax += 120
cmpq %rdi, %rax         //
jne .L6                 
```

A,B(见注释处)

C. The value of M: %rdx + 8 表示数组元素右移一位，然而%rax + 120 表示数组元素向下移一位，说明M的值为M = 120 / 8 = 15

##### 3.68

```assembly
p in %rdi, q in %rsi
setVal:
    movslq 8(%rsi), %rax        // %rax = *(8 + q)
    //
    addq 32(%rsi), %rax         // %rax += *(32 + q)

    movq %rax, 184(%rdi)        //我们可以从此语句推断出来(%rdi+184)对应p->y 因此根据struct的性质，4*A*B + space = 184
    //我们可以得到一个不等式 45<= A*B <= 46
		//因为两个结构体中都有long,因此都将按照8字节为最小单位来进行补齐。
		//所以我们可以推断出char array[B]所占用的字节数小于等于8
		//即 B <= 8;
		//由第二行addq指令我们可以推断出，short s[A]所占用字节数小于等于20
		//即 A <= 10;
		//由以上三个不等式我们可以解出唯一整数解： A = 9, B = 5。
```

##### 3.70

A.

| val     | offset |
| ------- | ------ |
| e1.p    | 0      |
| e1.y    | 8      |
| e2.x    | 0      |
| e2.next | 8      |

B.

16

两个结构体所占用的最大字节数均为16 所以这个联合体仅需要占用16 bytes

C.

Answer

```c
void proc(union ele *up) {
  up->e2.x = *( *(up->e2.next)->e1.p ) - *(up->e2.next)->e1.y
}

```



```c
  union ele {
  struct {
    long *p;
    long y;
  } e1;
  struct {
    long x;
    union ele *next;
  } e2;
};


 void proc(union ele *up)
 up in %rdi
proc:
  movq 8(%rdi), %rax
  movq (%rax), %rdx
  movq (%rdx), %rdx
  subq 8(%rax), %rdx
  movq %rdx, (%rdi)
    /* 
    由第一条和第二条汇编movq我们可以推断出，%rax寄存器将要存放一个地址，且这个地址为%rdi + 8 即为两个struct中的第二个元素。第二个元素存放地址的（指针）变量仅e2.next。所以%rax对应e2.next。
    由第三条汇编movq我们知道e2.next的第一个元素也是pointer,所以即为*p，
    所以括号内即为*(up->e2.next)->e1.p
    因为此时%rdx存放的是一个long，因此8（%rax）意味着，第二个元素必须为一个long。因此必e1.y，所以减号后面填  *(up->e2.next)->e1.y 
    而此时（%rdi）指向第一个元素，必须为long，所以可以推断出为e2.x
    */
  ret
```