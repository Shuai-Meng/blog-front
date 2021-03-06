---
title: Java总结——数据类型
abbrlink: 4899
date: 2019-03-02 15:35:26
tags: java
categories: 技术
---

### 基本数据类型与引用类型
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Java是面向对象的语言，在java中一切都可以视为对象。然而，为什么却会引入基本数据类型这样一个明显不是对象的元素呢？个人觉得有两个方面的原因：一是逻辑上的必然结果；二是对计算机本身结构的妥协。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;面向对象编程的基本原则：一切都是对象；对象可以分解为对象（我胡说的，没有验证过）。这意味着，对象的组成成分也是对象，每个对象都是由其他对象构成的。然而在实际编程时，却是自底向上的构建更复杂的对象，而不是自顶向下的逐步分解对象。但是在这个规则下，却没办法找到这样底层对象，因为它还可以分解为更底层的。这样，在逻辑上就陷入了寻找最细粒度对象的死循环中。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;解决办法就是破坏规则：
1. 引入一种不可分割的元素，它不是对象，并且最“底层”的对象完全由这种元素构成，这样就破坏了规则一；
2. 或者使一些对象不可以继续分割，其他对象都有这种对象构建而成，这就破坏了规则二。两种方案的共同点是终止“对象可以无限分割”这个死循环。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;显然Java采用了第一种方案，在语法中加入了基本数据类型这种非对象元素。Java的前辈，SmallTalk语言采用了方案二。表面上看，SmallTalk中似乎一切都是对象了，被称为“纯面向对象”语言。然而，这是构建在一种特殊对象基础之上的，这些基础对象是否真的符合对象的定义值得怀疑。其他所谓“纯面向对象”的语言同理。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Java采用方案一的另一个原因是受制于计算机的结构。长期实践中，编程领域基本上形成了一些基本的数据类型，如int，float，char等，这些基本数据类型可以被计算机很好的支持，甚至计算机中的很多指令都是和这些数据类型耦合的。所以任何一门编程语言，都逃脱不了基本数据类型的限制。Java在此基础上，规定了8种基本数据类型，作为所有对象的基础；当然java的基本数据类型是与平台无关的，屏蔽了底层细节，比如int类型无论在哪种机器上对外表现都是四个字节。其他“纯面向对象”的语言，最底层对象本质上就是这些基本数据类型，只不过稍微加了一层包装，让其看起来像个对象而已。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Java为什么没采用方案二呢？这是考虑到了执行效率，直接使用计算机原生的数据类型，比将其包装成对象运算要快得多——因为原生数据类型可以直接保存到栈上，而对象则保存在堆上，通过保存到栈上的指针或引用去获取——显然获取对象数据因为绕了一个弯，而损失了一部分效率。
 
<!-- more -->

### 基本数据类型
在java中一共有8种基本数据类型，其中整数类型有四种：byte，short，int，long；浮点数类型：float，double；布尔类型：boolean；字符类型：char。
#### byte
byte类型表示一个整数， 一个byte类型的变量可以存放一个字节的数据，一个字节等于8个比特，即1 Byte = 8 bit，所以可以表示的数的范围就是0000 0000 ～ 1111 1111。而java中所有的数字类型都是有符号数，于是开头的比特位用来表示正负，0代表正数，1代表负数。所以实际能表示的数的范围是：1111 1111 ～ 0111 1111，用十进制表示就是 -127 ～ 127。这好像跟教材说的不一样：难道不应该是 -128 ～ 127 ？这是因为刚才所述使用的是二进制的原码表示法，简单直接，容易理解。然而Java使用的是补码表示法，在形式上，负数需要在原码的基础上将各个数据bit位取反（不包括符号bit位），然后加1；而正数继续使用原码。于是 -127应该用二进制表示应该是1000 0001，127则是0111 1111。1000 0001可以继续减1得到1000 0000, 所以下限应该是 -128（即-127 - 1）；但0111 1111却不能再加1了，因为
0111 1111 + 1 = 1000 0000 = -127 
 这意味着，一旦采用了补码表示法（限定二进制长度为8位），从逻辑上，1000 0000只能表示-127, 而不是128。
