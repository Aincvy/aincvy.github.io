# Void - 一个Cursor 的开源代替产品 的体验记录


项目地址是：  https://github.com/voideditor/void

直接说结果把， 结果就是使用Ollama 搭建的各种小模型表现都不好。 但是 Deepseek 官方的 deepseek-reasoner 模型是可以使用的，表现也不错。 
笔者尝试的小模型有：
- qwen3:14b
- deepseek-r1:14b  

笔者尝试使用这个从0开始创建了一个简单的控制台程序，是可以正常创建的。 

如下操作花费了我2.5元的 deepseek：
- 创建一个学生管理系统， 使用控制台UI， 数据保存到csv文件里面。 
- 创建一个考试管理系统， 和学生管理系统类似的结构。
- 调整考试类型成使用lombok
- 添加一个ServiceManager类型， 并且将UI 拆分，先选择是考试还是学生管理， 再进入新的子项。

这些操作花费的时间并不是特别长，做出来的东西也能用。 如果我让它使用jpa， 以及提供spring boot 的restful 接口，花费和时间和现在这个应该也不会相差太多。 

合起来使用了55.1万token（包括命中缓存的输入，未命中缓存的输入，以及输出的总的）

*我感觉花费很高， 本地模型用不了真的太伤了。。 为什么我说花费很高呢，因为项目代码多了之后，每次修改和新增功能的时候，token使用量应该都暴增很多。。*    


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/void---cursor-%E7%9A%84%E5%BC%80%E6%BA%90%E4%BB%A3%E6%9B%BF%E4%BA%A7%E5%93%81-%E7%9A%84%E4%BD%93%E9%AA%8C%E8%AE%B0%E5%BD%95/  

