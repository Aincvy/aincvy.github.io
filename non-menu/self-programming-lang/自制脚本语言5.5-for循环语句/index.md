# 自制脚本语言[5.5] for循环语句


本篇来尝试描述一下`for`循环语句如何实现。

## 基本内容

### 语法树部分

先上图。 

![for 的语法树](/img/program/for_ast1.png)

笔者这里给出了一个简单的for语句， 并附上了一个示意的AST结构。

`for`语句看起来复杂， 但是还是比较简单的， 只是他的子节点比别的语句多了几个而已。

基本上来看， for 语句可以看做这样的形式 `for 小括号内 语句块` 。 而`小括号内` 又可以看成`第一阶段;条件语句;第三阶段` 。  把所有的元素展开合并之后就得到了上图的四个子元素。  其中第一阶段只在循环开始的时候执行1次， 而第三阶段则在每次循环体运行之后运行一次。

- `for_1`  第一阶段
- `condition` 条件语句
- `for_3`   第三阶段
- `loop_body`  循环体 （语句块）

`condition`条件部分 和其他的循环语句， `if`语句的一样， 在语法分析上，基本是可以共用一个部分。

for语句的实现思路也比较简单， 先执行`for_1`语句， 然后执行`condition`语句， 如果是true就执行`loop_body`块，然后执行`for_3`，否则跳出循环。

`for_1 -> condition -> loop_body -> for_3 -> condition -> loop_body -> for_3 -> condition(false) -> break`

*不知道读者能不能看得懂上面的流程。。。 :joy:*



### 直接在AST上进行运算

像往常一样， 在AST上运算的时候， 跳转不是很好做。 其次， 循环体里面的所有节点的值记得注意一下， 否则可能会出现BUG。 

实现思路就按照上面说的内容进行实现就可以了， 如果在实现的过程中发现了什么问题， 往往是节点执行流程的设计有问题， 重新矫正一下那部分即可。 



### 使用字节码进行运算

这部分， 笔者尝试写一下上面语句的字节码伪代码。 

指令附加阅读内容： 

- [ 字节码列表](../自制脚本语言附.1-字节码列表/)

```bash
# for

# for_1
seti i 0

# condition
start_of_loop:
# 检查 i 是否小于10
lt i 10
# 如果上面运算的值为false, 则跳出循环
cjmp end_of_loop

# loop_body
# println(i);
call println 
get i
execute

# for_3
add i 1

# back to start 
jmp start_of_loop

# out of loop.
end_of_loop: 
```

主要思路和while循环类似，主要是在 for_3 阶段后面添加一条无条件的跳转语句， 调整到循环开始的地方。 

关于 `for(; ; ) {}`  语句的话， 省略掉 `for_1`,`condition`，`for_3`的内容，下面给出一个字节码示例。

```bash
start_of_loop:

# loop_body
# println(i);
call println 
get i
execute

# back to start 
jmp start_of_loop

# out of loop.
end_of_loop: 
```





---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/non-menu/self-programming-lang/%E8%87%AA%E5%88%B6%E8%84%9A%E6%9C%AC%E8%AF%AD%E8%A8%805.5-for%E5%BE%AA%E7%8E%AF%E8%AF%AD%E5%8F%A5/  

