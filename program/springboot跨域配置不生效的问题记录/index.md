# Springboot跨域配置不生效的问题记录



先看一下下面的代码： 

```java
@Configuration
@RequiredArgsConstructor 
public class WebConfig implements WebMvcConfigurer {

    private final ApiAuthInterceptor apiAuthInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(apiAuthInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns(ApiAuthInterceptor.IGNORE_URI_SET.toArray(new String[0]));
    }

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "POST", "PUT", "DELETE")
                .allowedHeaders("*")
                .maxAge(3600);
    }
}
```

上面的代码看起来没什么问题， 只是添加了一个拦截器和配置了跨域。   

但是笔者使用这个代码的时候却发现跨域配置不生效， 经过仔细排查之后发现原因在于拦截器上。 

拦截器里面的代码大概如下： 

```java
	private void writeCode401(HttpServletResponse response) throws Exception {

        response.addHeader("Content-Type", "application/json");

        response.getWriter().write(code401Json);

    }


    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {

        if (handler instanceof HandlerMethod handlerMethod) {
        }
	   
	    writeCode401(response);
	    return false;
	}
```

原因在于这个拦截器除了拦截处理方法， 还会拦截其他过滤器。   上面的跨域配置添加了之后， spring 会创建一个 org.springframework.web.filter.CorsFilter 的对象， 用于实现这个跨域配置。  这个对象会被拦截器的 preHandle 方法处理，对象将作为 handler 参数传入。  

所以解决方案就是： 

```java
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {

        if (handler instanceof HandlerMethod handlerMethod) {
            // xxxxx
        }
	   
	    // writeCode401(response);
	    // return false;
        return true;
	}
```

参考阅读：   https://www.cnblogs.com/haoxianrui/p/17338196.html



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/springboot%E8%B7%A8%E5%9F%9F%E9%85%8D%E7%BD%AE%E4%B8%8D%E7%94%9F%E6%95%88%E7%9A%84%E9%97%AE%E9%A2%98%E8%AE%B0%E5%BD%95/  

