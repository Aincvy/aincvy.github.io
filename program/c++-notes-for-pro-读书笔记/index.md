# C++ Notes for Pro 读书笔记


笔者这几天抽空读完了这本`C++ Notes for Professionals book`， 所以写一篇读书笔记， 记录下笔者还记得的内容。

https://goalkicker.com/CPlusPlusBook/

这本书对于从c 转向 cpp 的读者挺有帮助的， 笔者感觉。



## 详细内容

*本篇内容基本上都是以 c++17的版本来写的。*

*绝大部分代码都是纯手写，可能会出现部分错误。*



### class, struct 关键字

- `class`关键字用于定义一个类， `struct`关键字用于定义一个结构。 

- 类和结构大体一致， 是可以在内部进行转换的。区别在于 类的默认访问修饰符是`private`，而结构是`public`。

- 并且在一些地方，两个关键字是可以互换的。比如： `enum class x{}; enum struct x{};`   

在类继承的时候， 如果没有给访问修饰符， 则是会一个默认的。 请看下面的示例。

```cpp
struct B1 {};
class B2 {};

struct D1: B1{};   // 等同于： struct D1: public B1{}; 因为 B1是一个 struct
struct D2: B2{};   // 等用于： struct D2: private B2{}; 因为 B2 是一个 class

class D3: B1{};    // 等同于 D1
class D4: B2{};    // 等同于 D2
```



### 模板的理解

- c++的模板在笔者的理解里面更偏向于是一种代码生成技术。

- 模板类的代码混合类型，模板函数的代码混合参数 生成一个具体的类，结构，函数。
- 很多模板类的代码很复杂， 难懂。
- 模板类可以在编译期计算一些值， 虽然笔者感觉好像没什么用。
- 模板的参数可以是具体的值，并不需要完全是类型。
- 模板的参数可以特定化。
- SFINAE;  限制特定的类型使用特定的模板。
- `decltype(e)` 用于获取 e 的类型。
- 模板类的实现应该放在头文件里面， 除非在极其少数的情况下。
- 而正常函数的实现应该在源代码文件里面， 因为 ODR 原则。 即只应该存在一处定义， 但是可以存在多处声明。
- 使用`forwarding reference` 可以实现完美转发。 



下面给一些代码示例。 

```cpp
// 模板特定化
// 
template<class T>
struct S {
	using type = T;		
}

// 将 char,short 都重定位到 int
template<>
struct S<char> {
	using type = int;		
}
template<>
struct S<short> {
	using type = int;		
}

// 使用例子
void test(){
  S<char>::type a = 1;
  S<float>::type b = 1.0f;
}

// 使用具体值的模板
template<class T, std::size_t N>
struct MyArray{
  T* arrayPointer = nullptr;
  MyArray() {
    arrayPointer = new T[N];
  }
  ~MyArray() {
    if(arrayPointer){
      delete[] arrayPointer;
      arrayPointer = nullptr;
    }
  }
  std::size_t size() const {
    return N;
  }
}

void test(){
  MyArray<int, 5> a1;      // 注意这里的5
  a1.arrayPointer[0] = 15;
  a1.arrayPointer[1] = 25;
  a1.size();
}


// SFINAE
// Substitution Failure Is Not An Error
// 翻译过来大致是 在模板里面的部分代码如果是无效的格式，只会从备选列表中移出该模板， 而不会产生一个编译错误
// 当然， 如果移出完了之后， 没有可用的模板的话， 仍然还是会产生一个错误
// 这个可用于排除某些模板
template <class T>
auto myBegin(T& t) -> decltype(t.begin()){
  return t.begin();
}

template <class T>
auto myBegin(T& t) -> decltype(t.start()){
  return t.start();
}

void test() {
  std::vector<int> notUsed;
  auto iter = myBegin(notUsed);
  int a = *iter;
}

// 上述代码中会调用第一个 myBegin() 因为 std::vector<int> 类型没有 start 函数，只有 begin 函数
// 而如果一个类型有 start 函数，没有 begin 函数的话， 应该就会使用第二个
// 如果一个类型既有 start 函数，又有 begin 函数的话， 应该就会报错了， 因为模糊不清的调用
// 上述示例只是一个说明示例， 一般情况下，不会这么写代码。
// 在 SFINAE 中， 比较常用的应该是 std::enable_if<>
// 后面的 = 0 是 模板参数的默认值
template<typename Int, std::enable_if_t<std::is_signed<Int>::value, int> = 0>
void incr2(Int& target, Int amount);
template<typename Int, std::enable_if_t<std::is_unsigned<Int>::value, int> = 0> 
void incr2(Int& target, Int amount);

// forwarding reference
template<class F, typename... Args>
void func(F f,Args&&... args){
  f(std::forward<Args>(args) ...);  
}
// 这里的 两个&& 表示 forwarding reference,而不是右值引用
// ... 是 fold 表达式， 用于展开参数列表

```

