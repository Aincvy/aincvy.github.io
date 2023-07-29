# 自制脚本语言[1] 词法分析与语法分析


本篇来描述如何做语法分析和词法分析。



有两种方式可以做词法分析和语法分析， 一是自己纯手写， 二是使用第三方工具。

- 手写的优势
  - 灵活性比较高
  - 可以提供较好的错误提示给脚本语言的用户
- 手写的劣势
  - 在没有基础的情况下，可能会需要更多的时间来完成这件事
  - 可能会出现一些BUG
- 使用第三方工具的优势
  - 较为出名的第三方工具往往都比较稳定
  - 只需要关心语法结构， 而不用特别关心如何实现。
- 使用第三方工具的劣势
  - 不够灵活，如果某些特性第三方工具没有的话，很可能就实现不了了。
  - 提供给脚本语言用户的错误提示可能不够精确。



一般来说，使用哪种方式看读者的喜好。 笔者下面介绍一下部分已知的工具是使用哪种方式实现的。

- 《两周自制脚本语言》 提供的是手写方式。
- lua  是使用的手写方式。
- 早期的mysql 应该是使用的第三方工具。  （*因为它给的提示特别烂。。*）



## 第三方工具

因为使用第三方工具比较简单， 所以笔者采用第三方工具的方式来实现langX的词法解析与语法解析。

下面介绍一下笔者知道的几种工具。

