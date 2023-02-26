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
      std::cout << v;   // ok
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

> - Easy to read, colorized output (colors auto-disable when the output is not an interactive terminal)
> - Prints file name, line number, function name and the original expression
> - Adds type information for the printed-out value
> - Specialized pretty-printers for containers, pointers, string literals, enums, `std::optional`, etc.
> - Can be used inside expressions (passing through the original value)
> - The `dbg.h` header issues a compiler warning when included (so you don't forget to remove it).
> - Compatible and tested with C++11, C++14 and C++17.

`dbg`是一个宏， 最终拓展开的时候，会调用下面的函数。

```cpp
namespace detail {
template <typename T>
using ostream_operator_t =
    decltype(std::declval<std::ostream&>() << std::declval<T>());

template <typename T>
struct has_ostream_operator : is_detected<ostream_operator_t, T> {};
}
// Specializations of "pretty_print"

template <typename T>
inline void pretty_print(std::ostream& stream, const T& value, std::true_type) {
  stream << value;
}

template <typename T>
inline void pretty_print(std::ostream&, const T&, std::false_type) {
  static_assert(detail::has_ostream_operator<const T&>::value,
                "Type does not support the << ostream operator");
}

template <typename T>
inline typename std::enable_if<!detail::is_container<const T&>::value &&
                                   !std::is_enum<T>::value,
                               bool>::type
pretty_print(std::ostream& stream, const T& value) {
  pretty_print(stream, value,
               typename detail::has_ostream_operator<const T&>::type{});
  return true;
}
```

完成代码可以在这里浏览： https://github.com/sharkdp/dbg-macro/blob/master/dbg.h



## 拓展阅读

- Issue in dbg:  [Type does not support the << ostream operator on custom type](https://github.com/sharkdp/dbg-macro/issues/118)
- https://gcc.gnu.org/bugzilla/show_bug.cgi?id=83035


---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/program/cpp-%E6%93%8D%E4%BD%9C%E7%AC%A6%E9%87%8D%E8%BD%BD%E5%87%BD%E6%95%B0%E5%9C%A8%E6%A8%A1%E6%9D%BF%E9%87%8C%E6%89%BE%E4%B8%8D%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98%E8%AE%B0%E5%BD%95/  

