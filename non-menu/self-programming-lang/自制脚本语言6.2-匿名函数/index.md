# 自制脚本语言[6.2] 匿名函数


本篇来描述一下匿名函数的实现。



## 基本内容

### langX 代码示例

```text
onEvent =&gt; (flag, callback) {
	// xxx
}

callEvent =&gt; (flag, data){
	local cb = findCallback(flag);
	if( cb ){
		cb(data);
	}
}

registerListeners =&gt; {
	// 这里的第二个参数就是一个匿名函数。
	onEvent(1, data =&gt; {
		println(&#34;[DEBUG] event 1 = &#34; &#43; data.toString());
	});
	onEvent(2, data =&gt; {
		println(&#34;[DEBUG] event 2 = &#34; &#43; data.toString());
	});
}
```

笔者在 langX 里面使用了类似函数声明的语法来声明一个匿名函数。 

当一个函数的参数需求另外一个函数的时候， 使用匿名函数很方便。

### 主要思路

设计了合适的语法之后， 解析该语法生成一个函数结构即可。  实现了函数这个功能之后， 这一部分做起来还是比较简单的。 

在处理函数名称的地方， 给一个空字符串或者 `nullptr`之类的即可。 

匿名函数除了没有名字之外， 其他行为和函数几乎一样。  

### 一些其他问题

在大多数语言里面， 匿名函数似乎都可以引用创建时候的变量环境的值。

举个🌰， cpp 里面有一个捕捉列表， 用于捕获声明时环境的值。 而 js 可以直接使用。 

看下面的 js 代码。

```javascript
function callWrapper(cb = Function()) {
  cb();
}

function test() {
  let foo = &#39;bar&#39;;
  callWrapper(function() {
    console.log(foo); // bar
    foo = 1;
  });
  console.log(foo);  // 1
}

test();
```

*点击链接 https://jsfiddle.net/qpgjLsrd/ 应该可以尝试一下。 控制台的输出结果在右下角的 Console 窗口里面，点一下Console标题会出现窗口。*

*直接打开 Chrome， 按下F12打开开发者工具，找到 Console,也能执行上面的代码。*

看上面的代码， `callWrapper`函数的参数是一个匿名函数， 该匿名函数内部并没有一个叫做 foo 的变量， foo 是出现在了 匿名函数的上面。

那么， 这个该如何实现呢？  *这里笔者仅提供几个思路，因为笔者并没有真正实现过。*

- 将匿名函数的执行环境从声明环境上拓展出来。 
  - 可以选择继承自 匿名函数的声明环境
  - 匿名函数自身内部的变量还是要储存在内部。
  - 让匿名函数在搜索变量的时候先从父环境中搜索。
- 在做语法分析的时候， 锁定变量出现的位置， 然后在真正创建匿名函数的时候引用变量



## 总结

匿名函数 似乎也被叫做 闭包， lambda 表达式等。 


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/non-menu/self-programming-lang/%E8%87%AA%E5%88%B6%E8%84%9A%E6%9C%AC%E8%AF%AD%E8%A8%806.2-%E5%8C%BF%E5%90%8D%E5%87%BD%E6%95%B0/  

