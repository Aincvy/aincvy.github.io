# 如何在java中使用etcd   Jetcd


## 主要内容

### etcd 的特点

- KV 数据库， github 描述为：  Distributed reliable key-value store for the most critical data of a distributed system
- 较为轻量， 和etcd交互的时候就是 操作（*查询，设置，删除，监听*）一个 字符串的 键值对
- etcd 支持集群  *但是笔者没有使用过。*
- 每个KEY可以设置TTL，  并且存在 mod revision, 即修改版本号。


### 应用场景

- 服务注册与发现
  - 一些读者可能觉得nacos 比较好， 因为它可以以一个非常小的编码操作来实现这个功能。  那么笔者认为可以继续这样使用， 没必要一定要换到etcd。 
  - 笔者没有选择 nacos 的原因是因为笔者觉得 nacos 可能比较庞大， 笔者想引入一个轻量级的库。   
  - 笔者实现的思路是：  
    - 每个服务器上报自己的信息到etcd上。  
      - KEY的格式是： `/servers/TYPE/ID`
      - 上传间隔60秒， KEY的过期时间为65秒
      - 只有当前服务器自己会上传信息到这些KEY， 别的服务器会拉取这些KEY的信息， 但是不会修改。
    - 网关服务器使用 key prefix 来查询其他所有在线的服务器， 并尝试连接他们。 
    - 登录服使用 key prefix 来查询其他所有的网关服务器， 并根据网关在线人数 进行负载均衡。 
- 配置表读取和热更新
  - 配置表的KEY格式为： `/config/GROUP/CONFIG_FILENAME`     *突然发现， 服务器的KEY也可以把GROUP带上*
    - GROUP是服务器分组， 当多个服务器需要使用同一个etcd的时候 可以使用GROUP 进行分组。 
  - 配置表数据拉取到之后， 将这个KEY的 mod revision 记录一下。
  - 定时轮询一下etcd ， 或者使用 WatchClient 来监视一下KEY，  如果key被修改了 就进行数据更新。   *如果轮询的话，可以对比mod revision* 

### 代码示例

```java
    @Data
    public static class ServerRunningInfo {
        private int id;
        private int onlineCount;
        private String group = &#34;test&#34;;
        private String host;
        private int port;
        private String uuid;
    }

    public static ByteSequence seq(String str) {
        return ByteSequence.from(str, StandardCharsets.UTF_8);
    }

    public static void main(String[] args) {

        ObjectMapper objectMapper = new ObjectMapper();
        Client client = Client.builder().endpoints(&#34;http://127.0.0.1:2379&#34;)
            .password(seq(&#34;123456&#34;)).build();
        
        KV kvClient = client.getKVClient();

        var option = GetOption.builder().isPrefix(true).build();
        kvClient.get(seq(&#34;/servers/Fight/&#34;), option).thenAccept(r -&gt; {
            if (r.getCount() &lt;= 0) {
                return;
            }

            for (KeyValue kv : r.getKvs()) {
                long oldModRevision = 1;      // 应该保存在某处， 在此地获取
                if (kv.getModRevision() == oldModRevision) {
                    continue;
                }
                try {
                    ServerRunningInfo runningInfo = objectMapper.readValue(kv.getValue().getBytes(), ServerRunningInfo.class);
                    // connect to server or some thing 
                    System.out.println(runningInfo);
                    oldModRevision = kv.getModRevision();
                } catch (StreamReadException e) {
                    e.printStackTrace();
                } catch (DatabindException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

        }).join();    // 当前主线程没其他事情的时候， 如果这里不 join的话， 程序就会自动退出了。
        
        Lease leaseClient = client.getLeaseClient();
        var leaseId = leaseClient.grant(65).join().getID();      // 设置65秒的 TTL

        // 模拟放置服务器信息， 假设目前是网关服务器
        ServerRunningInfo myInfo = new ServerRunningInfo();
        try {
            PutOption putOption = PutOption.builder().withLeaseId(leaseId).build();
            kvClient.put(seq(&#34;/servers/Gate/&#34; &#43; myInfo.getId()), 
                ByteSequence.from(objectMapper.writeValueAsBytes(myInfo)),
                putOption)
                .join();    // 如果主线程或者说当前线程有其他任务的话， 就不需要join 
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }

        Watch watchClient = client.getWatchClient();
        watchClient.watch(seq(&#34;/config/test/A.json&#34;), r -&gt; {
            for (WatchEvent e : r.getEvents()) {
                KeyValue keyValue = e.getKeyValue();
                long oldModRevision = 1;      // 应该保存在某处， 在此地获取
                if (keyValue.getModRevision() == oldModRevision) {
                    continue;
                }

                var jsonContent = keyValue.getValue().toString();
                // do something with jsonContent ...
                oldModRevision = keyValue.getModRevision();
            }
        });
        
    }
```

可以看到， 总共的代码行数也不是特别多， 但是需要的功能都可以实现。   

自己实现的好处是减少黑盒占比， 出现了问题的时候方便解决。 



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/%E5%A6%82%E4%BD%95%E5%9C%A8java%E4%B8%AD%E4%BD%BF%E7%94%A8etcd-jetcd/  

