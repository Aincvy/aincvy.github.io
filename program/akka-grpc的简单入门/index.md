# Akka Grpc的简单入门


## Akka Grpc 的简单入门

### 简介

`Akka Grpc` 是一个 `rpc` 框架。 基于`akka-http`,`google protobuf`  构建而成。

一些快速的链接。

- 官方文档：  https://doc.akka.io/docs/akka-grpc/current/index.html
- Github地址： https://github.com/akka/akka-grpc



### 详细内容

本篇是一个中文的使用流程把。 使用`Java` 语言编写代码。

`scala` 语言和其他类型的编译工具的代码可以在官方文档中找到。 



#### 协议文件

```protobuf
option java_multiple_files = true;
option java_package = &#34;example.myapp.helloworld.grpc&#34;;
option java_outer_classname = &#34;HelloWorldProto&#34;;

package helloworld;

// The request message containing the user&#39;s name.
message HelloRequest {
    string name = 1;
}

// The response message containing the greetings
message HelloReply {
    string message = 1;
}


////////////////////////////////////// The greeting service definition.
service GreeterService {

    //////////////////////
    // Sends a greeting //
    ////////*****/////////
    //      HELLO       //
    ////////*****/////////
    rpc SayHello (HelloRequest) returns (HelloReply) {}

    // Comment spanning
    // on several lines
    rpc ItKeepsTalking (stream HelloRequest) returns (HelloReply) {}

    /*
     * C style comments
     */
    rpc ItKeepsReplying (HelloRequest) returns (stream HelloReply) {}

    /* C style comments
     * on several lines
     * with non-empty heading/trailing line */
    rpc StreamHellos (stream HelloRequest) returns (stream HelloReply) {}
    
}

```



**简单的说明**

`message` 定义数据交互格式 。

`service` 生成rpc 调用 使用的 api

格式为： 

`rpc 方法名 (参数) returns (返回类型) {}`

参数和返回类型可以添加 `stream` 关键字， 表示这是一个数据流 。(持续输入， 或者持续输出)





#### 生成java代码

我使用 `Maven` 工具作为依赖管理工具。 

项目的 `pom.xml` 大致内容如下。 

```xml
&lt;project&gt;
  &lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt;
  &lt;name&gt;Project name&lt;/name&gt;
  &lt;groupId&gt;com.example&lt;/groupId&gt;
  &lt;artifactId&gt;my-grpc-app&lt;/artifactId&gt;
  &lt;version&gt;0.1-SNAPSHOT&lt;/version&gt;
  &lt;properties&gt;
    &lt;maven.compiler.source&gt;1.8&lt;/maven.compiler.source&gt;
    &lt;maven.compiler.target&gt;1.8&lt;/maven.compiler.target&gt;
    &lt;akka.grpc.version&gt;1.0.0-M1&lt;/akka.grpc.version&gt;
    &lt;grpc.version&gt;1.29.0&lt;/grpc.version&gt;
    &lt;project.encoding&gt;UTF-8&lt;/project.encoding&gt;
  &lt;/properties&gt;
  &lt;dependencies&gt;
    &lt;dependency&gt;
      &lt;groupId&gt;com.lightbend.akka.grpc&lt;/groupId&gt;
      &lt;artifactId&gt;akka-grpc-runtime_2.12&lt;/artifactId&gt;
      &lt;version&gt;${akka.grpc.version}&lt;/version&gt;
    &lt;/dependency&gt;
  &lt;/dependencies&gt;
  &lt;build&gt;
    &lt;plugins&gt;
      &lt;plugin&gt;
        &lt;groupId&gt;com.lightbend.akka.grpc&lt;/groupId&gt;
        &lt;artifactId&gt;akka-grpc-maven-plugin&lt;/artifactId&gt;
        &lt;version&gt;${akka.grpc.version}&lt;/version&gt;
        &lt;configuration&gt;
        	&lt;generatorSettings&gt;
          	&lt;serverPowerApis&gt;true&lt;/serverPowerApis&gt;
        	&lt;/generatorSettings&gt;
        &lt;/configuration&gt;
        &lt;executions&gt;
          &lt;execution&gt;
            &lt;goals&gt;
              &lt;goal&gt;generate&lt;/goal&gt;
            &lt;/goals&gt;
          &lt;/execution&gt;
        &lt;/executions&gt;
      &lt;/plugin&gt;
    &lt;/plugins&gt;
  &lt;/build&gt;
&lt;/project&gt;
```

这里添加一个叫`akka-grpc-maven-plugin`  的 Maven插件。 这个插件的`generate` 行为可以生成出java 代码。

单独执行生成的话， 可以试试运行下面的命令生成代码。

`mvn akka-grpc:generate`  

#### 编写 server 端代码

##### 在`server`端 需要实现 `GreeterService ` 接口。

