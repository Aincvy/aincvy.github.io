# 一种以0,0为原点的AOI九宫格的实现方式


## 简述


假如原点（0,0）的位置在一个角落， 比如左下角， 那么全部格子都将会在第一象限，不用考虑负数的形式， 这种做法在使用格子坐标进行遍历的时候应该是比较简单的。 

但是如果无法确定原点的位置，就是说在无法找到一个比较好的角落的情况下， 笔者认为有下面几种方法做处理。 
- 仍然完全使用第一象限， 但是对每个单位坐标都添加一个偏移量，使转换出来的格子坐标远离原点。   假如格子坐标脱离第一象限就直接报错。 
- 允许格子坐标(0,0) 的面积是其他格子的4倍。
- 将原点设置在中间， 并且没有(0,0)的格子，中间四个格子分别为： (1,1);(-1,1);(1,-1);(-1,-1)

其他
- 单位坐标： 场景内单位的坐标。
- 格子坐标： AOI格子的坐标。

## 主要内容

*本篇使用java语言进行代码编写。*

九宫格的核心想法是把地图划分为AxB个格子， 把N个单位放到一个格子里面， 在单位移动的时候更新他所在的格子， 在广播的时候只需要遍历周边的格子的单位列表。   
这样做的好处是减少不必要的计算量： 假如整个地图有1000个单位， 在玩家I旁边的单位只有70个。 
- 假如没有做AOI的话， 每次进行广播，或者其他位置计算的时候 都需要和其他999个单位进行距离计算， 这个计算量是非常大的， 因为计算距离的时候需要开方。  还有一个重要原因是过远的单位是没有必要计算的。
- 玩家I需要与其他999个单位进行距离计算， 玩家J也是， 这样会使计算量增长的十分恐怖。  
- 假如存在AOI的话，  在广播的时候就可以不计算距离， 直接按格子进行广播。 在需要计算距离的时候也仅仅是只需要计算70个， 而非999个。  计算量的减少是非常可观的。 

### 单位坐标转格子坐标

在把某个单位第一次放入AOI网格中， 或者某个单位进行了传送之后， 都是需要根据单位坐标求得格子坐标。 

下面先看一段代码： 
```java

class HelloWorld {
    
    public static void a(float x) {
        int offset = 15000;
        System.out.println((int)((x &#43; offset) / 15));
    }
    
    public static void b(float x) {
        System.out.println((int)((x) / 15));
    }
    
    public static void main(String[] args) {
        a(0.1f);           // 1000
        a(-0.1f);          // 999
        a(0f);             // 1000
        
        b(0.1f);           // 0
        b(-0.1f);          // 0
        b(0f);             // 0
 
        b(15.1f);          // 1
        b(-15.1f);         // -1
    }
}

```

- 方法a 展示了一下如何在换算的时候使用偏移量，以及在结果上偏移量是真的生效了的证明
- 方法b 展示了在不做任何处理的情况下，格子 0,0 将会是其他格子的4倍的情况。

现在， 笔者将说一下如何去掉格子0,0  让所有格子的单位面积都一致。 

看下面的一段代码： 
```java

    public static int c(float x) {
        float gridBorder = 15;
        if (x &gt;= 0) x &#43;= gridBorder;
        else x -= gridBorder;
        return (int)((x) / 15);
    }

    public static void main(String[] args) {
        System.out.println(c(0.1f));   // 1
        System.out.println(c(-0.1f));   // -1
        System.out.println(c(0f));   // 1
        System.out.println(c(15.1f));   // 2
        System.out.println(c(-15.1f));   // -2
    }
```

即： 往四个方向添加一段偏移量即可。   *笔者这里演示的是单轴，2D平面地图里面有2个轴，所以是4个方向。*

### 遍历

这里使用下面的结构来记录格子坐标与格子映射关系。  

```java

class AOIManager {
    // 行， 列， 格子
    com.google.common.collect.Table&lt;Integer, Integer, AOIGrid&gt; gridTable;
}

@Data
class AOIGrid {
    private int x;
    private int y;

    // 玩家列表，场景对象列表和其他属性省略
}
```

如果将原点放到一个角落， 并使用偏移量的形式来实现的话， 这里应该没什么问题，正常遍历即可。   
但是如果将原点设置在中间的话，就需要下面的方法来做格子坐标的计算。 

```java
    /**
     * @param a    格子坐标
     * @param b    偏移量
     */
    public static int addAndPassZero(int a, int b) {
        int x = a &#43; b;
        if(x == 0 || (a ^ x) &lt; 0) {
            return b &gt;= 0 ? x &#43; 1 : x - 1;
        } 
        return x;
    }

    public static void main(String[] args) {        
        System.out.println(addAndPassZero(1,1));      // 2
        System.out.println(addAndPassZero(1,-1));     // -1
        System.out.println(addAndPassZero(-1,-1));    // -2
        System.out.println(addAndPassZero(-3,5));     // 3
        System.out.println(addAndPassZero(-3,-5));    // -8
    }
```

对照一个2D平面视图就可以看到， 0 被跳过了。  参考视图： https://www.geogebra.org/calculator

当获取到一个 `AOIGrid` 对象之后， 使用`addAndPassZero`方法就可以获取到周边格子的正确坐标，再使用坐标转换成`AOIGrid` 对象就可以实现遍历了。 


### 移动

在某个单位移动之后，应该更新他所在的格子， 以及广播给一些格子 这个单位离开了， 广播给另外一些格子，这个单位进来了。 
下面是一些补充代码
```java
@Data
class AOIGrid {
    private int x;
    private int y;

    private float fuzzyRightTopPosX;
    private float fuzzyRightTopPosZ;
    private float fuzzyLeftBottomPosX;
    private float fuzzyLeftBottomPosY;

    private float rightTopPosX;
    private float rightTopPosY;

    // 玩家列表，场景对象列表和其他属性省略
}
```

- `fuzzy` 开头的属性表示了一个比格子面积大一些的范围， 当玩家离开这个范围的时候才视为离开了当前格子。 目前笔者给的值是8% 读者可以根据自己的需求修改。  使用 fuzzy的目的是 避免玩家的轻微移动会不停的切换格子。 
- `rightTopPosX, rightTopPosY` 在进行范围计算的时候， 直接进行减法， 然后和格子的边长进行比较即可。

在玩家I切换格子的时候， 是存在一个方向的。 比如从`-1, -1` 走到 `1, -1`  是x 从`-1` 变到了`1`， 那么方向是向右的， 或者说是向东的。 
- 那么玩家I的旧的广播范围的最西边的一列格子， 应该给这列格子的所有玩家都广播一条消息： 玩家I离开了。 让客户端把玩家I从场景中剔除
- 玩家I的新的广播范围的最东边的一列格子， 应该给这列格子的所有玩家都广播一条消息： 玩家I 进入了场景。


### 其他

每个`AOIGrid` 对象上带有一个周边格子的引用的话，在遍历周边格子的时候就不需要查询`gridTable` ， 理论上来说性能会快一些。 但是远距离的格子还是使用`gridTable` 更好一些。  同时， 建立引用的时候也是需要依赖`gridTable` 的。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/%E4%B8%80%E7%A7%8D%E4%BB%A500%E4%B8%BA%E5%8E%9F%E7%82%B9%E7%9A%84aoi%E4%B9%9D%E5%AE%AB%E6%A0%BC%E7%9A%84%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F/  

