# Java   Stream 中的 Map与flatMap 函数


map 函数的作用是映射， 或者说转换， 把对象A 转换到对象B， 可以切换类型。   
flatMap 函数的作用是先执行 map操作， 然后平铺。 

## 示例

下面看示例1.
```java
package com.example;

import java.util.Arrays;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class StreamTest {

    public static void main(String[] args) {

        var list1 = Stream.of(1, 2, 3, 4, 5).map(i -&gt; i * 2).collect(Collectors.toList());
        System.out.println(list1); // [2, 4, 6, 8, 10]

        var list2 = Stream.of(new int[] { 1, 2 }, new int[] { 3, 4 }, new int[] { 5 })
                .flatMap(array -&gt; Arrays.stream(array).boxed()).map(i -&gt; i * 2).collect(Collectors.toList());
        System.out.println(list2); // [2, 4, 6, 8, 10]
    }

}
```

`list1` 演示了 map 对整形数字的用法， 这里并没有切换数字的类型。 
`list2` 看起来稍微复杂一些，并且在 `flatMap` 之后， 又使用了一个 `map` 函数。 此例的  `flatMap` 的作用是转换 `Stream&lt;int[]&gt;` 到`Stream&lt;int&gt;` 。 

参考阅读： https://stackoverflow.com/a/27888447/11226492   
大致翻译一下： 
&gt; `Stream.of(intArray)` 将返回一个 `Stream&lt;int[]&gt;` 类型  
&gt; `Arrays.stream(intArr)` 将返回一个 `IntStream`  类型


下面再来看下示例2 
```java
package com.example;

import java.util.Arrays;
import java.util.Comparator;
import java.util.stream.Collectors;
import java.util.stream.Stream;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

public class StreamTest {

    /**
     * Desk
     */
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class Desktop {
        private String type;
        private String color;
        private int width;
        private int height;

        // ...
        // ...
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class Dimension {
        private String customString;
        private int width;
        private int height;

        public int area() {
            return width * height;
        }
    }

    public static void main(String[] args) {

        var desks = Arrays.asList(new Desktop(&#34;Desk1&#34;, &#34;white&#34;, 100, 100),
                new Desktop(&#34;Desk2&#34;, &#34;white&#34;, 110, 110),
                new Desktop(&#34;Desk3&#34;, &#34;black&#34;, 120, 120),
                new Desktop(&#34;Desk4&#34;, &#34;black&#34;, 130, 130),
                new Desktop(&#34;Desk5&#34;, &#34;black&#34;, 140, 140));

        var maxArea = desks.stream().map(d -&gt; new Dimension(d.getType(), d.getWidth(), d.getHeight()))
                .max(Comparator.comparingInt(Dimension::area));
        System.out.println(maxArea.get()); // StreamTest.Dimension(customString=Desk5, width=140, height=140)

        var deskArrayList = Arrays.asList(new Desktop[] {
                new Desktop(&#34;Desk1&#34;, &#34;white&#34;, 100, 100),
                new Desktop(&#34;Desk2&#34;, &#34;white&#34;, 110, 110)
                },
                new Desktop[] {
                        new Desktop(&#34;Desk3&#34;, &#34;black&#34;, 120, 120),
                        new Desktop(&#34;Desk4&#34;, &#34;black&#34;, 130, 130),
                },
                new Desktop[] {
                        new Desktop(&#34;Desk5&#34;, &#34;black&#34;, 140, 140)
                });

        maxArea = deskArrayList.stream()
                .flatMap(l -&gt; Stream.of(l).map(d -&gt; new Dimension(d.getType(), d.getWidth(), d.getHeight())))
                .max(Comparator.comparingInt(Dimension::area));
        System.out.println(maxArea.get());  //  StreamTest.Dimension(customString=Desk5, width=140, height=140)

    }

}

```

&gt; *有些读者看完代码可能会说 为什么Desktop不直接引用Dimension呢？*  
&gt; *因为这是一个示例，还可能因为是Desktop类型是已经入库的，不方便修改的。等等之类的*  
&gt; *还有诸如 `area` 方法为什么不放在 Desktop 中之类的， 大多都是因为这是一个示例。关注点应该是map和flatMap函数的使用和作用。*

在求解 `maxArea` 的地方，可以看到 `map` 函数把一个`Stream&lt;Desktop&gt;` 转换成了一个 `Stream&lt;Dimension&gt;` 类型。

在第二次求解 `maxArea` 的地方， 可以看到`flatMap`函数把一个 `Stream&lt;Desktop[]&gt;` 转换成了一个 `Stream&lt;Dimension&gt;` 类型。

*`flatMap`函数感觉一般用不着。*


## 其他

扩展阅读：
- [What&#39;s the difference between map() and flatMap() methods in Java 8? Answer1](https://stackoverflow.com/a/26684710)
- [What&#39;s the difference between map() and flatMap() methods in Java 8? Answer2](https://stackoverflow.com/a/26684582)
- [How can I create a stream from an array?](https://stackoverflow.com/a/27888447/11226492)




---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/java-stream-%E4%B8%AD%E7%9A%84-map%E4%B8%8Eflatmap-%E5%87%BD%E6%95%B0/  

