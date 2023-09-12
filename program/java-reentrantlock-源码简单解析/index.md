# Java ReentrantLock 源码简单解析


源码简单解析。    *源码来自 openjdk 17.0.7 2023-04-18*

## 主要内容

### 内部类以及构造方法

内部类
- Sync extends AbstractQueuedSynchronizer
  - FairSync
  - NonfairSync

构造方法 
```java
    /**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

    // 笔者：  其余操作基本上都是对 sync的一个封装。 
```

FairSync 和 NonfairSync 的不同之处在于 加锁的时候会判断有无等待中的队列
```java
    /**
     * Sync object for non-fair locks
     */
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        final boolean initialTryLock() {
            Thread current = Thread.currentThread();
            if (compareAndSetState(0, 1)) { // first attempt is unguarded
                setExclusiveOwnerThread(current);
                return true;
            } else if (getExclusiveOwnerThread() == current) {
                int c = getState() + 1;
                if (c < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(c);
                return true;
            } else
                return false;
        }

        /**
         * Acquire for non-reentrant cases after initialTryLock prescreen
         */
        protected final boolean tryAcquire(int acquires) {
            if (getState() == 0 && compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }
    }


    /**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        /**
         * Acquires only if reentrant or queue is empty.
         */
        final boolean initialTryLock() {
            Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedThreads() && compareAndSetState(0, 1)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            } else if (getExclusiveOwnerThread() == current) {
                if (++c < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(c);
                return true;
            }
            return false;
        }

        /**
         * Acquires only if thread is first waiter or empty
         */
        protected final boolean tryAcquire(int acquires) {
            if (getState() == 0 && !hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }
    }
```
公平锁在获取锁的时候总是会判断是否存在等待中的队列， 而非公平锁则不判断， ~~所以非公平的锁的性能会更好一些。~~  副作用是有些线程可能要一直睡眠了。   

刚刚又查看了一下代码，非公平锁的性能肯定会好一些， 但是感觉区别似乎不大啊。。。  
在释放锁的时候会直接调用 `signalNext()`  然后对第一个等待中的线程进行 `unpark` 操作， 随后第一个线程就会被唤醒，然后执行代码。 在失败几次之后，再次进入睡眠。 


### 加锁部分代码解析
JDK 内部的源代码。 
```java
    /**
     * Main acquire method, invoked by all exported acquire methods.
     *
     * @param node null unless a reacquiring Condition
     * @param arg the acquire argument
     * @param shared true if shared mode else exclusive
     * @param interruptible if abort and return negative on interrupt
     * @param timed if true use timed waits
     * @param time if timed, the System.nanoTime value to timeout
     * @return positive if acquired, 0 if timed out, negative if interrupted
     */
    final int acquire(Node node, int arg, boolean shared,
                      boolean interruptible, boolean timed, long time) {
        Thread current = Thread.currentThread();
        byte spins = 0, postSpins = 0;   // retries upon unpark of first thread
        boolean interrupted = false, first = false;
        Node pred = null;                // predecessor of node when enqueued

        /*
         * Repeatedly:
         *  Check if node now first
         *    if so, ensure head stable, else ensure valid predecessor
         *  if node is first or not yet enqueued, try acquiring
         *  else if node not yet created, create it
         *  else if not yet enqueued, try once to enqueue
         *  else if woken from park, retry (up to postSpins times)
         *  else if WAITING status not set, set and retry
         *  else park and clear WAITING status, and check cancellation
         */

        for (;;) {
            if (!first && (pred = (node == null) ? null : node.prev) != null &&
                !(first = (head == pred))) {
                if (pred.status < 0) {
                    cleanQueue();           // predecessor cancelled
                    continue;
                } else if (pred.prev == null) {
                    Thread.onSpinWait();    // ensure serialization
                    continue;
                }
            }
            if (first || pred == null) {
                boolean acquired;
                try {
                    if (shared)
                        acquired = (tryAcquireShared(arg) >= 0);
                    else
                        acquired = tryAcquire(arg);
                } catch (Throwable ex) {
                    cancelAcquire(node, interrupted, false);
                    throw ex;
                }
                if (acquired) {
                    if (first) {
                        node.prev = null;
                        head = node;
                        pred.next = null;
                        node.waiter = null;
                        if (shared)
                            signalNextIfShared(node);
                        if (interrupted)
                            current.interrupt();
                    }
                    return 1;
                }
            }
            if (node == null) {                 // allocate; retry before enqueue
                if (shared)
                    node = new SharedNode();
                else
                    node = new ExclusiveNode();
            } else if (pred == null) {          // try to enqueue
                node.waiter = current;
                Node t = tail;
                node.setPrevRelaxed(t);         // avoid unnecessary fence
                if (t == null)
                    tryInitializeHead();
                else if (!casTail(t, node))
                    node.setPrevRelaxed(null);  // back out
                else
                    t.next = node;
            } else if (first && spins != 0) {
                --spins;                        // reduce unfairness on rewaits
                Thread.onSpinWait();
            } else if (node.status == 0) {
                node.status = WAITING;          // enable signal and recheck
            } else {
                long nanos;
                spins = postSpins = (byte)((postSpins << 1) | 1);
                if (!timed)
                    LockSupport.park(this);
                else if ((nanos = time - System.nanoTime()) > 0L)
                    LockSupport.parkNanos(this, nanos);
                else
                    break;
                node.clearStatus();
                if ((interrupted |= Thread.interrupted()) && interruptible)
                    break;
            }
        }
        return cancelAcquire(node, interrupted, interruptible);
    }
```

