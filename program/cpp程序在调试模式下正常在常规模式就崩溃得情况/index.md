# Cpp程序在调试模式下正常，在常规模式就崩溃得情况


笔者前几天在使用cpp(*c++*)写代码得时候， 发现一个很蛋疼得问题， 在使用`lldb`进行调试得时候，程序完全正常，没有错误。 而退出调试模式， 使用命令行运行程序得时候， 则会触发内存错误。

经过一番查找之后发现， 笔者发现有些类得属性没有进行初始化。 。。

这可能是一个非常基础得错误。。 但是笔者一般写Java得时候 是不需要考虑这个问题得。。:joy::joy: :joy:

大致情况是这样得。 

```cpp
// 物品类， 占位使用
class Thing {
  
}

class Person {
public:
  void take(Thing* thing){
  	if( m_onHand == nullptr ){
      // do something ..
    }
    
    // do something ..
  }
private:
  // 手里正在拿得东西
  Thing* m_onHand;
}
```

在调试模式下， 上述代码得属性部分 `m_onHand` 是一个`nullptr`，这会产生预期的行为。

而非调试模式下，似乎就不是一个`nullptr` ， 则会产生不可预期得内存错误。 :cry::cry::cry:

笔者使用得解决办法就是在声明得时候这么写： `Thing* m_onHand = nullptr;`



除此之外， 笔者还搜索到了一种可能性。

链接在此： https://stackoverflow.com/questions/186237/program-only-crashes-as-release-build-how-to-debug/186285#186285

下面是原文引用： 

> In 100% of the cases I've seen or heard of, where a C or C++ program runs fine in the debugger but fails when run outside, the cause has been writing past the end of a function local array. (The debugger puts more on the stack, so you're less likely to overwrite something important.)





---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/program/cpp%E7%A8%8B%E5%BA%8F%E5%9C%A8%E8%B0%83%E8%AF%95%E6%A8%A1%E5%BC%8F%E4%B8%8B%E6%AD%A3%E5%B8%B8%E5%9C%A8%E5%B8%B8%E8%A7%84%E6%A8%A1%E5%BC%8F%E5%B0%B1%E5%B4%A9%E6%BA%83%E5%BE%97%E6%83%85%E5%86%B5/  