- 词法分析器： flex(lex)  [doc](http://dinosaur.compilertools.net/lex/index.html) 。  
- 语法分析器： yacc(bison)  [doc](https://www.gnu.org/software/bison/manual/html_node/index.html) 。  
- 语法分析+词法分析： tree-sitter  [doc](https://tree-sitter.github.io/tree-sitter/)  。



笔者这里统一使用 lex 称呼 flex/lex , 使用yacc 称呼 yacc/bison， lex+yacc是一起配合使用的。

Tree-sitter 则是一个单独使用的工具，  下面笔者就来更详细的介绍一下这几个工具。



### Tree-sitter

这里先说一下tree-sitter 这个工具。  这个工具使用js 来编写语法文件， 可以配合atom的插件来实现语法高亮。 

Atom 是一个代码编辑器， 和VS Code 类似。 Atom的功能没有大型IDE那么强大， 但是也勉强够用了。

Atom提供插件机制， 读者可以自行为读者的脚本语言编写一个语法高亮插件。 而Atom的语法高亮插件就是基于tree-sitter 提供的API的。  

[langX的Atom插件源码](https://github.com/Aincvy/langX-atom)  , [langX的tree-sitter源码](https://github.com/Aincvy/langX-tree-sitter)

tree-sitter是基于 Node.js 来运行的， 在编写语法的时候 只需要编辑一个叫做`grammer.js`的文件。

tree-sitter 可以使用一些函数来简化语法文件的编写， 比如 `optional()`,`repeat()`,`seq()`, `choice()` 等。 而yacc则不能使用函数， 只能写规则。 

tree-sitter可以生成在程序里面使用的语法分析文件(*\*.h,\**.c)， 但是笔者并没有使用相关的内容。

更多详细的内容可以阅读官方文档： https://tree-sitter.github.io/tree-sitter/

### lex + yacc

在 https://github.com/Aincvy/langX/tree/dev/extern  这个目录里面的 a.l 是词法分析文件(*lex*)， a.y是语法分析文件(*yacc*)。

这两文件可能乍一看 内容很长， 但是其实并不复杂，都是按照一定的规则进行编写的。



#### 词法分析工具lex

flex 在mac平台上可以使用 `brew install flex` 进行安装。 

在 debian/ubuntu 上可以考虑使用 `sudo apt install flex` 来进行安装。

lex的语法是使用 `%%` 符号进行分割, 整个文件将会被两个`%%` 分割成三块。

- 用户定义内容
- 词法规则
- 其他代码

下面的示例代码节选自 a.l 文件。 （*语法高亮可能存在问题*）

```plain
// 用户定义内容

%{
#include "y.tab.h"

// 当前行
int now_line = 1 ;
// 当前列
int column = 0;

void comment(void);
void count(void);

%}

%%

// 词法内容 
"/*"			{ comment(); }
"//"[^\n]*      { /* consume //-comment */  }

"auto" {count(); return KEY_AUTO;}
"if" {count(); return KEY_IF;}
"else" {count(); return KEY_ELSE;}
"while" {count(); return KEY_WHILE;}


\"(\\.|[^\\"\n])*\"   {count();  yylval.sValue = strdup(yytext); return TSTRING;}
[$_a-zA-Z][$_a-zA-Z0-9]*  {count(); yylval.sValue=strdup(yytext); return IDENTIFIER;}

0|([1-9][0-9]*)           {count();  yylval.iValue = atoi(yytext);  return TINTEGER; }
(0|[1-9][0-9]*)\.[0-9]+ { count(); yylval.dValue = atof(yytext); return TDOUBLE;}

">" {count(); return '>'; }
"<" {count(); return '<'; }
"{" {count(); return '{'; }
"}" {count(); return '}'; }

[\n]   {count(); now_line++; }

%%

// 其他代码内容

void comment(void)
{
	char c, prev = 0;

	while ((c = yyinput()) != 0)      /* (EOF maps to 0) */
	{
		if (c == '\n') {
			now_line++ ;
			column = 0;
		}
		if (c == '/' && prev == '*')
			return;
		prev = c;
	}
	//error("unterminated comment");
}

void count(void)
{
	int i;

	for (i = 0; yytext[i] != '\0'; i++) {
		if (yytext[i] == '\n'){
			column = 0;
		} else if (yytext[i] == '\t')
			column += 8 - (column % 8);
		else {
			column++;
		}
	}

	// printf("after count %s, column: %d\n", yytext, column);
}

```

其中 词法规则部分是匹配token 的关键地方。 token是语法分析的基本单位， 使用lex 识别的token会传递给yacc， 然后由yacc筛选和确定语法规则。

在 用户定义内容的地方可以使用 `%{` 和 `%}` 来插入c/cpp代码。  而 其他代码内容则不需要符号，直接编写c/cpp代码即可。

读者可以先看看 langX的这部分代码是如何编写的， 然后再使用搜索引擎搜索一下lex相关的文章就明白了。

编写文件之后， 可以直接使用命令 `lex a.l` 来生成`lex.yy.c` 文件。 （`lex [filename]`）

#### 语法分析工具 yacc

yacc 在mac平台上使用命令`brew install bison` 来进行安装。

在debian/ubuntu平台上，可以使用`sudo apt install bison`  来进行安装。

默认情况下， yacc使用LR(1) 的方式进行语法分析， 但是可以使用`%glr-parser` 标记使其使用glr的方式进行语法分析。

Yacc 同样使用符号`%%` 进行三段式分割，使用 `%{` 和`%}` 来插入c代码。

第一部分的部分内容的说明：

- `%union{}`  这个结构就是 在lex 中的变量`yylval` 的数据类型。
- `%token`  用于声明token，每一个token 将会生成一个宏定义。 在源码包含`y.tab.h` 头文件之后即可使用。 在lex 中return 的也是这个。
- `%type<X>` 用于声明一个并非token 的中间类型，主要方便语法文件的抽象。 `X` 则表示类型，具体值为`%union{}` 中的字段名
- `%nonassoc`   无关联性的优先级指示。
- `%left` 用于指定左结合的`%token` 。 例如： `,` `+` `-` 等等。
- `%right` 用于指定右结合的`%token`。 例如：`=` `+=`  `-=` 等等
- `%glr-parser` 指示 yacc使用 glr的方式解析语法。
- `%start`  指定一个语法规则的入口



第二部分就是纯粹的语法规则了。

第三部分可以写一些c代码。在langX中，笔者在这段就只写了一个`main `方法。 



下面看一段**语法规则**的代码。 （*即 第二部分*）

```plain
// 本部分内容只是一些示例，并不能直接使用
program
    : statement_list
    ;

statement_list
    : statement_list statement      { execAndFreeNode($2); }
    |
    ;

statement
    : _extra_nothing     { $$ = $1; }
    | out_declare_stmt    { $$ = $1; }
    | con_ctl_stmt   { $$ = $1; }
    | simple_stmt    { $$ = $1; }
    | try_stmt       { $$ = $1; }
    | TOKEN_END_OF_FILE { lexEOFWork();  $$ = nullptr; }
    ;

_extra_nothing
    : ';'         { $$ = NULL; }
    ;

out_declare_stmt
    : func_declare_stmt          { $$ = $1; }
    | class_declare_stmt         { $$ = $1; }
    | namespace_declare_stmt     { $$ = $1; }
    | namespace_ref_stmt        { $$ = $1; }
    ;
 
// 命名空间的声明语句
namespace_declare_stmt
    : KEY_SET KEY_PUBLIC '=' namespace_name_stmt { $$ = opr(OPR_CHANGE_NAME_SPACE, 1, $4); }
    ;

// 引用命名空间
namespace_ref_stmt
    : KEY_REF namespace_name_stmt       { $$ = opr(KEY_REF, 1, $2); }
    ;

namespace_name_stmt
    : id_expr  { $$ = opr(OPR_GET_NAME_SPACE, 2, NULL, $1); }
    | namespace_name_stmt '.' id_expr { $$ = opr(OPR_GET_NAME_SPACE, 2, $1, $3); }
    ;
 
int_expr
    : TINTEGER { $$ = intNode($1); }
    ;

double_expr
    : TDOUBLE { $$ = number($1); }
    ;

```

使用 分行是为了看起来更清晰方便， 下面是一些更详细的说明

- 语法规则基本上是 `规则名称：子规则1 { 碰到这个子规则执行的语句} | 子规则2 {碰到这个规则执行的语句} ;`
- `$$` 表示规则名称的结果， 即当前规则向上返回的时候 返回的内容
- `$1,$2...`  `$n`表示第n个元素。 
- `//` 表示单行注释
- `number($1);  intNode($1);`  这些是笔者自定义的函数， 用于生成一个`XNode*` 。 上述代码中的`double_expr/int_expr...` 等等基本上都是使用 `%type<node>` 进行声明的，所以都是一个`XNode*` 类型。
- 上述内容选自 文件：  https://github.com/Aincvy/langX/blob/dev/extern/a.y    配合该文件一起看这篇博文可能更容易理解一些。

- 这里还有一些需要注意的地方是，仔细看`statement_list` 规则， 它可以是`statement_list statement` 或者一个空。
  - 当`statement_list`  是空的时候， 会匹配只有一条`statement` 的规则。即: `statement_list = statement`
  - 当匹配两条语句的时候就会出现下面的情况。 
    - `statement statement`       每个`statement` 表示一条语句
    - `EMPTY statement statement`   EMPTY和 空是一样的，这么写是为了更清晰的表达这里可以有一个空。
    - `statement_list statement`     ③将`EMPTY statement` 归约成一个`statement_list`
    - `statement_list`  继续归约。  
  - 当匹配三条及三条以上语句的时候和两条语句类似。
  - 归约就是 `reduce` 。  上面的③可能不是很好理解。 但其实很简单。
    -  因为`statement_list`可以是一个EMPTY。 对原语句进行替换，所以可以写成 `statement_list statement statement`
    - 使用规则 `statement_list: statement_list statement;` 对上面那条语句进行替换，就可以得到 `statement_list statement`  。  （*reduce前两个元素变成一个元素。*）



当语法文件编写结束之后可以使用命令`yacc -dv a.y`  来生成`y.tab.h`,`y.tab.c` 这两个文件和一些分析冲突使用的调试文件。



### 其他

Tree-sitter 的文档还是比较丰富的， 只是需要一定的英文水平， 并且tree-sitter提供了一些开源的示例可以参考，所以难度应该不是很大。在笔者编写langX的时候好像还没有tree-sitter这个工具，所以当时就选择了lex+yacc, 笔者目前只是使用tree-sitter来做语法高亮。

在使用lex+yacc 的时候， 建议刚开始从一些小的文件开始， 慢慢熟悉结构了之后 自然就会把文件写的较为复杂了。  当复杂到一定程度之后 可能会碰到很多冲突(*shift/reduce,reduce/reduce*)的警告，此时就需要考虑对语法文件进行重构， 笔者建议使用思维导图进行整理思路。 *注： 部分警告不是一定要解决的，但是建议能解决的都解决掉。*

语法分析会产生一个抽象语法树，之后的操作大多是基于这个抽象语法树的。所以理论上来说词法分析和语法分析的实现是可替换的，即你目前可以选择lex+yacc,后面不爽了可以换成纯手写的。 （*替换会产生成本就是了:joy:*） 



### 拓展阅读

本文的内容有点偏于介绍性，而非入门型， 所以建议笔者同时阅读下面的文章以深入了解lex&&yacc。

- https://segmentfault.com/a/1190000000396608
- https://www.jianshu.com/p/728e4011a61e



第三方词法分析器列表： https://en.wikipedia.org/wiki/Comparison_of_parser_generators

*随便截取几个放到下方*

- C# Flex, C# Lex
- DFA
- Golex, JLex
- ANTLR3, ANTLR4
- Bison, byacc
- AXE
- LPG, LISA
- AustenX
- BNFlite
- Canopy
- IronMeta



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/non-menu/self-programming-lang/%E8%87%AA%E5%88%B6%E8%84%9A%E6%9C%AC%E8%AF%AD%E8%A8%801-%E8%AF%8D%E6%B3%95%E5%88%86%E6%9E%90%E4%B8%8E%E8%AF%AD%E6%B3%95%E5%88%86%E6%9E%90/  

