# 自制脚本语言[4.1] 数值运算


本篇来描述一下数值运算相关的内容。 

*本篇描述的只是最简单的实现方式， 并没有提供什么优化技巧。*



## 详细内容

数值运算应该编程里必不可少的一部分， 所以一个脚本语言引擎应该需要实现数值运算的功能。

下面来看一段代码。

```plain
a = 1;
b = 2;

c = a &#43; b;
d = a - b;
e = a / d;
f = a * b; 
g = a % d;

h = a ^ b;
i = a | b;
j = a &amp; b;
k = ~ a ;
```

大概需要实现的运算类型就是  加减乘除，取模以及位运算。 读者可以根据自己的需求只实现部分的运算符号， 或者实现更多的符号。  （*自定义新的符号*）



### 基本逻辑

这里只讨论单个运算符， 不讨论多个运算符的情况。 比如 `d = a &#43; b &#43; c` 这种暂时不讨论。

基本逻辑其实很简单。

1. 取出N个数字
2. 进行运算
3. 将结果放回去

几个运算符的差异部分，基本就是第二步，进行运算的部分。 

N个操作数， 产生一个结果的运算符（*&#43;，-，\*，/等*）的第一步和第三步的内容基本是一致的。



### 基于Map 的实现逻辑 

下面尝试解析一下 `c = a &#43; b` 的实现过程。

1. 从Map中取出key为a, b 的数字。 

2. 在宿主语言中进行运算 (*c/cpp/java/...*) 然后获得结果

3. 将结果保存到Map中。 （Key为 c) 

   

下面看一段示例代码 

```java
// java 代码， 代码并没有经过测试，只是解释思路使用
// c = a &#43; b; 

public class Test {
 
  public void test (){
    // key: 变量名，  value: 变量值
    Map&lt;String, Object&gt; map = new HashMap&lt;&gt;();
    // 第一个操作数
    String first = &#34;a&#34;;
    // 第二个操作数
    String second = &#34;b&#34;;
    
    String result = &#34;c&#34;;
    
    // 1
    Object a = map.get(first);
    Object b = map.get(second);
    
    // null check 和 类型check 这里就省略了。 
    
    // 2
    // Number 是一个自定义类型， 因为篇幅问题，对声明进行了省略。
    Number c = Number.add( ((Number) a), ((Number) b) );
    
    // 3
    map.put( result, c );
  }
  
}
```



### 基于栈的实现逻辑

*本部分笔者并没有实现过，只是基于自己的知识和理解来写的内容。*

下面同样尝试解析一下 `c = a &#43; b` 的实现过程。

1. 取出 a 对应的数值， 进行压栈
2. 取出 b 对应的数值， 进行压栈
3. 执行 加号指令   | 这里应该是基于栈的核心部分
   1. 取出栈顶的2个元素
   2. 进行相加
   3. 将结果压栈
4. 将栈顶元素出栈，放入到c对应的位置。 



这里使用局部变量表和栈进行配合使用， 而不使用Map。 

下面看示例代码

``` java
// 同样使用 Java做示例， 代码只是演示说明使用
// c = a &#43; b ;

public class Test {
 
  // 局部变量表
  private Object[] localVarTable = new Object [30];
  
  // 栈
  private Stack&lt;Object&gt; stack = new Stack&lt;&gt;();
  
  /**
   * 根据变量名获取变量对应的 局部变量表的索引
   */
  public int getIndex(String name){
    // todo 
    return 0;
  }
  
  /**
   * 执行加号指令
   */
  public void add(){
    // check size 和别的check 暂时都直接省略了
    
    Number a = (Number) stack.pop();
    Number b = (Number) stack.pop();
    
    // 运算， 并将结果入栈
    stack.push( Number.add(a,b) );
    
  }
  
  public void test(){
   
    // 第一个操作数
    String first = &#34;a&#34;;
    // 第二个操作数
    String second = &#34;b&#34;;
    
    String result = &#34;c&#34;;
    
    // 1, 2
    // 因为要节省篇幅， 所以大部分的 error check 就不做了
    Object a = localVarTable[getIndex(first)];
    Object b = localVarTable[getIndex(second)];
    
    // 入栈
    stack.push(a);
    stack.push(b);
    
    // 3
    add();
    
    // 4
    Object c = stack.pop();
    localVarTable[getIndex(result)] = c;
    
  }
  
}

```



### 基于 寄存器的实现逻辑

*本部分笔者并没有实现过，只是基于自己的知识和理解来写的内容。*

这里假设存在3个寄存器：  a, b, c  。  每一个寄存器都可以放任意类型的值。

1. 取出变量a的值， 放入a 寄存器
2. 取出变量b的值， 放入b 寄存器
3. 执行加法运算， 将结果放入c寄存器
4. 取出c寄存器的值， 放回局部变量表
5. 清空所有寄存器



下面看示例代码

```java
// c = a &#43; b ;

public class Test {
 
  // 局部变量表
  private Object[] localVarTable = new Object [30];
  
  // 寄存器
  private Object a = null;
  private Object b = null;
  private Object c = null;
  
  /**
   * 根据变量名获取变量对应的 局部变量表的索引
   */
  public int getIndex(String name){
    // todo 
    return 0;
  }
  
  /**
   * 清理寄存器 
   */
  public void clearReg(){
  	a = b = c = null;
  }
  
  /**
   * 执行加法运算， 运算逻辑总是将 寄存器 a,b 相加，并把值放入寄存器 c
   */
  public void add(){
    // 将会省略 check 
    
    c = Number.add( (Number) a, (Number) b ) ;
  }
  
  /**
   * 这里的变量名， 改用传参的形式
   */
  public void test(String first,String second, String result ){
   
    // 1, 2
    a = localVarTable[getIndex(first)];
    b = localVarTable[getIndex(second)];
    
    // 3
    add();
    
    // 4
    localVarTable[getIndex(result)] = c;
    
    // 5 
    clearReg();
  }
  
}

```





## 其他

本篇介绍了几种实现机制， 读者可以根据自己的需求，能力进行选择。



关于复合运算(*d = a &#43; b &#43; c*)， 将会在下节进行介绍。



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/non-menu/self-programming-lang/%E8%87%AA%E5%88%B6%E8%84%9A%E6%9C%AC%E8%AF%AD%E8%A8%804.1-%E6%95%B0%E5%80%BC%E8%BF%90%E7%AE%97/  