模板的 SFINAE规则可能找不到一个不同命名空间的函数，具体可看： [cpp 操作符重载函数在模板里找不到的问题记录](./cpp 操作符重载函数在模板里找不到的问题记录)

### 让函数返回多个值

可以使用下列类型： 

- `std::tuple<A,B,C,D,E>`
- 自定义结构
- `std::pair<a,b>`
- 使用 c++17的结构绑定 可以很方便的使用上面的返回值
- `std::vector<x>`
- 使用参数。 （指针， 或者回调函数）

下面给一些示例

```cpp
// std::tuple<> 在 ≥ c++17 的时候， 是一个比较不错的选择
std::tuple<int, float, char> randomData(){
  return {1, 1.0f, 'A'};   // 也可以用 std::make_tuple<>()
}
// 使用 stuple 传递引用
std::tuple<int&> xxx(){
  static int a = 1;
  return std::forward_as_tuple(a);  
}

void test(){
  // ≥ c++17 结构绑定
  auto [i,f,c] = randomData();
  // 如果不使用结构绑定， 应该是使用 std::get<0>(tuple), std::get<1>(tuple) 这种方式
  std::out << i << ',' << f << ',' << c << std::endl;
  
  auto [ref] = xxx();
  ref = 15;
}

// 自定义结构返回
auto customStructReturns(){
  struct { int x = 0; int y = 0; } point;
  point.x = 15; 
  point.y = 20;
  return point;
}

void test(){
  auto [x,y] = customStructReturns();
  std::out << x << std::endl;
  std::out << y << std::endl;
}
// 这种方式使用起来没有 std::tuple  那么清晰， 因为使用 auto 的话，调用者不知道结构的内容

```

### lambda 表达式

- lambda 表达式是一个语法糖，代表了一个匿名函数。
- 格式为：  [捕获列表]\(参数列表) -> 返回值 { 代码块}
- 返回值的部分可以省略， 让编译器自行推断。
- 默认情况下， 按值捕获的变量不可修改， 可以使用`mutable`关键字改变这个情况。
- Generic lambda ， 当成模板的 lambda 表达式。



下面是一些示例代码

```cpp
 void test(){
   auto abc = [](int i) {
     std::cout << i << std::endl;
   };
   abc(1);
   
   int a = 0; int b = 1;
   auto la1 = [a,&b](){     // a 是按值捕获， b 是按引用捕获
     a = 10;
     b = 20;
   };
   la1();   // 在调用的时候 不需要传递捕获部分的变量， 只需要传递参数即可。
   std::cout << a;     // 0
   std::cout << b;     // 20 
   
   auto la2 = [](bool b) -> float {
     return b ? 1 : 1.5f;
   }
   la2(true);
   
   auto la3 = [a]() mutable -> float{
     return ++a;
   }
   la3();   // 1 
   la3();   // 2, 虽然这里是2， 但是 当前作用域里面的 a 的值并无变化
   std::cout << a;    // 0 
   // 上面的 la3 的内容 参考自  https://blog.csdn.net/Trouble_provider/article/details/90521215
   
   // generic lambda
   auto gl1 = [](auto a, auto b) {
     return a + b;
   }
   
   gl1(1, 2);   // 3
   gl1(1.5f, 2.5f);  // 4
   
   auto lamb1 = [](int &&x) {return x + 5;}; 
   auto lamb2 = [](auto &&x) {return x + 5;}; 
   int x = 10;
   lamb1(x);    // 非法， 因为 x 不是一个右值， 需要使用 `std::move(x)` 
   lamb2(x);    // 合法，  x 会变成一个 int&
 }
```



### friend, mutable 关键字

- 使用 friend 关键字指定一个类为友元类， 一个函数为友元函数
- 友元类和友元函数都可以访问该类的私有属性
- 友元类并不继承。
- mutable 指示一个类的非静态成员变量可以在 const 函数中修改。

### 多态性

