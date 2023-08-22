# Llama Gpt 项目的简单介绍以及切换CUDA的方法

llama-gpt 是一个 需要自己架设，离线，类似ChatGPT的聊天机器人，由Llama 2 提供支持。

官方 github 地址： https://github.com/getumbrel/llama-gpt.git

官方简介：  
> A self-hosted, offline, ChatGPT-like chatbot. Powered by Llama 2. 100% private, with no data leaving your device.

## 主要内容

### 体验 
这是笔者第一次体验gpt类的工具， 之前并没有尝试过chatGPT， 感觉这个llama-gpt 勉强可用， 不过更多的还是期待它未来的表现。

- 仓库的版本是 纯CPU 运算的， 所以速度很慢。 笔者的U 是amd 5600, 一个问题需要几分钟才能开始响应， 到响应结束还需要一段时间。 *2023年8月21日*
- 不过可以自己切换到N卡的 CUDA 上进行运算， 有点麻烦， 但是可以切换。
- 响应的内容 没有排版，这个很难受，并且笔者目前没有找到办法去解决。   *后续更新可见： https://github.com/getumbrel/llama-gpt/issues/28*
- 可以用中文提问， 但是默认情况下会回复英文
- 会无视一些输入提示词， 比如 想让它省略解释说明部分， 但是没用，它一定会给你解释一下。

下面看几张图。 

![llama-gpt1](/img/periphery/llama-gpt1.png)

虽然排版很难受， 但是生成出现的东西， 还是可以考虑的。 

```cpp 
#include <iostream>

#include "imgui.h"

int main() {
  ImGui::CreateContext();
  ImGui::StyleColor( & ImGuiCol_Text, ImVec4(1.0 f, 1.0 f, 1.0 f, 1.0 f));
  ImGui::Begin("Form", nullptr);
  ImGui::Text("Name:");
  ImGui::SameLine();
  ImGui::InputText("name", name, IM_ARRAYSIZE(name));
  ImGui::Text("Age:");
  ImGui::SameLine();
  ImGui::InputText("age", age, IM_ARRAYSIZE(age));
  ImGui::Button("Submit");
  ImGui::Button("Reset");
  ImGui::End();
  ImGui::DestroyContext();
  return 0;
}
```
*使用 https://codebeautify.org/cpp-formatter-beautifier 自动格式化*

```cpp
#include <iostream>

#include "imgui.h"

int main() {
  ImGui::CreateContext();
  ImGui::StyleColor( & ImGuiCol_Text, ImVec4(1.0 f, 1.0 f, 1.0 f, 1.0 f));
  ImGui::Begin("Form", nullptr);
  ImGui::Text("Name:");
  ImGui::SameLine();
  ImGui::InputText("name", name, IM_ARRAYSIZE(name));
  ImGui::Text("Age:");
  ImGui::SameLine();
  ImGui::InputInt("age", age);
  if (ImGui::Button("Submit")) {
    std::cout << "Submit button pressed!" << std::endl;
  }
  if (ImGui::Button("Reset")) {
    name[0] = '\0';
    age = 0;
  }
  ImGui::End();
  ImGui::DestroyContext();
  return 0;
}
```

第二版代码比第一版本多了一些内容， 但是仍然没帮我声明 name, age 两个变量 😆


偶尔的情况下， 会给一个很好的排版。 

![llama-gpt2](/img/periphery/llama-gpt2.png)


可以生成 unity的脚本, 这里还可以看到是存在上下文语境的。  ~~但是我看了一下日志， 感觉就是把聊天记录都发了一遍过去。。~~

![llama-gpt3](/img/periphery/llama-gpt3.png)

### 应用

- 小段代码生成
- 一些常见的算法代码生成
  - 生成的冒泡排序可以使用， 但是 快速排序却有语法问题， 且没给我解决。。
- 中文翻译成英文
  - 但是无法从英文翻译中文
- 问一些乱七八糟的
- 生成  CURD 相关的代码。 


### 切换部分内容到GPU上运行

本文描述的方法是使用 N卡的CUDA 进行运算。  更换之后， 大多数问题在几秒内就可以开始回答了。

*这只是一个临时的办法， 官方可能会在不久的将来支持 CUDA。 可见 https://github.com/getumbrel/llama-gpt/issues/6*


*笔者在 Debian 11 上进行了测试， 成功运行了 13B模型。*

*更多的说明在最后部分*

**简单版本**
- 编译 llama-cpp-python 的 cuda 版本 镜像
- 手动把模型映射到容器中
- 调整并使用 `llama-gpt/api/run.sh` 文件来启动

**长版本**

