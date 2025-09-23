# @TransactionalEventListener 调用别得Service,事务不保存得问题记录



先看一段代码： 

```java

@Service
public class A {

    private SomeRepository repo

    @Transactional
	public void m1(){
	  m2();
	}
	
	@Transactional
	public void m2(){
	  Some s = xxxx;
	  repo.save(s);
	}

}

@Service
@AllArgsConstructor
public class B {

    private final A a;
    private final ApplicationEventPublisher eventPublisher;
    
    @TransactionalEventListener(fallbackExecution = true)
    public void onEvent(Event e){
	    a.m1();
    }
    
    public void doSomething(){
        Event e = new Event();
        eventPublisher.publishEvent(e);
    }

}

@Service
@AllArgsConstructor
public class C {

    private final B b;
    
    @Transactional
    public void finish(){
	    b.doSomething();
    }

}

@RestController()
@RequestMapping("/v1")
@AllArgsConstructor
@Slf4j
public class Controller {

    private final C c;
    
    @PostMapping("/test")
    public void test(){
	    c.finish();
    }

}


```

在这种调用链路之下，当一个 `/v1/test`请求过来得时候， 最后得 a service 得 m1, m2 方法都成功调用了， 日志也有。 但是数据s 就是无法入库。    
从别的路线调用  a service 得 m1方法的时候， 数据会入库。

笔者使用的版本是： 
- spring boot 2.7.18
- spring 5.3.31
- hibernate 5.6.15

经过调试和探查发现在 @TransactionalEventListener 注解得方法添加上  `@Transactional(propagation = Propagation.REQUIRES_NEW)` 就可以了。。 


比如：
```java
    @TransactionalEventListener(fallbackExecution = true)
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void onEvent(Event e){
	    a.m1();
    }
```

笔者怀疑是A.m1(), A.m2()  添加到了一个已经结束得事务中了。  这样得话这个已经结束得事务不会再次被提交。。  但是这个怀疑可能也是不准确得。。 因为笔者原本在 B.onEvent 还有这样一段代码： 

```java
        boolean isInTransaction = TransactionSynchronizationManager.isActualTransactionActive();
        if (!isInTransaction) {
          log.warn("[onCallWebhookEvent] 事务没有开启，执行了 webhook: {}", event);
        }
```
这个警告 会被打印出来。。  这就说明当前不在一个事务里面。。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/@transactionaleventlistener-%E8%B0%83%E7%94%A8%E5%88%AB%E5%BE%97service%E4%BA%8B%E5%8A%A1%E4%B8%8D%E4%BF%9D%E5%AD%98%E5%BE%97%E9%97%AE%E9%A2%98%E8%AE%B0%E5%BD%95/  

