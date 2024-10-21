# Msvc,cmake在cpp项目中不编译c文件的问题的处理


## 2024年10月6日更新 
```cmake
project(name
    VERSION 0.0.1
    DESCRIPTION &#34;A WIP vs code extension.&#34;
    LANGUAGES CXX C
)
```
这里可以使用 LANGUAGES 选项设置启用的语言是什么， 添加上C 就可以了。 

## 原文

先看一段 CMakeLists.txt 的文件内容： 
```cmake
add_executable(bead-server program.cpp)

target_sources(bead-server PRIVATE
    src/network.cpp
    src/bead.cpp
    src/db.cpp
    src/ast.cpp
    src/log.cpp
    src/completion.cpp
)

target_sources(bead-server PRIVATE
    dependencies/tree-sitter-cpp/src/parser.c
    dependencies/tree-sitter-cpp/src/scanner.c
)
```

使用上面的配置生成 vcxproj 文件的内容将会如下： 

```xml
  &lt;ItemGroup&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\program.cpp&#34; /&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\src\network.cpp&#34; /&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\src\bead.cpp&#34; /&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\src\db.cpp&#34; /&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\src\ast.cpp&#34; /&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\src\log.cpp&#34; /&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\src\completion.cpp&#34; /&gt;
    &lt;None Include=&#34;G:\nodeProjects\bead\server\dependencies\tree-sitter-cpp\src\parser.c&#34;&gt;
    &lt;/None&gt;
    &lt;None Include=&#34;G:\nodeProjects\bead\server\dependencies\tree-sitter-cpp\src\scanner.c&#34;&gt;
    &lt;/None&gt;
  &lt;/ItemGroup&gt;
```

可以看到两个c文件相关的内容是  `None` 而不是 `ClCompile`， 这样就会产生一个问题， 在编译的时候不会编译这两个文件， 从而会产生符号未定义的情况。 

在 CMakeLists.txt 文件中添加下面两行即可解决问题： 
```cmake
set_source_files_properties(dependencies/tree-sitter-cpp/src/parser.c PROPERTIES LANGUAGE C )
set_source_files_properties(dependencies/tree-sitter-cpp/src/scanner.c PROPERTIES LANGUAGE C)
```

注意要设置成 C ， 而不是CXX， 否则会因为符号问题 而出现链接错误。 

现在再来看一下 vcxproj 文件的内容， 已经变成了 
```xml
  &lt;ItemGroup&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\program.cpp&#34; /&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\src\network.cpp&#34; /&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\src\bead.cpp&#34; /&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\src\db.cpp&#34; /&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\src\ast.cpp&#34; /&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\src\log.cpp&#34; /&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\src\completion.cpp&#34; /&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\dependencies\tree-sitter-cpp\src\parser.c&#34;&gt;
      &lt;CompileAs Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;Debug|x64&#39;&#34;&gt;CompileAsC&lt;/CompileAs&gt;
      &lt;ScanSourceforModuleDependencies Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;Debug|x64&#39;&#34;&gt;false&lt;/ScanSourceforModuleDependencies&gt;
      &lt;CompileAs Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;Release|x64&#39;&#34;&gt;CompileAsC&lt;/CompileAs&gt;
      &lt;ScanSourceforModuleDependencies Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;Release|x64&#39;&#34;&gt;false&lt;/ScanSourceforModuleDependencies&gt;
      &lt;CompileAs Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;MinSizeRel|x64&#39;&#34;&gt;CompileAsC&lt;/CompileAs&gt;
      &lt;ScanSourceforModuleDependencies Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;MinSizeRel|x64&#39;&#34;&gt;false&lt;/ScanSourceforModuleDependencies&gt;
      &lt;CompileAs Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;RelWithDebInfo|x64&#39;&#34;&gt;CompileAsC&lt;/CompileAs&gt;
      &lt;ScanSourceforModuleDependencies Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;RelWithDebInfo|x64&#39;&#34;&gt;false&lt;/ScanSourceforModuleDependencies&gt;
    &lt;/ClCompile&gt;
    &lt;ClCompile Include=&#34;G:\nodeProjects\bead\server\dependencies\tree-sitter-cpp\src\scanner.c&#34;&gt;
      &lt;CompileAs Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;Debug|x64&#39;&#34;&gt;CompileAsC&lt;/CompileAs&gt;
      &lt;ScanSourceforModuleDependencies Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;Debug|x64&#39;&#34;&gt;false&lt;/ScanSourceforModuleDependencies&gt;
      &lt;CompileAs Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;Release|x64&#39;&#34;&gt;CompileAsC&lt;/CompileAs&gt;
      &lt;ScanSourceforModuleDependencies Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;Release|x64&#39;&#34;&gt;false&lt;/ScanSourceforModuleDependencies&gt;
      &lt;CompileAs Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;MinSizeRel|x64&#39;&#34;&gt;CompileAsC&lt;/CompileAs&gt;
      &lt;ScanSourceforModuleDependencies Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;MinSizeRel|x64&#39;&#34;&gt;false&lt;/ScanSourceforModuleDependencies&gt;
      &lt;CompileAs Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;RelWithDebInfo|x64&#39;&#34;&gt;CompileAsC&lt;/CompileAs&gt;
      &lt;ScanSourceforModuleDependencies Condition=&#34;&#39;$(Configuration)|$(Platform)&#39;==&#39;RelWithDebInfo|x64&#39;&#34;&gt;false&lt;/ScanSourceforModuleDependencies&gt;
    &lt;/ClCompile&gt;
  &lt;/ItemGroup&gt;
```

可以看到， 已经是 `ClCompile`  标签了。 

*PS: 可能和笔者使用了vcpkg有关系，也可能没关系。*

---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/msvccmake%E5%9C%A8cpp%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%B8%8D%E7%BC%96%E8%AF%91c%E6%96%87%E4%BB%B6/  

