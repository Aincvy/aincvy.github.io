# Vscode开发c&#43;&#43; 介绍


笔者也是刚开始使用vs-code， 部分技巧还不是很熟练， 不过感觉没有CLion 那么好用。

笔者是使用Mac版本的 vs code，不过应该都差不多把。



## 拓展

vscode 本身能做的事情比较少， 还是要借助拓展来实现功能。

- llvm-vs-code-extensions.vscode-clangd
  - https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd
  - llvm 出的插件， 基于 `clangd` ，使用LSP通信。
  - 这个插件的自动补全感觉比微软出的c/c&#43;&#43;插件好一些。
  - 自动完成
  - 编译错误和警告
  - 跳转到定义/声明
  - 代码格式化 （`clang-format`)
  - 简单的重构  （提取变量， 重命名什么的）
  - 这个插件附带 `clang-tidy`和`clang-format`， 分别用于代码分析和格式化。 （CLion应该也是基于`clangd`的， 这个插件提供的警告部分还不错。 格式化功能，笔者觉得略逊于 CLion。）
- ms-vscode.cpptools-extension-pack
  - https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-extension-pack
  - 这个是一个插件包， 包含了几个插件。 如果觉得这个插件包的内容太臃肿了， 可以考虑不装。
- ms-vscode.cpptools
  - https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools
  - 微软官方的 cpp插件， 这个和 clangd 一起安装的时候要禁用这个插件的 IntelliSense 功能。
  - 这个插件好像提供 debug 功能 。 （笔者没有使用过， 笔者基本上是使用lldb命令）
- twxs.cmake        
  - https://marketplace.visualstudio.com/items?itemName=twxs.cmake
  - cmake 语法支持
- ms-vscode.cmake-tools
  - https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools
  - 编译， 清理 cmake 工程
  - 用于提供cmake的数据给别的拓展？
- deerawan.vscode-dash
  - https://marketplace.visualstudio.com/items?itemName=deerawan.vscode-dash
  - dash 相关的插件， 如果读者不使用dash, 可以不安装。  
  - 使用快捷键可以直接转到dash里面搜索接口
- streetsidesoftware.code-spell-checker
  - https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker
  - 拼写检查工具
- xyz.local-history
  - https://marketplace.visualstudio.com/items?itemName=xyz.local-history
  - 本地文件历史，  会产生一个叫`.history`目录， 里面存放了很多临时文件。 记得添加目录到`.gitignore`文件里面
  - 可以安装， 也可以不安装把，笔者觉得。 
- pkief.material-icon-theme
  - https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme
  - 文件，文件夹 的icon
- tdennis4496.cmantic
  - https://marketplace.visualstudio.com/items?itemName=tdennis4496.cmantic
  - 根据声明生成定义
  - 生成 getter/ setter
  - 修改函数签名
  - 添加头文件保护
  - 比较建议安装这个拓展。
- eamodio.gitlens
  - https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens
  - 这是一个GIT拓展，如果读者和别人一起工作，则可以考虑安装。 
  - 这个拓展可以显示一个函数，类，命名空间的作者，和修改人，修改时间等
  - 类似 VS里面的那个函数上面的虚字。
- wayou.vscode-todo-highlight
  - https://marketplace.visualstudio.com/items?itemName=wayou.vscode-todo-highlight
  - 可以高亮 todo 等字符的拓展。  （字符可以自定义）
  - 并且可以全文件搜索并显示成一个列表。
  - 注意： 这个拓展也会搜索`xyz.local-history`产生的临时文件。记得把`.history`文件夹添加到这个拓展的`Exclude`配置项里。
- jeff-hykin.better-cpp-syntax
  - https://marketplace.visualstudio.com/items?itemName=jeff-hykin.better-cpp-syntax
  - 好像是个主题拓展， 想装就装把。



## 配置项

### clangd

即`llvm-vs-code-extensions.vscode-clangd`拓展。

这个插件需要 Cmake 生成一个额外的文件指示编译命令。 可以在`CMakeLists.txt`文件添加下面这行生成。 

```cmake
set(CMAKE_EXPORT_COMPILECOMMANDS ON)
```

如果提示插件找不到 `clangd`，可以让插件自动下载一个。 

参数可以使用类似下面的内容

```json
&#34;clangd.arguments&#34;: [
        &#34;-background-index&#34;,
        &#34;--compile-commands-dir=./build&#34;,
        &#34;-clang-tidy&#34;,
        &#34;-log=verbose&#34;,
        &#34;-pretty&#34;,
        &#34;-suggest-missing-includes&#34;,
        &#34;-header-insertion=iwyu&#34;,
        &#34;-completion-style=detailed&#34;,
        &#34;--query-driver=/usr/bin/clang&#43;&#43;&#34;
    ]
```

其中`--compile-commands-dir=./build` 指定了 cmake 的 build文件夹， `--query-driver=/usr/bin/clang&#43;&#43;` 指定了一个额外的 clang&#43;&#43; 用于查询 include目录。

笔者刚开始使用这个插件的时候， c&#43;&#43;的库文件找不到， 提示 `&#39;string&#39; file not found/ &#39;filesystem&#39; file not found`等的这类， 但是笔者的项目是可以编译的， 添加了这个`query-driver`选项之后就好了。

在项目的根目录下， 创建下面文件可以指定一些额外的配置项。

- `.clangd`   clangd的额外选项。   https://clangd.llvm.org/config
- `.clang-format`  用于指定代码格式化的规则。 https://clang.llvm.org/docs/ClangFormatStyleOptions.html

每个人对格式的感觉不一样， 建议手动从头撸一份配置。

这里有一点 笔者要说一下。  `clang-format`不能处理这种情况。

