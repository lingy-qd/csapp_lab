# CSAPP Lab1: DataLab

记录：  
实验完成于2023/4/18，耗时约两天  
实验文档完成于2023/4/19


#### 1. bitXor
求异或，德摩根律
```c
int bitXor(int x, int y) {
  return ~(x&y) & ~(~x&~y);
}
```

#### 2. tmin
32位有符号整数的最小值：最高位是1、其余均0，在只能使用8位的情况下使用0x80u左移24位
```c
int tmin(void) {
  return 0x80u<<24;
}
```
#### 3. isTmax
32位有符号整数的最大值：最高位是0、其余均1，即0x7FFFFFFF  
Tmax+1=0x80000000，~Tmax=0x80000000，有Tmax+1=~Tmax的性质  
但是这里要排除x=-1的情况  
```c
int isTmax(int x) {
  /*
   * 利用Tmax+1=~Tmax
   * 但是要排除x=-1的情况
   */
  return !((~(x+1)^x))&!!(x+1);
}
```
#### 4.allOddBits
检查x的每一个奇数位是否都为1，直观想法是使用奇数位全为1的mask进行按位与操作，相当于检查每一个奇数位是否为1，再和这个mask进行异或操作，如果全0说明x的所有奇数位均为1  
获得奇数位全为1的mask的方法：0xAA先左移8位、或上0xAA得到0xAAAA，再左移16位、或上0xAAAA得到0xAAAAAAAA  
```c
int allOddBits(int x) {
  int a_8=0xAA;
  int a_16=(a_8<<8)|a_8;
  int a_32=(a_16<<16)|a_16;
  return !((x&a_32)^a_32);
}
```


#### 5. negate
输出相反数，这个很简单，按位取反再加一  
```c
int negate(int x) {
  return ~x+1;
}
```

#### 6. isAsciiDigit
判断x是否是Ascii数字，即判断0x30<=x<=39是否成立  
通过和0x30和0x39分别相减并判断符号位的方式来判断：x>=0x30 and x<=0x39  
```c
int isAsciiDigit(int x) {
  int sign1=x+(~0x30+1)>>31; // x>=0x31
  int sign2=0x39+(~x+1)>>31; // x<=0x39
  return !sign1&!sign2; // sign1>=0 and sign2>=0
}
```

#### 7. conditional
条件运算法（x?y:z）：if x return y else return z  
!!x相当于对x进行正则化，如果x非零，!x=0，!!x=1，mask=0xFFFFFFFF；如果x为零，!x=1，!!x=0，mask=0x00000000  
```c
int conditional(int x, int y, int z) {
  int sign=!!x;
  int mask=~sign+1;
  return (mask&y)|(~mask&z);
}
```

#### 8. isLessOrEqual
判断x<=y是否成立，因为x-y有可能溢出，所以不能直接判断(x-y)的符号位，需要分情况讨论  
溢出情况：若x>0、y<0，可能上溢导致(x-y)<0；若x<0、y>0，可能下溢导致(x-y)>0  
case1: if x<0 and y>0: return 1  
case2: if x>0 and y<0: return 0  
case3: if x<0 and y<0 and x-y<=0: return 1  
case4: if x>0 and y>0 and x-y<=0: return 1  
总结：  
case1&case2：x与y异号，不用计算x-y，根据x的符号位判断即可  
case3&case4：x与y同号，需要计算（x-y）的结果并根据符号位计算  
```c
int isLessOrEqual(int x, int y) {
  /*
   * 分情况讨论
   * if x<0 and y>0: return 1
   * if x>0 and y<0: return 0
   * if x<0 and y<0 and y-x>=0: return 1 
   * if x>0 and y>0 and y-x>=0: return 1
  */
  int signx=(x>>31)&0x01;
  int signy=(y>>31)&0x01;
  int c=signx^signy; // c=0 means x and y have the same symbol, c=1 means x and y have different symbols
  int res=y+(~x+1); // res=y-x
  int signr=(res>>31)&0x01;
  return (c&signx)|(!c&!signr);
}
```

#### 9. logicalNeg
实现逻辑运算符！：计算x|(~x+1)，只有x=0时式子为0，其他所有情况下最高位一定是1  
简单验证一下：  
x=0, ~x+1=0, x|(~x+1)=0  
x=Tmin，~x+1=Tmin，x|(~x+1)的最高位一定是1，因为Tmin是负数  
其他所有情况，x和它的相反数（~x+1）的最高位一定不同，因此x|(~x+1)的最高位一定是1  
```c
int logicalNeg(int x) {
  /*
  * if x=0, x|(~x+1)=0, hope to return 1
  * else the highest bit of x|(-x+1) must be 1, hope to return 0
  */
  int a = x|(~x+1);
  int sign=a>>31;
  return sign+1;
}
```

