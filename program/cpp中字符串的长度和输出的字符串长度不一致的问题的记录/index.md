# Cpp中字符串的长度和输出的字符串长度不一致的问题的记录


## 主要内容

### 现象

先来看一段日志 
```cpp
[..t/sight/sight_js.cpp:996 (parseGraph)] finalSource = "varName = 23.5 + -69.5;" (std::string)
[..t/sight/sight_js.cpp:996 (parseGraph)] finalSource.length() = 80 (unsigned long)
[..t/sight/sight_js.cpp:996 (parseGraph)] finalSource.size() = 80 (unsigned long)
[..t/sight/sight_js.cpp:997 (parseGraph)] finalSource.data() = varName (char*)
[..t/sight/sight_js.cpp:998 (parseGraph)] finalSource.c_str() = "varName" (const char*)
varName
varName = 23.5 + -69.5;
```
前面三行可以看到字符串 `finalSource` 的长度是80， 但是内容只打印出 `varName = 23.5 + -69.5;` 这些是23个字符，与输出的长度并不一致。 

经过调试之后， 我发现是因为字符串里面存在大量的0, 这些0在打印的时候 什么也不会输出。

```cpp
[..t/sight/sight_js.cpp:996 (parseGraph)] finalSource.length() = 80 (unsigned long)
[..t/sight/sight_js.cpp:996 (parseGraph)] finalSource.size() = 80 (unsigned long)
[..t/sight/sight_js.cpp:997 (parseGraph)] finalSource.data() = varName (char*)
[..t/sight/sight_js.cpp:998 (parseGraph)] finalSource.c_str() = "varName" (const char*)
varName
11897114789710910100000000000000000000000000000000000000000000000000000000032613250514653324332455457465359
```

第一段日志的最后一行是使用 `%c` 打印出来的， 上一段日志的最后一行是使用`%d` 打印出来的。   
经过查询得知， 0 对应的 字符描述是 `Null character`  *from https://www.ascii-code.com/*   
所以在打印的时候 什么都没有显示出来应该也是正常的。 

打印使用的代码如下： 
```cpp
printf("%s\n", finalSource.c_str());
for (const auto &item : finalSource) {
    printf("%d",item);
}
```

### 问题原因

这些字符串是使用 v8pp 注入cpp 字符串到js里面产生的。  
当注入字符串类型是 `char[]` 的时候， 它会把全部的内容都注入到js 里面。

v8pp 的函数是这样的： 
```cpp
template<size_t N>
v8::Local<v8::String> to_v8(v8::Isolate* isolate,
	char const (&str)[N], size_t len = N - 1)
{
	return convert<string_view>::to_v8(isolate, string_view(str, len));
}
```
作者使用了模板函数获取字符串数组的长度， 然后把所有内容都注入进去了。 

解决方法如下： 
```cpp
char tmp[1024] = {0};
sprintf(tmp, "abcd");

v8pp::to_v8(isolate, tmp);       // 这有问题
v8pp::to_v8(isolate, (const char*) tmp);    // 这样就可以了。 
```

作者的解释可以看这里： https://github.com/pmed/v8pp/issues/174#issuecomment-921982794

作者这样的设计的目的可能是允许其他人注入二进制数据到js里面。 


### 其他 

编译时的字符串拼接 `test("hello," "world!" )`


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/cpp%E4%B8%AD%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%9A%84%E9%95%BF%E5%BA%A6%E5%92%8C%E8%BE%93%E5%87%BA%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2%E9%95%BF%E5%BA%A6%E4%B8%8D%E4%B8%80%E8%87%B4%E7%9A%84%E9%97%AE%E9%A2%98%E7%9A%84%E8%AE%B0%E5%BD%95/  

