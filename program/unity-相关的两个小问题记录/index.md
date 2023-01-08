# Unity 相关的两个小问题记录




## 主要内容



### 动画播放结束会模型会卡住一段时间的问题

动画播放是使用 Animator Controller 实现的， 笔者碰到的问题是在一个受击动画播放结束之后，没有马上切换到 Idle 状态， 而是会在受击状态结束后卡住一会才会切换。 

因为是受击动画，所以笔者没有使用条件，而是使用了 `Has Exit Time`

经过一段时间的查找， 笔者发现原因是： 当动画的 `Exit Time` 比实际的动画要更长的时候， 就会出现这个问题。 

解决方法是： 找到受击状态与 Idle 状态的过渡线， 点开`Settings` ， 里面有个 `Exit Time` ,改小一些就可以了。 



### 使用代码修改粒子特效的属性

```c#
// item.emission.enabled = false;
// Cannot modify the return value of 'ParticleSystem.emission' because it is not a variable [Assembly-CSharp]
// item.enableEmission = false;
// 'ParticleSystem.enableEmission' is obsolete: 'enableEmission property is deprecated. Use emission.enabled instead.' [Assembly-CSharp]
// item:  ParticleSystem
var e = item.emission;
e.enabled = false;
```

上面两种方式一种会产生报错， 一种会产生警告， 使用最后那种方式修改就好了。 

修改其他属性的时候， 也可以使用类似的方式， 比如 `item.main` 等。

---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/program/unity-%E7%9B%B8%E5%85%B3%E7%9A%84%E4%B8%A4%E4%B8%AA%E5%B0%8F%E9%97%AE%E9%A2%98%E8%AE%B0%E5%BD%95/  