1. 安装 并且设置好 docker， 包括 [nvidia container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
2. git clone https://github.com/abetlen/llama-cpp-python.git
3. 准备好 模型
   1. llama-2-7b-chat.bin - https://huggingface.co/TheBloke/Nous-Hermes-Llama-2-7B-GGML/resolve/main/nous-hermes-llama-2-7b.ggmlv3.q4_0.bin
   2. llama-2-13b-chat.bin - https://huggingface.co/TheBloke/Nous-Hermes-Llama2-GGML/resolve/main/nous-hermes-llama2-13b.ggmlv3.q4_0.bin
   3. 我觉得 7b 或者 13b 的就足够了
   4. 下载好的模型放到宿主机的某个文件夹， 比如： `/home/debian/docker/llama-models`
   5. 我使用的是 `llama-2-13b-chat.bin` 这个模型
4. 查阅文档：  https://github.com/abetlen/llama-cpp-python/tree/main/docker#cuda_simple
   1. 不需要执行最后的 docker run 命令， 使用 `docker build -t cuda_simple .` 构建好镜像即可
   2. 修改版本的 Dockerfile: https://github.com/Aincvy/llama-cpp-python/blob/dockerfile-cn-mirrors/docker/cuda_simple/Dockerfile-CN
      1. 使用了 阿里的 apt 源
      2. 使用了 清华大学的 python 源
5. 执行命令 `sudo docker tag cuda_simple llama-cpp-python-cuda:latest`  
6. 寻找一个目录作为 docker compose 的目录， 比如： `/home/debian/docker-compose/llama-gpt-cuda`
7. 新建文件 `docker-compose.yaml` ,并填充下面的内容
```yaml
version: '3.6'

services:
  llama-gpt-api-13b:
    # image: 'ghcr.io/getumbrel/llama-gpt-api-llama-2-13b-chat:latest'
    image: 'llama-cpp-python-cuda:latest'
    # build:
    #   context: ./api
    #   dockerfile: 13B.Dockerfile
    restart: on-failure
    environment:
      MODEL: '/models/llama-2-13b-chat.bin'
      USE_MLOCK: 1
    volumes:
      - '/home/debian/docker/llama-models:/models/'
      - './data:/home'
    cap_add:
      - IPC_LOCK
      - SYS_RESOURCE
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
    command: /bin/sh /home/run.sh

  llama-gpt-ui:
    image: 'getumbrel/llama-gpt-ui:latest'
    container_name: 'llama-gpt-ui'
    ports: 
      - 3000:3000
    restart: on-failure
    environment:
      - 'OPENAI_API_KEY=sk-XXXXXXXXXXXXXXXXXXXX'
      - 'OPENAI_API_HOST=http://llama-gpt-api-13b:8000'
      - 'DEFAULT_MODEL=/models/llama-2-13b-chat.bin'
      - 'WAIT_HOSTS=llama-gpt-api-13b:8000'
      - 'WAIT_TIMEOUT=600'

```
8. 创建一个目录： `data` 
9. 创建一个文件： `data/run.sh` 并填充下面的内容
*从 llama-gpt/api/run.sh 复制，然后调整的*
```shell
#!/bin/bash

# make build

# Get the number of available threads on the system
n_threads=$(grep -c ^processor /proc/cpuinfo)

# Define context window
n_ctx=4096

# Offload everything to CPU
n_gpu_layers=37      

# Define batch size
n_batch=2096
# If total RAM is less than 8GB, set batch size to 1024
total_ram=$(cat /proc/meminfo | grep MemTotal | awk '{print $2}')
if [ $total_ram -lt 8000000 ]; then
    n_batch=1024
fi

echo "Initializing server with:"
echo "Batch size: $n_batch"
echo "Number of CPU threads: $n_threads"
echo "Number of GPU layers: $n_gpu_layers"
echo "Context window: $n_ctx"

exec python3 -m llama_cpp.server --n_ctx $n_ctx --n_threads $n_threads --n_gpu_layers $n_gpu_layers --n_batch $n_batch
```  
   
10. 调整 `n_gpu_layers`参数， 过大可能会导致 OOM， 我是3060 12G， 37已经非常极限了。
11. `sudo docker compose up`  查看能否正常运行， 如果没有什么问题的话， 则使用 `sudo docker compose up -d ` 进入后台运行。


其他说明
- GPU 的占用好像没有跑满， 大约在 30% 附近。  CPU还是100% 在运行， 
- 速度确实提升了不少， ~~但是没有到达立即响应的地步。~~  很多问题可以在几秒内开始回答。
- 聊天记录越多， 越容易 崩溃， 暂时没看到崩溃原因，只看到了 exit code 0
  - 可能和docker 容器关闭再启动也有点关系把。
- 模型的路径好像只能用 `/models/llama-2-13b-chat.bin` ，或者只能是`/models/` 目录， 否则 UI 程序会出现问题。



## 其他

尝试了一下  https://github.com/oobabooga/text-generation-webui ， 同样安装有点麻烦， 但是这个排版很好。 *使用的同一个模型*

---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/llama-gpt-%E9%A1%B9%E7%9B%AE%E7%9A%84%E7%AE%80%E5%8D%95%E4%BB%8B%E7%BB%8D%E4%BB%A5%E5%8F%8A%E5%88%87%E6%8D%A2cuda%E7%9A%84%E6%96%B9%E6%B3%95/  