#### 10. howManyBits
// todo：为什么负数要按位取反，为什么最后还要加+1  
这里没想出来，参考了别人的答案和同学讨论了一下才想明白，需要巧妙地使用二分的思想  
首先，如果x是正数则保持不变，如果x是负数则按位取反，这样将正数与负数的处理统一起来  
下面分析一下二分思想：  
step1：如果高16位不为零，则b16=16，否则b16=0。  
step2：然后根据b16的情况进行移位：case1：如果高16位不为零，说明下面需要分析高16位的情况，直接右移16位，将高16位左移到低16位的位置；case2：如果高16位均为0，说明下面需要分析低16位的情况，此时不需要右移，因为需要分析的部分已经在低16位上了。  
```c
int b16=!!(x>>16)<<4; // 如果高16位不为零，b16=16，否则b16=0
x=x>>b16; // 如果高16位不为零，意味着不需要考虑低16位的情况，右移16位即可
```
下面使用相同的思路计算b8、b4、b2、b1、b0并进行移位。  
二分最后的汇总：  
b16=16说明高16位不为零、至少需要16位来进行存储，因此在程序末尾需要计算b16+b8+b4+b2+b1+b0+1  
```c
int howManyBits(int x) {
  int sign=x>>31;
  x=(~sign&x)|(sign&~x); // 如果x是正数则不变；如果x是负数按位取反
  int b16=!!(x>>16)<<4; // 如果高16位不为零，b16=16，否则b16=0
  x=x>>b16; // 如果高16位不为零，意味着不需要考虑低16位的情况，右移16位即可
  int b8=!!(x>>8)<<3;
  x=x>>b8;
  int b4=!!(x>>4)<<2;
  x=x>>b4;
  int b2=!!(x>>2)<<1;
  x=x>>b2;
  int b1=!!(x>>1);
  x=x>>b1;
  int b0=x;
  return 1+b0+b1+b2+b4+b8+b16;
}
```

下面都是浮点数的题目，浮点数的存储比较复杂，四种情况：  
1. 规格化浮点数：1<=exp<=254，尾数部分有隐含1  
2. 非规格化浮点数：exp=0；0.0：exp=0，frac=0  
3. 无穷、NaN：exp==255，无穷大：frac=0，NaN：frac!=0  

#### 11. floatScale2
输入按位表示的uf，返回2\*uf的按位表示  
需要分情况讨论：首先先取出sign、exp、frac  
case1：exp=0，说明是非规格化数，frac\*2即可  
case2：exp全1，说明是无穷大或者NaN，返回uf  
case3：规格化数，exp\*2，检查exp是否溢出  
```c
unsigned floatScale2(unsigned uf) {
  unsigned sign=uf&0x80000000;
  unsigned exp=uf&0x7F800000;
  unsigned frac=uf&0x007FFFFF;
  if(exp==0x0) return sign|frac<<1; // non-normalized：frac*2^(-126),所以frac*2即可, |sign保证符号相同
  if (exp==0x7F800000) return uf; // uf: infinite or NaN
  // normalized float
  exp=exp+0x00800000;
  if (exp==0x7F800000) return sign|0x7F800000; // overflow: return infinite
  return sign|exp|frac;
}
```
#### 12. floatFloat2Int
相当于强制类型转换，同样需要分情况讨论，首先取出sign、exp和frac，注意这里的frac补上了隐含1，实际上相当于右移了23位，指数E=exp-127  
case1：E<0，指数是负数，说明uf是个小于1的数，return 0  
case2：E>31，溢出，return 0x80000000  
case3：可以表示，但是需要判断E和23的关系（因为frac补上了隐含1），并根据关系进行调整，最后根据符号位返回即可  
```c
int floatFloat2Int(unsigned uf) {
  // 强制类型转换：float to int
  unsigned sign=uf&0x80000000;
  unsigned exp=uf&0x7F800000;
  unsigned frac=uf&0x007FFFFF|0x00800000; //补上隐含1，相当于默认左移了23位
  int E=(exp>>23)-127;
  int res=0;
  if (E<0) return 0; // 2^E的小数舍入为0
  if (E>31) return 0x80000000; // overflow
  res= (E<23) ? frac<<(23-E) : frac<<(E-23);
  if(sign) return ~res+1;
  return res;
}
```

#### 13. floatPower2
输入x，返回2^x的浮点数按位表示  
考虑非规格化数的情况下，可以表示的数据范围是多少？尾数部分的阶数是固定的：E=1-127=-126，尾数部分有23位，因此单精度浮点数能够表示的最小的数量级为2\^(-126-23)=2\^(-149)  
同样分情况讨论：  
case1：x<-149，精度不够无法表示，return 0  
case2：x<-126，非规格化数部分，用frac部分表示  
case3：x<128，规格化数，用exp表示  
case4：上溢出，返回+INF  
```c
unsigned floatPower2(int x) {
    if(x<-149) return 0; // 单精度浮点数精度不够
    if(x<-126) return 1<<(x+149); // non-normalized
    if(x<128) return (x+127)<<23; // normalized
    return 0xFF<<23; // overflow, return infinite
}
```
