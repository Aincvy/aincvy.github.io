# 谷歌v8引擎的尝试


谷歌V8是一个用于执行js脚本的引擎。



## 详细内容

### Isolate

> 一个 Isolate 是一个独立的虚拟机。对应一个或多个线程。但同一时刻 只能被一个线程进入。所有的 Isolate 彼此之间是完全隔离的, 它们不能够有任何共享的资源。如果不显式创建 Isolate, 会自动创建一个默认的 Isolate。

下面给出一些相关代码。

```cpp
// Initialize V8.
v8::V8::InitializeICUDefaultLocation(execPath);
v8::V8::InitializeExternalStartupData(execPath);
std::unique_ptr<v8::Platform> platform = v8::platform::NewDefaultPlatform();
v8::V8::InitializePlatform(platform.get());
v8::V8::Initialize();
// Create a new Isolate and make it the current one.
v8::Isolate::CreateParams createParams;
createParams.array_buffer_allocator =
  v8::ArrayBuffer::Allocator::NewDefaultAllocator();
auto isolate = v8::Isolate::New(createParams);
auto isolate_scope = std::make_unique<v8::Isolate::Scope>(isolate);
g_V8Runtime = new V8Runtime{
  std::move(platform),
  isolate,
  createParams.array_buffer_allocator,
  std::move(isolate_scope),
};

// 销毁
dbg("destroy js engine..");
g_V8Runtime->isolateScope.reset();
if (g_V8Runtime->isolate) {
  g_V8Runtime->isolate->Dispose();
  g_V8Runtime->isolate = nullptr;
}

v8::V8::Dispose();
v8::V8::ShutdownPlatform();

if (g_V8Runtime->arrayBufferAllocator) {
  delete g_V8Runtime->arrayBufferAllocator;
  g_V8Runtime->arrayBufferAllocator = nullptr;
}

```

`Initialize V8.` 部分的代码， 整个程序只需要执行一次， 之后便可以创建`Isolate` 。 

在多线程环境下，可以选择每个线程都创建一个单独`Isolate` ，对于笔者目前的项目来说， 选择的是采用一个单独的线程执行js脚本。当其他线程有执行脚本的需求的时候都会选择将任务发送给这个线程。

在线程结束之前应该将 `Isolate` 销毁。 

在程序结束之前， 所有的`Isolate`都销毁了之后， 应该执行下面的语句销毁v8引擎。

```cpp
v8::V8::Dispose();
v8::V8::ShutdownPlatform();
```



另外，Isolate 类的构造函数和析构函数都是删除了的。

```cpp
// v8.h

Isolate() = delete;
~Isolate() = delete;
Isolate(const Isolate&) = delete;
Isolate& operator=(const Isolate&) = delete;
// Deleting operator new and delete here is allowed as ctor and dtor is also
// deleted.
void* operator new(size_t size) = delete;
void* operator new[](size_t size) = delete;
void operator delete(void*, size_t) = delete;
void operator delete[](void*, size_t) = delete;
```



### 回调函数

并不是所有的场景都是执行单独一次js文件就完事了， 部分场景下， 我们需要执行js的回调函数， 而且可能会执行多次。

现在就简单的介绍一下如何实现这部分内容。 

使用这个类型即可`v8::Persistent` 即可。

下面给一个示例。 

```cpp
using namespace v8;

struct MyStruct {
  v8::Persistent<v8::Function, v8::CopyablePersistentTraits<v8::Function>> callback;
}

void regCallback (const FunctionCallbackInfo<Value> &args){
  auto isolate = args.GetIsolate();
  Local<Function> f = args[0].As<Function>();
  MyStruct s;
  s.callback = Persistent<Function, CopyablePersistentTraits<Function>>(isolate,f);
}

void useCallback(MyStruct const& s, Isolate* isolate){
  auto function = s.callback.Get(isolate);
  auto context = isolate->GetCurrentContext();
  
  // function->Call(xxx)  
}
```



### Context

这个是上下文环境，可以把全局的函数，成员，类型注册进去。

不过在使用中， 笔者发现了些许问题。 

Context如果出现多个的情况， 会产生层级， 然后子层级是无法访问父层级的内容的。 在父层级注册了的函数，在子层级需要重新注册一遍。

在使用函数 `v8::Script::Compile()` 编译执行一个脚本的时候， 源码里面所有的内容会放入Context里面。 

比如， 读者在脚本里面声明了一个函数`abc`， 那么这个`abc`会保存到Context，在执行别的脚本的时候就可以调用这个`abc`函数， 同时别的脚本则不能再声明一个叫`abc`的函数。 当然，此脚本也无法再次执行。

为了可以多次运行同一脚本，可以使用函数`v8::ScriptCompiler::CompileFunctionInContext()`， 这样把源码全部编译到一个函数里面。



### Scope

```cpp
v8::HandleScope handle_scope(isolate);
```

这个是作用域， 建议在合适的位置上都使用一下上面的语句。

这条语句会生成一个新的作用域给 isolate， 当离开`handle_scope`的cpp作用域的时候， 这个v8的作用域会被自动释放掉。 (*析构函数*)

 

### 其他

`node.js` 这个工具是基于谷歌v8引擎的， 这个工具的文档稍多一些，并且`node.js`本身是开源的项目，碰到问题的时候，也可以尝试从`node.js`里面寻找答案。 



## 拓展阅读

https://fantasyplayer.link/program/cpp%E7%A8%8B%E5%BA%8F%E5%B5%8C%E5%85%A5v8%E5%BC%95%E6%93%8E%E7%9A%84%E8%AE%B0%E5%BD%95%E8%B4%B4%E8%B8%A9%E5%9D%91/

https://www.cnblogs.com/imlucky/p/3270306.html

https://stackoverflow.com/questions/21239249/storing-handles-to-objects-in-a-hashmap-or-set-in-googles-v8-engine

https://yjhjstz.gitbooks.io/deep-into-node/content/chapter2/chapter2-0.html

https://github.com/nodejs/node/tree/master/src

https://zhuanlan.zhihu.com/p/25404844


---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/program/%E8%B0%B7%E6%AD%8Cv8%E5%BC%95%E6%93%8E%E7%9A%84%E5%B0%9D%E8%AF%95/  

