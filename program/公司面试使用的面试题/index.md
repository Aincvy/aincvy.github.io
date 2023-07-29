# 公司面试使用的面试题


## 简述

前些天，公司缺了一个做web后台的人，因为公司做后端的人就剩我一个了，所以面试的任务就交给我了。

笔者个人来说， 是第一次面试别人， 且笔者对 web 技术并不熟悉。 因为笔者是做游戏开发的，对 web 开发并不是很感兴趣，所以了解的不多。

仔细考虑了一下之后，笔者决定出两道很基本面试题来过滤一下面试者，笔者认为如果能完成这两道题，则说明基本逻辑的编写应该没有问题，那样就可以干活了。



## 面试题内容

一、 有下面代码

```java
// file: Student.java
// 学生信息类
@Data
public class Student {
    // 学号
    private int no;
    // 姓名
    private String name;
    // 年龄
    private int age;
    // 年级 1-16
    private int grade;
}

// file: StudentFilter.java
// 学生信息筛选的方法
public class StudentFilter{
    
    public static void test(){
        var list = new ArrayList<Student>();
        list.add(new Student(1,"学生1", 7, 2));
        // 此处省略添加 学生信息 N 个
        
        // 问题： 请输出8~10岁（都包含） 在上三年级的学生数量，以及信息
        // PS： 将grade = 3 视作三年级即可
        // 在下面空白处作答 即可。
    }
}
```





二、 SQL语句问题

```sql
-- 年级表
CREATE TABLE `grade` (
	id INT(11) PRIMARY KEY ,
    `name` VARCHAR(255) NOT NULL COMMENT '年级的名字',
    `type` INT(11) NOT NULL DEFAULT 1 COMMENT '1小学，2初中，3....'
);

-- 学生表
CREATE TABLE `student` (
	`no` INT(11) PRIMARY KEY COMMENT '学号',
    `name` VARCHAR(255) NOT NULL COMMENT '学生姓名',
    `age` INT(11) NOT NULL DEFAULT 1 COMMENT '年龄',
    `grade` INT(11) NOT NULL DEFAULT 1 COMMENT '年级表id'
);

-- 问题: 请查询出小学生的数量 （写SQL语句即可）
```



## 面试题解答

### 第一题

第一题是很简单的内容，基本上有些经验的人都能完成。 但是笔者发现很多自称两年，三年经验的人并不能完成这份笔试。。

本题主要考察下面几个点：

- Lombok。  自动生成 `Getter` ,`Setter`,`toString()` 的工具
- 基本逻辑思维以及代码编写经验。
  - 写过大量java 代码的程序员， 我觉得是可以记住这个问题里所使用的 API 的。
- Java8 - stream
  - 方便操作集合的工具。
  -   过滤，以及转换成 list。
- 认真的阅读需求的能力  
  - 这里希望首先输出数量，其次输出学生的信息。 
  - 绝大部分完成的人都是先输出的信息，之后再输出数量。（他们并没有使用 stream，而是使用一个临时变量记录数量，问题也算不大把。）
- Java10 - var 关键字
  - 用于局部的类型推导。

笔者大概试了10个人左右，并没有发现一个满足全部需求的人， 基本上都是缺了一些内容。

而且还有部分来面试的人 做不出来这题。（*讲真，我感觉到很惊讶😐*）

🥳 下面是示例的解题代码。  （*纯手写， 并没有经过 IDE 测试*）

```java
var result = list.stream().filter(s -> s.getGrade() == 3)      // 过滤年级
  .filter(s -> s.getAge() >= 8 && s.getAge() <= 10)    // 过滤年龄
  .collect(Collectors.toList());

// 输出数量以及信息
System.out.println(result.size());
System.out.println(result);

```

这里在额外多说几句， 写注释也是一个非常重要的内容。如果你的方法名是自解释的，则可以考虑不写。 但是其他情况，尤其是逻辑比较长且复杂的时候， 一定要写上注释。

### 第二题

两表联合查询的 sql，也是非常简单的。  *本题参考了上家公司的面试题。*

本题主要考察下面几个点：

- sql 语句的阅读能力
-  两表联合查询的能力

下面给出一个示例的解题 sql 语句。（*同样纯手写， 并没有经过 IDE 测试*）

```mysql
SELECT COUNT(1) FROM student s,grade g
WHERE s.grade = g.id AND g.type = 1
```



## 后续

这里说一个小插曲。 

有一个人和我约了面试之后，在面试当日和我说他不来了。 不来嘛就算了，但是呢，大约20天左右之后吧，那个人突然来到我们公司要求进行面试。 这人来之前并没有和我取得任何联系， 同事和我说有面试的时候，我是极其惊讶的，因为我记得我并没有约任何一个人哇！因为笔者脾气还算 OK，所以还是打印简历让他进行面试了， 结果嘛，不用说，当然是 PASS 掉了。 

我有问过他为什么又要过来面试了， 他回答说：“我后来又看了一下你们公司职位描述，感觉你们公司挺好的，所以就来试试。” 

这种有点我行我素的人， 笔者基本是不会考虑录用的。如果碰到一个脾气大的面试官，可能直接就让他走了， 面试都不面试。 （*笔者基本上也是走个流程，随便问问。虽然只有这个人的第一题输出是按照规则来的，但是笔者仍然不会选择录用他。*） 

这里笔者也建议你不要这样做， 因为基本上是在浪费自己的时间。 如果你爽约了，还想再尝试一次，你**一定**要先和面试官取得联系。



总之，笔试题是一个很好用的过滤器， 如果你也需要面试别人， 建议你使用一下。 



**🎉🎉🎉晚安，中秋节快乐，国庆节快乐。🎉🎉🎉**

---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/%E5%85%AC%E5%8F%B8%E9%9D%A2%E8%AF%95%E4%BD%BF%E7%94%A8%E7%9A%84%E9%9D%A2%E8%AF%95%E9%A2%98/  

