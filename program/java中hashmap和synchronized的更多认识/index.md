# Java中HashMap和synchronized的更多认识


## 主要内容

### HashMap.keySet()
这个方法返回一个内部类 `HashMap.KeySet`的实例， 多次调用这个方法将获取同一个对象。
- 调用这个对象的 `remove` 方法 将删除HashMap 里面的元素。
- 调用这个对象的 `clear` 方法将清空 HashMap

可见源码： 
```java
public Set&lt;K&gt; keySet() {
    Set&lt;K&gt; ks = keySet;
    if (ks == null) {
        ks = new KeySet();
        keySet = ks;
    }
    return ks;
}

final class KeySet extends AbstractSet&lt;K&gt; {
    public final int size()                 { return size; }
    public final void clear()               { HashMap.this.clear(); }
    public final Iterator&lt;K&gt; iterator()     { return new KeyIterator(); }
    public final boolean contains(Object o) { return containsKey(o); }
    public final boolean remove(Object key) {
        return removeNode(hash(key), key, null, false, true) != null;
    }
    public final Spliterator&lt;K&gt; spliterator() {
        return new KeySpliterator&lt;&gt;(HashMap.this, 0, -1, 0, 0);
    }
    public final void forEach(Consumer&lt;? super K&gt; action) {
        Node&lt;K,V&gt;[] tab;
        if (action == null)
            throw new NullPointerException();
        if (size &gt; 0 &amp;&amp; (tab = table) != null) {
            int mc = modCount;
            for (Node&lt;K,V&gt; e : tab) {
                for (; e != null; e = e.next)
                    action.accept(e.key);
            }
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
}
```

这个方法的返回值一般用于遍历 keys, 如果对这个Set做一些操作，但是并不想修改HashMap的数据的话， 可以这么做： 
```java
Map&lt;String, Long&gt; map = new HashMap&lt;&gt;();
Set&lt;String&gt; keys = new HashSet&lt;&gt;(map.keySet());
// 这么做之后， 修改 `keys` 里面的元素就不会影响到 `map` 了
// 但是如果修改keys的元素的值， 那么就会影响到 map里面的key, 这个自然不必多说。
// 在本例里面 元素的类型是 String, 我们都知道String是不可变的， 所以无法修改它的内部属性
// 这里在额外说一些内容 
// 如果 Map&lt;K,V&gt; 中的 K是一个 自定义的复合类型， 在放入map中之后就尽量不要修改它的值
// 详情看下面的代码
```

如果 `Map&lt;K,V&gt;` 中的 K是一个 自定义的复合类型， 那么就尽量不要修改它的值， 因为如果你修改了， 则可能会发生一些预期之外的行为。 其实最好的还是不使用复合类型作为KEY， 仅仅使用简单类型和String，或者使用一些 不可变的对象 。
下面是笔者测试使用的代码。 

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Map.Entry;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.NoArgsConstructor;
import lombok.ToString;

public class HashMapTest {
    
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @EqualsAndHashCode
    @ToString
    public static class Point{
        private int x;
        private int y;
    }

