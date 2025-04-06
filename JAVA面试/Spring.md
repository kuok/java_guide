## Spring

---

### @Component Bean的Name默认是什么

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-01/553237788806541.png?Expires=4897071916&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=Dj139iBaMdjauCpDMxLrPw7Rhzo%3D)

---

### AutoWired 与 Resource 区别
1. 来源不同  
@Autowired 和 @Resource 来自不同的“父类”，其中 `@Autowired` 是 `Spring2.5` 定义的注解
同时宣布支持@Resource ，而 `@Resource` 是 `Java` 定义的注解，它来自于 `JSR-250`（Java 250 规范提案）

2. 依赖查找的顺序不同
   * @AutoWired注解  
   @Autowired 注解在查找要注入的 bean 时，首先会`按照类型`进行匹配。如果有多个匹配的 bean，就会根据名称进行匹配。  
   此外，可以在 @Autowired 注解中使用 @Qualifier 注解来指定要注入的bean的名称，如果不使用 @Qualifier 注解就会使用属性名。
   
   ```java
   @Autowired
   @Qualifier("userService2")
   private UserService userService;
   ```
   
   * @Resource注解  
   @Resource注解既没有指定name属性，也没有指定type属性，那么它会`默认按照名称`来查找对应的bean，并将其注入到被注解的属性或者方法参数中。
   ```java
   @Resource(name = "myBean")
   private MyBean myBean;
   
   @Resource(type = MyBean.class)
   private MyBean myBean;
   ```

3. 支持的参数不同  
   @Autowired 和 @Resource 在使用时都可以设置参数，但二者支持的参数以及参数的个数完全不同，其中 @Autowired 只支持设置一个 required 的参数，而 @Resource 支持 7 个参数，支持的参数如下图所示：

   ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-01/554076428603125.png?Expires=4897072754&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=2rYZ7s2%2FhL1dhQoBMDMRtRuv10I%3D)

   ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-01/554081898301500.png?Expires=4897072760&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=gztWQXos5KtnFsXc%2Fe3yVyLeD4Y%3D)

4. 依赖注入的用法支持不同  
   @Autowired 支持属性注入、构造方法注入和 Setter 注入，而 @Resource 只支持属性注入和 Setter 注入

---

### Bean的实例化和Bean的初始化
Spring在创建一个Bean对象时，会先创建出来一个Java对象，会通过反射来执行类的构造方法从而得到一个Java对象，而这个过程就是Bean的`实例化`。
> 实例化指将对象创建到JVM中

得到Java对象后，会进行依赖注入，依赖注入之后就会进行初始化了，而Bean的`初始化`就是调用前面创建出来的Java对象中特定的方法，比如Java对象实现了InitializingBean接口，那么初始化的时候就会执行Java对象的afterPropertiesSet()，Spring只会执行这个方法，并不关心方法做了什么，我们可以在这个方法中去对某个属性进行验证，或者直接给某个属性赋值都是可以的，反正Bean的初始化就是执行afterPropertiesSet()方法，或者执行init-method指定的方法，
> 初始化指对对象进行属性注入后的可使用状态。

---

### Bean创建的生命周期
Spring中一个Bean的创建大概分为以下几个步骤：
1. 推断构造方法
2. 实例化
3. 填充属性，也就是依赖注入
4. 处理Aware回调
5. 初始化前，处理@PostConstruct注解
6. 初始化，处理InitializingBean接口
7. 初始化后，进行AOP

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-01/556237622664625.png?Expires=4897074916&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=WF4186bkQgkAVH%2Ft1ypR6dfdZ90%3D)


---

### Spring中的事务是如何实现的
1. Spring事务底层是基于数据库事务和AOP机制的
2. 首先对于使用了@Transactional注解的Bean，Spring会创建一个代理对象作为Bean
3. 当调用代理对象的方法时，会先判断该方法上是否加了@Transactional注解
4. 如果加了，那么则利用事务管理器创建一个数据库连接
5. 并且修改数据库连接的autocommit属性为false，禁止此连接的自动提交，这是实现Spring事务非常重要的一步
6. 然后执行当前方法，方法中会执行sql
7. 执行完当前方法后，如果没有出现异常就直接提交事务
8. 如果出现了异常，并且这个异常是需要回滚的就会回滚事务，否则仍然提交事务
9. Spring事务的隔离级别对应的就是数据库的隔离级别
10. Spring事务的传播机制是Spring事务自己实现的，也是Spring事务中最复杂的
11. Spring事务的传播机制是基于数据库连接来做的，一个数据库连接一个事务，如果传播机制配置为需要新开一个事务，那么实际上就是先建立一个数据库连接，在此新数据库连接上执行sql

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-01/554625170827458.png?Expires=4897073304&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=7aXcq2fEZSeYxnM5yPeclVDfvBc%3D)

