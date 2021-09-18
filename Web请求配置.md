# 1 拦截器

## 1.1 多个拦截器顺序执行

​		拦截器只需要去实现HandlerInterceptor接口，HandlerInterceptor当中有三大方法

1. preHandle：请求在进入controller前，进入Servet后执行，实现机制是基于反射机制来实现

2. postHandle：在执行controller方法后执行，DispatcherServlet进行视图的渲染之前，也就是说在这个方法中你可以对ModelAndView进行操作

3. afterCompletion：DispatcherServlet进行视图的渲染之后

   如果需要多个拦截器顺序执行，则按顺序注册就行了：

```java
@Configuration
public class LoginInterptorConfig implements WebMvcConfigurer {
    public void addInterceptors(InterceptorRegistry registry) {
        System.out.println("拦截器配置开始加载");
        //注册TestInterceptor拦截器
        InterceptorRegistration registration = registry.addInterceptor(new LoginInterceptor()).addPathPatterns("/api/*");
        registry.addInterceptor(new LogDataInterceptor()).addPathPatterns("/api/base");
    }

}
```



## 1.2 在拦截器中去注入Bean

​		如果采用上面方法：在addInterceptors去注册拦截器，这个拦截器是无法被Spring管理的，也就是说拦截器LogDataInterceptor中的bean是无法被注入的，如果想要在拦截器中注入bean：

1. 手动注册拦截器LogDataInterceptor，并使用@Bean生成一个个Bean实例去交给Spring容器管理，在这个过程中，**Spring会提前去扫描主启动类所在包，以及子包下的类的注解，将被注解标注的类纳入容器**，因为拦截器其实是在Servlet内使用反射去实现的。

从HTTP请求进来开始，整个链路加载顺序为：

1. Tomcat容器
2. Filter过滤器
3. Servlet
4. Interceptor拦截器
5. Controller控制器



>  **/ 与 /* 的区别**

​	以http://localhost:8080/为例子

- 在配置拦截路径为 【**/**】时，只会拦截 http://localhost:8080/，而不会去拦截 http://localhost:8080/api，总结来说就是，只会匹配【**/**】路径，只要请求根路径是【**/api/a/b**】这类多级请求路径就不会被拦截
- 【**/***】是会拦截所有的请求，不管放行了什么请求，就算配置了放行【**/a.html**】也会被拦截，【/**】就更是一样了
- /* / api / * 拦截的是二级请求路径包含api的字符串的，如**/api/a**是不会被拦截的



> @Compnent、@Service等注解

​		像这类注解，注入到Spring容器中的实例的bean名称，都是以小写的字母开头注入到容器中的。

```
// Spring容器中的bean名称其实是：user而不是User
@Compnonet
public class User{
}


public class UserDao{
}

// Spring容器中的bean名称其实是：userDaoImpl而不是UserDaoImpl
@Compnonet
public class UserDaoImpl implements UserDao{

}
```



> **@Autowired 与 @Resource的区别**

1. @Autowired是按照类型ByType注入的，如果允许null值，可以设置它的required属性为false。如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用。
2.  @Resource是按照名称ByName来注入的，如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不制定name也不制定type属性，这时将通过反射机制使用byName自动注入策略。

