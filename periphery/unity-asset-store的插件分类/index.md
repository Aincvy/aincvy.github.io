# Unity Asset Store 一些插件的介绍


*插件这东西并不是越多越好， 还是符合自己的需求最好。*

- 🈚  免费
- 🈶  要钱
- 📖  开源

## 主要内容


### 代码工具类

#### 日志插件： 🈚📖 [Unity File Debug](https://assetstore.unity.com/packages/tools/utilities/unity-file-debug-72250)

![log viwer](https://assetstorev1-prd-cdn.unity3d.com/package-screenshot/4488db8c-b006-4ea0-b498-7b3177337081.webp)

简介： 

这个插件可以记录 console log 到文件里面， 文件类型可以是 csv, json, txt 。  
插件提供一个 `DebugWrapper`的类，所以使用的时候 不需要修改旧的代码。    
*就是这个类缺了函数`LogWarningFormat` 自己手动添加一下即可。*

此外，插件还提供一个日志查看使用的html文件， 图示就是那个html 文件。    
如果是开发手游的话， 在调试的时候就可以把生成的日志文件复制到电脑里面， 然后用这个html 文件进行查看。   

使用方法：  安装插件之后， 拖拽一个 Prefab 到场景里面， 设置一下输出日志的文件名就可以了。   
日志的默认输出目录是 [Application.persistentDataPath](https://docs.unity3d.com/ScriptReference/Application-persistentDataPath.html)



#### 工具： 🈚📖 [UniRx - Reactive Extensions for Unity](https://assetstore.unity.com/packages/tools/integration/unirx-reactive-extensions-for-unity-17276)

![xx](https://assetstorev1-prd-cdn.unity3d.com/package-screenshot/bc8d37ac-b3f7-49ea-a045-2bfabdbd7227.webp)

简介：

这个应该是一个unity的响应式拓展， 下面放出一些 作者在Github 上的示例 。

```csharp
var clickStream = Observable.EveryUpdate()
    .Where(_ => Input.GetMouseButtonDown(0));

clickStream.Buffer(clickStream.Throttle(TimeSpan.FromMilliseconds(250)))
    .Where(xs => xs.Count >= 2)
    .Subscribe(xs => Debug.Log("DoubleClick Detected! Count:" + xs.Count));

// Another example
public static IObservable<float> ToObservable(this UnityEngine.AsyncOperation asyncOperation)
{
    if (asyncOperation == null) throw new ArgumentNullException("asyncOperation");

    return Observable.FromCoroutine<float>((observer, cancellationToken) => RunAsyncOperation(asyncOperation, observer, cancellationToken));
}

static IEnumerator RunAsyncOperation(UnityEngine.AsyncOperation asyncOperation, IObserver<float> observer, CancellationToken cancellationToken)
{
    while (!asyncOperation.isDone && !cancellationToken.IsCancellationRequested)
    {
        observer.OnNext(asyncOperation.progress);
        yield return null;
    }
    if (!cancellationToken.IsCancellationRequested)
    {
        observer.OnNext(asyncOperation.progress); // push 100%
        observer.OnCompleted();
    }
}

// usecase
Application.LoadLevelAsync("testscene")
    .ToObservable()
    .Do(x => Debug.Log(x)) // output progress
    .Last() // last sequence is load completed
    .Subscribe();


// MessageBroker is Rx based in-memory pubsub system filtered by type.
public class TestArgs
{
    public int Value { get; set; }
}

---

// Subscribe message on global-scope.
MessageBroker.Default.Receive<TestArgs>().Subscribe(x => UnityEngine.Debug.Log(x));

// Publish message
MessageBroker.Default.Publish(new TestArgs { Value = 1000 });
```

笔者用的比较简单， 主要用于同步事件。  


#### 工具： 🈚📖 [Query for Unity](https://assetstore.unity.com/packages/tools/query-for-unity-42015)

![image](https://assetstorev1-prd-cdn.unity3d.com/package-screenshot/5e7c5a67-e512-4b72-a882-75c3414d489f.webp)

简介： 

代码工具。 

- [Github](https://github.com/npruehs/unity-query)
- [Cheat Sheet](https://raw.githubusercontent.com/npruehs/unity-query/master/Source/UnityQuery/Assets/UnityQuery/UnityQuery%20Cheat%20Sheet.pdf)


#### 节点编辑器基础：  🈚📖  [xNode](https://github.com/Siccity/xNode)

![image](https://user-images.githubusercontent.com/6402525/53689100-3821e680-3d4e-11e9-8440-e68bd802bfd9.png)

简介： 

这个也是可视化编辑工具。   
从Github 上获取不要钱， 但是从 asset store 上获取则需要10刀。 

> xNode is a very powerful and intuitive node editor framework ideal for coding your own dialogue systems, state machines, procedural generation, behaviour trees etc. 

笔者使用这个工具来做 行为树。 笔者使用的是老版本， 当节点到达 500+ 的时候， 编辑节点就会感觉到卡顿， 不知道最新版有没有变化。  

代替品考虑：  
- https://github.com/alelievr/NodeGraphProcessor      *笔者暂时没有使用过*

#### Inspector工具： 🈚📖 [NaughtyAttributes](https://assetstore.unity.com/packages/tools/utilities/naughtyattributes-129996)

```csharp
public class NaughtyComponent : MonoBehaviour
{
	[Button]
	private void MethodOne() { }

	[Button("Button Text")]
	private void MethodTwo() { }
}
```

![image](https://raw.githubusercontent.com/dbrizov/NaughtyAttributes/master/Assets/NaughtyAttributes/Documentation%7E/Button_Inspector.png)

简介：

> NaughtyAttributes is an extension for the Unity Inspector.

给组件添加一些注解之后，  Inspector 上的渲染方式会发生改变。  


#### json && bson: 🈚 [JSON .NET For Unity](https://assetstore.unity.com/packages/tools/input-management/json-net-for-unity-11347#description)

简介： 

> JSON .NET brings the power of Json and Bson serialization to Unity with support for 4.7.2 and up and is compatible with both .NET and IL2CPP backends.

这个好像是 Newtonsoft.Json  
没什么好说的， 就是json 和bson 的序列化和反序列化工具。 


### 3D 模型相关

#### 动画编辑工具： 🈶 [UMotion Pro - Animation Editor](https://assetstore.unity.com/packages/tools/animation/umotion-pro-animation-editor-95991)

- 原价：  $77
- 存在一个免费的 社区版本 [UMotion Community - Animation Editor](https://assetstore.unity.com/packages/tools/animation/umotion-community-animation-editor-95986)

![image](https://assetstorev1-prd-cdn.unity3d.com/package-screenshot/8125953d-0770-4caf-8975-7ca40cd32aad.webp)

简介：  

可以在unity 里面查看，编辑， 创建 动画*AnimationClip*的一个工具。    
*也许在 blender里面修改 更简单，而且还不要钱？ 笔者不会blender 所以不太清楚。。*


#### 过渡动画工具： 🈶 [DOTween Pro](https://assetstore.unity.com/packages/tools/visual-scripting/dotween-pro-32416)

![image](https://assetstorev1-prd-cdn.unity3d.com/key-image/d28cf7c5-1e07-4494-81e3-bc3ca7539da6.webp)

- 原价：  $16.50
- 存在一个免费的 社区版本 https://assetstore.unity.com/packages/tools/animation/dotween-hotween-v2-27676

简介：  

动画过渡工具。  在线文档：  http://dotween.demigiant.com/documentation.php

- 位置过渡动画
- 旋转过渡动画
- 值过渡

每个过渡还有一些事件可以使用
- onKill
- onComplete
- ...


## 待验证

- 低模可动城市人口   https://assetstore.unity.com/packages/3d/characters/humanoids/humans/simple-modular-human-100162
- 低模城市  带有夜景   https://assetstore.unity.com/packages/3d/environments/urban/city-package-107224
- 2d ui 图标  https://assetstore.unity.com/packages/2d/gui/icons/2d-casual-ui-hd-82080
- 低模环境包   https://assetstore.unity.com/packages/3d/environments/landscapes/lowpoly-environment-pack-99479
- 3d ui 菜单   https://assetstore.unity.com/packages/tools/gui/3d-modern-menu-ui-116144
- 音效已经bgm管理  https://assetstore.unity.com/packages/tools/audio/fmod-for-unity-161631
- 免费shader  https://assetstore.unity.com/packages/vfx/shaders/ultimate-10-shaders-168611

---

> 作者: hideDragon  
> URL: https://fantasyplayer.link/periphery/unity-asset-store%E7%9A%84%E6%8F%92%E4%BB%B6%E5%88%86%E7%B1%BB/  

