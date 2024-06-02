# 作为术士，我要应付月中测验 - 技术细节


游戏类型：  单人， VR， 战斗。


## 主要内容

游戏引擎： Unity 2018,  Unity 2019

美术部分：  Unity asset store
- 模型
- 动画
- 粒子特效

Unity 插件使用以及目的
- MapMagic
  - 地形生成工具， 用于生成地形
  - 还包括树木， 石头， 花草的自动放置。
- UMotion
  - 动画创造，编辑工具。 
  - 用于修改已有动画， 主要是调节 Z轴和Y轴。
- Dotween
- xNode
  - 主要是用于制作AI 行为树
- Newtonsoft
  - Json 序列化， 反序列化
- Pdollarplus
  - 图案识别
- UniRx
  - 事件
- VRTK
  - vr 操作的基础工具集
  - 传送，抓取物品
  - 摄像头， 手部控制器的位置同步相关
- steamVR
- Facepunch.Steamworks
  - steam API 
- TinyScreenCapture
  - 运行时截屏工具
- TextMesh Pro
  - 高质量文本显示

编辑器相关的工具 
- NaughtyAttributes
  - Inspector 工具
- HierarchyPro
  - Hierarchy 美化工具
  - 给工具栏添加自定义按钮
- UnityFileDebug
  - 日志查看工具， 方便分类，筛选等


### 技能和实体和AI
此处的实体并非 ECS的实体， 而是表示一个可受伤的单位。

#### 实体类结构
- Entity
  - LivingEntity
    - NPCEntity
      - MyNormalMonster
        - MyBossMonster
          - BOSSA
          - BOSSB
        - MonsterA
        - MonsterB
      - Servant
      - Monster  *已经弃用*
        - NormalMonster   *已经弃用*
    - Player

Entity 是所有实体的基类，是一个抽象类。
- 给当前单位应用一个持续力。     *包含击退*
- 攻击的伤害产生部分
- 。。。

LivingEntity 是关键类.  表示一个活着的实体。 
- 具有血量， 蓝量（蓝量并没有使用）
- 能受伤， 死亡， 眩晕 。   
- 具有移动，跳跃能力，飞行能力
- 伤害抗性
- 。。。

NPCEntity  表示一个 Npc单位
- 主要是动画部分的应用
- AI 行为树的更新。
- 。。。

#### 技能部分
- SpellManager
- SpellMetaInfo
  - 技能元信息  大多数的技能配置都在这里
  - 名称，描述，伤害，对伤害过的单位产生一个新的力，  prefab 路径
  - 冷却时间， 蓝量    *都未使用*
  - 技能类型， 技能移动类型， 元素类型， 能否被反击
  - 挂载点， 瞄准点
  - 坚定优先级  *即是否使目标进入受伤动画的优先级*
  - 技能读条信息
    - 读条时间
    - 能否被打断， 打断条件是什么
    - 。。。
  - 技能额外信息
    - 一些其他参数
- SpellForm
  - Prefab 挂载的脚本
  - 主要目的是组合各个模块， 产生一个合适的效果
- SpellAnimation
  - 让施法者播放一个动画
- SpellDamage
  - 技能伤害 配置
    - 持续时间， 伤害间隔等
  - 伤害目标判定
    - 主要是碰撞体的方式
    - 还有 Physics.XXX() 的形式    *这个用的比较少*
- SpellCustom
  - 施放一个另外的技能
  - 给施法者添加一个新的力
  - 添加buff
  - 。。。
- ParticleHelper
  - 技能特效


### AI 部分

- KXAIGraph
  - 表示一个AI图
  - 所有节点的数据
  - 运行中的节点数据
  - trigger list, property list, expire list
- AINodeEnter
- AINodeSequence
  - Selector    
    - 遇到第一个 return true的节点就结束序列， 并return true
  - Sequence   
    - 从头往下执行，每次Update 最多执行一个节点。
    - 遇到 return false的节点就中断整个序列 [可选项]
  - Parallel
    - 并行，  一次Update 全部执行完毕， 不管子节点的返回值
- AICheckNode
- AIWorkNode
  - 具体的实体行为
  - 移动到某个单位附近， 移动到某个点
  - 使用技能
  - 给自己添加BUFF
  - 等等
- AIEventNode
  - 事件节点， 优先级高于其他类型的Node， 但是不参与常规Update
  - KXAIGraph 初始化事件
  - 技能在读条的时候被打断的事件
  - 玩家使用了技能的事件
  - 等等
- AINodeChanceWork
  - 权重随机


### 编辑器工具部分

- SpellCreator
- SpellTest
- BuffCreator
- EntityEditorHelper
- 等等


### 反思

技能部分 应该是可以有一个更好的组成方式。 在构想中， 使用xNode的类似插件， 应该可以更好的设计和配置技能。

技能特效与技能效果与技能的生命周期 直接的协调， 还需要再考虑一下。

随机数的生成， 还需要考虑一个新的方案。 这个对技能和AI的影响比较大。 


## 其他工具

- GoldWave
  - 音频编辑工具
  - 免费版有导出文件次数限制， 感觉还不如使用 adobe家族的工具
- Shotcut
  - 视频编辑工具
  - 免费， 不要钱
  - 有点难用， 但是也还能用
  - 可以用于steam宣传片， 或者游戏内的视频
- GIMP
  - 图片编辑工具
  - 免费，不要钱
  - 做宣传图的时候可以考虑使用
  - 游戏内的美术资源修改  也可以考虑使用
- VS Code
  - 文本编辑工具
  - 代码编写工具
  - 断点调试工具
  - 免费， 不要钱







---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E4%BD%9C%E4%B8%BA%E6%9C%AF%E5%A3%AB%E6%88%91%E8%A6%81%E5%BA%94%E4%BB%98%E6%9C%88%E4%B8%AD%E6%B5%8B%E9%AA%8C-%E6%8A%80%E6%9C%AF%E7%BB%86%E8%8A%82/  

