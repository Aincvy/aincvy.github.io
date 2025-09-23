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
option java_package = "example.myapp.helloworld.grpc";
option java_outer_classname = "HelloWorldProto";

package helloworld;

// The request message containing the user's name.
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
<project>
  <modelVersion>4.0.0</modelVersion>
  <name>Project name</name>
  <groupId>com.example</groupId>
  <artifactId>my-grpc-app</artifactId>
  <version>0.1-SNAPSHOT</version>
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <akka.grpc.version>1.0.0-M1</akka.grpc.version>
    <grpc.version>1.29.0</grpc.version>
    <project.encoding>UTF-8</project.encoding>
  </properties>
  <dependencies>
    <dependency>
      <groupId>com.lightbend.akka.grpc</groupId>
      <artifactId>akka-grpc-runtime_2.12</artifactId>
      <version>${akka.grpc.version}</version>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>com.lightbend.akka.grpc</groupId>
        <artifactId>akka-grpc-maven-plugin</artifactId>
        <version>${akka.grpc.version}</version>
        <configuration>
        	<generatorSettings>
          	<serverPowerApis>true</serverPowerApis>
        	</generatorSettings>
        </configuration>
        <executions>
          <execution>
            <goals>
              <goal>generate</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
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
  public CompletionStage<HelloReply> sayHello(HelloRequest in) {
    System.out.println("sayHello to " + in.getName());
    HelloReply reply = HelloReply.newBuilder().setMessage("Hello, " + in.getName()).build();
    return CompletableFuture.completedFuture(reply);
  }

  @Override
  public CompletionStage<HelloReply> itKeepsTalking(Source<HelloRequest, NotUsed> in) {
    System.out.println("sayHello to in stream...");
    return in.runWith(Sink.seq(), mat)
      .thenApply(elements -> {
        String elementsStr = elements.stream().map(elem -> elem.getName())
            .collect(Collectors.toList()).toString();
        return HelloReply.newBuilder().setMessage("Hello, " + elementsStr).build();
      });
  }

  @Override
  public Source<HelloReply, NotUsed> itKeepsReplying(HelloRequest in) {
    System.out.println("sayHello to " + in.getName() + " with stream of chars");
    List<Character> characters = ("Hello, " + in.getName())
        .chars().mapToObj(c -> (char) c).collect(Collectors.toList());
    return Source.from(characters)
      .map(character -> {
        return HelloReply.newBuilder().setMessage(String.valueOf(character)).build();
      });
  }

  @Override
  public Source<HelloReply, NotUsed> streamHellos(Source<HelloRequest, NotUsed> in) {
    System.out.println("sayHello to stream...");
    return in.map(request -> HelloReply.newBuilder().setMessage("Hello, " + request.getName()).build());
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
    // important to enable HTTP/2 in ActorSystem's config
    Config conf = ConfigFactory.parseString("akka.http.server.preview.enable-http2 = on")
            .withFallback(ConfigFactory.defaultApplication());

    // Akka ActorSystem Boot
    ActorSystem sys = ActorSystem.create("HelloWorld", conf);

    run(sys).thenAccept(binding -> {
      System.out.println("gRPC server bound to: " + binding.localAddress());
    });

    // ActorSystem threads will keep the app alive until `system.terminate()` is called
  }

  public static CompletionStage<ServerBinding> run(ActorSystem sys) throws Exception {
    Materializer mat = ActorMaterializer.create(sys);

    // Instantiate implementation
    GreeterService impl = new GreeterServiceImpl(mat);

    return Http.get(sys).bindAndHandleAsync(
      GreeterServiceHandlerFactory.create(impl, sys),
      ConnectHttp.toHost("127.0.0.1", 8080),
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

    String serverHost = "127.0.0.1";
    int serverPort = 8080;

    ActorSystem system = ActorSystem.create("HelloWorldClient");
    Materializer materializer = ActorMaterializer.create(system);

    // Configure the client by code:
    GrpcClientSettings settings = GrpcClientSettings.connectToServiceAt("127.0.0.1", 8080, system);

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
      System.out.println("Status: " + e.getStatus());
    } catch (Exception e)  {
      System.out.println(e);
    } finally {
      if (client != null) client.close();
      system.terminate();
    }

  }

  private static void singleRequestReply(GreeterService client) throws Exception {
    HelloRequest request = HelloRequest.newBuilder().setName("Alice").build();
    CompletionStage<HelloReply> reply = client.sayHello(request);
    System.out.println("got single reply: " + reply.toCompletableFuture().get(5, TimeUnit.SECONDS));
  }

  private static void streamingRequest(GreeterService client) throws Exception {
    List<HelloRequest> requests = Arrays.asList("Alice", "Bob", "Peter")
        .stream().map(name -> HelloRequest.newBuilder().setName(name).build())
        .collect(Collectors.toList());
    CompletionStage<HelloReply> reply = client.itKeepsTalking(Source.from(requests));
    System.out.println("got single reply for streaming requests: " +
        reply.toCompletableFuture().get(5, TimeUnit.SECONDS));
  }

  private static void streamingReply(GreeterService client, Materializer mat) throws Exception {
    HelloRequest request = HelloRequest.newBuilder().setName("Alice").build();
    Source<HelloReply, NotUsed> responseStream = client.itKeepsReplying(request);
    CompletionStage<Done> done =
      responseStream.runForeach(reply ->
        System.out.println("got streaming reply: " + reply.getMessage()), mat);

    done.toCompletableFuture().get(60, TimeUnit.SECONDS);
  }

  private static void streamingRequestReply(GreeterService client, Materializer mat) throws Exception {
    Duration interval = Duration.ofSeconds(1);
    Source<HelloRequest, NotUsed> requestStream = Source
      .tick(interval, interval, "tick")
      .zipWithIndex()
      .map(pair -> pair.second())
      .map(i -> HelloRequest.newBuilder().setName("Alice-" + i).build())
      .take(10)
      .mapMaterializedValue(m -> NotUsed.getInstance());

    Source<HelloReply, NotUsed> responseStream = client.streamHellos(requestStream);
    CompletionStage<Done> done =
      responseStream.runForeach(reply ->
        System.out.println("got streaming reply: " + reply.getMessage()), mat);

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
Publisher publisher = new SubmissionPublisher<HelloReply>();
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

