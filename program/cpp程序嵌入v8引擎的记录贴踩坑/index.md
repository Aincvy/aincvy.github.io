# Cpp程序嵌入v8引擎的(踩坑)记录贴


笔者最近打算做一个工具， 考虑使用v8引擎做js语言的解析和执行。  在嵌入的过程中踩了一些小坑， 本文用来记录一下。 



## 长话短说

笔者使用的是Mac 系统，使用命令 `brew install v8` 即可安装v8库。 

安装成功之后，在这里应该可以找到需要的文件： ` /usr/local/opt/v8/libexec/`  。

`/usr/local/opt/v8` 是一个软连接， 它的真实目录这里就不介绍了， 如果读者想知道的话， 可以自行使用命令查看一下。 

可以使用命令`d8` 来检查一下安装是否成功了， 那是一个可以执行js语句的命令行程序。 



### cmake

如果使用cmake 的话， 可能还需要这个 `https://github.com/metacall/core/blob/develop/cmake/FindV8.cmake` 

可以使用类似如下的代码使用这个`FindV8.cmake`

```cmake
# 先使用 wget 或者其他工具把该文件下载到本地， 并保存到项目目录下的cmake 子目录。 
# add directory 
set(CMAKE_MODULE_PATH APPEND "${CMAKE_SOURCE_DIR}/cmake/" )

# sight 是笔者正在做的一个工具。 
add_executable(sight program.cpp)

# V8
set(V8_DIR "/usr/local/opt/v8/libexec/")
find_package(V8 REQUIRED)
target_include_directories(sight PRIVATE ${V8_INCLUDE_DIR})

# 如果没有这个定义 在程序运行的时候 好像会报错 
target_compile_definitions(sight PRIVATE V8_COMPRESS_POINTERS V8_31BIT_SMIS_ON_64BIT_ARCH)

# 链接库
target_link_libraries(sight PRIVATE ${V8_LIBRARIES})
```

关于`V8_COMPRESS_POINTERS`宏定义的拓展阅读部分： https://stackoverflow.com/questions/62921373/embedder-side-pointer-compression-is-disabled/62921689#62921689



### 常规方式

如果不使用cmake的话， 可以手动添加目录。 

下面介绍一下相关的目录内容。 

- 头文件目录： `/usr/local/opt/v8/libexec/include`
- 库文件目录： `/usr/local/opt/v8/libexec/`
  - `libv8.dylib`  
  - `libv8_libbase.dylib`   引用了上面的那个， 似乎可以不引用这个
  - `libv8_libplatform.dylib`  
  - 以下库都非必须，在需要的时候引用即可
  - `libicuuc.dylib` 
  - `libicui18n.dylib`
  - `libchrome_zlib.dylib`
  - `icudtl.dat`   这个不是动态库，在部分应用场景下需要复制到自己的程序目录里面，在初始化v8的时候，让v8引擎加载这个文件。 



### v8pp

这里再介绍一个方便 绑定cpp和js的库， v8pp。 

https://github.com/pmed/v8pp 



## 长篇大论

在使用homebrew 之前， 笔者尝试自己编译v8源码， 结果发现一些问题， 死活没弄好。。 

本部分就是相关的记录。 



### 从源码获取到编译

官方文档： https://v8.dev/docs/build-gn

步骤如下： 

1. 获取 depot_tools.
2. 获取 v8 源码
3. 调整编译参数
4. 编译



```shell
# 1. 
cd somewhere
git clone 'https://chromium.googlesource.com/chromium/tools/depot_tools.git'

# 添加环境变量 
# add path , you may need add to ~/.bashrc or ~/.zshrc or config.fish 
export PATH=somewhere/depot_tools:$PATH

# 更新 depot_tools
cd somewhere/depot_tools
gclient

# 2.
cd somewhere
fetch v8
cd v8

# 3. 

# 自动参数编译  |  不推荐使用自动的方式， 因为编译出来的是静态库。  当然，静态库可能也行吧。。 
gm x64.release
# 如果想附带测试就执行下面这个 | 两个命令执行1个即可。 
gm x64.release.check
# 以上任意一个命令都会附带编译过程。 

# 手动调整参数   | 和上面自动参数编译 二选一执行即可。 
# 下面这个命令会打开一个 vi/vim 编辑器， 让你手动填写参数
gn args out/x64.release

# 读者可以使用下面的命令附加参数  | 此命令和上面那个使用1个即可。 
gn gen out/x64.release --args='is_debug=false target_cpu="x64" v8_target_cpu="arm64"'

# 可以使用这个命令查看所有可用的参数  
gn args out/x64.release --list

# 因为参数会很多， 所以建议读者把输出重定向到文件， 然后阅读文件  
gn args out/x64.release --list > args.txt 

# 注意   参数 `is_component_build = true`  会编译出动态库， 如果需要动态库就加上这条配置。 
# 然后根据args.txt 文件里面的内容 选取想要的参数， 添加到 out/x64.release/args.gn 里面即可。 

# 4. 

# 生成配置
gn gen out/x64.release

# 编译 所有目标   | 如果想编译特定的目标， 就在后面追加参数即可。 
ninja -C out/x64.release

# 等待一段时间， 应该就可以编译成功了。 
```

默认情况下， v8 应该会使用自带的`clang`编译器进行编译， 在笔者的环境下是 `clang 13.0` 。  该执行程序位于`somewhere/v8/third_party/llvm-build/Release+Asserts/bin/` 。

这个和笔者的系统默认编译器不是同一个， 笔者的默认编译器是`clang 12.0` 。 

在上述的环境下编译`hello-world.cc`这个示例文件， 可能会出现 `std::unique_ptr<v8::Platform> platform = v8::platform::NewDefaultPlatform();` 这行语句报错的情况。 

笔者就卡死在处理`NewDefaultPlatform()`语句的情况下了。。 

使用`c++filt -n xxxxx` 可以还原符号的内容， 会发现`libv8_libplatform.dylib` 文件里面的符号是 `v8::platform::NewDefaultPlatform(int, v8::platform::IdleTaskSupport, v8::platform::InProcessStackDumping, std::__1::unique_ptr<v8::TracingController, std::__1::default_delete<v8::TracingController> >)` 

*使用nm 命令可以查看符号， 然后使用c++filt 命令还原。*

他引用的命名空间是 `std::__1` ， 然后查看我们自己的文件， 会发现它引用的命名空间是`std::__cr` 。

笔者考虑的办法有以下几个。 

- 使用v8带的编译工具编译我们自己的文件
  - 笔者没有尝试这个
- 使用我们自己的编译工具编译v8 
  - 笔者尝试了好几次， 总是无法编译。。
  - 在使用`ninja` 工具的地方出错
- 什么都不修改编译成功之后， 在相应的目录会生成一个叫做`libc++.dylib`的文件， 使用这个动态链接库代替系统的。 
  - 笔者没有找到合适的资料讲述如何完成这个步骤，所以没有尝试
  - 在尝试了homebrew 之后， 笔者成功了， 所以就没有继续考虑这个该怎么做。 



## 其他

使用 `make VERBOSE=1` 可以打印出编译使用的命令。  *笔者是使用cmake生成的文件尝试的。*



https://v8.dev/docs/embed  是官网文档， 讲述如何嵌入V8的。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/cpp%E7%A8%8B%E5%BA%8F%E5%B5%8C%E5%85%A5v8%E5%BC%95%E6%93%8E%E7%9A%84%E8%AE%B0%E5%BD%95%E8%B4%B4%E8%B8%A9%E5%9D%91/  