```java
package example.myapp.helloworld;

import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.CompletionStage;
import java.util.stream.Collectors;

import akka.NotUsed;
import akka.stream.Materializer;
import akka.stream.javadsl.Sink;
import akka.stream.javadsl.Source;

import example.myapp.helloworld.grpc.*;

public class GreeterServiceImpl implements GreeterService {
  private final Materializer mat;

  public GreeterServiceImpl(Materializer mat) {
   this.mat = mat;
  }

  @Override
  public CompletionStage&lt;HelloReply&gt; sayHello(HelloRequest in) {
    System.out.println(&#34;sayHello to &#34; &#43; in.getName());
    HelloReply reply = HelloReply.newBuilder().setMessage(&#34;Hello, &#34; &#43; in.getName()).build();
    return CompletableFuture.completedFuture(reply);
  }

  @Override
  public CompletionStage&lt;HelloReply&gt; itKeepsTalking(Source&lt;HelloRequest, NotUsed&gt; in) {
    System.out.println(&#34;sayHello to in stream...&#34;);
    return in.runWith(Sink.seq(), mat)
      .thenApply(elements -&gt; {
        String elementsStr = elements.stream().map(elem -&gt; elem.getName())
            .collect(Collectors.toList()).toString();
        return HelloReply.newBuilder().setMessage(&#34;Hello, &#34; &#43; elementsStr).build();
      });
  }

  @Override
  public Source&lt;HelloReply, NotUsed&gt; itKeepsReplying(HelloRequest in) {
    System.out.println(&#34;sayHello to &#34; &#43; in.getName() &#43; &#34; with stream of chars&#34;);
    List&lt;Character&gt; characters = (&#34;Hello, &#34; &#43; in.getName())
        .chars().mapToObj(c -&gt; (char) c).collect(Collectors.toList());
    return Source.from(characters)
      .map(character -&gt; {
        return HelloReply.newBuilder().setMessage(String.valueOf(character)).build();
      });
  }

  @Override
  public Source&lt;HelloReply, NotUsed&gt; streamHellos(Source&lt;HelloRequest, NotUsed&gt; in) {
    System.out.println(&#34;sayHello to stream...&#34;);
    return in.map(request -&gt; HelloReply.newBuilder().setMessage(&#34;Hello, &#34; &#43; request.getName()).build());
  }
   
}
```

实现`GreeterService ` 接口， 完成自己的需求即可。



##### 启动类

```java
package example.myapp.helloworld;

import akka.actor.ActorSystem;
import akka.http.javadsl.*;
import akka.stream.ActorMaterializer;
import akka.stream.Materializer;
import com.typesafe.config.Config;
import com.typesafe.config.ConfigFactory;

import example.myapp.helloworld.grpc.*;

import java.util.concurrent.CompletionStage;

class GreeterServer {
  public static void main(String[] args) throws Exception {
    // important to enable HTTP/2 in ActorSystem&#39;s config
    Config conf = ConfigFactory.parseString(&#34;akka.http.server.preview.enable-http2 = on&#34;)
            .withFallback(ConfigFactory.defaultApplication());

    // Akka ActorSystem Boot
    ActorSystem sys = ActorSystem.create(&#34;HelloWorld&#34;, conf);

    run(sys).thenAccept(binding -&gt; {
      System.out.println(&#34;gRPC server bound to: &#34; &#43; binding.localAddress());
    });

    // ActorSystem threads will keep the app alive until `system.terminate()` is called
  }

  public static CompletionStage&lt;ServerBinding&gt; run(ActorSystem sys) throws Exception {
    Materializer mat = ActorMaterializer.create(sys);

    // Instantiate implementation
    GreeterService impl = new GreeterServiceImpl(mat);

    return Http.get(sys).bindAndHandleAsync(
      GreeterServiceHandlerFactory.create(impl, sys),
      ConnectHttp.toHost(&#34;127.0.0.1&#34;, 8080),
      mat);
  }
}
```

需要在配置文件里面指定下面的参数

`akka.http.server.preview.enable-http2 = on`



#### 编写 client 代码

