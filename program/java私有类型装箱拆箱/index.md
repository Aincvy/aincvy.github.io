# Java私有类型装箱拆箱




### 简述

在尽可能的情况下， 都应该使用 java的**私有类型**， 而装箱类型应该少用。

因为 装箱的类型 会占用更多的内存。



### 内存占用相关的内容

| 私有类型 | 装箱类型 | 内存占用(私有类型/ 装箱类型) |
| -------- | -------- | :--------------------------- |
| boolean  | Boolean  | 1 byte / 16 bytes            |
| byte     | Byte     | 1 byte / 16 bytes            |
| short    | Short    | 2 bytes / 16 bytes           |
| char     | Char     | 2 bytes / 16 bytes           |
| int      | Integer  | 4 bytes / 16 bytes           |
| long     | Long     | 8 bytes / 16 bytes           |
| float    | Float    | 4 bytes / 16 bytes           |
| double   | Double   | 8 bytes / 16 bytes           |

> 1 byte 是一个字节 
>
> 1 byte = 8 bit
>
> 1 KB = 1024 byte

由表中的数据可知， 装箱类型的内存占用比私有类型大很多， 因为 装箱类型都是 `java class` 。每个对象都需要存储类信息。 

因此 从内存占用的角度来看，我们也应该避免使用装箱类型。 

- 一个数组 `float[5]` 只会占用 `32`个字节
- 一个数组`Float[5]` (全部都是非null对象) 则会占用`112`个字节。 如果是64位非压缩指针，则需要`152` 个字节。

由此可见 使用装箱类型则会使用很多不必要的内存占用。

- 使用较多的装箱类型 将会引发更频繁的 **GC** 。
- 更频繁的 **GC** 将会产生性能问题。 
- *附： 使用对象池也是为了尽可能的减少GC次数*



### 如果非要使用的话

*在使用泛型的时候， 我们不可避免的需要使用装箱类型，这个我暂时不知道如何更好的处理。*



使用 `valueOf()` 方法， 避免使用 `new` 关键字。  因为装箱类型在JVM 里面会有缓存，所以这样做会有效的减少内存的占用。

```java
var num1 = new Integer(10);      // in jdk13, 你会看到 'Integer(int)' is deprecated  这样的说明， 并且会建议你 使用 valueOf(x) 方法
var num2 = new Integer(10);     
var num3 = new Integer(10); 
// 此时会有 3个 Integer 的对象。  占用 差不多 3 x 8 = 24 个字节。

var num11 = Integer.valueOf(10);  
var num12 = Integer.valueOf(10);  
var num13 = Integer.valueOf(10);  
// 此时 num11,num12,num13   都是使用的同一个缓存的对象。 占用 1 x 8 = 8 个字节
```

默认情况下， JVM 会缓存 -128 ~ +127 这个数字范围的 Integer 对象。



### 其他内容

#### 关于浮点数 精度的问题

```java
var num1 = 1f;
var num2 = 0.99f;
var num3 = num1 - num2;
System.out.println("num3: " + num3);        // num3: 0.00999999
```

在上面的代码里面 , num3应该等于`0.001` 才对， 但是实际上输出的结果并不是。

这和java的浮点数实现机制有关系， 虽然大部分情况下， 数据都是正常的，但是如果用的多， 那肯定会有误差。

这时该怎么办呢？ 

**方法一： 提升高度， 使用`int`或者`long`。**

比如 你在写一个交易系统的时候， 假如你使用`float`类型来存放用户的余额， 并且使用`元` 作为基本单位。 

*比如， 1.5  就是1块钱5角钱*

此时你想修改的话， 就把基本单位 修改成`分`, 数据类型修改成`long` 就好了。

*比如， 1.5元就 记录成 150。 这样就会避免计算错误的情况*

因为`int/long` 类型不是浮点数， 没有这样的计算问题，所以就可以避免了。 

**方法二： 使用`BigDecimal` 类型**

这个方法在使用的时候需要很多 `BigDecimal`实例， 除非用于计算很大的数字，否则我建议你使用方法一。 



#### 关于反射

`int.class` 和 `Integer.class` 是不同的，其他类型应该也是一样。 



#### 随意吐槽

*学习编程相当长一段时间后，我才知道原来`java`和`javascript`根本没有半毛钱关系，那你们干嘛起类似的名字呀😂😂😂*









---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/java%E7%A7%81%E6%9C%89%E7%B1%BB%E5%9E%8B%E8%A3%85%E7%AE%B1%E6%8B%86%E7%AE%B1/  

