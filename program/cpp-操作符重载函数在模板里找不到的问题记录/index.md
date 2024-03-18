# Cpp 操作符重载函数在模板里找不到的问题记录


## 主要内容

笔者此前在使用 [dbg-macro](https://github.com/sharkdp/dbg-macro) 库的时候碰到了一个奇怪的问题， 就是操作符的重载函数无法被找到。

准确的说是被模板的 SFINAE 原则忽略了。 

大致的情况是这样的：

- 有一个结构体叫做`ImVec2`，它处于全局命名空间里面。  *来自库[imgui](https://github.com/ocornut/imgui)*

- 操作符的重载函数放在一个叫做`sight`的命名空间里面。

- ```cpp
  namespace sight {
    void test(){
      ImVec2 v;
      std::cout &lt;&lt; v;   // ok
      dbg(v);           // fail
    }
  }
  ```

- 大致代码如上面所述。

- **解决办法**是把 操作符重载函数和类型放入同一个命名空间， 比如把操作符的重载函数移到全局命名空间里面。

- 在类型和操作符重载函数在不同的命名空间的时候， ADL 规则是无法生效的。  这里的问题是，似乎模板函数在生成的时候没有搜寻当前的命名空间， 但是正常函数调用的时候却会搜寻。

- 可能是一个编译器 bug: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=83035.  不过笔者使用的是 clang， 也是存在这个问题的。



### dbg(v) 的解释

`dbg-macro`是一个用于方便输出调试信息的库， 特性大致如下

&gt; - Easy to read, colorized output (colors auto-disable when the output is not an interactive terminal)
&gt; - Prints file name, line number, function name and the original expression
&gt; - Adds type information for the printed-out value
&gt; - Specialized pretty-printers for containers, pointers, string literals, enums, `std::optional`, etc.
&gt; - Can be used inside expressions (passing through the original value)
&gt; - The `dbg.h` header issues a compiler warning when included (so you don&#39;t forget to remove it).
&gt; - Compatible and tested with C&#43;&#43;11, C&#43;&#43;14 and C&#43;&#43;17.

`dbg`是一个宏， 最终拓展开的时候，会调用下面的函数。

```cpp
namespace detail {
template &lt;typename T&gt;
using ostream_operator_t =
    decltype(std::declval&lt;std::ostream&amp;&gt;() &lt;&lt; std::declval&lt;T&gt;());

template &lt;typename T&gt;
struct has_ostream_operator : is_detected&lt;ostream_operator_t, T&gt; {};
}
// Specializations of &#34;pretty_print&#34;

template &lt;typename T&gt;
inline void pretty_print(std::ostream&amp; stream, const T&amp; value, std::true_type) {
  stream &lt;&lt; value;
}

template &lt;typename T&gt;
inline void pretty_print(std::ostream&amp;, const T&amp;, std::false_type) {
  static_assert(detail::has_ostream_operator&lt;const T&amp;&gt;::value,
                &#34;Type does not support the &lt;&lt; ostream operator&#34;);
}

template &lt;typename T&gt;
inline typename std::enable_if&lt;!detail::is_container&lt;const T&amp;&gt;::value &amp;&amp;
                                   !std::is_enum&lt;T&gt;::value,
                               bool&gt;::type
pretty_print(std::ostream&amp; stream, const T&amp; value) {
  pretty_print(stream, value,
               typename detail::has_ostream_operator&lt;const T&amp;&gt;::type{});
  return true;
}
```

完成代码可以在这里浏览： https://github.com/sharkdp/dbg-macro/blob/master/dbg.h



## 拓展阅读

- Issue in dbg:  [Type does not support the &lt;&lt; ostream operator on custom type](https://github.com/sharkdp/dbg-macro/issues/118)
- https://gcc.gnu.org/bugzilla/show_bug.cgi?id=83035


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/cpp-%E6%93%8D%E4%BD%9C%E7%AC%A6%E9%87%8D%E8%BD%BD%E5%87%BD%E6%95%B0%E5%9C%A8%E6%A8%A1%E6%9D%BF%E9%87%8C%E6%89%BE%E4%B8%8D%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98%E8%AE%B0%E5%BD%95/  

