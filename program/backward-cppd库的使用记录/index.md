# Backward Cpp库的使用记录


## 基本内容

项目地址： https://github.com/bombela/backward-cpp

这个库是一个用于在程序崩溃的时候打印堆栈信息和代码信息的 c++库。

笔者是在 mac 系统上尝试使用的，该库的尝试结果如下：

- 可以打印函数名，但无法打印文件名和行号，  有时候连函数名也无法打印。
- 详情可见： https://github.com/bombela/backward-cpp/issues/231
- 据说在 linux 平台上是可以完美打印信息的， 笔者并没有尝试过。

*至于怎么使用这个库， 作者在主页上写的很清楚， 这里就不再赘述了。*



## 其他

### libdwarf

笔者寻找的到的库在 mac 上编译有一点点问题，处理一下就好了。 **注意，即使编译好了这个库，也无法打印文件名和行号**

libdwarf 在mac平台上编译有问题， 添加 `target_link_libraries(${target} PUBLIC ${LIBELF_LIBRARIES} "z")`   "z" 库的链接即可。 *添加到 Cmake 的配置文件里面。*

安装的时候， include 会提示有问题， 笔者是采取手动复制的情况。  即手动复制文件夹到 `/usr/local/include/libwarf` 里。
`build/config.h` 不知道有没有用，一起复制过去了。 



### 输出一个宏定义的值

```c++
#define XSTR(x) STR(x)
#define STR(x) #x

#pragma message "The value of BACKWARD_HAS_BACKTRACE_SYMBOL: " XSTR(BACKWARD_HAS_BACKTRACE_SYMBOL)
#pragma message "The value of BACKWARD_HAS_UNWIND: " XSTR(BACKWARD_HAS_UNWIND)
#pragma message "The value of BACKWARD_HAS_BACKTRACE: " XSTR(BACKWARD_HAS_BACKTRACE)
```

可见上述代码， 定义了`XSTR, STR` 两个宏之后， 使用 `XSTR(name)` 的形式即可以打印一个宏的值。


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/backward-cppd%E5%BA%93%E7%9A%84%E4%BD%BF%E7%94%A8%E8%AE%B0%E5%BD%95/  

