# 如何让ebean处理自定义类型成json


## 简介

如果数据库字段类型是 json/jsonb ， 那么使用ebean 的时候可以自动把字段序列化成json 。  
但是 ebean 内置支持的java类型的只有： 
- List\<Long\>, List\<String\>
- Set\<Long\>, Set\<String\>
- Map<String, Object>  

如果把 `@DbJson/@DbJsonb`  注解添加到了一个不支持的类型， 那么启动的时候 ebean 就会报错。 所以简单来说，就只使用上面的类型最好。

但是如果你特别想使用别的类型（这是正常需求）， 可以考虑下面的做法。

## 主要内容

笔者阅读了ebean 的源代码， 然后做了一些测试， 笔者发现，给项目添加下面的代码就可以实现这个需求。 


**CommonEbeanJsonMapper**
```java
import io.ebean.core.type.ScalarJsonMapper;
import io.ebean.core.type.ScalarJsonRequest;
import io.ebean.core.type.ScalarType;
public class CommonEbeanJsonMapper implements ScalarJsonMapper{
    @Override
    public ScalarType<?> createType(ScalarJsonRequest request) {
        return new CommonStringJsonScalarType<>(request.beanType());
    }
    @Override
    public <A extends Annotation> @Nullable Class<A> markerAnnotation() {
        return (Class<A>)MMStringDBValue.class;
    }
}
```

**MMStringDBValue**
```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
@Retention(RetentionPolicy.RUNTIME)
public @interface MMStringDBValue {
}
```

**CommonStringJsonScalarType<T>**
```java
import io.ebean.config.dbplatform.DbPlatformType;
import io.ebean.core.type.ScalarTypeBaseVarchar;
import lombok.extern.slf4j.Slf4j;
@Slf4j
public class CommonStringJsonScalarType<T> extends ScalarTypeBaseVarchar<T> {
	private ObjectMapper objectMapper = new ObjectMapper();

    public CommonStringJsonScalarType(Class<T> type) {
        super(type, false, DbPlatformType.JSONB);
    }
    @Override
    public String formatValue(T v) {
        return convertToDbString(v);
    }
    @Override
    public T parse(String value) {
        return convertFromDbString(value);
    }
    @Override
    public T convertFromDbString(String dbValue) {
        try {
            return objectMapper.readValue(dbValue, type);
        } catch (JsonMappingException e) {
            log.error("dbValue=" + dbValue, e);
        } catch (JsonProcessingException e) {
            log.error("dbValue=" + dbValue, e);
        }
        return null;
    }
    @Override
    public String convertToDbString(T beanValue) {
        try {
            return objectMapper.writeValueAsString(beanValue);
        } catch (JsonProcessingException e) {
            log.error("dbValue=" + beanValue, e);
        }
        return null;
    }
}
```

然后创建一个文件 `src/main/resources/META-INF/services/io.ebean.core.type.ScalarJsonMapper`  ， 值是 类型``CommonEbeanJsonMapper`` 的全限定名称，比如 ``com.abc.ebean.CommonEbeanJsonMapper``   
这个文件的作用是使用SPI 告诉ebean ， 我提供了一个 json 映射器， 当你找不到类型对应的的转换类的时候， 就使用这个。

然后在自己的字段上添加自己的注解就可以了， 比如：
```java
    @DbJsonB
    @MMStringDBValue
    private List<AcFanCombination> acFanCombinations;
```

下面对实现做一些说明：
- MMStringDBValue 这个注解可以随便修改名字， 这个就是告诉ebean 我只处理有这个额外注解的字段。
- CommonEbeanJsonMapper 这个类型是关联了注解和字段的解析类型
- CommonStringJsonScalarType  这个则是解析类型， 即如何实现解析。 

---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/%E5%A6%82%E4%BD%95%E8%AE%A9ebean%E5%A4%84%E7%90%86%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B1%BB%E5%9E%8B%E6%88%90json/  