---

### Spring事务的传播机制
1. REQUIRED(默认)：如果当前没有事务，则自己新建一个事务，如果当前存在事务，则加入这个事务
2. SUPPORTS：当前存在事务，则加入当前事务，如果当前没有事务，就以非事务方法执行
3. MANDATORY：当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常。
4. REQUIRES_NEW：创建一个新事务，如果存在当前事务，则挂起该事务。
5. NOT_SUPPORTED：以非事务方式执行,如果当前存在事务，则挂起当前事务
6. NEVER：不使用事务，如果当前事务存在，则抛出异常
7. NESTED：如果当前事务存在，则在嵌套事务中执行，否则和REQUIRED的操作一样（开启一个事务）

---

### 哪些情况下会导致Spring事务失效
1. **方法内的自调用**：Spring事务是基于AOP的，只要使用代理对象调用某个方法时，Spring事务才能生效，而在一个方法中调用使用this.xxx()调用方法时，this并不是代理对象，所以会导致事务失效。
   * 解放办法1：把调用方法拆分到另外一个Bean中
   * 解决办法2：自己注入自己
   * 解决办法3：AopContext.currentProxy()+@EnableAspectJAutoProxy(exposeProxy = true)
2. **方法是private的**：Spring事务会基于CGLIB来进行AOP，而CGLIB会基于父子类来失效，子类是代理类，父类是被代理类，如果父类中的某个方法是private的，那么子类就没有办法重写它，也就没有办法额外增加Spring事务的逻辑。
3. **方法是final的**：原因和private是一样的，也是由于子类不能重写父类中的final的方法
4. **单独的线程调用方法**：当Mybatis或JdbcTemplate执行SQL时，会从ThreadLocal中去获取数据库连接对象，如果开启事务的线程和执行SQL的线程是同一个，那么就能拿到数据库连接对象，如果不是同一个线程，那就拿到不到数据库连接对象，这样，Mybatis或JdbcTemplate就会自己去新建一个数据库连接用来执行SQL，此数据库连接的autocommit为true，那么执行完SQL就会提交，后续再抛异常也就不能再回滚之前已经提交了的SQL了。
5. **没加@Configuration注解**：如果用SpringBoot基本没有这个问题，但是如果用的Spring，那么可能会有这个问题，这个问题的原因其实也是由于Mybatis或JdbcTemplate会从ThreadLocal中去获取数据库连接，但是ThreadLocal中存储的是一个MAP，MAP的key为DataSource对象，value为连接对象，而如果我们没有在AppConfig上添加@Configuration注解的话，会导致MAP中存的DataSource对象和Mybatis和JdbcTemplate中的DataSource对象不相等，从而也拿不到数据库连接，导致自己去创建数据库连接了。
6. **异常被吃掉**：如果Spring事务没有捕获到异常，那么也就不会回滚了，默认情况下Spring会捕获RuntimeException和Error。
7. **类没有被Spring管理**
8. **数据库不支持事务**

---

### 过滤器（Filter）和拦截器（Interceptor）

参考：<https://juejin.cn/post/7097781804827934733?share_token=bc432f68-aeb2-43c7-87cf-5e797984aba9>

---

#### 过滤器（Filter）

---

1. 介绍：  
   过滤器 Filter 是 Sun 公司在 Servlet 2.3 规范中添加的新功能，其作用是对客户端发送给 Servlet 的请求以及对 Servlet 返回给客户端的响应做一些定制化的处理，例如校验请求的参数、设置请求/响应的 Header、修改请求/响应的内容等。  
   Filter 引入了过滤链（Filter Chain）的概念，一个 Web 应用可以部署多个 Filter，这些 Filter 会组成一种链式结构，客户端的请求在到达 Servlet 之前会一直在这个链上传递，不同的 Filter 负责对请求/响应做不同的处理。
2. 创建  
   OncePerRequestFilter 是一个由 Spring 提供的抽象类，在项目中，我们可以采用继承 OncePerRequestFilter 的方式创建 Filter，然后重写 doFilterInternal 方法定义 Filter 的处理逻辑，重写 shouldNotFilter 方法设置 Filter 的放行规则。对于多个 Filter 的执行顺序，我们也可以通过添加 @Order 注解进行设置。当然，若要使 Filter 生效，还需添加 @Component 注解将其注册到 Spring 容器。
   
   ```java
   @Component
   @Order(1)
   public class CSpringFilter extends OncePerRequestFilter {
       @Override
       protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
           // 处理逻辑
       }
   
       @Override
       protected boolean shouldNotFilter(HttpServletRequest request) throws ServletException {
           // 放行规则
       }
   }
   ```

