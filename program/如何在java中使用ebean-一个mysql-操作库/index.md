# 如何在java中使用ebean


## 简介

笔者在寻找一个轻量级 ORM 工具的时候， chat gpt 告诉笔者 ebean 是比较轻量的， 所以笔者就尝试了一下ebean 这个库。 

体验下来感觉使用上是比较方便的，但是配置起来则比较令人头大， 所以写一篇博文记录一下相关内容。 

## 主要内容

### 单模块项目使用ebean 
一些链接：
- ebean 官网：  https://ebean.io/
- 如何排查问题：  https://ebean.io/docs/trouble-shooting


以下是一些基本信息：
- 项目基于Maven 进行配置。 
- 使用Mysql 数据库。   *如果使用别的数据库也只是需要修改一些依赖项就可以了。*
- 除了引用依赖之外， 还需要使用maven 插件。 


Maven 依赖库： 
```xml
        &lt;!-- https://mvnrepository.com/artifact/jakarta.persistence/jakarta.persistence-api --&gt;
        &lt;dependency&gt;
            &lt;groupId&gt;jakarta.persistence&lt;/groupId&gt;
            &lt;artifactId&gt;jakarta.persistence-api&lt;/artifactId&gt;
            &lt;version&gt;${jakarta.version}&lt;/version&gt;
        &lt;/dependency&gt;

        &lt;!-- https://mvnrepository.com/artifact/io.ebean/ebean --&gt;
        &lt;dependency&gt;
            &lt;groupId&gt;io.ebean&lt;/groupId&gt;
            &lt;artifactId&gt;ebean-core&lt;/artifactId&gt;
            &lt;version&gt;${ebean.version}&lt;/version&gt;
        &lt;/dependency&gt;

        &lt;!-- https://mvnrepository.com/artifact/io.ebean/ebean-platform-mysql --&gt;
        &lt;dependency&gt;
            &lt;groupId&gt;io.ebean&lt;/groupId&gt;
            &lt;artifactId&gt;ebean-platform-mysql&lt;/artifactId&gt;
            &lt;version&gt;${ebean.version}&lt;/version&gt;
        &lt;/dependency&gt;

        &lt;!-- https://mvnrepository.com/artifact/io.ebean/ebean-querybean --&gt;
        &lt;dependency&gt;
            &lt;groupId&gt;io.ebean&lt;/groupId&gt;
            &lt;artifactId&gt;ebean-querybean&lt;/artifactId&gt;
            &lt;version&gt;${ebean.version}&lt;/version&gt;
        &lt;/dependency&gt;

        &lt;dependency&gt;
            &lt;groupId&gt;io.ebean&lt;/groupId&gt;
            &lt;artifactId&gt;ebean-agent&lt;/artifactId&gt;
            &lt;version&gt;${ebean.version}&lt;/version&gt;
        &lt;/dependency&gt;
```

- `ebean.version` 变量用于指定ebean的版本， 示例值为： `15.1.0`   
- ebean 使用了JPA的注解来标记实体类， 比如 `@Entity, @Id, ...`
- `ebean-platform-mysql`  是表示操作mysql数据库的， 如果是其他数据库则使用其他的版本
- `ebean-querybean` 是关于Bean的附属类型 `QBean` 的支持， 主要是用于查询的。 
- `ebean-agent` 是修改Bean的字节码的， 实现一些字节码增强技术。 


Maven 插件： 
```xml
            &lt;plugin&gt;
                &lt;groupId&gt;org.apache.maven.plugins&lt;/groupId&gt;
                &lt;artifactId&gt;maven-compiler-plugin&lt;/artifactId&gt;
                &lt;configuration&gt;
                    &lt;annotationProcessorPaths&gt;
                        &lt;path&gt;
                            &lt;groupId&gt;org.projectlombok&lt;/groupId&gt;
                            &lt;artifactId&gt;lombok&lt;/artifactId&gt;
                            &lt;version&gt;${lombok.version}&lt;/version&gt;
                        &lt;/path&gt;
                        &lt;path&gt;
                            &lt;!-- generate ebean query beans --&gt;
                            &lt;groupId&gt;io.ebean&lt;/groupId&gt;
                            &lt;artifactId&gt;querybean-generator&lt;/artifactId&gt;
                            &lt;version&gt;${ebean.version}&lt;/version&gt;
                        &lt;/path&gt;
                    &lt;/annotationProcessorPaths&gt;
                &lt;/configuration&gt;
            &lt;/plugin&gt;

            &lt;?m2e execute onConfiguration,onIncremental?&gt;
            &lt;plugin&gt;
                &lt;!-- perform ebean enhancement --&gt;
                &lt;groupId&gt;io.ebean&lt;/groupId&gt;
                &lt;artifactId&gt;ebean-maven-plugin&lt;/artifactId&gt;
                &lt;version&gt;${ebean.version}&lt;/version&gt;
                &lt;extensions&gt;true&lt;/extensions&gt;
                &lt;executions&gt;
                &lt;execution&gt;
                    &lt;?m2e execute onConfiguration,onIncremental?&gt;
                        &lt;id&gt;enhance&lt;/id&gt;
                        &lt;phase&gt;process-classes&lt;/phase&gt;
                        &lt;goals&gt;
                            &lt;goal&gt;enhance&lt;/goal&gt;
                        &lt;/goals&gt;
                    &lt;/execution&gt;
                &lt;/executions&gt;
            &lt;/plugin&gt;
```