```cpp
if (f1.fileType == ProjectFileType::Directory &amp;&amp; f2.fileType == ProjectFileType::Regular) {
  return true;
} else if ((f1.fileType == ProjectFileType::Directory &amp;&amp; f2.fileType == ProjectFileType::Directory) 
           || (f1.fileType != ProjectFileType::Directory &amp;&amp; f2.fileType != ProjectFileType::Directory)) {
  return f1.filename &lt; f2.filename;    
}
```

第二个`else if`的条件，  因为条件代码太多， 所以笔者拆成了两行，但是在格式化之后会自动合并到一行，且笔者并没有找到办法处理这个。

当 `clangd`出现缓存问题的时候，可以考虑下面的方式。

- `cmd/ctrl&#43;shift&#43;p` 输入 clangd, 选择 restart language server
- `cmd/ctrl&#43;shift&#43;p` 输入 reload window
- 重启 vs code.

### vs-code

vs-code 本身也有一些可以调整的选项。

```json
// 自动保存   5s保存一次
&#34;files.autoSave&#34;: &#34;afterDelay&#34;,
&#34;files.autoSaveDelay&#34;: 5000,
// 粘贴之后自动对齐格式
&#34;editor.formatOnPaste&#34;: true,
// 使用 代码段等提示的时候 也会产生代码补全
&#34;editor.suggest.snippetsPreventQuickSuggestions&#34;: false,
```



## 编译和运行

### 添加配置

**如果使用了 Cmake 插件， 这一步可以不做。**

左侧工具栏里有一个 编译和运行的按钮，点一下就能看到调试界面了。

点击上方的下拉框之后，选择“添加配置” 或者点击右侧的齿轮按钮应该都能到`launch.json`文件的编辑界面，只不过点击“添加配置”的时候， 会自动添加一个示例的配置项。

笔者是 Mac 系统下， 使用 lldb 进行调试的，下面给出一下笔者的配置内容，给读者一个参考。  

```json
{
  &#34;name&#34;: &#34;(lldb) 启动&#34;,
  &#34;type&#34;: &#34;cppdbg&#34;,
  &#34;request&#34;: &#34;launch&#34;,
  &#34;program&#34;: &#34;${command:cmake.launchTargetPath}&#34;,
  &#34;args&#34;: [],
  &#34;stopAtEntry&#34;: false,
  &#34;cwd&#34;: &#34;${workspaceFolder}&#34;,
  &#34;environment&#34;: [
    {
      // add the directory where our target was built to the PATHs
      // it gets resolved by CMake Tools:
      &#34;name&#34;: &#34;PATH&#34;,
      &#34;value&#34;: &#34;${env:PATH}:${command:cmake.getLaunchTargetDirectory}&#34;
    },
  ],
  &#34;externalConsole&#34;: true,
  &#34;MIMode&#34;: &#34;lldb&#34;
}
```

笔者使用了 `cmake`插件提供的值， 而没有具体指定。

### 启动和调试

~~添加了正确的配置项之后， 程序应该就可以在 VS Code 中调试了。~~

使用 cmake 插件的时候，可以使用下面的方式启动程序。

- 使用 `Ctrl&#43;F5` 或者点击底部状态栏的小虫子🐞按钮 打开程序进行调试。
- 使用`Shift&#43;F5` 或者点击底部状态栏的播放按钮 ▶正常运行程序。

如果没有使用 cmake 插件的话， 就需要按照上一部分添加配置项。

- 之后还是在底部状态栏， 可能看到配置项的名字，点一下应该就可以了。

可以在行号左侧点击一下，以添加断点。 

在程序碰到致命错误或者断点的时候会暂停执行程序，当程序暂停之后可以在调试界面看到更多信息。

底部还有一个`DEBUG CONSOLE` ，这是一个调试用控制台，输入变量名用于查看变量的值， 还能执行部分函数。但是好像不支持其他命令。

将鼠标放到对应代码上， 也可以看到当前的值。

这个`DEBUG CONSOLE`有一个问题， 那就是打印出来的文字并不好看。  每一行基本都是使用`@&#34;内容&#34;`的形式。 其中内容字符串里面的转义字符并不会被转义，而是直接打印出来。

成功启动调试之后， 会有一个小小的工具栏在VS Code 窗口的中间正上方的位置，可以重启，结束调试，步进，按行执行等。

### 拓展阅读

- https://github.com/microsoft/vscode-cmake-tools/blob/develop/docs/debug-launch.md#debug-using-a-launchjson-file
- https://code.visualstudio.com/docs/editor/debugging



## 总结

有了上面的这些插件之后， 就可以正常工作了。

虽然还是有一些不太严重的不足：

- ~~switch 不能自动补全缺失的 case 项~~
  - 似乎是`cmantic`插件带有的功能， 可以补全缺少的 case 项目。 具体操作是鼠标点击一下 switch 关键字中间的任意位置，之后左侧应该会出现一个蓝色的按钮，点击一下会有一个按钮，再点一下按钮就会补全 case 项了。
  - 但是 补全的列表，只有最后一项有 `break`，其他项目需要自己手动添加。
  - 如果存在 `default`关键字，就不会追加缺少的 case项。
- 代码段的功能不是很好用  (if/for/while之类的)
- 代码格式化有点蛋疼， 不像 Jetbrains家族的那些， 自带的就很舒服。
  - 如果 clang-format 可以满足读者的需求， 那么可以无视这条。



~~笔者目前是使用命令行手动make 和run的，所以编译运行相关的功能还没有测试过，不知道是否好用。~~

总体来说，还是比较满意的。   *有部分插件，笔者还没来得及也没有细究。*



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/vscode%E5%BC%80%E5%8F%91c&#43;&#43;%E7%9A%84%E4%BD%93%E9%AA%8C/  

