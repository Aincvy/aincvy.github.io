# 自制脚本语言[5.4] while循环语句


本篇来尝试描述一下`while`循环语句如何实现。 

## 基本内容

### 语法树部分

先上图。

![while 的语法树](/img/program/while_ast1.png)



笔者这里给出了一个简单的while语句， 并附上了一个示意的AST结构。

`while`语句下 分为两个部分， 一个`condition`, 一个 `loop_body`

思路也比较简单， 先执行 condition块， 获取结果， 如果是true就执行loop_body块， 否则跳出循环。

*这里没有讨论do-while结构， 目前笔者不打算讨论这个结构。*

*笔者给的示例图是一个非常简单的语句，并没有包括跳转用的关键字。*

### 直接在AST上进行运算

这里有几个麻烦的地方， 先列出来给读者看一下。 

- 循环里面可能会出现`break/continue` 等跳转用的关键词。   在AST上运算的时候， 跳转往往是一个比较麻烦的问题，  没有使用字节码方便。 具体做法类似if-else语句的在AST上进行运算的部分。 
- 因为是循环， 所以循环体可能会执行多次。 在每次执行循环体之前，需要把节点的值都设置成NULL。在循环体执行之后设置应该也可以。  

剩余内容使用很普通的思路进行实现即可。   



### 使用字节码进行运算

这部分， 笔者尝试写一下上面语句的字节码伪代码。 

指令附加阅读内容： 

- [ if-else分支语句里的指令说明](../自制脚本语言5.1-分支语句/#使用字节码进行运算)

```bash
# a = 0;
seti a 0

# while
start_of_loop: 
# &#43;&#43;a &lt;= 10
# add = 将 某个变量的值添加N
# lte = less than equals ( 小于等于， 此指令会将一个bool 类型的结果压栈)
add a 1 
lte a 10
cjmp end_of_loop

# println(a);
call println 
get a
execute

# back to start 
jmp start_of_loop

# out of loop.
end_of_loop: 
```



和`if-else`语句的区别就是在循环体的最后追加一条`jmp start_of_loop`指令，这条无条件跳转指令将回到循环开始的地方，继续判断条件，执行循环体。 


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/non-menu/self-programming-lang/%E8%87%AA%E5%88%B6%E8%84%9A%E6%9C%AC%E8%AF%AD%E8%A8%805.4-while%E5%BE%AA%E7%8E%AF%E8%AF%AD%E5%8F%A5/  