下面是代码的一些解析 
- `Thread.onSpinWait()`   
  - 解释： https://stackoverflow.com/a/44637990/11226492
  - 大致是说， 当前线程还是处于运行状态， 但是指示CPU 分配给当前线程的资源少一些
- `LockSupport.park()`    
  - > Disables the current thread for thread scheduling purposes unless the permit is available.
- 尝试获取锁的时候会调用 `tryAcquireShared` 或者`tryAcquire` , 但是 AQS 本身没实现这两个方法， 是交给子类去实现的
- 队列应该是使用了一个双向链表实现的， 类是 `AbstractQueuedSynchronizer.Node`, 头尾的字段分别是 `head, tail`
- 关于 `acquire`方法中的`Node node` 参数， 只有 `public class ConditionObject implements Condition, java.io.Serializable` 类里调用的时候会传值，在AQS 内部进行调用的时候， 都是传入的null 
- 下面的逻辑有点多， 但是并不难
- 下面分析一下 `ReentrantLock.tryLock(long timeout, TimeUnit unit)` 的调用链
  - `ReentrantLock.Sync.tryLockNanos(long nanos) throws InterruptedException`
    - `initialTryLock() || tryAcquireNanos(1, nanos)`
    - 分支1： `initialTryLock()`       见上述源码部分即可
    - 分支2： `tryAcquireNanos(1, nanos)`
      - `if (tryAcquire(arg)) return true;`  tryAcquire(arg) 见上述源码即可
      - `acquire(null, arg, false, true, true, System.nanoTime() + nanosTimeout)`   
        - 重点参数
          - `Node node` 参数的传入值是 null 
          - `boolean shared`   传入值false
        - 第一轮循环
          - 第一段主要 if  不会执行， 因为 `node == null`
            - `if (!first && (pred = (node == null) ? null : node.prev) != null &&
                  !(first = (head == pred)))`
          - 第二段 主要 if 会执行， 因为 ` pred == null `
            - 因为 `shared==false`, 所以实际上会调用 `acquired = tryAcquire(arg)` 语句
              - `NonfairSync/FairSync` 的 `tryAcquire`  方法
            - 拿到锁之后 这里就直接返回 1 了
          - 没拿到锁就会执行第三段 if,  因为 `node == null`   随后进入第二轮循环
        - 后续循环的假设场景是  线程A 拿到了锁资源， 且没释放。 我们来看一下线程B的执行流程。
        - 第二轮循环
          - 在第一轮循环中没有拿到锁， 就会进入第二轮循环， 循环的起始语句是 `for(;;)`
          - 参数更新：  `node=new ExclusiveNode()`
          - 第一段主要 if 不会执行，  因为 `((pred = node.prev) != null)`  的条件不满足
          - 第二段主要 if 会执行， 因为 ` pred == null` 
            - 这里和第一次循环一样， 如果拿到锁就直接返回1了
          - 第三段主要if 会执行这个分支 `else if (pred == null)`
            - 简单来说， 就是把当前线程放入到 node.waiter 属性上， 然后把node 入队列， 此时会更新 node.prev 属性
            - 然后进入第三轮循环
        - 第三轮循环
          - 参数更新： `node.prev`  指向队列中的前一个node, 或者 当前队列的头部
            - 假如队列中没有其他元素的话， 那么就指向的队列头部， 也是当前的状态
          - 第一段主要 if 不会执行， 但是 first 会更新成 true 
          - 第二段主要 if 会执行， 因为 `first == true `
            - 假如没有拿到锁，则会进入 第三段主要if 
          - 第三段主要 if 会进入到 `else if (node.status == 0)` 这一部分
            - 将会执行语句 `node.status = WAITING` 
            - `static final int WAITING   = 1;          // must be 1`
          - 然后进入第四轮循环
        - 第四轮循环
          - 参数更新：  无
          - 第一段主要 if 不会执行， 因为 `first == true `
          - 第二段主要 if 会执行， 因为 `first == true `
            - 假如没有拿到锁，则会进入 第三段主要if 
          - 第三段主要 if 会进入到 最终的 `else` 部分
            - 执行 `spins = postSpins = (byte)((postSpins << 1) | 1);`， 结果是 `spins = postSpins = 1`
            - 因为 `timed == true`,以及目标超时时间大于当前时间， 所以会执行 `LockSupport.parkNanos(this, nanos);`
              - 如果目标超时时间已经过掉了， 则终止循环， 调用 `cancelAcquire(node, interrupted, interruptible);`
            - `LockSupport.parkNanos`  将会暂时停止调度当前线程， 直到等待的时间到达了。  这个约等于 `Thread.sleep()` 但是别的线程可以调用 `LockSupport.unpark(Thread thread)` 来恢复调度当前线程
            - `node.clearStatus();`   好像是把 status 设置成0 
            - 检测当前线程有没有中断， 如果已经中断，就跳出循环， 调用 `cancelAcquire` ，否则进入下一轮循环。
              - 调用线程的`interrupt` 方法可以中断该线程
            - 进入第五轮循环
        - 第五轮循环
          - 参数更新：  `spins = 1`
          - 第一段主要 if 不会执行， 因为 `first == true `
          - 第二段主要 if 会执行， 因为 `first == true `
            - 假如没有拿到锁，则会进入 第三段主要if 
          - 第三段主要 if 会进入到 `else if (first && spins != 0)`   因为目前 spins 是1
            - `--spins; ` 
            - 会调用 `Thread.onSpinWait();`  指示 runtime  现在处于 忙等待状态
          - 随后进入第六轮循环
        - 第六轮循环
          - 参数更新：  `spins = 0`
          - 第一段主要 if 不会执行， 因为 `first == true `
          - 第二段主要 if 会执行， 因为 `first == true `
            - 假如没有拿到锁，则会进入 第三段主要if 
          - 第三段主要if , 会因为时间过期， 而中断循环， 随后调用 `cancelAcquire` 取消尝试拿锁
    - 经过分析得知， 即使当前线程进入睡眠， 也是在自旋失败个几次之后。
 - 下面尝试分析一下 释放锁的调用链
   - `ReentrantLock.unlock()`    
     - 这里简化一下调用链， 只说明一些关键的地方
     - `ReentrantLock.Sync.tryRelease(int arg)`
       - 加锁次数 - 1。   *state - 1, 1是传入的参数arg*
       - 加锁次数 变成0的时候，  释放锁， 同时调用 `signalNext` 给队列中的下一个线程发信号
     - `signalNext`
       - 使用 `LockSupport.unpark(s.waiter)`   唤醒目标线程， 对应 `LockSupport.park(this)`  语句的调用。




---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/java-reentrantlock-%E6%BA%90%E7%A0%81%E7%AE%80%E5%8D%95%E8%A7%A3%E6%9E%90/  

