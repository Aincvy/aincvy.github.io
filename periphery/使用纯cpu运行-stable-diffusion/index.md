# 使用纯cpu运行 Stable Diffusion


## 主要内容

直接安装 webui： https://github.com/AUTOMATIC1111/stable-diffusion-webui

修改文件 `webui-user.bat/webui-user.sh` 找到 `COMMANDLINE_ARGS=`   在后面追加参数`--use-cpu all`即可。 

参考配置：  

```shell
# Linux版
export CUDA_VISIBLE_DEVICES=-1
export COMMANDLINE_ARGS="--use-cpu all --no-half --precision full --skip-torch-cuda-test"
```

下面是一些额外的信息
- 添加 `--listen` 参数开启局域网访问
- 添加 `--enable-insecure-extension-access`  开启局域网安装拓展
- 使用脚本尝试一键安装  （linux）
  - 因为网络问题， 一键安装可能会失败
  - 先使用conda 创建一个环境  `conda create -n webui python=3.10.6 anaconda`
  - 下载并执行脚本： https://raw.githubusercontent.com/AUTOMATIC1111/stable-diffusion-webui/master/webui.sh
    - ```shell
        wget -q https://raw.githubusercontent.com/AUTOMATIC1111/stable-diffusion-webui/master/webui.sh
        chmod +x webui.sh
        ./webui.sh
        # Check `webui-user.sh` for options.
  - `files.pythonhosted.org`  的连接速度可能会非常慢，需要加速一下。 

------

注意：这个只能玩一玩， 体验一下， 因为出图非常之慢。  
- 768*768， 迭代步数30， 采样方法 euler a
- batch count 3 需要1个多小时
- 正面提示词： big sun, blue sky, clouds, beautiful elf girl, master piece
- 负面提示词： worst quality
- CPU E5-2650 v2 @ 2.60GHz * 2， 虚拟机CPU核心数： 28， 内存： 24G

笔者依稀记得， 一张512*512的图需要15分钟， 而3060 12G 只需要15秒 ， 所以这个真的只能体验一下。 

‌
## 参考阅读

- [https://ivonblog.com/posts/stable-diffusion-running-on-cpu/](https://ivonblog.com/posts/stable-diffusion-running-on-cpu/ "smartCard-inline")

- [https://github.com/microsoft/TaskMatrix/issues/250#issuecomment-1477468422](https://github.com/microsoft/TaskMatrix/issues/250#issuecomment-1477468422 "smartCard-inline")

- [https://blog.csdn.net/qq_27825451/article/details/89237574](https://blog.csdn.net/qq_27825451/article/details/89237574 "‌")

- [https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/10117#issuecomment-1537294185](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/10117#issuecomment-1537294185 "smartCard-inline")



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/%E4%BD%BF%E7%94%A8%E7%BA%AFcpu%E8%BF%90%E8%A1%8C-stable-diffusion/  