3. 应用场景
* **解决跨域访问**：前后端分离的项目往往存在跨域访问的问题，Filter 允许我们在 response 的 Header 中设置 "Access-Control-Allow-Origin"、"Access-Control-Allow-Methods" 等头域，以此解决跨域失败问题。
* **设置字符编码**：字符编码 Filter 可以在 request 提交到 Servlet 之前或者在 response 返回给客户端之前为请求/响应设置特定的编码格式，以解决请求/响应内容乱码的问题。
* **记录日志**：日志记录 Filter 可以在拦截到请求后，记录请求的 IP、访问的 URL，拦截到响应后记录请求的处理时间。当不需要记录日志时，也可以直接将 Filter 的配置注释掉。
* **校验权限**：Web 服务中，客户端在发送请求时会携带 cookie 或者 token 进行身份认证，权限校验 Filter 可以在 request 提交到 Servlet 之前对 cookie 或 token 进行校验，如果用户未登录或者权限不够，那么 Filter 可以对请求做重定向或返回错误信息。
* **替换内容**：内容替换 Filter 可以对网站的内容进行控制，防止输入/输出非法内容和敏感信息。例如在请求到达 Servlet 之前对请求的内容进行转义，防止 XSS 攻击；在 Servlet 将内容输出到 response 时，使用 response 将内容缓存起来，然后在 Filter 中进行替换，最后再输出到客户浏览器（由于默认的 response 并不能严格的缓存输出内容，因此需要自定义一个具备缓存功能的 response）。

4.实现原理  
如果将函数（C++ 中的函数指针，Java 中的匿名函数、方法引用等）作为参数传递给主方法，那么这个函数就称为回调函数，主方法会在某一时刻调用回调函数。  
使用回调函数的好处是能够实现函数逻辑的解耦，主方法内可以定义通用的处理逻辑，部分特定的操作则交给回调函数来完成。例如 Java 中 Arrays 类的 sort(T[] a, Comparator<? super T> c) 方法允许我们传入一个比较器来自定义排序规则，这个比较器的 compare 方法就属于回调函数，sort 方法会在排序时调用 compare 方法。

---

#### 拦截器（Interceptor）

1. 介绍  
   拦截器 Interceptor 是 Spring MVC 中的高级组件之一，其作用是拦截用户的请求，并在请求处理前后做一些自定义的处理，如校验权限、记录日志等。这一点和 Filter 非常相似，但不同的是，Filter 在请求到达 Servlet 之前对请求进行拦截，而 Interceptor 则是在请求到达 Controller 之前对请求进行拦截，响应也同理。 与 Filter 一样，Interceptor 也具备链式结构，我们在项目中可以配置多个 Interceptor，当请求到达时，每个 Interceptor 根据其声明的顺序依次执行。

2. 创建  
   创建 Interceptor 需要实现 org.springframework.web.servlet.HandlerInterceptor 接口，HandlerInterceptor 接口中定义了三个方法：
  * preHandle：在 Controller 方法执行前被调用，可以对请求做预处理。该方法的返回值是一个 boolean 变量，只有当返回值为 true 时，程序才会继续向下执行。
  * postHandle：在 Controller 方法执行结束，DispatcherServlet 进行视图渲染之前被调用，该方法内可以操作 Controller 处理后的 ModelAndView 对象。
  * afterCompletion：在整个请求处理完成（包括视图渲染）后被调用，通常用来清理资源。
    注意，postHandle 方法和 afterCompletion 方法执行的前提条件是 preHandle 方法的返回值为 true。
    
     ```java
     @Component
     public class TestInterceptor implements HandlerInterceptor {
         @Override
         public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
             System.out.println("Interceptor 拦截到了请求: " + request.getRequestURL());
             return true;
         }
    
         @Override
         public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    
             System.out.println("Interceptor 操作 modelAndView...");
         }
    
         @Override
         public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
             System.out.println("Interceptor 清理资源...");
         }
     }
     ```
    
    Interceptor 需要注册到 Spring 容器才能够生效，注册的方法是在配置类中实现 WebMvcConfigurer 接口，并重写 addInterceptors 方法:
    
     ```java
     @Configuration
     public class TestInterceptorConfig implements WebMvcConfigurer {
    
         @Autowired
         private TestInterceptor testInterceptor;
    
         @Override
         public void addInterceptors(InterceptorRegistry registry) {
             registry.addInterceptor(testInterceptor)
                     .addPathPatterns("/*")
                     .excludePathPatterns("/**/*.css", "/**/*.js", "/**/*.png", "/**/*.jpg", "/**/*.jpeg")
                     .order(1);
         }
     }
     ```