- 基类应该声明一个 虚的析构函数， 否则可能会产生未定义的行为。
- 使用`using`关键字引用父类的函数
- 函数覆写时应该标记一个`override`
- 非覆写的同名函数会隐藏父类的函数，除非使用`using`关键字。
- 对一个虚函数后面加上` = 0` 可以使该函数变成一个纯虚函数。
  - 纯虚函数也可以添加实现。 
  - 拥有纯虚函数的类无法被实例化。 
- 最好使用`dynamic_cast`来向下转换指针。



看下面这段代码：

```cpp
class Base {
  Base() = default;
  virtual ~Base() = default;
};

class D : Base {
  int* pointer = nullptr;
  
  ~D() {
    if(pointer){
      delete pointer;
      pointer = nullptr;
    }
  }
};

void test(){
  Base *b = new D();
  auto d = dynamic_cast<D*>(b);   // 向子类指针转换， 如果无法转换，则d 是一个 nullptr
  
  delete b;        // 如果这里的 Base::~Base() 不是虚函数的话， D::~D() 可能就不会调用，就会产生内存泄露
  b = nullptr; 
}

// 函数覆写
class Base {
  void func(int i );
  void func(float f);
  
  virtual void abc(int i);
  virtual void foo(int i);
};

class D : Base {
  
  void abc(int i) override; 
  void foo(std::string i) override;   // 这里编译器应该会给出警告，这个函数并没有覆写任何一个父类的函数
  
  void func(std::string msg);   
  // 上面那个语句应该会隐藏父类的两个 func 函数
  // 除非使用下面的 using 语句显示的引入。
  using Base::func;
};

```



### RAII

**Resource Acquisition Is Initialization**  *暂时没去搜索中文是什么意思。*

- 简单来说，就是使用构造函数和析构函数完成对资源的占用和释放。

看下面的代码

```cpp
// v8::Context::Scope 的源代码
class V8_NODISCARD Scope {
  public:
  explicit V8_INLINE Scope(Local<Context> context) : context_(context) {
    context_->Enter();
  }
  V8_INLINE ~Scope() { context_->Exit(); }

  private:
  Local<Context> context_;
};
// 可以看到，在构造函数里面 Enter, 在 析构函数里面 Exit
// 除此之外， 这类类型一般也会把 拷贝，移动函数都禁用掉。 


// 下面给出另外一个示例
class Lock{
  public:
  void lock(){}
  void unlock(){}
}
class LockHelper {
  Lock& lock;      
  
  LockHelper(Lock& lock) : lock(lock){
    lock.lock();
  }
  ~LockHelper(){
    lock.unlock();
  }
  
  // 禁用拷贝函数， 移动函数
  LockHelper() = delete;
  LockHelper(LockHelper const&) = delete;
  void operator=(LockHelper const&) = delete;
  
  LockHelper(LockHelper&&) = delete;
  void operator=(LockHelper&&) = delete;
  
};

void test(){
  Lock l;
  LockHelper(l);    // auto lock and unlock.
  // do something.
  // 如果使用 java 的话， 就需要使用 try-catch-finally 的 finally 部分。
}
```



### 左值和右值

- `lvalue` left value: 左值。 一般可具有名字的都是左值
- `xvalue` expiring value: 将亡值。     `std::move()`函数的返回值 
- `prvalue` pure right value: 纯右值。  没有名字的表达式的。 
  - 一个临时对象 `std::string("123")`
  - 函数的返回值 （除了引用）
  - 字面量  `1, true,  0.5f, 'a'`
  - lambda 表达式
- `rvalue` right value: 右值， `xvalue` 和 `prvalue`的统称。
- `glvalue`  `lvalue` 和`xvalue`的统称
- 函数参数中， 可以用`Type&&` 表示需要一个右值。
- 使用右值对应的 move 语义可以提高程序性能。

看下面的示例代码： 

