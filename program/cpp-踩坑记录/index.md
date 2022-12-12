# Cpp 踩坑记录




### \#param once 的缺点： 相同文件内容的不同位置会视为两个文件

当出现这个状况的时候， 可能会引发重定义的错误。

更多解释可见：  https://stackoverflow.com/a/1946730/11226492

一个可能的场景是这样的： 有一个项目A， 有一个项目B， 项目B依赖项目的A的头文件，且两个项目都由同一个人开发。

现在项目B是在项目A内部的， 且项目B使用的是相对路径引用的项目A的头文件。

大致的文件夹结构如下所示。

```plain
A
+ source
+ include
+ modProject
+ + B
+ + + include
+ + + source
+ README.md
```

目前的情况是： 在B的源代码里面使用诸如 `../../include/abc.h` 的形式引用A的头文件。

开发这些项目的开发者发觉这样写不太美观，而且不方便拓展。所以决定将项目A的头文件安装到`/usr/local/include/A` 目录下 。

这时，就需要把项目B里面的所有引用位置都纠正了，否则就可能出现重定义的情况。 （*这里只讨论使用 #param once 的情况，使用宏定义的方式应该不会出现这样的情况。* ）



---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/program/cpp-%E8%B8%A9%E5%9D%91%E8%AE%B0%E5%BD%95/  

