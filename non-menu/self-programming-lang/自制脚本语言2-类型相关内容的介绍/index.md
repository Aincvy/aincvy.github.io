# 自制脚本语言[2] 类型相关内容的介绍


本篇来描述一下 类型相关的内容。



脚本语言引擎需要提供一些基本类型和至少一个复杂类型的抽象(类，表等)。



## 基本类型

就拿JAVA语言的基本类型来举例： `boolean,byte,char,short,int,long,float,double` 

其中的每一个类型都是基本元素，基本上在使用的时候不会拆分成更小的类型。

除了上述的关于数值的类型之外， 常用的还有一个字符串类型。  （*在Java中 字符串不算基本类型。算引用类型。但在读者自己开发的脚本语言中，则可以自己定义。*）

笔者在langX中定义的基本类型有下面几个：

- 数值类型 （整数， 浮点数， 布尔）
- null 类型
- 字符串类型
- 数组
- 函数

别的复杂类型基本上都需要使用上述的类型进行组成。



## 复杂类型的抽象

基本上可以理解为JAVA里的类， lua里的表。   

下面请看一个JAVA的代码片段。

```java
@Data
public class Student {
  // 唯一编号
  private int id;
  // 姓名
  private String name;
  // 年龄
  private int age;
  // 是否毕业了
  private boolean graduation;
  
  public boolean isGraduation(){
  	return graduation;
  }
  
}
```

上面描述一个叫 `Student`的类， 即一个复杂类型。这个类由字段，方法，注解组成，其中大部分字段使用的基本类型， 还有部分字段使用的是复杂类型（*字符串*）。 

这个类是一个具体的复杂类型， 为了使脚本语言引擎可以支持这个复杂类型， 我们需要实现一套机制。 这套机制可以表述一个复杂类型。 

下面是一段示例代码（*伪代码* ）

```plain
字段类 {
	访问限制符 ;
	字段类型 ;
	字段名 ;
}

类型类 {
	字段列表 ;
	方法列表 ;       // 节省篇幅，省略了方法类 
	注解列表 ;       // 同上  （其实就是懒 🤣🤣🤣 ）
}
```

*因为笔者的脚本语言引擎使用的 c++实现， 所以在名称后面加上了一个“类”。以表述该伪代码描述的是一个c++类*

使用上面的伪代码可以表述 `Student`类， 基本上也就可以表示任何的复杂类型。这样，脚本语言引擎就可以支持复杂类型了。 

如果读者只是想实现一个非常简单的脚本语言，或者DSL用作配置的话， 则可以考虑不实现这个， 而是实现具体的类型。之后在解释的时候， 一个字段对应一个字段的读取。 

如果只是用作于配置文件， 并且读者使用JAVA的话， 使用 json/yaml 之类的可能更合适。 （因为JAVA存在反射机制，所以可以很方便的自动赋值。）

但是如果存在逻辑的话， 可能json/yaml 就不是很好使用了。（比如分支语句， 循环语句等。）



## 总结

本篇只是大概的介绍了一下类型应该怎么做。 具体怎么样去做可能需要读者花时间去尝试。



这里在实现基本类型的时候 有一些参考性的指导内容。 

- 使用类继承的方式， 比如定义一个 `Object`的基类。 之后定义一个`Number` 类，表示数值，并继承自`Object` 。 定义一个`Null` 类， 继承 `Object`

  - 数组，函数， 对象 可能有点特殊。 笔者在实现langX的时候使用了下面的做法。 
  - 分别定义`Array/Function/langXObject` 表示对应的具体数据， 但是这些类并不继承`Object`。 
  - 定义新的类型`ArrayRef/FunctionRef/langXObjectRef` 来表示对上述类型的引用。 这些引用则继承自`Object`。
  - 这样做的原因在后面章节说到 “节点的值” 的时候会进行解释。

- 使用一些内存技巧。 

  - c/c++ 的内存布局里面没有多余的信息，并且可以进行强制类型转换。 

  - ```cpp
    // 下面代码的灵感来自于 lua源代码的阅读， 笔者并未真实实践
    // 这里只是给读者一些思路
    
    struct NumberType {
      int type;
      double value;
    }
    
    struct NullType {
      int type;
    }
    
    struct StringType{
      int type;
      int len;
      char *value;
    }
    
    #define isNull(x)  ((struct NullType*)x)->type == 1 
    #define isNumber(x)  ((struct NullType*)x)->type == 2
    #define isString(x)  ((struct NullType*)x)->type == 3
    
    void *ptr = ... ;   // ... 获取值的代码进行了省略
    
    if(isNull(ptr) ){
    	// null
    } else if (isNumber(ptr)){
    	auto value = ((struct NumberType*)ptr)->value;
      // ...
    } else if (isString(ptr)) {
      auto stringPtr = (struct StringType*) ptr;
    	// ...
    }
    
    ```

  -  使用上面的技巧 可以避免定义类型。  简单来说，这个技巧就是： 使结构的前几个字段类型相同。







---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/non-menu/self-programming-lang/%E8%87%AA%E5%88%B6%E8%84%9A%E6%9C%AC%E8%AF%AD%E8%A8%802-%E7%B1%BB%E5%9E%8B%E7%9B%B8%E5%85%B3%E5%86%85%E5%AE%B9%E7%9A%84%E4%BB%8B%E7%BB%8D/  

