# Java字节操作的简单描述


## byte[]和 int 互相转换

```java
// 将 byte[] 转换成 int
byte[] input = {(byte)0x65, (byte)0xFF, (byte)0x31 };

// 使用大端的方式转换成 int ：  x == 0x65FF31
int x = (input[0] &amp; 0xFF) &lt;&lt; 16
  	| (input[1] &amp; 0xFF) &lt;&lt; 8
  	| (input[2] &amp; 0xFF) &lt;&lt; 0;

// 使用小端的方式转换成 int :  y  == 0x31FF65
int y = (input[0] &amp; 0xFF) &lt;&lt; 0
  	| (input[1] &amp; 0xFF)  &lt;&lt; 8
  	| (input[2] &amp; 0xFF)  &lt;&lt; 16;
```

#### 介绍一些概念

- 一个 int 由4个字节组成，分别记作  x0,x1,x2,x3 
- 一个字节由8个位(bit) 组成， 每个 bit的值只能为1或者0 
- 一个 int 由 32个位(bit) 组成。
- `|` 是按位或的操作符， 当第一项的第n位或者第二项的第n位的值为1 ，结果的第 n 位就是1 
- `&amp;` 是按位与的操作符， 当两项中的第 n 位的值都是1， 则结果的第 n 位的值就是1

#### 解释

在大端模式下， 组成 int 的字节数量不够的时候， 省略左边的值（*我猜的:joy:*）。

所以此时 `x0 = 0`  就无视了。 (*因为此时只存在3个字节*)

- 将第一个 byte 左移16位， 会让它占据在第二个字节的位置 （x1)。
- 将第二个 byte 左移8位， 会让它占据在第三个字节的位置（x2)。
- 将第三个 byte 左移0位， 会让它占据在 第四个字节的位置（x3)。 
- 使用按位或操作符 拼一下 就得到了一个正确的结果值。
- `0xFF` 的二进制就是 `1111 1111` 
- 将 `input[0]` 与 `0xFF` 进行按位与操作得到的应该就是`input[0]` 原值 (*还是我猜测的*)
- 如果需要给 x0赋值， 则将该字节 左移 24位即可。



在小端模式下， 只是字节放置的顺序变化了。 



参考阅读：

- [大小端模式 - 百度百科](https://baike.baidu.com/item/%E5%A4%A7%E5%B0%8F%E7%AB%AF%E6%A8%A1%E5%BC%8F)
- [Java 位运算(移位、位与、或、异或、非）](https://blog.csdn.net/xiaochunyong/article/details/7748713)



```java
// 将 int  转换成 byte[]
int x = 0x31FF65;

// 使用大端的方式转换:  { 0x65, 0xFF, 0x31}
byte[] c = {
  (byte) ( x &gt;&gt; 16 ),
  (byte) ( x &gt;&gt; 8),
  (byte) ( x &amp; 0xFF)
};


// 使用小端的方式转换:  { 0x31, 0xFF, 0x65 }
byte[] d = {
  (byte) ( x &amp; 0xFF),
  (byte) ( x &gt;&gt; 8),
  (byte) ( x &gt;&gt; 16)
};
```

#### 解释

- 右移 8位 转换成 `byte`类型， 会得到第二个字节。
- 右移 16位 转换成 `byte`类型， 会得到第三个字节。
- 右移 24位 转换成 `byte`类型， 会得到第四个字节。
- 因为 `int -&gt; byte` 的转变，是向下类型转换，所以会进行截断。 
- `x &amp; 0xFF` 会得到 第一个字节。  因为 `0xFF` 的值是 `1111 1111` 而按位与需要两项第 n 位都为1的时候结果才为1。 `0xFF` 可以理解成 `0x000000FF` 所以在这8个1之前还有24个0 ，那些值将会被忽略。



## 操作位

```java
bool f = (i &amp; 1 &lt;&lt; n ) != 0;      // 检测第 n 位是否为1
i |= 1 &lt;&lt; n;          		   // 将第 n 位设置为 1
i &amp;= ~(1 &lt;&lt; n);     		   // 将第 n 位设置为 0
i ^= 1 &lt;&lt; n;     		   // 切换第 n 位的值 （1变0， 0变1）
```



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/java%E5%AD%97%E8%8A%82%E6%93%8D%E4%BD%9C%E7%9A%84%E7%AE%80%E5%8D%95%E6%8F%8F%E8%BF%B0/  