0 = -127 + 127 = 1000 0001 + 0111 1111 = 1 0000 0000 = 0000 0000 （溢出，舍去最高位）
-1 = -128 + 127 = 1000 0000 + 0111 1111 = 1111 1111
注意，这里说的仅仅是补码表示法的表面运算规则，而底层原理请参考[这篇文章](https://www.cnblogs.com/zhangziqiu/archive/2011/03/30/ComputerCode.html)。
byte类型是最小的数据类型，只有一个字节，即8个比特。虽然表示信息的最小单位是比特，但是计算机操作信息的最小单位却是字节，即每次运算时，都是一次性操作至少一个字节，因为以比特为操作单位的话，太浪费效率了。所以，无论是内存还是硬盘，或者CPU，传输信息时，都是以字节为单位，即使存在一定的浪费（有些信息用不了一个字节的容量）。
在实际编程中，很少有直接操作byte变量的场景；使用byte一般都是直接定义一个byte数组，用来存放二进制数据，比如典型的IO操作。直接操作byte的画风可能是这样的：
```java
public class Byte {
    public static void main(String[] args) {
        byte b = 128;
    }
}
```
编译结果如下：
```shell
Byte.java:5: 错误: 不兼容的类型: 从int转换到byte可能会有损失
        byte b = 128;
                 ^
1 个错误
```
这是因为整型数字的字面量默认都是int类型（4个字节）的，如果赋值给比int类型短的的类型（byte和short等），这个字面值的范围必须要在要求类型的范围内，比如byte类型的变量只能赋值-128～127之间的整数，超出这个范围就会编译报错。
另外，还可以用十六进制数字直接给整型变量赋值，这是从Java SE7开始有的特性，如：
```
byte a = 0x1f
```
语法上，十六进制数以“0x”开头，大小写随意，后面的十六进制数字同样如此。这里稍微说一下字节和十六进制的关系，一个字节8个位，而一个十六进制数字最多要用4个位表示（2的 4次 方，正好16），故而一个字节可以容纳两个十六进制数字。

#### short
short类型表示一个有符号整数， 一个short类型的变量可以存放两个字节的数据，所以可以表示的数的范围就是0000 0000 0000 0000 ～ 1111 1111 1111 1111，翻译成补码形式的十进制就是-32768~32767。
short类型在平时的编码中还是很少用到的，没有必要为了节省空间而特意使用short类型。
下面写个程序验证一下，short是不是真的只占两个字节。
```
package com.explore;

public class Size {
    public static void main(String[] args) {
        System.out.println(Runtime.getRuntime().freeMemory());
        short[] tmp = new short[10000];
        for (int i = 0; i < tmp.length; i++) {
            tmp[i] = 1;
        }
        System.out.println(Runtime.getRuntime().freeMemory());
    }
}
```
上面的程序中，Runtime.freeMemory()方法可以获取到当前jvm进程所拥有的空闲内存。注意，这个内存是总内存，包括堆内存和栈内存等。第一次打印到第二次打印之间，占用内存的包括一个short类型数组以及这个数组的引用。其中引用，也就是tmp变量分配在栈上，数组本身分配在堆中。而for循环中定义的临时变量i，在for循环结束后就释放了（google一下局部变量的作用域与生存期），相当于没有占用内存。
执行过程如下：
```
➜  java javac com/explore/Size.java                    
➜  java java -Xms1025k -Xmx1025k -cp . com.explore.Size
1174728
1154712
```
执行的时候，通过-Xms和-Xmx参数限定了jvm的堆内存就是1025K，这个数值是能让jvm启动的最小值（还没研究过这个值是怎么来的）。打印结果表明数组以及数组引用一共占据了20016个字节的空间。可以猜到，引用占了16个字节，数组占了20000个字节；而数组中存放了10000个short类型的整数，所以每个short类型的变量占用两个字节。
为了证明上述猜想，将程序中的数组长度改为5000,再次执行的结果如下：
```
➜  java javac com/explore/Size.java                    
➜  java java -Xms1025k -Xmx1025k -cp . com.explore.Size
1174728
1164712
```
就是一个二元一次方程组，解出来即可。
注意，上述程序中，数组的长度不可太小，否则测试将出现偏差，具体原因尚未明确。同样的道理也可以测出其他基本类型的实际容量或大小。 另外，在JDK1.8中，基本类型的包装类有个BYTES属性，直接指明了其对应的基本类型所占据的字节数；JDK1.5添加的SIZE属性也指明了所占据的位数。

#### int
int属于最常使用的整数类型，而且java中规定了整数的字面量默认就是int类型的。int类型占用四个字节，其能表示的整数范围是：0000 0000 0000 0000 0000 0000 0000 0000 ～ 1111 1111 1111 1111 1111 1111 1111 1111，翻译成十进制就是 -2147483648 ～ 2147483647，大概是-21.47亿到21.47亿，大部分情况下用到的数不会超出这个范围。
上面提到了字面量这个概念，什么是字面量？说白了，字面量就是值。我们常说给变量赋值，变量的本质是内存的一块空间，而字面量就是这块空间存放的具体内容。字面量表示的是一个固定的值，代表了程序中初始的输入或状态。一个程序中，不可能所有的元素都是变量，它总是需要从外界获取初始的输入——网卡，键盘，或者直接在代码中预先定义。离开计算机领域，字面量就是人类日常书写用到的全部文字，包括数字以及字符串；只是到了计算机领域后，需要严格的和变量等其他编程元素区别开来。字面量是人类和计算机直接沟通交流的方式。

#### long
long类型也是比较经常使用的整数类型，尤其是数目比较大以至于int无法容纳时。long类型占用8个字节，能表示的整数范围是：-9223372036854775808~9223372036854775807，大概是922.33亿亿，真的好大，我写代码遇到过使用long类型的场合一般也就是各种数据库表的id字段了。
从编译器的角度，源代码就是一堆字符串，字面量和变量都是这堆字符串中的符号而已，只是有的符号被编译成变量有的被编译成字面量，取决于编译器的实现。上面提到java中整数类型的字面量，其默认的类型是int。那么给long类型的变量赋值的时候，字面量的值如果超过了int类型多能表示的范围时，需要在数字后面添加l或L(建议大写，容易识别)，以表明这个数字字面量是long类型的。如下：
```
long a = 2147483648L
```

#### char
char类型的变量代表一个字符，虽然在底层存储的时候还是二进制，然而取出来解读的时候，却表示成一个字符。比如下面这段代码：
```
char a = 't';
System.out.println(a);
```
最终打印到屏幕上的，会是字符“t”。这其中用到了两种技术：字符编码和字符集，那是另一个话题了。
在语法上，字符字面量需要用单引号括起来，这是因为源代码本身已经是字符（串）了，所以要在源码中表示另外的‘字符’，需要加以区别，告诉编译器这个字符是字符字面量，而不是其他语法元素。字符串用双引号括起来是一样的道理。

char类型和byte类型的大小是一样的，都是1个字节。
#### boolean
boolean类型表示布尔变量，在java中，它的值只能是两个字面量之一：true或false，或者这个两个单词的全大写。这种类型表示逻辑判断中的真假，在C语言中，任何非0值都可以表示为true，而0表示false；或者null表示false，非null表示true；但是在Java中，真值被单独抽象出来，成为一个类型，不再用其他类型“兼职”。boolean类型的大小同样可以通过上面的程序测试出来，结果是1 byte。

#### float
float类型的变量用来存放浮点数，大小为4个字节。先说下浮点数在计算机中的二进制表示法，再确定它所能表示的数的范围。参考了[阮一峰的文章](http://www.ruanyifeng.com/blog/2010/06/ieee_floating-point_representation.html)，浮点数二进制采用科学计数法，公式如下：
V = (-1) ^ s * M * 2 ^ E
其中：
(-1) ^ s决定正负，s为0时是正数；s为1时是负数；
(2) 2 ^ E 代表指数，其中E是无符号整数，也就是只包含正数和0；
(3) M 代表有效数字，且 1<= M < 2。
float类型占用4个字节，也就是32个bit，它们是这样分配的：第一个bit代表符号，就是s的值；接下来的8位代表指数，也就是E的值；剩下的23位代表有效数字，也就是M的值。

由于M>=1，而E>=0，所以M * 2 ^ E >= 1，这样就没办法表示0和1之间的小数，所以E的取值必须包含负数，方法就是增加一个偏移量P，P的大小是2 ^ (t - 1) - 1，t表示E所占用的bit位数，偏移的方向是负。也就是，在计算指数部分的时候，实际的大小为2 ^ (E - P)。对于float类型，P=2^(8-1) -1 = 127，而E的范围是0～255，所以幂数e的范围是-127～128。

1<= M < 2的意思是，通过将原数左移或右移，使得其总是只保留一位整数数字，例如原数是11.01011，可以将其右移一位变成1.101011，对应的指数部分同时扩大2倍，整体大小保持不变。由于整数部分是固定的1,于是可以将其省略，M只保存小数部分，取值的时候，自动加1即可，这样可以多出来一个bit，提高精度。
但是有几种特殊情况，指数取偏移的方法就行不通了，需要特殊处理：
（1）E全为1,如果此时M也全为0,则V表示正负无穷大；否则，M不全为0,则V不是一个数，用NaN表示。这意味着，上面的e不可能取到128。
（2）E全为0,则此时e = 1 - P = -126，计算M时不再加1,这样是为了表示0以及非常接近0的数。

举个例子：
有个整数转换成二进制后是101.11001，那么存储它的时候，首先移位：1.0111001 * 2 ^ 2，然后指数部分的E = 2 + 127 = 129，尾数部分的M = 0111001，符号是0，所以V = 01000000 10111001 00000000 00000000
逆运算也是简单的，这里就不举例了。然后谈一谈二进制和十进制小数之间的转换。比如101.11转换成十进制应该是：1 * 2 ^ 2 + 0 * 2 ^ 1 + 1 * 2 ^ 0 + 1 * 2 ^ (-1) + 1 * 2 ^ (-2) = 5.75，显然小数部分是一个无穷级数1/2 + 1/4 + 1/8 + 1/16 + 1/32 + ……，用这个级数去无限逼近想要的小数，但是除了一些特殊的小数之外，绝大部分都只是近似，而且由于M的长度是有限的，所以也不可能无限逼近，只能有限逼近。比如计算机其实无法真正的表达0.1，只能用0.0625 + 0.03125 + ...去近似，这也算是二进制的弊端吧。

了解了浮点数在计算机中的存储机制后，下面将探讨一下float类型的范围，方便起见，只讨论V > 0的情况，负数只需取反即可。首先需要确定float可以表示的最小的正数，显然指数的下限只能是2 ^ (-126)，尾数M此时小于1，所以只保留最后一个bit位是1即可（其他bit都是0），由上面的例子可知，此时M = 2 ^ (-23)，从而得出V的最小值是2 ^ (-149)。小于这个数float就无能为力了；当然，即使大于这个数，float能表示的数也是不连续的，绝大部分只能近似。对于最大值，指数部分不可能取到 2 ^ 128，最大只能是2 ^ 127，而尾数M全为1，整体（加1后）也是小于2的（这个无穷级数收敛到1），于是最大值就是2 ^ 128(小于)。
参考[文章](http://cenalulu.github.io/linux/about-denormalized-float-number/)
综上，float变量V的取值范围是  -2 ^ 128 < V <= -2 ^ (-149)  || V = 0 ||  2 ^ (-149) <= V < 2 ^ 128 
另外在语法上，浮点数的字面量，其默认类型为double，所以给float类型的变量赋值时，需要在字面量的后面加上大写或小写的字母“f”。
计算机的存储资源是有限的，而浮点数也就是数学中的实数是无限的，所以计算机中的浮点数其实是对实数的近似，而非完全相等，这就产生了精度的问题。比如，两个浮点数之间做比较，不能直接使用==运算符，这将产生误差。如下：
```
System.out.println(1f == 0.99f);
System.out.println(1f == 0.99999999f);
```
将打印如下结果：
```
false
true
```
再比如浮点数的运算结果不能轻易取用：
```
System.out.println(0.01f + 0.05f);
System.out.println(1.0f - 0.9f);
```
直接取用可能会造成严重后果，尤其是在商业计算中：
```
0.060000002
0.100000024
```
#### double
double类型同float类型一样，用来表示浮点数，但是它有8个字节，所以能表示的数的范围更大一些，精度也更高。double类型的8个字节是这么使用的： 首bit依旧表示符号，指数部分占据11位，剩下的52位是尾数。
计算规则同float，所以可以得出double类型浮点数的范围是：
- 2 ^ 1024 < V <= - 2 ^ (- 1074) || V =0 || 2 ^ (- 1074) <= V < 2 ^ 1024 

关于上面提到的精度问题，有以下几种解决方案：
（1）运算使用BigDecimal类，牺牲一定的运算速度，但是提升了精度；
（2）比较大小，可以直接使用“<”和“>”；
（3）比较相等，不能直接使用“==”，可以考虑Math.abs()方法，包装类的一些api，如：
        Double.compare(d1, d2) == 0)
        Double.doubleToLongBits(d1) == Double.doubleToLongBits(d2)
        Double.valueOf(d1).equals(d2)

### 引用类型（对象）
引用类型的变量并不是对象的本身，可以将其视为保存了一个地址，通过这个地址，就能找到对象在内存中的起始位置。地址也是数据，所以引用类型的变量也需要占据一定的内存空间，但是这个空间有多大呢？这个问题可以等同于，内存到底有多大？
但是内存的大小不是确定的，随时可以通过添加内存条的方式来扩大内存，这时内存地址显然更多了，那么原来的引用类型变量是否足够大以便容纳新增的地址范围呢？
通过上面的测试程序，可以得出在我的电脑上，引用类型变量的大小是4个字节。然而，java是跨平台的语言，所以无论在什么样的机器上，引用类型的变量的大小是固定的，都将是4个字节。
然而，固定的空间大小如何来应对内存的变化呢？我猜一种可能是，引用类型变量所存储的地址其实是抽象的，不是真实的物理地址，最后将由操作系统映射到真实物理地址上，所以不用担心内存大小的变化。

