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

### 地形与树木

首先说说树木的碰撞体吧。   
给树木的prefab 添加碰撞体之后， 需要在地形的 TerrainCollider 组件上 勾选 EnableTreeColliders 属性， 这样树木的碰撞体就生效了。  
默认情况下， EnableTreeColliders 属性是开启的。  所以只需要给树木的prefab 添加碰撞体就可以了。   
如果把树木当成普通的prefab 使用， 那么就不需要这个。

现在来说一下 树木的 layer。  
使用上面的方式添加碰撞体之后， 树木的碰撞体和layer 将会与地形混合在一起。  即当你与树木的碰撞体碰撞之后， 你获取到的GameObject 是TerrainCollider 所挂载的GameObject。  
GameObject name, collider, GameObject layer, 都是地形碰撞体的。   
在地形编辑器里面有一个属性叫做 `PreserveTreePrototypeLayers`   ，勾选它， 似乎就可以获取到 树木的layer 。   
但是 笔者在测试后发现， 这个属性没用。  似乎有些版本有效， 有些没用。  笔者目前测试的版本是 Unity 2019.4.21f1   
有一些其他版本的用户也说了这个问题
- https://forum.unity.com/threads/cannot-get-layer-id-of-terrain-tree-prefab-bug.986496/
- https://forum.unity.com/threads/preservetreeprototypelayers-broken.1406575/
- https://forum.unity.com/threads/trigger-sound-on-terrain-tree-collider.1050500/

上面的链接中 有一个人提出了一个曲线救国的方法， 使用碰撞位置和地形高度进行比较， 笔者没有尝试， 感觉使用起来比较麻烦，而且不知道会不会有性能问题。 

此外，树木的碰撞体在场景里是看不到的， 如果树木的碰撞体比网格大的话， 当带有rigidbody的碰撞体靠近树木的时候可能会被弹开。

然后是树木和风  
添加一个 WindZone 可以给树木施加风的效果。   
但是， 并不是所有的树木都是可以的， 需要使用地形工具的Paint Tree 画出来的树木才可以。 并且在Edit Tree 中需要把 Bend Factor 属性的值修改成大于0的值。  
如果Edit Tree 中没有这个属性， 则说明该树木不支持 Wind 的效果。   
如果你是购买了一个树木资源包， 那么， 那个资源包可能会包含一个 Wind_prefab 之类的东西。 具体情况可以去询问资源包的作者。   

吐槽一下
- ~~经过简单的测试， 笔者感觉 wind 的效果并不怎么样- -~~
  - 又试着修改了一下， 效果还行， 也算可以使用吧。
- WindZone 如果设置成 Directional ， 那么将影响整个场景， 想设置一个范围都不行- - 
  
### trigger collider 的回调函数

```csharp
protected virtual void OnTriggerEnter(Collider other) {

}

protected virtual void OnTriggerExit(Collider other) {

}
```

当一个碰撞体 触发了 `OnTriggerEnter` 回调的时候， 你不能预期它的 `OnTriggerExit` 回调一定会执行。  
如果当前的 GameObject 被禁用了的话， OnTriggerExit 就永远不会被调用了。 

这里有一个脚本提供了可靠性的调用保证： https://forum.unity.com/threads/fix-ontriggerexit-will-now-be-called-for-disabled-gameobjects-colliders.657205/
> // OnTriggerExit is not called if the triggering object is destroyed, set inactive, or if the collider is disabled. This script fixes that


但是笔者并没有使用这个脚本， 而是选择了 `OnTriggerStay` 函数。


### UNITY 回调函数的执行顺序 

官方文档地址：  https://docs.unity3d.com/Manual/ExecutionOrder.html
 
![unity callback execute order](https://docs.unity3d.com/uploads/Main/monobehaviour_flowchart.svg)

`Awake` 和 `OnEnable` 最先执行， 之后是 `Reset`  ， 但是`Reset` 只在编辑器里面会调用， 在添加组件的时候。 

然后是 `Start` 函数， 然后是 物理部分。   
物理部分 一帧可能会调用多次， 当 fixed time step 小于帧的 update time  的时候。  
物理部分 先是 `FixedUpdate` ，最后是  `OnTriggerXXX` 和 `OnCollisionXXX`。  
`OnTriggerXXX` 要先于 `OnCollisionXXX` 调用。

`Update` 要在物理和 输入事件部分之后调用, 之后是 Coroutine 相关的部分。   
`LateUpdate` 在 `GameLogic` 的最后一部分执行，但是之后还有渲染和其他一些内容。

`OnDisable` 和 `OnDestroy` 是在最后的最后执行的。



### DOTween  插件

这个插件是闭源的， 所以出现了问题之后 无法调试，是一个黑盒子。 可以在asset store 上寻找一些免费的代替品， 这些免费代替品会给你源代码。
此外，这个插件实现的内容其实并不是特别多，如果需要自己实现的话， Ease 可能麻烦点，其他的应该还好。


如果你一定要用的话， 出了问题可以尝试联系作者， 他可能会回复你。 

*说是这么说， 但是如果你在上班的时候出现了问题，你发了邮件给作者，作者基本上不可能立即回复你的，而且可能还会有时差。而这个时候你也不可能等他把，不然工作进度慢了，还是需要加班。*

### 光照探针组 无法编辑探针位置

尝试重置编辑器布局成默认布局。 

参考阅读： https://forum.unity.com/threads/cant-select-light-probes.716924/#post-5309718

### 网格剔除

笔者下意识的以为摄像机看不到的网格都会自动被剔除， 不会渲染， 也不会接受灯光效果。  

这几天发现这个想法错了。 只要物体在摄像机的角度内， 就会渲染， 也会接受光照效果， 所以如果地图比较大的话， 就需要一个物体管理的脚本 以优化性能。 

比如 A 是一个很大的物体，B是一个很小的物体， A 挡住了B之后， 笔者原本以为B就不会被渲染了。 *渲染指渲染方面的计算， set pass calls, draw calls 等*

但是实际上， 只要B出现在摄像机的角度里面， B 的渲染还是会进行计算， 只是不会显示出来。 


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/unity-%E7%9B%B8%E5%85%B3%E7%9A%84%E4%B8%A4%E4%B8%AA%E5%B0%8F%E9%97%AE%E9%A2%98%E8%AE%B0%E5%BD%95/  

