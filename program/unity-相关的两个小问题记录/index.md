# Unity 相关的几个小问题记录


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


### transform.position 获取的位置和看到的位置不一样的问题记录。

主要是 y 轴不一样，  x,z 轴实际和看到的是一样的。

因为 怪物身上同时具有 rigidbody 和 NavMeshAgent 两个组件。

NavMeshAgent 会把怪物强制到地形上面， 而 rigidbody 添加重力的时候会让怪物不断下沉。

将 NavMeshAgent 禁用掉， 就可以看到怪物实际的位置了。  

主要触发原因可能是 怪物的初始 y 轴的值 略微低于地面。 当这种情况下，把怪物的 y轴 调到地形上面就可以了。  
或者更好的处理 两者的关系。   
使用 NavMeshAgent 的时候就设置 Rigidbody.isKinematic = true.  
使用 Rigidbody 的时候就设置，  NavMeshAgent.enable = false.  
 
### 更改 Asset store 的下载目录

*windows 操作系统下的方法， 其实各个系统都可以使用软链接来做。*

windows 下的默认目录是： `c:/Users/[用户名]/AppData/Roaming/Unity`  
先将这个目录移动到别的位置， 比如：  `D:\RoamingUnity`  
随后执行下面的命令 建立一个软链接就可以了。   
```shell
cd c:/Users/[用户名]/AppData/Roaming
mklink /d Unity D:\RoamingUnity
```

### layer 和 layer mask 的一点内容
layer 和 layer mask 是两个东西， gameobject.layer  使用的是layer， 发射射线使用的是 layer mask。

基本上来说， layer 是一个整形数字 具体值好像是 0 ～ 31 。   
而 layer mask 是移位 移出来的。 `layer mask = 1 << layer`   
layer 是唯一的， 只能表示一个，  但是 layer mask 可以同时表示多个layer *该位为1,即表示包含。*  

可以使用的 api 有： 
```csharp
var layer = LayerMask.NameToLayer("Entity");
var layerMask = LayerMask.GetMask("Entity"); 

Debug.Log(layer);
Debug.Log(layerMask);
```


参考阅读：  https://docs.unity3d.com/Manual/layers-and-layermasks.html


### 动画过渡相关的一点内容

动画过渡的 settings 里面的 exit time 是比例(normalized time)， 不是秒

duration 可以设置是 秒还是 比例。

### NavMesh和 NavMeshAgent 相关的一点内容
`NavMesh.GetAreaFromName` 返回的结果是 0 ~ 31

要设置给agent的时候使用 `navMeshAgent.areaMask = 1 << NavMesh.GetAreaFromName("Walkable");`    

默认情况下，agent type 只有 Humanoid 可用。 如果设置了其他类型 就会无法寻路。 
想要使用其他的agent type可以参考下面的链接 
- https://answers.unity.com/questions/1358023/nav-mesh-agent-type.html
- https://github.com/Unity-Technologies/NavMeshComponents

似乎以`Mask`结尾的东西都是移位之后的数值。 

*笔者在自己测试的时候发现，修改NavMeshAgent的 radius,height 属性之后 似乎没有什么作用。具体原因笔者没有深究。*

### Unity Message 
unity 的消息函数， 如： 
- OnCollisionEnter
- Awake
- Start
- Update

都是使用反射的方式调用的。  如果在设计类结构的时候使用了继承机制， 那么就要小心这些消息函数的访问修饰符。

如果在代码里都是使用的`private` 修饰符的话， 比如像下面的代码。
```csharp
private void OnCollisionEnter(Collision other) {
        // 检测是否是碰到了玩家
        Log("OnCollisionEnter");
}
```
如果子类也实现了这个函数的话， 那么基类的函数就不会得到调用。  可能会产生一些不太好寻找的BUG。 
并且， 在VS Code 里面的话， 也不会得到一丝提示。 

~~如果去掉访问修饰符， 或者使用 `protected` `public` 的话， 在 VS Code 里面就可以看到一个警告。~~  
如果使用修饰符 `protected` 或者 `public` 的话， 在 VS Code 里面就可以看到一个警告。

拓展阅读：  https://forum.unity.com/threads/monobehaviour-inheritance-and-awake-start-onenable-etc.303834/#post-1985265

### 动画预览

打开Animator 窗口之后， 点击一个Transition 之后， 可以在右下角看到一个preview 界面， 这个是预览动画。

预览界面的底部中间有一行字，这行字的前半部分是 0:00  ， 这个是表示的是帧数还是时间， 笔者还没有搞清楚。  
但是笔者知道， 前面一个0 是秒数， 后面一个则是 0 ~ 59 之间的数值。 即 0:58, 0:59, 1:00   这样的。

如果需要手动填写动画时间的时候， 需要注意一个差别， 1秒 等于 1000 毫秒，  即 预览里看到的是 0:30 的时候， 时间上应该填写 0.5s  
不过， 笔者发现 手动填写动画时间 并不靠谱，更应该考虑动画事件， 这样在时间上更精确一些。   
只是动画事件的回调，在 transition的过程中也会调用， 这是一个问题。  



---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/program/unity-%E7%9B%B8%E5%85%B3%E7%9A%84%E4%B8%A4%E4%B8%AA%E5%B0%8F%E9%97%AE%E9%A2%98%E8%AE%B0%E5%BD%95/  

