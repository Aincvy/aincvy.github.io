# 如何在steam上发行自己的游戏

如果你没有注册过 steam works， 那么在开始注册到可以售卖之间可能需要2个月的时间。 

## 主要内容

### 注册账号

注册账号：  https://partner.steamgames.com/   
如果你之前有 steam 账号， 那么可以直接登录， 只不过还是要填写一些信息。   
根据提示填写就可以了，  不是很难。   

个人游戏，公司名要填写自己本人的名字。因为之后银行卡的名字要和这个名字一模一样，所以不要乱写。必须是你的银行卡汇款的账号名。

### 验证税务信息

点击  用户与权限 / 公司详细资料

可能需要补充一下 银行详情信息。 

税务信息在最下面， 点击更改税务信息。   这时会跳到一个新的网站， 让你填写税务信息。 
永久居住地址的地方 一定要填写身份证上的地址， 最好采用机翻的形式。  
邮政编码也要正确填写，   联系地址直接保持一致就可以了 。 

TIN 的地方 选择我有， 然后填写外国的 TIN， 具体数值就是身份证号码。 

填写完成之后， 在提交的时候注意下，需要提供您在加入 Steamworks 流程中使用的帐户所链接的电子邮件地址。

提交完成之后， 税务验证信息 可能会变成验证通过， 但是这个是假的。 只是临时的验证通过， 一定要多注意自己的邮箱。

基本上在1天之内， 就会收到邮件结果， 可能会变成真正的验证通过， 也可能会要求补充信息。  
下面是笔者收到的邮件。 
> Hello,
> 
> At this time, we are unable to validate your Steam account. This is because the permanent and or mailing address you entered in the tax interview cannot be verified
> 
> You must now provide us with proof of your address.
> 
> Please provide one of the following documents, showing the same address as you have entered it in the tax interview:
> 
> ·        Recent utility bill (if it shows your address)
> 
> ·        Recent bank statement (if it shows your address)
> 
> ·        Government ID Card (if it shows your address)
> 
> ·        Tenant Lease (if it shows your address)
> 
> ·        OR for a company / business you can provide a company registration certificate (if it shows your address)
> 
> Please do not give us the address of your bank or utility company.
> 
> Forward your documents as an attachment to this email.
> 
> Your Steam account will not be validated until we can verify the address.
> 
> Please note that Valve uses Lilaham, a third-party tax provider, to gather tax information.
> 
> Important Notice: Information in this document does not constitute tax, legal, or other professional advice. If you have other questions, please contact your tax, legal, or other professional advisor.
> 
>  
> Best regards,   
> XXXX | Valve

这里可以看到， 允许提供 `Government ID Card` 就是身份证照片，  笔者把身份证正反面拍照了之后提交上去了。 
文字部分笔者是这么写 
> This is my China  Government ID Card.
> 
> Address part is [英文机翻地址，和提交的地址保持一致 （  中文地址部分 ）。]   
> 
> Best wishes,   
> [自己的名字]

笔者是英渣， 看着写就可以了。 


在几个工作日之后， 笔者收到了 税务信息验证通过的邮件。 
语气更好一些， 可能用的时间会短一些。 

### 支付费用 
在 steam works 的主面板里， 有一栏是 `支付Steam Direct 费` ， 没什么好说的， 点开之后付钱， 支持支付宝。   *2023年4月26日看了一下价格是 638 。 受到汇率影响。*

这个费用不能用 steam 钱包支付。   
这个费用在销售金额到达1000刀的时候 可以退还好像。 

### 准备商店页面
付钱之后 就可以创建应用了。 

创建应用之后， 就可以准备商店页面了。 
在商店编辑页面底部有一段文字 
> Valve 保密
> 本页含有 Valve 的机密信息，设有访问限制。您必须与 Valve 签订涵盖机密信息条款的保密协议和/或许可协议，方能使用和访问本页面。

所以详情内容， 笔者就不透露了。 

下面说一些注意事项
- 准备好之后就可以提交审核了， 商店页面和程序包是分开审核的
- 宣传片和宣传图相关
  - 注意， 图片中不要包含 评分， 折扣等信息， 具体可以查看： https://partner.steamgames.com/doc/store/assets/rules
  - https://partner.steamgames.com/doc/store/assets/standard
  - https://partner.steamgames.com/doc/store/assets/libraryassets
- 关于此游戏的地方 可以多写写玩家可以做什么，有什么特性。

审核成功之后， 会有邮件通知。 

### 准备程序包

steam work 的设置页面 每个标签看过去， 带*的都写上就差不多了。 

对了， 小于2GB的 压缩包可以通过 web界面上传， 但是注意， 压缩包要直接包含 可启动的exe程序。    
比如， 设置的启动程序是： Sorcerer's mid-month exam.exe    
那么压缩包的内容可以是
- MonoBleedingEdge
- Sorcerer's mid-month exam_Data
- Sorcerer's mid-month exam.exe
- UnityCrashHandler64.exe
- UnityPlayer.dll

但是不能是
- Sorcerer's mid-month exam\MonoBleedingEdge
- Sorcerer's mid-month exam\Sorcerer's mid-month exam_Data
- Sorcerer's mid-month exam\Sorcerer's mid-month exam.exe

大于 2GB的程序 使用 sdk上传。 

设置好， 上传完， 设置分支之后就可以提交审核了。    
对了， 成人内容审查记得填写一下。
还有就是， 游戏内容要包含商店中 关于此游戏 列出的特性。  

审核成功之后， 会有邮件通知。 


## 其他

只要交了钱， 在 steam 上发行游戏很简单， 但是困难的是 怎么卖出更多的游戏。 


### steam 的unity 库
- https://github.com/Facepunch/Facepunch.Steamworks
- https://steamworks.github.io/

看情况使用一个就可以了。 

### 拓展阅读
- https://indienova.com/u/%25E7%2596%25AF%25E7%258E%258B%25E5%25AD%2590/blogread/1388
- https://zhuanlan.zhihu.com/p/482830658


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E5%A6%82%E4%BD%95%E5%9C%A8steam%E4%B8%8A%E5%8F%91%E8%A1%8C%E8%87%AA%E5%B7%B1%E7%9A%84%E6%B8%B8%E6%88%8F/  