- 如果项目中没有使用lombok, 则可以去掉相关的部分。   
- 插件顺序换位可能会产生一些问题。
- `querybean-generator` 是用于自动生成 `QBean` 代码使用的。 
- `ebean-maven-plugin` 是用于编译之后自动做字节码增强使用的。 


如果是单模块项目的话， 有了上面的配置文件内容之后就可以认为差不多算是配置好了。  

*以下代码部分是手写， 部分是从官网或者其他网站上复制而来。*

```java
@Data
@MappedSuperclass
public abstract class BaseModel {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    protected Long id;
    
    @Version
    protected Long version;
    
    @WhenCreated
    protected Instant createdTime;
    
    @WhenModified
    protected Instant modifiedTime;

}


@Data
public class Order extends BaseModel{

    private long buyer;

    private long goodsId;

    private int status = 0;

}


public class Test {

    private Database database;

    public void setup() {
        // datasource
        DataSourceConfig dataSourceConfig = new DataSourceConfig();
        dataSourceConfig.setUsername(&#34;root&#34;);
        dataSourceConfig.setPassword(&#34;123456&#34;);
        dataSourceConfig.setUrl(&#34;jdbc:mysql://localhost:3306/mydb&#34;);


        // configuration ...
        DatabaseConfig config = new DatabaseConfig();
        config.setDataSourceConfig(dataSourceConfig);
        

        // create database instance
        database = DatabaseFactory.create(config);
    }

    public void testSave(){
        Order o = new Order();
        o.setBuyer(1);
        o.setGoodsId(15001);
        o.setStatus(0);

        database.save(o);
    }

    public void testUpdate(){
        List&lt;Order&gt; orders = new QOrder().goodsId.equalTo(15001).findList();

        for( Order o : orders) {
            if(o.getStatus() == 0) {
                o.setStatus(100);
                database.save(o);    // 或者循环结束之后， 使用saveAll， 这里的代码仅做示意作用。
            }
        }
    }

}

```

- `Version, WhenCreated, WhenModified` 由ebean 自行维护， 表示版本号， 创建时间， 最后的修改时间
- `QOrder` 是基于`Order` 生成的， 是插件`querybean-generator`  进行生成的。
- 除了`save` 还可以考虑显式的调用`insert/update` 方法


有了上面的代码， 就可以考虑运行程序了。   但是运行的话， 需要点辅助装置。  读者有2个选项， 一个是安装插件（idea, eclipse）, 或者添加jvm参数（任何编辑器都可用）

- 在 https://ebean.io/docs/getting-started/ 页面里面找到IDE Plugin 部分，然后Install 以下就可以了。 
  - 编辑器插件在打包之后的运行是不需要的。
- 手动添加jvm参数的话， 需要先将 `ebean-agent` 的jar 复制到项目路径， 随后在启动配置中添加jvm 参数 `-javaagent:${ebean-agent-jar-path}` 即可。
  - `ebean-agent` 的jar文件可以在 本地的.m2 目录里面找到。 路径应该是 `XXXXX/.m2/io/ebean/ebean-agent/xxx.jar` 
  - vs code 添加jvm参数需要修改 `launch.json` 配置文件，添加内容： `&#34;vmArgs&#34;: &#34;-javaagent:${ebean-agent-jar-path}&#34;` 
  - idea 需要在 `VM options` 那一栏里添加， 新版本这个选项默认隐藏了， 需要打开一下。 


### 问题处理

Q: java.lang.IllegalStateException: Bean class _ is not enhanced?  

因为 ebean-agent 没配置好， 编辑器下应该是插件问题， 或者 JVM参数没指定好。 

Q: NullPointerException using Query beans 

基本上问题是同上 。


### 多模块项目中使用ebean 


如果读者的多模块项目都使用ebean 库作为ORM工具， 那么相关配置直接配置到父模块即可。  但是如果读者的一个子模块不需要使用ebean, 比如是spring boot 项目， 那么需要使用下面的方式进行配置。 

- 将插件放到父模块的`build/pluginManagement/plugins`  节点下面
- 在每个需要启用插件的 子模块上将插件放到 `build/plugins` 节点下面， 只需要 `groupId, artifactId` 这两个属性即可。 其他属性都会从父模块上继承。 
- bean模块和 调用 bean 的模块都需要添加上面的插件。 


### 打包

在VS Code 开启过程中打包的话， 可能会出现一些问题，因为VS Code 的Java插件，Maven插件也会对源代码进行编译， 但是在编译的时候，他们是不应用ebean-maven-plugin 的， 所以如果在VS Code 开启的时候执行`mvn clean package` ，打出来的包可能是无法使用的。 

关闭VS Code 即可解决这个问题。 

## 总结

ebean 的配置确实繁琐了一些。。   但是它也有它自己的优点。 
    


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/%E5%A6%82%E4%BD%95%E5%9C%A8java%E4%B8%AD%E4%BD%BF%E7%94%A8ebean-%E4%B8%80%E4%B8%AAmysql-%E6%93%8D%E4%BD%9C%E5%BA%93/  