```cpp
 class Foo {
   int i = 0;
   
   int get() const { return i; }
   
   Foo() = default;
   ~Foo() = default;
   
   // copy
   Foo(Foo const& rhs){
     this.i = rhs.i;
   }
   Foo& operator=(Foo const& rhs){
     if(this == &rhs){
       return;  // 防止 自我拷贝赋值  除了这样写，还可以使用 copy-swap 的写法
     }
     this.i = rhs.i;
     return *this;
   }
   
   // move
   Foo(Foo&& rhs){
     this.i = rhs.i;
     rhs.i = 0;
     // 这里相当于把 rhs 的数据偷了过来。  当前类里面并没有包含动态内存， 所以效果不明显
     // 如果存在动态内存的话， 可以防止多次无用的内存申请。 
     // 在把 rhs 的数据偷过来之后， 要保证 rhs 是能够正常析构和复制的。
   }
   
   // 这里是 Foo::operator=() 函数的另外一个重载。 区别于 Foo const&
   Foo& operator=(Foo&& rhs){
     // 这里应该可以不做 自我移动赋值的判断。 
     // 除非有人这样写：  Foo f;  f = std::move(f); 
     
     // 这里的 i 是 int 类型，所以调用 std::move 应该是没有什么用
     // 但是其他类型的话，应该是有用的。 
     // 这里应该是同样可以使用 copy-swap 的。
     this.i = std::move(rhs.i);  
     return *this;
   }
 };

void bar(Foo &&f){
  std::cout << f.get();
}

void test(){
  
  Foo f1;
  Foo f2;
  f2 = f1;    // Foo::Foo(Foo const& rhs)  复制构造函数
  f1 = {};    // Foo::operator=(Foo&& rhs)   移动
  Foo f3 = {};  // Foo::Foo(Foo&& rhs)    移动构造函数
  Foo f4 = std::move(f2);      // Foo::Foo(Foo&& rhs) 
  // 现在 f2 应该是无法继续使用了。 如果使用的话， 可能会产生一个未定义的行为
  
  bar(Foo());
  // bar(f3)      // 这应该会报错
}
```



### 循环, auto 等

- auto 可以用于类型推断
- 范围的 for 循环是基于对象的 `begin(), end()`函数的。

看下面的代码：

```cpp
void test(){
  auto i = 1;   // i = int
  auto c = 'a'; // c = char
  
  std::vector<int> v1;
  auto begin = v1.begin();    // begin = std::vector<int>::iterator
  
  auto& ri = i ;    // ri = int&
  
  // 相当于  
  // for(auto iter = v1.begin(); iter != v1.end(); ++iter) {
  //   auto const& item = *iter;
  //   ...  代码
  // }
  for( auto const& item : v1){
    std::cout << item;
  }
}
```



### 指针的运算和比较

- 指针的比较只能在同一个数组里面， 否则会产生 未定义的行为
- 指针最多可以到数组最后一个元素的地址 +1个元素 的位置。 
- 有效的指针进行相减的时候会得到元素个数



### std 里的一些工具函数

```cpp
void test(){
  int a[10];
  std::size(a);    // 10
  std::begin(a);   // 类似迭代器 [begin]
  std::end(a);     // 类似迭代器 [end]
  
  std::vector<int> b;
  std::begin(b);
  std::end(b);  
  
  auto iter1 = std::find(b.begin(), b.end(), 1);    // 寻找元素
  // 使用函数查找元素
  auto iter2 = std::find_if(std::begin(a), std::end(b), [](int i) { return i > 5; }); 
  
  std::vector v;
  v.push_back(1);
  v.push_back(2);
  std::swap(v[0], v[1]);    // 交换元素 OR 调整元素顺序
  
  
}
```

### 位操作

- 1bit 就是1位， 值只有0或者1
- 一个 char 有8位
- 一个 int 通常上是32位 *并非绝对，取决于平台。*
- 位操作基本上就是改变第 N 位的值。

操作符列表

- `|` 按位或操作， 两个操作数的同一个位 有一个为1 则该位的值为1.

  - 0 0 0 1 0 0 1 0
  - 1 0 0 1 1 0 0 0 |
  - 1 0 0 1 1 0 1 0   *结果* 

- `^` 按位异或， 两个操作数的同一个位的值相同时，该位的值为0 不同时为1.

  - ```cpp
    int a = 42;
    int b = 64;
    
    // 使用按位异或的方式交换数值
    a ^= b;
    b ^= a;
    a ^= b;
    ```

- `&` 按位与，两个操作数的同一个位的值都为1的时候，值为1，否则为0.

- `<<` 左移，  将操作数的每一个位的值都左移 N 位， 多的部分会被省略

  - N 不能为负数， 也不能大于类型所占的位数。

- `>>` 右移，  将操作数的每一个位的值都右移 N 位， 多的部分会被省略

`std::bitset`

- 这个是一个工具类？  用于快捷操作位的。

- `bitset::set()  ` 设置所有位的值为1

- `bitset::flip(N)`    反转第 N 位的值

- `bitset::test(N)` 检查第 N 位的值是否为1

- ```cpp
  std::bitset<10> x;
  x.set(); // Sets all bits to '1'
  ```

  

