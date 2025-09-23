# 在windows Cpp 开发中嵌入v8等库


## 主要内容

### 下载dll
*编译v8是一件很麻烦的事情，所以如果有预编译的dll就好了， 这么想着之后， 笔者就开始搜索了起来， 并且真的找到了一个可用的预编译库。*

仓库的地址是：  https://github.com/pmed/v8-nuget  

注意，这个仓库包含的是构建脚本， 作者把构建成功的dll文件都上传到了nuget上， 可以nuget上进行下载。 

- 头文件： https://www.nuget.org/packages/v8-v143-x64/#dependencies-body-tab
- 库文件： https://www.nuget.org/packages/v8.redist-v143-x64/

可以使用nuget 进行安装，也可以下载到本地之后手动添加进项目。  

在尝试nuget 命令失败之后， 笔者就采取了手动下载的方式进行使用。  这里需要注意的是头文件和dll库需要下载**相同版本**的， 否则会无法链接， 或者无法运行。 

这里插播一个关于cmake添加nuget包的方式： 
```cmake
# cmake version >= 3.15
set_property(TARGET MyApplication
    PROPERTY VS_PACKAGE_REFERENCES "v8-v143-x64_11.9.169.4"
)
```

名字和版本号之间用下划线进行连接， 原文是： https://stackoverflow.com/a/56093754

不过笔者没有尝试过这个方法， 笔者是使用`target_include_directories` 和`target_link_libraries` 来引用本地文件的。


### 其他

- 在cpp里调用v8 就是很普通的方式， 这里就不再赘述了。 
- 在Windows中使用这个库的时候 不能使用clang 编译器， 否则可以成功编译，但是运行的时候会出错。需要使用msvc进行编译。 
- Build Tools
  - 这个是vs2022 内部的一个工具， 可以安装编译工具，但是不需要安装vs2022， 可以节省磁盘空间
  - 下载地址可以参考：
    - https://visualstudio.microsoft.com/zh-hans/downloads/
    - https://download.visualstudio.microsoft.com/download/pr/db3d4c0f-3622-4e9b-bc48-7b4d831a33a7/86341ae0fda1f685efa3b1949b6dcf52fca81a85265cf1c7b8c62dd65fa4e8cf/vs_BuildTools.exe


## 其他的库
 
- yaml:  https://www.nuget.org/packages/fluid.yaml-cpp
  - 如果链接的时候出现了问题，则在include yaml.h 前面添加一行  `#define YAML_CPP_API`
- libuv:  https://www.nuget.org/packages/aerospike-client-c-libuv


这些库都比较小型， 下载源代码自己编译应该也是OK的。




---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/%E5%9C%A8windows-cpp-%E5%BC%80%E5%8F%91%E4%B8%AD%E5%B5%8C%E5%85%A5v8%E7%AD%89%E5%BA%93/  

