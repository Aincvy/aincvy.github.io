# 自制脚本语言[3.2]抽象语法树的简单介绍


## 基本内容

在进行了正确的语法分析之后，会生成抽象语法树。  

一般情况下是一段代码生成一个抽象语法树。 *（常见的情况是一个文件或者一块代码。）*

本篇来简单的描述一下抽象语法树的内容， 由于笔者的知识储备并不是特别的深，所以可能还需要读者阅读更多的资料。

抽象语法树就是一个树，树是一种基本的数据结构， 主要是由节点构成。

*下面定义来自维基百科。*

- 每个节点都只有有限个子节点或无子节点；
- 没有父节点的节点称为根节点；
- 每一个非根节点有且只有一个父节点；
- 除了根节点外，每个子节点可以分为多个不相交的子树；
- 树里面没有环路(cycle)



词法分析会从字符流中生成出词法单元， 然后由语法分析进行组装，产生一个AST（*抽象语法树*）。 基本上来说，树上的每一个节点都是一个词法单元。 

下面来看一些示例图。(*点击图片可以查看大图*)

**数值运算**

![AST1](/img/program/node_a&#43;b&#43;c.png)



**if语句**

![if-else语法树](/img/program/if-else语法树.png)

实际上的树要比笔者给的图更加复杂， 因为在设计的时候， 会考虑如何复用， 所以会进行抽象。 再进行了一定的抽象之后， 会多出一些层次， 但是那些多出来的内容在理解AST的时候是不太重要的。 

只要能够组织基本的AST， 那么复杂的其实也不难。 基本上来说， 复杂的AST只是一个量变和正常量变带来的些许复杂度。



## 代码示例

下面给出一些langX的代码示例， 辅助读者理解这部分内容。 

```cpp
struct Node
{
  NodeType type;
  // 在执行结束之后是否进行 free 操作
  bool freeOnExecuted;
  // 当前节点的值 .  如果当前结点是一个常量数字， 则free 的时候会施放这个值得内存
  Object *value;
  // 万能指针 ， 主要用于放置参数什么的
  void *ptr_u;
  // 节点状态
  NodeState state;
  // 当前节点所在的文件信息
  NodeFileInfo fileinfo;
  // 后置值 ， 如果当前节点为一个 变量节点， 当前属性则为有用
  Object *postposition;

  Variable *var_obj;
  Constant *con_obj;
  Operator *opr_obj;
  ArrayInfo *arr_obj;
};

// 节点的文件信息
struct NodeFileInfo
{
  // 文件名
  // const char * filename;
  // 修改成 std::string 试试，上面的那个类型要处理一下
  std::string filename;
  // 行号
  int lineno;
};

struct Constant
{
  int iValue;
  double dValue;
  char* sValue;
  char cValue;

  // 当前常量是否是一个 bool 类型得元素
  bool flagBool = false;
};

// 数组声明时使用
struct XArrayNode
{
  // 数组名字
  char * name;
  // 数组长度
  int length;
  // 变量长度索引 . 如果变量索引节点存在， 则使用这个， 否则使用 length 
  Node * lengthNode;
};

struct Operator
{
  // 子节点列表  | 实现树的主要部分内容
  struct Node** op;
  int op_count;
  // 操作符是什么
  int opr;

};

```

*本部分内容的代码在 `Object.h`文件里面。*

如果读者阅读了langX的 代码， 会发现还有一个类似的结构， 叫做`NodeLink`。  这个不用深入探究，这个是因为笔者当时的知识水平有限， 误入了歧路而产生的东西。 



### 接上一篇的一些内容

由上面的代码示例可以看出， 每一个节点都有一个`Object *value;`的属性。 这个属性用于存放该节点的运算结果。 

这种方式是属于直接再AST上进行运算， 不建议读者这么使用， 因为这个方式并不好， 会产生很多问题。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/non-menu/self-programming-lang/%E8%87%AA%E5%88%B6%E8%84%9A%E6%9C%AC%E8%AF%AD%E8%A8%803.2-%E6%8A%BD%E8%B1%A1%E8%AF%AD%E6%B3%95%E6%A0%91%E7%9A%84%E7%AE%80%E5%8D%95%E4%BB%8B%E7%BB%8D/  

