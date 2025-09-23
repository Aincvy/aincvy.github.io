# Libuv 异步通知的实现


## 主要内容

### 主要思路

问题是： 假设有一个线程 A， 有一个线程 B， 现在想让线程 B 给线程 A 发送消息。 

本篇来说明一下如何使用 libuv 实现， 以及这样做有哪些不足。 



笔者没有并没有深入研究过 libuv 这个库， 所以背后原理并不是很了解， 这里只给出如何做。

下面是一些代码内容：

```cpp
// 纯手写代码， 并没有经过测试
#include "uv.h"
#include <iostream>
#include <thread>

static struct {
  uv_loop_t* uvLoop = nullptr;
  uv_async_t* uvAsync = nullptr;
  
} status;

// 回调函数
void callback(uv_async_t *handle){
  auto p = (int*)handle->data;
  std::cout << *p << std::endl;
  free(p);
}

// 线程 A 的函数
void threadA(){
  status.uvAsync = new uv_async_t();
  // uv loop init
  auto uvLoop = (uv_loop_t*)malloc(sizeof(uv_loop_t));
  status.uvLoop = uvLoop;
  uv_loop_init(uvLoop);
  uv_async_init(uvLoop, status.uvAsync, callback);
  
  // 循环执行事件
  uv_run(uvLoop, UV_RUN_DEFAULT);
  //while(true){
  //  uv_run(uvLoop, UV_RUN_NOWAIT);    // 只执行1个事件，且无堵塞
  //}
  
  // 释放内存
  // free uv
  uv_loop_close(status->uvLoop);
  free(status->uvLoop);
  status->uvLoop = nullptr;
  delete status->uvAsync;
  status->uvAsync = nullptr;
}

// 线程 B 的函数
void threadB(){
  // 睡眠500毫秒 等待uv 初始化
  std::this_thread::sleep_for(std::chrono::milliseconds(500));
  
  // 
  auto async = g_UIStatus->uvAsync;
  // 准备数据
  auto p = (int*)calloc(1, sizeof(int));
  *p = 1;
  async->data = p;
  // 发送给另外一个线程
  uv_async_send(async);
}

int main(void){
  std::thread  t(threadB);
  threadA();
  t.join();
  return 0;
}

```

先初始化一个`uv_loop_t`和 一个`uv_async_t` ，之后在附属线程先准备数据， 然后使用`uv_async_send()`函数发送数据即可。

**注意：** 这样做有一个缺点， 即 `uv_async_t` 一次只能保存1个数据， 如果前一个数据没有被使用，然后又来了一个数据，此时前一个数据会被顶掉。 *可能会产生内存泄露。*

选择一个线程安全的队列可以避免这个问题， 但是笔者并没有找到合适的线程安全的队列库，估计要使用互斥锁，条件变量自己手写一个。 



## 拓展阅读

- [C++创建对象时区分圆括号( )和大括号{ }](https://zhuanlan.zhihu.com/p/268894227)
- [GitHub - fffaraz/awesome-cpp: A curated list of awesome C++ (or C) frameworks, libraries, resources, and shiny things. Inspired by awesome-... stuff.](https://github.com/fffaraz/awesome-cpp#standard-libraries)
- [uv_run()函数的第二个参数详解](https://stackoverflow.com/a/17329626/11226492)
- [Threads — libuv documentation](http://docs.libuv.org/en/v1.x/guide/threads.html#synchronization-primitives)
- [Can I do custom events with libuv?](https://stackoverflow.com/a/15718043/11226492)


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/libuv-%E5%BC%82%E6%AD%A5%E9%80%9A%E7%9F%A5%E7%9A%84%E5%AE%9E%E7%8E%B0/  