### 操作符重载

```cpp
// 结构外重载
T operator+(T lhs, const T& rhs) {
  lhs += rhs;
  return lhs; 
}
T& operator+=(T& lhs, const T& rhs) {
  //Perform addition
  return lhs; 
}

// 结构内重载
T operator+(const T& rhs)
{
    *this += rhs;
    return *this;
}
T& operator+=(const T& rhs) {
    //Perform addition
    return *this;
}
```

部分操作符既可以在结构内重载， 也可以在结构外重载。 但是部分操作符只能 在结构内重载。 *类和结构一样的。*

更多内容可以阅读原书， 或者查阅：[https://en.cppreference.com/w/cpp/language/operators](https://en.cppreference.com/w/cpp/language/operators)

可以重载的操作符大致有下面这些

- 算数操作符 `+,-,*,/,%,...`
- 逻辑判断操作符 `>,<,==,...`
- 位操作运算符 `&,|,^,<<,>>,...`
- 函数调用的操作符 `()`
- 方括号的这个操作符 `[]`
- 指针相关的操作符 `->, ->*`
- 类型转换的操作符
- *new 和 delete 操作符也能重载？* 

### 文件操作

- `std::ofstream, std::ifstream, std::fstream`  使用写模式，读模式，读写模式打开文件
- 上面的类型可以使用操作符 `>>` 读取内容， `<<` 写入内容
- `imbue()`  函数可以用于修改 Locale
- c++17 新增了一个叫做`std::filesystem` 的命名空间 *好像是来自 boost?*
- `std::filesystem::exists(path)`  检测文件是否存在
- `std::filesystem::copy_file(path1, path2)`  复制文件
- `std::filesystem::directory_iterator`  目录迭代器  
-  *更多 API 请自行查阅文档*

部分示例代码

```cpp
std::ofstream output("foo.txt");
output << "text!";
output.flush();     // 刷新流
output.close();     // 关闭流，  这一步应该会自动刷新一遍，  如果没有手动调用这个函数的话，析构函数应该会调用这个

std::ifstream file("file3.txt"); 
std::vector<std::string> v;
std::string s;
while(file >> s) // 一直读取文件内容
{
	v.push_back(s); 
}

// 遍历目录
using directory_iterator = std::filesystem::directory_iterator;
for (auto it = directory_iterator {"./plugins"}; it != directory_iterator {}; ++it) {
	std::cout << it->path().string() << std::endl;
}
```



### cmake 

cmake 项目的编译比较简单， 执行诸如下面的命令就可以了。 

```shell
mkdir build && cd build
cmake ..
make
```

如果想应用 cmake 到自己的项目里面的话， 则需要编写`CMakeLists.txt`

大概使用下面的代码

```cmake
cmake_minimum_required(VERSION 3.10)     # 设置最低可运行的版本

# 设置一些编译属性
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_EXPORT_COMPILECOMMANDS ON)

# 添加一个新的 target 叫做 sight, 类型是可执行程序
add_executable(sight program.cpp)

# 给 sight 这个 target 添加一些源码文件
target_sources(sight PRIVATE
        sight_ui.cpp
        sight_ui_node_editor.cpp
        sight_ui_project.cpp
        sight_nodes.cpp
        sight_js.cpp
        sight_js_parser.cpp
        sight_project.cpp
        sight.cpp
        sight_plugin.cpp
        sight_widgets.cpp
        sight_keybindings.cpp
        sight_external_widgets.cpp
        sight_network.cpp
        sight_undo.cpp
        # dependencies/imgui/imgui_demo.cpp           # for debug
)

# 给 sight 这个 target 添加一些 Include 目录 
target_include_directories(
        sight PRIVATE
        ${PROJECT_SOURCE_DIR}
)

# yaml-cpp
find_package(yaml-cpp REQUIRED)      ##  查找 yaml-cpp 这个包
target_include_directories(sight PRIVATE YAML_CPP_INCLUDE_DIR)   # 添加 Include 目录
target_link_libraries(sight PRIVATE ${YAML_CPP_LIBRARIES})       # 链接 yaml 的动态库
MESSAGE(STATUS "Found yaml-cpp at: ${YAML_CPP_INCLUDE_DIR}")     # 输出日志信息
```





## 总结

笔者写出来的内容解释的不够详细， 所以读者如果感兴趣的话， 可以自行下载文档查阅。 *pdf 格式，无需注册账户，直接下载。*


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/c++-notes-for-pro-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/  

