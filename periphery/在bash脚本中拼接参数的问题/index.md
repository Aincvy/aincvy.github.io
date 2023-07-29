# 在bash脚本中拼接参数的问题


在脚本里面使用`rsync` 这个命令的时候， 需要根据文件的情况动态的给予参数，所以考虑使用变量来实现这个需求。  

笔者刚开始在做这件事的时候碰到了一个比较蛋疼的事情。

拼接的参数会自动使用一个引号括起来， `rsync` 识别不到那么多参数， 好像只是识别到了第一个，后面的都被忽略掉了。

我刚开始好像是使用一个单一的变量来尝试控制参数。（*大约1，2个月以前的东西​，记得不是很清楚:joy:*)

```bash
# 本部分代码没有经过测试， 只是给个示例来说明一下。
include_arg='--include="*.jar"'

# 如果本地存在 config.json 文件， 就发送过去
if [[ test -e "$folder/config.json" ]] 
then
	include_arg='$include_arg --include="*config.json"'
fi
sshpass -e rsync -avz "$include_arg" --exclude="*" ./ root@xx:/root
```

上述代码中， 最后一段里面的 `"$include_arg"` 变量，经过我的测试，基本都会被解释成`"--include=*.jar --include=*config.json"` 。

这个参数肯定是有问题的，`rsync`识别不出来。

经过在`stackOverflow` 上的搜索，发现使用数组可以处理掉这个问题。

代码修改成如下所示的情况， 就可以正常使用了。

```bash
# 本部分代码没有经过测试， 只是给个示例来说明一下。
include_arg=(--include="*.jar" --include="lib")

# 如果本地存在 config.json 文件， 就发送过去
if [[ test -e "$folder/config.json" ]] 
then
	include_arg+=(--include="*config.json")
fi
sshpass -e rsync -avz "${include_arg[@]}" --exclude="*" ./ root@xx:/root
```

修改成这样之后， `${include_arg[@]}` 会展开成3个独立的参数传给`rsync`。 

---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E5%9C%A8bash%E8%84%9A%E6%9C%AC%E4%B8%AD%E6%8B%BC%E6%8E%A5%E5%8F%82%E6%95%B0%E7%9A%84%E9%97%AE%E9%A2%98/  

