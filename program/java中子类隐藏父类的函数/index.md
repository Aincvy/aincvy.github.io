# Java中子类隐藏父类的静态函数



```java
class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Try programiz.pro");
        
        Basic.printNumber();
        Child.printNumber();
    }
}

class Basic {
    public static int getNumber(){
        return 1;
    }
    
    public static void printNumber(){
        System.out.println(getNumber());
    }
}

class Child extends Basic {
    public static int getNumber(){
        return 2;
    }
}
```

上面的代码， 将会输出下面的结果：  
```
Try programiz.pro  
1  
1  
```

说明在静态函数里面 不存在覆盖一说，只是单纯的隐藏而已。 如果是实例方法的话， 父类的方法将会被覆盖， 但是静态方法不会被覆盖。  



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/java%E4%B8%AD%E5%AD%90%E7%B1%BB%E9%9A%90%E8%97%8F%E7%88%B6%E7%B1%BB%E7%9A%84%E5%87%BD%E6%95%B0/  

