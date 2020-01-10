# First Homework of CSAPP

##### 2.63

 Function srl: After the first step of the srl func, the most significant k bits will have k ones or zeros. Since performing logical right shift, we need to keep 0 as 0 and transform 1 to 0(in the k new bits), what's more make other bits of xsra remain unchanged. We can implement this idea by create k bits of 1in the most significant bits and do xor with xsra. 
Using -1<<w-k can create k bits of 1 in the most significant bits. 

Function sra: First, we need to know the most significant bit of x is 1 or 0. We can use x&INT_MIN to determine. If it is 0, then we can just return xsrl. Otherwise, we need to change k bit of 0 to 1 using | with mask.

```c
#include<stdio.h>
#include<limits.h>

unsigned srl(unsigned x, int k)
{
  unsigned xsra = (int) x >> k;
  int width = 8*sizeof(int);
  int mask = (-1)<<(width-k);
  xsra &= ~mask;
  return xsra;
}

int sra(int x, int k ){
  /*Perform shift logically */
  int xsrl = (unsigned) x >> k;
  /*if(x&INT_MIN)
  {
    int width = 8*sizeof(int);
    int mask = (-1)<<width-k;
    xsrl |= mask;
  }*/
   int width = 8*sizeof(int);
   int mask = (int)-1<<(width-k);
  (x&INT_MIN)&& (xsrl |= mask);
  return xsrl;
}
```

##### 测试代码：

```c
int main()
{
	printf("%X\n", srl(0xffffffff,8));
	printf("%X\n", sra(0xffffffff,8));
	printf("%X\n", sra(0x7fffffff,8));
	printf("%X\n", srl(0x7fffffff,8));
	printf("%X\n", srl(0x8fffffff,8));
	printf("%X\n", sra(0x8fffffff,8));
	
}
```

##### 测试结果：


As for the first two test, MSB is 1 so srl just add 0 to the first 8 bits while sra will add 1 to the first 8 bits.

As for the 7fffffff, MSB is 0 so srl and sra just add 0 to the first 8 bits.

As for the 8fffffff,MSB is 1 so srl just add 0 to the first 8 bits while sra will add 1 to the first 8 bits.



##### 2.71

A. If the 8-bits int is a negative number, after we operate(word>>(bytenum<<3)), MSB maybe all 1.

However after we use & with 0xff, the MSB will be 0 and it will turn into a postive number, which is wrong.

B.  First we need to use << to eliminate all the bits on the left side of the int that we want.

3 - bytenum represent the bytenum on the left and we use << 3 to make it to bits level.

Second we need to use >> to eliminate all the bits ont the right side of the int.

Since the 8 bits we want is on the 8MSB and there are 24 bits on the right. They are useless for us so we just >> 24.

```c
#include<stdio.h>
typedef unsigned packed_t;

int xbyte(packed_t word,int bytenum)
{
	int right_move = (3 - bytenum)<<3;
	int left_move = 24;
	word = word << right_move;
	word = word >> left_move;
	return word;
}
```

#####  测试用例

```c
int main()
{
	printf("%X\n",xbyte(0xc431e3ff,0));
	printf("%X\n",xbyte(0xc431e3ff,1));
	printf("%X\n",xbyte(0xc431e3ff,2));
	printf("%X\n",xbyte(0xc431e3ff,3));
	return 0;
}
```

##### 结果


We can see that the function return what we want.

##### 2.72

A: Because sizeof(val) will return size_t, which is always be unsigned int. Since the maxbytes is int, so it will cause implicit casting to unsigned and make the expression be true.

B: We should forbid the implicit casting from int to unsigned so we must explicitly cast unsigned to int.

```c
if(maxbytes - (int)sizeof(val) >= 0)
```



##### 2.73

If it is positive overflow, MSB of sum would be 1 while MSB of x and y would be 0. So if these two conditional test is true we just assign TMax to sum.

If it is negative overflow,MSB of sum would be 0 while MSB of x and y would be 1. So if these two conditional test is true we just assign TMin to sum.

Others, we just return sum.

```c
int saturating_add(int x,int y)
{
  int sum = x + y;
  int w = sizeof(sum)<<3;
  int mask = (-1)<<(w-1);//used to judge postive(0) and negtive(1)
  (sum&mask)&&(!(x&mask))&&(sum = TMax);
  !(sum&mask)&&(x&mask)&&(sum = TMin);
  return sum;
}
```

##### 测试用例：

```c
int main()
{
	printf("%d\n", saturating_add(10,15));
	printf("%d\n", saturating_add(0x7fffffff,99999999));
	printf("%d\n", saturating_add(0x80000000,-9999999));
	printf("%d\n",TMax);
	printf("%d\n",TMin);
}
```

If the sum is overflowed it will get TMax or TMin as expected.

##### 2.82

A: (x < y) == (-x > -y)		False. If x = INT_MIN, x == -x == INT_MIN. 取反加一.

B, True  (x+y) <<4 + y - x = x<<4 + y<<4 + y - x= 16x + 16y + y - x = 17y + 15x  so it is equal.

C, True, Since we know that -x == ~x + 1 .we can add 1 to both sides of the equation and then  we can get -x - (-y) == -(x+y)

D,True, ux - uy=unsigned(x - y) = - (unsigned)(y - x)

E,Ture, We know that we may lose some 1 during the >> while << just give back some 0 so the outcome of the operation will be <= x itself.







##### 2.88

| Format A  |          | Format B  |          |
| --------- | -------- | --------- | -------- |
| Bits      | Value    | Bits      | Value    |
| 101111001 | -9/8     | 101110010 | -9/8     |
| 010110011 | 11*2^4   | 011100110 | 11*2^4   |
| 100111010 | -5*2^-10 | 100001010 | -5*2^-10 |
| 000000111 | 7*2^-15  | 000000000 | 0        |
| 111100000 | -1*2^13  | 111101111 | -248     |
| 010111100 | 3*2^7    | 011110000 | +∞       |



##### 2.89

A: (float)x == (float)dx		True. 

B: dx - dy == (double)(x-y)	Faluse. x-y may cause overflow while dx - dy would not cause overflow. EG: x = INT_MIN and y = INT_MIN.

C: (dx + dy) + dz == dx + (dy + dz)		True. Because double can represent up to 2^53 of int precisely  and + can't overflow these number.

D: (dx * dy) * dz == dx * (dy * dz)		False. Because * may overflow 2^53 so it may cause problem. 

E: dx/dx == dz/dz		False. dx == 0 or dz == 0.

##### 2.90

```c
float fpwr2(int x) {
  /* Result exponent and fraction */
  unsigned exp, frac;
  unsigned u;

  if (x < -149) {//-23-126 = -149
    /* too small. return 0.0 */
    exp = 0;
    frac = 0;
  } else if (x < -126) {//(1-2^-24)*(2^-126)约等于 = 2^-126
    /* Denormalized result */
    exp = 0;
    frac = 1<<(149+x);//因为x = -126-23+k,k表示从右数第k位，所以需要1<<k位
  } else if (x < 128) {//2^8 -1 - 2^7 +1 = 128
    /* Normalized result */
    exp = x + 127; // E = exp - bias so exp = E + bias. E is x.
    frac = 0;
  } else {
    /* Too big, return +oo */
    exp = 0xFF;
    frac = 0;
  }
```




