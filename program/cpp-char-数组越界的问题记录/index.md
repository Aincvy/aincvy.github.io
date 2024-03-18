# Cpp Char 数组越界的问题记录


看下面的代码：

```cpp
void uiChangeGraph(std::string_view path) {
  char tmp[FILENAME_BUF_SIZE]{ 0 };
  auto t = currentProject()-&gt;openGraph(path, tmp);
}

SightNodeGraph* Project::openGraph(std::string_view path, char* pathWithoutExtOut) {
  std::string targetPath{path};
  std::filesystem::path temp(targetPath);
  if (temp.has_extension()) {
    std::string ext = temp.extension().generic_string();
    if (ext == &#34;.json&#34; || ext == &#34;.yaml&#34;) {
      targetPath = std::string(targetPath, 0, targetPath.rfind(&#39;.&#39;));
    }
  }
  if (pathWithoutExtOut) {
    sprintf(pathWithoutExtOut, &#34;%s&#34;, targetPath.c_str());
  }
  return currentGraph();
}
```

这段代码在函数`uiChangeGraph`结束的时候可能会出现内存错误， 原因是因为 tmp的内存不足，越界了。

增大 tmp 的大小， 或者做长度检测就可以解决问题了。

或者给`openGraph`函数添加一个长度参数， 然后使用 `snprintf()`函数。





---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/cpp-char-%E6%95%B0%E7%BB%84%E8%B6%8A%E7%95%8C%E7%9A%84%E9%97%AE%E9%A2%98%E8%AE%B0%E5%BD%95/  