    public static void main(String[] args) {
        
        Map&lt;Point, Integer&gt; map = new HashMap&lt;&gt;(8);
        Point point1 = new Point(1,1);
        map.put(point1, 1);
        System.out.println(map.get(new Point(1, 1)));    // 1
        point1.setX(2);
        point1.setY(2);
        System.out.println(map.get(point1));     // null
        System.out.println(map.get(new Point(2, 2)));    // null
        point1.setX(1);
        point1.setY(1);
        System.out.println(map.get(point1));    // 1

        point1.setX(2);
        point1.setY(2);
        System.out.println(map.get(point1));    // null

        Point point3 = new Point(3, 3);
        map.put(point3, 3);
        for (Entry&lt;Point, Integer&gt; entry : map.entrySet()) {
            System.out.println(entry);  
            // HashMapTest.Point(x=2, y=2)=1
            // HashMapTest.Point(x=3, y=3)=3
        }
        for (Point point : map.keySet()) {
            System.out.println(point.toString() &#43; &#34;=&#34; &#43; map.get(point));  
            // HashMapTest.Point(x=2, y=2)=null
            // HashMapTest.Point(x=3, y=3)=3
        }   

        System.out.println(map.get(point1));    // null
        for (int i = 0; i &lt; 160; i&#43;&#43;) {
            Point point = new Point(i,i*2);
            map.put(point, i&#43;1);
        }
        System.out.println(map.size());         // 162
        System.out.println(map.get(point1));    // null

        map.put(new Point(1, 1), 10);
        System.out.println(map.size());         // 163
        System.out.println(map.get(point1));    // null
        System.out.println(map.get(new Point(1, 1)));    // 10
        System.out.println(new Point(1, 1).equals(point1));    // false
        System.out.println(new Point(1, 1).hashCode() == point1.hashCode());  // false

        System.out.println(map.get(new Point(2, 2)));   // null
        System.out.println(new Point(2, 2).equals(point1));   // true
        System.out.println(new Point(2, 2).hashCode() == point1.hashCode());   // true

        point1.setX(1);
        point1.setY(1);
        System.out.println(map.get(point1));       // 1
        System.out.println(map.size());            // 163
        System.out.println(new Point(1, 1).equals(point1));   // true
        System.out.println(new Point(1, 1).hashCode() == point1.hashCode());   // true
        
        System.out.println(map.get(new Point(1, 1)));    // 1
    }

}


```

前面可以看出， 假如 map.put(b,c), map.get(a). 如果 a.hashCode()和b.hashCode() 相等，并且a.equals(b) 返回true, 即使是两个不一样的对象， 也可以获取到 c.   
  
从代码中可以看到 假如 point1的值修改了， 那么你就永远的失去了它，除非你把它修改回去。  
- map.put(point1, 1); 
- 修改 point1的值
- map.get(point1);   // null
- map.get(new Point(2, 2));   // null

而修改回去又会产生新的问题， 假如在修改之前， put了一个使用原来key相同的值， 那么修改回去之后， 后面put的那个值将会永远丢失。 
- map.put(new Point(1, 1), 10);
- 修改回  point1的值
- System.out.println(map.get(point1));   // 1
- System.out.println(map.get(new Point(1, 1)));   // 1
- 可以看到， 我们的10 丢失了， 但是实际上 map里面的元素数量并没有减少。

原因可能和 `HashMap.Node` 类的实现有关系。 
可以看下面的代码 *节选自 java.util.HashMap*
```java
static class Node&lt;K,V&gt; implements Map.Entry&lt;K,V&gt; {
        final int hash;
        final K key;
        V value;
        Node&lt;K,V&gt; next;

        Node(int hash, K key, V value, Node&lt;K,V&gt; next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key &#43; &#34;=&#34; &#43; value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry&lt;?,?&gt; e = (Map.Entry&lt;?,?&gt;)o;
                if (Objects.equals(key, e.getKey()) &amp;&amp;
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```
这里的 Node 会缓存 put时的key的 hashCode。 *其实是一个间接值，即并不是原来的hashCode,但产生自原来的hashCode.*

也许还有更多的测试值得我们去做， 但是笔者认为现在已经拿出足够的理由使我们不使用可变的自定义复合类型作为Map.KEY .
*下面的相关内容是没有做测试的部分*
- containsKey
- remove  

### synchronized 
TODO 待补充


&lt;!-- 
据笔者所知 synchronized 是一个java的关键字， 用于加锁， 相比于`ReentrantLock`， synchronized 更加轻便， 没有选项， 但是存在一些优化措施 。   
比如所谓的锁升级机制就是指的 synchronized， ReentrantLock 则没有这种机制。   
- 锁升级机制是说 一个对象的 对象头(cpp部分) 有一个叫做 mark word 的东西， 那个字段有2个位是表示锁的状态的的。    
- 它有四种状态 无锁， 偏向锁， 轻量锁， 重量级锁。     
- 锁只能升级， 不能降级。 

当一个对象没有被任何线程锁住的时候，它是无锁的。 此时一个线程A 请求给这个对象加锁， 那么线程A会假设没有别的线程和它争抢， 它将对象头修改成 偏向锁，然后进入下一步操作。 但是此时，并没有真正的锁产生。 这么做当然是为了效率， 听说加锁是一个消耗比较重的操作。 
后来， 线程B 也想给这个对象加锁，那么，线程B会看到这个对象处于 偏向锁的状态，随后它修改对象头成轻量级锁， 并且进行一段时间的自旋。自旋就是空转， 这时线程B是没有进入休眠的， 线程B 假设线程A很快就会释放这个对象的锁。 
假如线程A真的很快的释放了锁， 那么线程B就会拿到锁， 并且进入下一步操作。 此时还是没有真正的 锁产生， 效率是比较高的。 但是假如线程A并没有很快的释放锁， 那么自旋到一定的次数或者时间， 线程B会把锁升级成重量级锁， 并且进入休眠， 此时就产生了一个真正的锁。

*对象头对于各个线程的同步机制在cpp层面是如何实现的， 笔者并不清楚。。 因为java的线程和cpp的线程听说是一对一的*   

`wait` 方法会释放掉 synchronized 持有的锁。
*以下代码节选自 java.util.TimerThread*
```java
/**
 * The main timer loop.  (See class comment.)
 */
private void mainLoop() {
    while (true) {
        try {
            TimerTask task;
            boolean taskFired;
            synchronized(queue) {
                // Wait for queue to become non-empty
                while (queue.isEmpty() &amp;&amp; newTasksMayBeScheduled)
                    queue.wait();
                if (queue.isEmpty())
                    break; // Queue is empty and will forever remain; die

                // Queue nonempty; look at first evt and do the right thing
                long currentTime, executionTime;
                task = queue.getMin();
                synchronized(task.lock) {
                    if (task.state == TimerTask.CANCELLED) {
                        queue.removeMin();
                        continue;  // No action required, poll queue again
                    }
                    currentTime = System.currentTimeMillis();
                    executionTime = task.nextExecutionTime;
                    if (taskFired = (executionTime&lt;=currentTime)) {
                        if (task.period == 0) { // Non-repeating, remove
                            queue.removeMin();
                            task.state = TimerTask.EXECUTED;
                        } else { // Repeating task, reschedule
                            queue.rescheduleMin(
                                task.period&lt;0 ? currentTime   - task.period
                                            : executionTime &#43; task.period);
                        }
                    }
                }
                if (!taskFired) // Task hasn&#39;t yet fired; wait
                    queue.wait(executionTime - currentTime);
            }
            if (taskFired)  // Task fired; run it, holding no locks
                task.run();
        } catch(InterruptedException e) {
        }
    }
}
```
通过这段代码 我们可以看到， 他是在 while 内部进行 synchronized的， 假如 `queue.wait();` 这行代码没有释放锁的话 ， 那么其他线程就永远拿不到`queue`的锁了。
而 再看看 `Timer` 类的实现内容。
```java
private void sched(TimerTask task, long time, long period) {
    if (time &lt; 0)
        throw new IllegalArgumentException(&#34;Illegal execution time.&#34;);

    // Constrain value of period sufficiently to prevent numeric
    // overflow while still being effectively infinitely large.
    if (Math.abs(period) &gt; (Long.MAX_VALUE &gt;&gt; 1))
        period &gt;&gt;= 1;

    synchronized(queue) {
        if (!thread.newTasksMayBeScheduled)
            throw new IllegalStateException(&#34;Timer already cancelled.&#34;);

        synchronized(task.lock) {
            if (task.state != TimerTask.VIRGIN)
                throw new IllegalStateException(
                    &#34;Task already scheduled or cancelled&#34;);
            task.nextExecutionTime = time;
            task.period = period;
            task.state = TimerTask.SCHEDULED;
        }

        queue.add(task);
        if (queue.getMin() == task)
            queue.notify();
    }
}
```
这里的代码说 在添加任务 之前， 它需要先执行  `synchronized(queue)` 语句。
这个方法肯定和 `TimerThread` 是在不同的线程执行的。 
假如 `wait` 方法不释放锁的话， 这里就会死锁了， 实际上并没有死锁发生， 所以可以说明 `wait` 会释放锁。 
*这种反推是不是不太合理啊。。。*


ReentrantLock 有两个 `tryLock`方法，用于尝试加锁， 给予一定的等待时间。
```java
public boolean tryLock(long timeout, TimeUnit unit)
        throws InterruptedException {
    return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}
```
还有下面的一些方法
- isLocked    是否被任一线程持有？
- isHeldByCurrentThread   是否被当前线程持有？
- isFair     是否公平？ 

可重入式锁是指 同一个 线程可以对同一个锁进行多次加锁操作。 但是每次加锁都需要对应一次解锁操作。 

--&gt;

---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/java%E4%B8%ADhashmap%E5%92%8Csynchronized%E7%9A%84%E6%9B%B4%E5%A4%9A%E8%AE%A4%E8%AF%86/  

