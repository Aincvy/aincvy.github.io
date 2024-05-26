# 在权重随机中使用遮罩位来实现去重


## 简介

需求： 在权重随机中需要去重处理。

主体思想： 
- 使用遮罩来确定某个元素是否已经被随机出来了。 
- 笔者认为使用遮罩比起使用一个新的列表的好处是会减少GC的压力。 

## 主要内容

*实现语言采取Java， 以下代码仅做示意使用。* 

```java
@Data
public class WeightObject&lt;T&gt; {
    private T obj;
    private int weight;
}

public final class Util {

    public static int mask(int v, int i) {
        return v | (1 &lt;&lt; i);
    }

    public static boolean isMask(int v, int i) {
        return (v &amp; (1 &lt;&lt; i)) != 0;
    }

    /**
     * 计算总权重
     */
    public static&lt;T&gt; int totalWeight(List&lt;WeightObject&lt;T&gt;&gt; list, int mask){
        int total = 0;
        for (int i = 0; i &lt; list.size(); i&#43;&#43;) {
            if (isMask(mask, i)) {
                continue;
            }

            total &#43;= list.get(i).getWeight();
        }
        return total;    
    }

    public static int random(int max) {
        return 0;   // 省略
    }

    public static&lt;T&gt; List&lt;T&gt; weightRandomDistinct(List&lt;WeightObject&lt;T&gt;&gt; list, int count) {
        int mask = 0;

        if(list.size() &gt;= 32) {
            // 此处可以考虑使用 long类型， 或者使用 BitSet, 或者回归var tmp = new ArrayList(list);  tmp.remove(j) 的形式。
            throw new IllegalArgumentException(&#34;size must be less than 32&#34;);
        }

        var result = new ArrayList&lt;T&gt;(count);
        for (int i = 0; i &lt; count; i&#43;&#43;) {
            var total = totalWeight(list, mask);
            var randomWeight = random(total);
            var currentWeight = 0;
            for (int j = 0; j &lt; list.size(); j&#43;&#43;) {
                if (isMask(mask, j)) {
                    continue;
                }

                var element = list.get(j);
                currentWeight &#43;= element.getWeight();
                if (currentWeight &gt;= randomWeight) {
                    mask = mask(mask, j);
                    result.add(element.getObj());
                    break;
                }
            }
        }

        return result;
    }
}

```

核心思想就是计算总权重的时候使用遮罩去除某些元素，  在遍历元素的时候， 也使用遮罩去除某些元素。 

如果使用 `var tmp = new ArrayList(list);  tmp.remove(j)` 的形式， 每次进行权重随机的时候都需要创建一个新的 ArrayList ， 使用之后就抛弃， 应该会产生较大的GC压力。  比如 16随3，  30 随3 的情况下， 使用遮罩之后，GC方面应该就会好很多。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/%E5%9C%A8%E6%9D%83%E9%87%8D%E9%9A%8F%E6%9C%BA%E4%B8%AD%E4%BD%BF%E7%94%A8%E9%81%AE%E7%BD%A9%E4%BD%8D%E6%9D%A5%E5%AE%9E%E7%8E%B0%E5%8E%BB%E9%87%8D/  

