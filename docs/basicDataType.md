### 二进制简介
假设有一 int 类型的数，值为5，那么，我们知道它在计算机中表示为：  
```
00000000 00000000 00000000 00000101
5转换成二制是101，不过int类型的数占用4字节（32位），所以前面填了一堆0。
```  
#### 负数的二进制(补码=反码+1)
-5在计算机中如何表示？  
```
在计算机中负数以其正值的补码形式表达
比如 00000000 00000000 00000000 00000101 是 5的 原码。
反码：将二进制数按位取反，所得的新二进制数称为原二进制数的反码。
11111111 11111111 11111111 11111010 是 00000000 00000000 00000000 00000101 的反码。
补码：反码加1称为补码。
也就是说，要得到一个数的补码，先得到反码，然后将反码加上1，所得数称为补码。
11111111 11111111 11111111 11111010 + 1 = 11111111 11111111 11111111 11111011  
-5在计算机中表示为：11111111 11111111 11111111 11111011  

```  

### 进制间转换
```
十进制转二进制
转二进制就除2把余数都标在右侧
最下面的余数放在最左面
例20的二进制 
20...0  20除2商10余0
10...0  10除2商5余0
5....1  5除2商2余1
2....0  2除2商1余0
.....1

--------->0000 10100


十六进制转二进制  
0x12--->0001 0010
0x1234-->0001 0010 0011 0100

二进制转十六进制  
0001 0010 0011 0100-->0x1234

```

### 基本数据类型占的位数
```
bit: 1 bit位 = 1 二进制数据

byte: 1 byte = 8 bit位 （-128 ~ 127）

字母: 1 字母 = 1 byte = 8 bit(位)

short: 16位

char: Unicode字符，16bit位

int: 32bit位，比如int 类型占用4个字节，32位

long: 64bit位

float: 32bit位

double: 64bit位

string:

汉字：1 汉字 = 2 byte = 16 bit 

《《***这里不是很准确，当编码不同的时候，1个汉字所占的字节数也会有所不同，有些编码是占 2个字节，有些则不是，可能是 3个或者 4个***》》
```  


###  与运算&0XFF
```
&作为位运算 （还可以作为逻辑运算符）  
操作数的二进制形式，同为1结果才是1，其它则为0，操作数可以是二进制八进制十六进制
0xff的十进制是255，八位二进制11111111,一个数&上它应该是不变的，但是

系统监测到byte作为int类型向控制台输出的时候，会自动将byte的内存空间高位补1扩充到32位。&0xff即是取了低八位，尽管十进制数不同但是保持了二进制补码的一致

例：
0x1234   十六进制
0001 0010 0011 0100  二进制
0xff---->1111 1111
左侧补0---->0000 0000 1111 1111
运算结果取低八位 0000 0000 0011 0100


```  
有符号整数，正数要去除一个符号位，负数不用
```
计算机用位（bit）来作为基础单位
byte它的范围是-127~128,用8个bit可表示 1（2^8）~2^8-1 
1111 1111
short,同char
char，它的范围是0~2^15所以只能用16个bit,即两个byte
int,他的范围是-2^-31~2^31-1
long double  8byte 64bit 取值-(2^64)~2^63
float 4byte 32bit  
```  

```
byte类型的-127，其计算机存储的补码是10000001（8位），将其作为int类型向控制台输出的时候，jvm作了一个补位的处理，因为int类型是32位，所以补位后的补码就是1111111111111111111111111 10000001（32位），这个32位二进制补码表示的也是-127.虽然byte->int（八位扩展到32位），计算机背后存储的二进制补码由10000001（8位）转化成了1111111111111111111111111 10000001（32位）很显然这两个补码表示的十进制数字依然是相同的。

```  
### 移位运算  
```
<<:左移运算符，num << 1,相当于num乘以2
>>:右移运算符，num >> 1,相当于num除以2
>>>:无符号右移，忽略符号位，空位都以0补齐!
```
### 位异或运算符
```
位异或运算符为^，其运算规则是：参与运算的数字，低位对齐，高位不足的补零，如果对应的二进制位相同（同时为 0 或同时为 1）时，结果为 0；如果对应的二进制位不相同，结果则为 1。
11^7=12  (化成二进制然后计算)
```