```java
package example.myapp.helloworld;

import java.util.Arrays;
import java.util.List;
import java.util.concurrent.CompletionStage;
import java.util.concurrent.TimeUnit;
import java.util.stream.Collectors;
import java.time.Duration;

import io.grpc.StatusRuntimeException;

import akka.Done;
import akka.NotUsed;
import akka.actor.ActorSystem;
import akka.stream.ActorMaterializer;
import akka.stream.Materializer;
import akka.stream.javadsl.Source;
import akka.grpc.GrpcClientSettings;

import example.myapp.helloworld.grpc.*;

class GreeterClient {
  public static void main(String[] args) throws Exception {

    String serverHost = &#34;127.0.0.1&#34;;
    int serverPort = 8080;

    ActorSystem system = ActorSystem.create(&#34;HelloWorldClient&#34;);
    Materializer materializer = ActorMaterializer.create(system);

    // Configure the client by code:
    GrpcClientSettings settings = GrpcClientSettings.connectToServiceAt(&#34;127.0.0.1&#34;, 8080, system);

    // Or via application.conf:
    // GrpcClientSettings settings = GrpcClientSettings.fromConfig(GreeterService.name, system);

    GreeterServiceClient client = null;
    try {
      client = GreeterServiceClient.create(settings, materializer, system.dispatcher());

      singleRequestReply(client);
      streamingRequest(client);
      streamingReply(client, materializer);
      streamingRequestReply(client, materializer);


    } catch (StatusRuntimeException e) {
      System.out.println(&#34;Status: &#34; &#43; e.getStatus());
    } catch (Exception e)  {
      System.out.println(e);
    } finally {
      if (client != null) client.close();
      system.terminate();
    }

  }

  private static void singleRequestReply(GreeterService client) throws Exception {
    HelloRequest request = HelloRequest.newBuilder().setName(&#34;Alice&#34;).build();
    CompletionStage&lt;HelloReply&gt; reply = client.sayHello(request);
    System.out.println(&#34;got single reply: &#34; &#43; reply.toCompletableFuture().get(5, TimeUnit.SECONDS));
  }

  private static void streamingRequest(GreeterService client) throws Exception {
    List&lt;HelloRequest&gt; requests = Arrays.asList(&#34;Alice&#34;, &#34;Bob&#34;, &#34;Peter&#34;)
        .stream().map(name -&gt; HelloRequest.newBuilder().setName(name).build())
        .collect(Collectors.toList());
    CompletionStage&lt;HelloReply&gt; reply = client.itKeepsTalking(Source.from(requests));
    System.out.println(&#34;got single reply for streaming requests: &#34; &#43;
        reply.toCompletableFuture().get(5, TimeUnit.SECONDS));
  }

  private static void streamingReply(GreeterService client, Materializer mat) throws Exception {
    HelloRequest request = HelloRequest.newBuilder().setName(&#34;Alice&#34;).build();
    Source&lt;HelloReply, NotUsed&gt; responseStream = client.itKeepsReplying(request);
    CompletionStage&lt;Done&gt; done =
      responseStream.runForeach(reply -&gt;
        System.out.println(&#34;got streaming reply: &#34; &#43; reply.getMessage()), mat);

    done.toCompletableFuture().get(60, TimeUnit.SECONDS);
  }

  private static void streamingRequestReply(GreeterService client, Materializer mat) throws Exception {
    Duration interval = Duration.ofSeconds(1);
    Source&lt;HelloRequest, NotUsed&gt; requestStream = Source
      .tick(interval, interval, &#34;tick&#34;)
      .zipWithIndex()
      .map(pair -&gt; pair.second())
      .map(i -&gt; HelloRequest.newBuilder().setName(&#34;Alice-&#34; &#43; i).build())
      .take(10)
      .mapMaterializedValue(m -&gt; NotUsed.getInstance());

    Source&lt;HelloReply, NotUsed&gt; responseStream = client.streamHellos(requestStream);
    CompletionStage&lt;Done&gt; done =
      responseStream.runForeach(reply -&gt;
        System.out.println(&#34;got streaming reply: &#34; &#43; reply.getMessage()), mat);

    done.toCompletableFuture().get(60, TimeUnit.SECONDS);
  }

}
```

主要是构造一个`GreeterServiceClient` 对象， 用于进行 `rpc` 调用即可。

基本上使用 就是上面的内容。 官方文档还提供一种添加 `Metadata` 参数的方法。 有兴趣的可以看看官方文档，作者回复`issue` 的频率很高， 有问题可以去问问。 

### 生产者/消费者模式

我的经验告诉我， 这个rpc库不能像常规的socket那样互发消息， 比如服务端要推送消息给客户端的时候就不太好用。

如果服务端不能推送消息给客户端的话， 那么就只能让客户端不停的轮询服务器，这里是比较浪费流量和性能的。 :baby_chick::baby_chick:

我考虑的解决方法是 *生产者/消费者模式 :smiley_cat:*

`Source` 类可以通过一个叫[fromPublisher](https://doc.akka.io/japi/akka/current/akka/stream/scaladsl/Source.html#fromPublisher(org.reactivestreams.Publisher)) 的方法把一个`Publisher` 转成`Source` 。

[Usage](https://doc.akka.io/docs/akka/current/stream/operators/Source/fromPublisher.html) 

给一段代码 应该就可以明白怎么使用了。 

```java
Publisher publisher = new SubmissionPublisher&lt;HelloReply&gt;();
// publisher.offer(xx)  即可发送消息

return JavaFlowSupport.Source.fromPublisher(publisher);
```

`Source.fromPublisher()` 方法需要使用`akka-stream` 库，有兴趣的朋友可以试着研究下。



### FAQ

#### `akka 2.6.0` 生成代码后报错。

使用更高的版本， 比如 `akka 2.6.4 ` 即可以解决问题。

如果非要使用 2.6.0 的话， 需要自己手动修改生成出来的代码， 不推荐这种方式。



#### 如果我完全不懂`akka` 我可以使用`akka-grpc` 吗

可以是可以。。  但是难度不小， 这个库是基于 `akka`,`akka-stream`,`akka-http` 的。 其中异步响应式流框架`akka-stream` 有很多API， 在`akka-grpc` 里面应该都是可用的。

所以想好好使用这个框架的话， 可能也需要花时间去学习另外一个框架。













---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/akka-grpc%E7%9A%84%E7%AE%80%E5%8D%95%E5%85%A5%E9%97%A8/  