3. 应用场景  
   Interceptor 的应用场可以参考上文中介绍的 Filter 的应用场景，可以说 Filter 能做到的事 Interceptor 都能做。由于 Filter 在 Servlet 前后起作用，而 Interceptor 可以在 Controller 方法前后起作用，例如操作 Controller 处理后的 ModelAndView，因此 Interceptor 更加灵活，在 Spring 项目中，如果能使用 Interceptor 的话尽量使用 Interceptor。

4. 实现原理
   
   ```java
   protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
       // 省略部分代码
       try {
           ModelAndView mv = null;
           Exception dispatchException = null;
   
           try {
               processedRequest = checkMultipart(request);
               multipartRequestParsed = (processedRequest != request);
               // 根据请求查找处理链 HandlerExecutionChain
               mappedHandler = getHandler(processedRequest);
               if (mappedHandler == null) {
                   noHandlerFound(processedRequest, response);
                   return;
               }
   
               // 获取适配器 HandlerAdapter, HandlerAdapter 用于处理请求
               HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
   
               // 省略部分代码
   
               // 依次执行所有拦截器的 preHandle 方法
               if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                   return;
               }
   
               // 执行 Controller 方法, 返回 ModelAndView
               mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
   
               if (asyncManager.isConcurrentHandlingStarted()) {
                   return;
               }
   
               applyDefaultViewName(processedRequest, mv);
               // 依次执行所有拦截器的 postHandle 方法, 顺序和 preHandle 方法相反
               mappedHandler.applyPostHandle(processedRequest, response, mv);
           }
           catch (Exception ex) {
               dispatchException = ex;
           }
           catch (Throwable err) {
               dispatchException = new NestedServletException("Handler dispatch failed", err);
           }
           // 进行视图渲染, 结束后执行拦截器的 afterCompletion 方法
           processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
       }
       // 如果发生异常, 那么执行拦截器的 afterCompletion 方法并抛出异常
       catch (Exception ex) {
           triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
       }
       catch (Throwable err) {
           triggerAfterCompletion(processedRequest, response, mappedHandler,
                   new NestedServletException("Handler processing failed", err));
       }
       // 省略部分代码
   }
   ```

* 首先获取当前请求对应的处理链 HandlerExecutionChain，HandlerExecutionChain 中包含了当前请求匹配的 Interceptor 列表。
* 然后根据 HandlerExecutionChain 获取适配器 HandlerAdapter，HandlerAdapter 用于处理当前请求。
* 依次执行所有 Interceptor 的 preHandle 方法，如果某个 preHandle 方法返回 false，那么程序将不再向下执行。
* 调用 HandlerAdapter 处理请求，返回 ModelAndView。
* 请求处理完成后，依次执行所有 Interceptor 的 postHandle 方法，执行的顺序与 preHandle 方法相反。
* 进行视图渲染，渲染结束后执行所有 Interceptor 的 afterCompletion 方法，执行顺序与 postHandle 方法相同。
* 如果以上操作发生异常，那么执行所有 Interceptor 的 afterCompletion 方法并抛出异常。

如果 Controller 抛出异常，那么 postHandle 方法将不会执行，afterCompletion 方法则一定执行。

---

#### Filter 和 Interceptor 的区别

1. **规范不同**  
   Filter 在 Servlet 规范中定义，依赖于 Servlet 容器（如 Tomcat）；Interceptor 由 Spring 定义，依赖于 Spring 容器（IoC 容器）。
2. **适用范围不同**  
   Filter 仅可用于 Web 程序，因为其依赖于 Servlet 容器；Interceptor 不仅可以用于 Web 程序，还可以用于 Application、Swing 等程序。
3. **触发时机不同**  
   Filter 在请求进入 Servlet 容器，且到达 Servlet 之前对请求做预处理；在 Servlet 处理完请求后对响应做后处理。
   Interceptor 在请求进入 Servlet，且到达 Controller 之前对请求做预处理；在 Controller 处理完请求后对 ModelAndView 做后处理，在视图渲染完成后再做一些收尾工作。
   ![触发时机](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-13/31987163644200.png?Expires=4895467240&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=yMqH%2BINjIq4dGXnW%2Bh8INNKTycA%3D)
4. **实现不同**  
   过滤器是基于方法回调实现的，而拦截器是基于动态代理（底层是反射）实现的。
5. **使用的场景不同**  
   因为拦截器更接近业务系统，所以拦截器主要用来实现项目中的业务判断的，比如：登录判断、权限判断、日志记录等业务。 而过滤器通常是用来实现通用功能过滤的，比如：敏感词过滤、字符集编码设置、响应数据压缩等功能。

---
