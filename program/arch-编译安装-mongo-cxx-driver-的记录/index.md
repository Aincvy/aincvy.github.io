# Arch 编译安装 Mongo Cxx Driver 的记录


## 基本内容

先从 github 上下载库的源码。
切换分支到 `releases/v3.7`  *切换到自己需要的分支即可*
命令参考： 
```bash
git clone https://github.com/mongodb/mongo-cxx-driver.git
cd mongo-cxx-driver
git checkout releases/v3.7
```

使用命令 `cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_STANDARD=17 -DCMAKE_INSTALL_PREFIX=/opt/mongo -DCMAKE_INSTALL_RPATH=/opt/mongo ..`  进行配置
这个命令将会把 mongo-cxx-driver 安装到 `/opt/mongo` 目录里面。 
成功配置后使用 `make` 命令进行编译
编译成功之后使用 `sudo make install` 进行安装。 
命令参考： 
```bash
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_STANDARD=17 -DCMAKE_INSTALL_PREFIX=/opt/mongo -DCMAKE_INSTALL_RPATH=/opt/mongo ..
make
sudo make install
```

随后，需要手动把 mongo-cxx-driver 的编译结果添加到 cmake 里。
`CMakeLists.txt` 文件内容参考
```cmake
SET(CMAKE_PREFIX_PATH "/opt/mongo/lib/cmake/")

SET(EXTRA_LIBS 
    mongo::bsoncxx_shared
    mongo::mongocxx_shared
)

target_link_libraries(target PRIVATE ${EXTRA_LIBS})
```


## 参考链接

- https://mongocxx.org/mongocxx-v3/installation/linux/


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/arch-%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85-mongo-cxx-driver-%E7%9A%84%E8%AE%B0%E5%BD%95/  

