# java字符串格式化


## 简介

本文主要会介绍下面几种格式化的方式。

- `String.format()`
- `System.out.printf()`      严格来说， 这并不是一种字符串格式化的方法。
- `MessageFormatter.format()`



### String.format()

这是一个 较为常用的 格式化方法， 类似 C语言的 `sprintf()` 函数。 

先给出几个示例 。

```java
// in jdk10&#43; ，低于jdk10的时候， var 换成 String 即可
int a = 1, b = 2 ;
var str1 = String.format(&#34;%d &#43; %d = %d&#34;, a , b , (a &#43; b) );  // 1 &#43; 2 = 3

float f = 1.5678f;
var str2 = String.format(&#34;f = %f&#34;, f);          // f = 1.5678
var str3 = String.format(&#34;f = %.2f&#34; , f);       // f = 1.56

var tmp1 = &#34;jack&#34; ;
var str4 = String.format(&#34;hello,%s&#34;,  tmp1);     // hello,jack

var p = new PlayerInfo();
var str5 = String.format(&#34;a player: %s&#34; , p) ;   // a player: xxxxx
// 此处 p 的内容取决于 以下几个情况
// 当 p = null 的时候， 输出 null
// 当 p != null 的时候，  PlayerInfo 实现了 toString() 方法， 则输出 toString() 方法的返回值
// 			 否则输出 PlayerInfo@xxxx  这种字符串 。 

var str6 = String.format(&#34;a: %d\tb: %d\t%%.2f= %.2f%n&#34;);
// a: 1    b: 2    %.2f= 1.56
// \t 表示制表符， 一般是 4个或者8个空格。 %% 表示 一个%  %n 表示换行(\n 或者\r\n  取决于系统)
```

看了上面的代码， 应该对这个方法有了一定的了解了， 下面来详细说明以下。 

`%`开头的 叫`占位符` , `\`开头的叫做`转义字符` 。

本文出现的 转移字符如下 。 

| 转移字符 | 含义                         |
| -------- | ---------------------------- |
| \t       | 制表，一般表示4个或者8个空格 |
| \n       | 换行   LF (Line Feed)        |
| \r       | 回车   CR (Carriage Return)  |
| \r\n     | windows 换行  CRLF           |

本文出现的 占位符如下

| 占位符 | 含义                        |
| ------ | --------------------------- |
| %%     | %                           |
| %s     | 字符串占位                  |
| %d     | 数字占位 (int,long,short)   |
| %f     | 浮点数 占位 (float, double) |
| %n     | 换行， 自动适应系统         |
| %b     | 布尔值                      |

**其他说明**

`%.2f` 是一种略微高级的用法， 用于保留两位小数 。 

看看下面的这段代码。

```java
public class Format {
    public static void main(String[] args) {
        System.out.format(&#34;%f, %1$&#43;020.10f %n&#34;, Math.PI);
    }
}

// output
// 3.141593, &#43;00000003.1415926536
```

`%1$&#43;020.10f` 这一长段的含义是 。

- %  格式开始的符号 ，也可以理解为占位符开始的地方。
- 1$   参数索引，也可以用 `&lt;` 符号来指定 前一个变量
- &#43;0   flags
- 20    宽度，低于这个宽度自动补0
- .10   浮点数长度
- f     转换字符， 代表 浮点数 



**日期**

格式化日期 ， 先看几个示例。

```java
Date date = new Date();
System.out.printf(&#34;%tT%n&#34;, date);     // 13:51:15

System.out.printf(&#34;hours %tH: minutes %tM: seconds %tS%n&#34;, date, date, date);
// hours 13: minutes 51: seconds 15

System.out.printf(&#34;%1$tH:%1$tM:%1$tS %1$tp %1$tL %1$tN %1$tz %n&#34;, date);
// 13:51:15 pm 061 061000000 &#43;0400
```

详细说明下 `%t` 后面跟的内容

| 字符  | 含义                                         |
| ----- | -------------------------------------------- |
| H,M,S | 小时,分钟,秒                                 |
| L,N   | 时间的毫秒和纳秒部分                         |
| p     | am/pm   上午，下午的字符串内容               |
| z     | 时区 偏移的小时数                            |
| A,B,Y | 星期几(英文),  月份的名字(英文) , 年份(数字) |
| d     | 两位数字显示的 今天是当月的第几天            |
| m     | 两位数字显示的月份                           |
| y     | 两位数字显示的年份                           |





### System.out.printf()

本方法是格式化字符串，然后输出到控制台。 

使用方法约等于 `String.format()`  。  



### MessageFormatter()

我时候的时候非常简单， 像下面这种方式使用

```java
MessageFormat.format(&#34;hello,{0}&#34;, &#34;jack&#34;);   // hello,jack
```



`MessageFormat.format()`方法里面的第一个参数和`String.format`()的第一个参数的格式并不一样。

似乎在简单使用的话， 只需要使用 `{Index}` 这个占位符就好了。 

这里有一篇详细介绍此内容的文章 https://vence.github.io/2016/04/29/javamethod-messageformat/ 





## 参考链接

https://vence.github.io/2016/04/29/javamethod-messageformat/ 

https://docs.oracle.com/javase/tutorial/essential/io/formatting.html

https://www.runoob.com/w3cnote/java-printf-formate-demo.html  菜鸟教程，中文内容

https://www.baeldung.com/java-printstream-printf



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/java%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%A0%BC%E5%BC%8F%E5%8C%96/  

