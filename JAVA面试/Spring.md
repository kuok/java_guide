# Spring

---

## Spring IOC 的实现机制
1. **反射**：Spring使用Java的反射机制来实现动态创建和管理Bean对象。通过反射，Spring可以在运行时动态地实例化Bean对象、调用Bean的方法和设置属性值。
2. **配置元数据**：Spring使用配置元数据来描述Bean的定义和依赖关系。配置元数据可以通过XML配置文件、注解和Java代码等方式进行定义。Spring在启动时会解析配置元数据，根据配置信息创建和管理Bean对象。
3. **Bean定义**：Spring使用Bean定义来描述Bean的属性、依赖关系和生命周期等信息。Bean定义可以通过XML配置文件中的<bean>元素、注解和Java代码中的@Bean注解等方式进行定义。Bean定义包含了Bean的类名、作用域、构造函数参数、属性值等信息。
4. **Bean工厂**：Spring的Bean工厂负责创建和管理Bean对象。Bean工厂可以是BeanFactory接口的实现，如DefaultListableBeanFactory。Bean工厂负责解析配置元数据，根据Bean定义创建Bean对象，并将其放入容器中进行管理。
5. **依赖注入**：Spring使用依赖注入来解决Bean之间的依赖关系。通过依赖注入，Spring容器负责将Bean所依赖的其他Bean实例注入到它们之中。Spring使用反射和配置元数据来确定依赖关系，并在运行时进行注入。

---

## Spring AOP and AspectJ AOP 有什么区别

1. 实现方式
   * Spring AOP：  
   基于代理：Spring AOP主要通过动态代理技术（如JDK动态代理和CGLIB代理）来实现。  
   运行时织入：在运行时通过代理对象将切面逻辑插入到目标对象的方法中。
     > 比喻：你有一个秘书，帮你处理一些日常事务，这样你就可以专注于更重要的事情。秘书是在你需要的时候才出现的。
   * AspectJ AOP：  
   基于字节码操作：AspectJ AOP通过字节码操作技术（如编译时织入、类加载时织入）来实现。  
   编译时/类加载时织入：在编译阶段或类加载阶段将切面逻辑直接插入到目标类的字节码中。
     > 比喻：你在建造房子的时候，就已经把电线、水管等基础设施埋好了，这样房子建好后可以直接使用这些设施。
2. 支持的切点表达式
   * Spring AOP：  
   有限的切点表达式：Spring AOP支持的方法拦截主要是基于方法的调用，不能对字段访问、构造函数等进行拦截。
     > 比喻：你只能在门口安装监控摄像头，不能在每个房间里都安装。
   * AspectJ AOP：  
   丰富的切点表达式：AspectJ AOP支持的方法拦截、字段访问、构造函数调用等多种切点表达式。
     > 比喻：你可以在门口、窗户、每个房间里都安装监控摄像头。
3. 性能
   * Spring AOP：  
   性能稍差：由于是运行时通过代理对象实现，每次方法调用都会有一定的开销。
     > 比喻：每次你需要秘书帮忙的时候，秘书都需要先找到你，再开始工作。
   * AspectJ AOP：  
   性能更好：由于是编译时或类加载时直接修改字节码，性能更高。
     > 比喻：你在建造房子的时候就已经把所有设施都安排好了，所以房子建好后可以直接使用，不需要额外的时间。
4. 配置和使用
   * Spring AOP：  
   配置简单：Spring AOP的配置相对简单，可以通过注解或XML配置来定义切面和切点。
     > 比喻：你只需要告诉秘书你需要他做什么，秘书就会帮你做好。
   * AspectJ AOP：  
   配置复杂：AspectJ AOP的配置相对复杂，需要编写切面类，并且可能需要额外的编译步骤。
     > 比喻：你需要详细规划房子的设计图，然后找专业的建筑工人来建造。


---

## @Component Bean的Name默认是什么

![beanname](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-01/553237788806541.png?Expires=4897071916&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=Dj139iBaMdjauCpDMxLrPw7Rhzo%3D)

---

## AutoWired 与 Resource 区别

1. 来源不同  
   @Autowired 和 @Resource 来自不同的“父类”，其中 `@Autowired` 是 `Spring2.5` 定义的注解  
   而 `@Resource` 是 `Java` 定义的注解，它来自于 `JSR-250`（Java 250 规范提案）

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

## Bean的实例化和Bean的初始化
Spring在创建一个Bean对象时，会先创建出来一个Java对象，会通过反射来执行类的构造方法从而得到一个Java对象，而这个过程就是Bean的`实例化`。
> 实例化指将对象创建到JVM中

得到Java对象后，会进行依赖注入，依赖注入之后就会进行初始化了，而Bean的`初始化`就是调用前面创建出来的Java对象中特定的方法，比如Java对象实现了InitializingBean接口，那么初始化的时候就会执行Java对象的afterPropertiesSet()，Spring只会执行这个方法，并不关心方法做了什么，我们可以在这个方法中去对某个属性进行验证，或者直接给某个属性赋值都是可以的，反正Bean的初始化就是执行afterPropertiesSet()方法，或者执行init-method指定的方法，
> 初始化指对对象进行属性注入后的可使用状态。

---

## Bean的生命周期
Spring中一个Bean的创建大概分为以下几个步骤：

1. 实例化
   * 通过反射去推断构造函数进行实例化
   * 实例工厂、 静态工厂
2. 依赖注入（DI）
   * 解析自动装配（byname bytype constractor none @Autowired）
3. 初始化
   * 调用很多Aware回调方法
   * 调用BeanPostProcessor.postProcessBeforeInitialization
   * 调用生命周期回调初始化方法
   * 调用BeanPostProcessor.postProcessAfterInitialization, 如果bean实现aop则会在这里创建动态代理
4. 使用
5. 销毁
   * 在spring容器关闭的时候进行调用
   * 调用生命周期回调销毁方法

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-01/556237622664625.png?Expires=4897074916&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=WF4186bkQgkAVH%2Ft1ypR6dfdZ90%3D)

---

## Spring容器启动流程是怎样的
1. 在创建Spring容器，也就是启动Spring时：
2. 首先会进行扫描，扫描得到所有的BeanDefinition对象，并存在一个Map中
3. 然后筛选出非懒加载的单例BeanDefinition进行创建Bean，对于多例Bean不需要在启动过程中去进行创建，对于多例Bean会在每次获取Bean时利用BeanDefinition去创建
4. 利用BeanDefinition创建Bean就是Bean的创建生命周期，这期间包括了合并BeanDefinition、推断构造方法、实例化、属性填充、初始化前、初始化、初始化后等步骤，其中AOP就是发生在初始化后这一步骤中
5. 单例Bean创建完了之后，Spring会发布一个容器启动事件
6. Spring启动结束
7. 在源码中会更复杂，比如源码中会提供一些模板方法，让子类来实现，比如源码中还涉及到一些BeanFactoryPostProcessor和BeanPostProcessor的注册，Spring的扫描就是通过BenaFactoryPostProcessor来实现的，依赖注入就是通过BeanPostProcessor来实现的
8. 在Spring启动过程中还会去处理@Import等注解

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-09/95934290955458.png?Expires=4897782349&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=Dg4GePnjClWWb%2FrZxf6iUVVdNiQ%3D)

---



## BeanFactory与FactoryBean的区别

---

### BeanFactory
BeanFactory，以Factory结尾，表示它是一个工厂类(接口)， 它负责生产和管理bean的一个工厂。在Spring中，BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现，如 `DefaultListableBeanFactory`、`XmlBeanFactory`、`ApplicationContext`。

包含以下功能：
* 提供Bean的生命周期管理
* 实现依赖注入
* 支持Bean的装配和延迟初始化

> BeanFactory是容器，管理所有Bean

---

### FactoryBean
FactoryBean 是一个特殊的 Bean，它是一个工厂对象，用于创建和管理其他 Bean 的实例。FactoryBean 接口定义了一种创建 Bean 的方式，它允许开发人员在 Bean 的创建过程中进行更多的自定义操作。通过实现 FactoryBean 接口，开发人员可以创建复杂的 Bean 实例，或者在 Bean 实例化之前进行一些额外的逻辑处理。  
它是实现了FactoryBean<T>接口的Bean，根据该Bean的ID从BeanFactory中获取的实际上是FactoryBean的getObject()返回的对象，而不是FactoryBean本身，如果要获取FactoryBean对象，请在id前面加一个&符号来获取。

FactoryBean在Spring中最为典型的一个应用就是用来创建AOP的代理对象。

* 复杂对象的初始化：如JDBC连接、第三方框架对象
* 动态代理对象创建：MyBatis中的Mapper接口代理对象就是通过FactoryBean创建的
* 条件化Bean创建：根据条件创建不同实现的Bean

> FactoryBean是特殊的Bean，用于创建复杂Bean

---

## Spring的三级缓存

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/119299005951958.png?Expires=4897855998&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=I30pxp0luRaMDTOCk4cCO2fjkBY%3D)


```java
/** Cache of singleton objects: bean name --> bean instance */
/** 一级缓存：用于存放完全初始化好的 bean **/
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);

/** Cache of early singleton objects: bean name --> bean instance */
/** 二级缓存：存放原始的 bean 对象（尚未填充属性），用于解决循环依赖 */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

/** Cache of singleton factories: bean name --> ObjectFactory */
/** 三级级缓存：存放 bean 工厂对象，用于解决循环依赖 */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);

/** Names of beans that are currently in creation. */
// 这个缓存也十分重要：它表示bean创建过程中都会在里面呆着~
// 它在Bean开始创建时放值，创建完成时会将其移出~
private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap<>(16));

/** Names of beans that have already been created at least once. */
// 当这个Bean被创建完成后，会标记为这个 注意：这里是set集合 不会重复
// 至少被创建了一次的  都会放进这里~~~~
private final Set<String> alreadyCreated = Collections.newSetFromMap(new ConcurrentHashMap<>(256));
```

### 二级缓存解决普通循环依赖

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/119433962905958.png?Expires=4897856133&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=tVO8o%2FPr9Kgi1IDmUkLop6%2FYQWQ%3D)

可以看到，二级缓存就能解决一个循环依赖问题，但是不能解决AOP代理的循环依赖问题，因为AOP后最终放进容器中的是代理对象而不是原对象。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/119750807687791.png?Expires=4897856449&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=d57bcOdrTaV2bUSMMXQSSm%2F3ZyQ%3D)

继续使用二级缓存就会出现问题。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/119680523184458.png?Expires=4897856379&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=e2I51JrgKujQyKATquPz%2F4ebhjY%3D)

---

### 三级缓存解决AOP循环依赖

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/119862569458708.png?Expires=4897856562&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=tAz56CTnWSy40J%2BHZDZ21%2BgdOEs%3D)

---

### 是否可以移除第三级缓存

去掉三级缓存之后，Bean 直接创建 earlySingletonObjects， 看着好像也可以。
如果有代理的时候，在 earlySingletonObjects 直接放代理对象就行了。
但是会导致一个问题：**在实例化阶段就得执行后置处理器**，判断有 AnnotationAwareAspectJAutoProxyCreator 并创建代理对象。

这么一想，是不是会对 Bean 的生命周期有影响。
同样，先创建 singletonFactory 的好处就是：在真正需要实例化的时候，再使用 singletonFactory.getObject() 获取 Bean 或者 Bean 的代理。相当于是延迟实例化。

---

### 是否可以移除第二级缓存

如果去掉了二级缓存，则需要直接在 singletonFactory.getObject() 阶段初始化完毕，并放到一级缓存中。

那有这么一种场景，B 和 C 都依赖了 A。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/120465777399000.png?Expires=4897857164&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=K9NGYoK1Ib%2BET%2Fy3iJu7VyJ5CDs%3D)

---

要知道在有代理的情况下 singletonFactory.getObject() 获取的是代理对象。
而多次调用 singletonFactory.getObject() 返回的代理对象是不同的，就会导致 B 和 C 依赖了不同的 A。

### 解决循环依赖条件

在 Spring 中，只有同时满足以下两点才能解决循环依赖的问题：

* **必须是单例**  
  依赖的 Bean 必须都是单例
  因为原型模式都需要创建新的对象，不能跟用以前的对象

* **不能全是构造器注入**  
  依赖注入的方式，必须不全是构造器注入，且 beanName 的字母顺序在前的不能是构造器注入
  
  > 在 Spring 中创建 Bean 分三步:
  > * 实例化，createBeanInstance，就是 new 了个对象
  > * 属性注入，populateBean， 就是 set 一些属性值
  > * 初始化，initializeBean，执行一些 aware 接口中的方法，initMethod，AOP代理等
  >
  > 如果全是构造器注入，比如A(B b)，那表明在 new 的时候，就需要得到 B，此时需要 new B 。但是 B 也是要在构造的时候注入 A ，即B(A a)，这时候 B 需要在一个 map 中找到不完整的 A ，发现找不到。
  > 为什么找不到？因为 A 还没 new 完呢，所以找到不完整的 A，因此如果全是构造器注入的话，那么 Spring 无法处理循环依赖。
  
  一个set注入，一个构造器注入能否成功？
  
  * 假设我们` A 是通过 set 注入 B，B 通过构造函数注入 A`，此时是成功的  
    我们来分析下：实例化 A 之后，可以在 map 中存入 A，开始为 A 进行属性注入，发现需要 B，此时 new B，发现构造器需要 A，此时从 map 中得到 A ，B 构造完毕。
    B 进行属性注入，初始化，然后 A 注入 B 完成属性注入，然后初始化 A。
    整个过程很顺利完成。
  * 假设 `A 是通过构造器注入 B，B 通过 set 注入 A`，此时是失败的  
    我们来分析下：实例化 A，发现构造函数需要 B， 此时去实例化 B。
    然后进行 B 的属性注入，从 map 里面找不到 A，因为 A 还没 new 成功，所以 B 也卡住了，然后就 失败
    看到这里，仔细思考的小伙伴可能会说，可以先实例化 B 啊，往 map 里面塞入不完整的 B，这样就能成功实例化 A 了啊。  
    但是
  > Spring 容器是按照字母序创建 Bean 的，A 的创建永远排在 B 前面

---

#### Spring中Bean的顺序
spring容器载入bean顺序是不确定的，在一定的范围内bean的加载顺序可以控制。
spring容器载入bean虽然顺序不确定，但遵循一定的规则：
* 按照字母顺序加载（同一文件夹下按照字母顺序；不同文件夹下，先按照文件夹命名的字母顺序加载）
* 不同的bean声明方式不同的加载时机，顺序总结：@ComponentScan > @Import > @Bean
  这里的ComponentScan指@ComponentScan及其子注解，Bean指的是@configuration + @bean
* Component及其子注解申明的bean是按照字母顺序加载的
* @configuration + @bean是按照定义的顺序依次加载的
* @import的顺序，就是bean的加载顺序
* 在xml中，通过<bean id="">方式声明的bean也是按照代码的编写顺序依次加载的
* 同一类中加载顺序：Constructor >> @Autowired >> @PostConstruct >> @Bean
* 同一类中加载顺序：静态变量 / 静态代码块 >> 构造代码块 >> 构造方法（需要特别注意的是静态代码块的执行并不是优先所有的bean加载，只是在同一个类中，静态代码块优先加载）

---

#### 更改加载顺序
1. 构造方法依赖 (推荐)

    ```java
    @Component
    public class CDemo1 {
        private String name = "cdemo 1";
    
        public CDemo1(CDemo2 cDemo2) {
            System.out.println(name);
        }
    }
    @Component
    public class CDemo2 {
        private String name = "cdemo 2";
    
        public CDemo2() {
            System.out.println(name);
        }
    }
    ```
   CDemo2在CDemo1之前被初始化。
2. 参数注入

    ```java
    @Bean
    public BeanA beanA(BeanB beanB) {
        System.out.println("bean A init");
        return new BeanA();
    }
    @Bean
    public BeanB beanB() {
        System.out.println("bean B init");
        return new BeanB();
    }
    ```
3. @DependsOn(“xxx”)

    ```java
    @Component("dependson02")
    public class Dependson02 {
    
        Dependson02() {
            System.out.println(" dependson02 Success ");
        }
    }
    @Component
    @DependsOn("dependson02")
    public class Dependson01 {
    
        Dependson01() {
            System.out.println("Dependson01 success");
        }
    }
    ```

4. BeanDefinitionRegistryPostProcessor接口  
    通过实现BeanDefinitionRegistryPostProcessor接口，在postProcessBeanDefinitionRegistry方法中通过BeanDefinitionRegistry获取到所有bean的注册信息，将bean保存到LinkedHashMap中，并从BeanDefinitionRegistry中删除，然后将保存的bean定义排序后，重新再注册到BeanDefinitionRegistry中，即可实现bean加载顺序的控制。
5. 执行顺序@Order  
   注解@Order或者接口Ordered的作用是定义Spring IOC容器中Bean的执行顺序的优先级，而不是定义Bean的加载顺序，Bean的加载顺序不受@Order或Ordered接口的影响，@Order不控制Spring初始化顺
    ```java
    @Component
    @Order(0)
    public class Test01 {
       ...
    }
    
    @Component
    @Order(1)
    public class Test02 {
       ...
    }
    
    @Component
    @Order(2)
    public class Test03 {
       ...
    }
    ```
6. 延迟注入@Lazy  
   在类A中使用 @Lazy 注解，将类A延迟加载，这样在启动应用程序时，Spring容器不会立即加载类A，而是在需要使用类A的时候才会进行加载。
    ```java
    @Component
    public class A {
        private final B b;
        public A(@Lazy B b) {
            this.b = b;
        }
        //...
    }
    
    @Component
    public class B {
        private final A a;
        public B(A a) {
            this.a = a;
        }
        //...
    }
    ```

---

## Spring中的事务是如何实现的
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

## Spring事务的传播机制
1. REQUIRED(默认)：如果当前没有事务，则自己新建一个事务，如果当前存在事务，则加入这个事务
2. SUPPORTS：当前存在事务，则加入当前事务，如果当前没有事务，就以非事务方法执行
3. MANDATORY：当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常。
4. REQUIRES_NEW：创建一个新事务，如果存在当前事务，则挂起该事务。
5. NOT_SUPPORTED：以非事务方式执行,如果当前存在事务，则挂起当前事务
6. NEVER：不使用事务，如果当前事务存在，则抛出异常
7. NESTED：如果当前事务存在，则在嵌套事务中执行，否则和REQUIRED的操作一样（开启一个事务）

---

## 哪些情况下会导致Spring事务失效
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

## 过滤器（Filter）和拦截器（Interceptor）

参考：<https://juejin.cn/post/7097781804827934733?share_token=bc432f68-aeb2-43c7-87cf-5e797984aba9>

---

### 过滤器（Filter）

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

4. 实现原理  
如果将函数（C++ 中的函数指针，Java 中的匿名函数、方法引用等）作为参数传递给主方法，那么这个函数就称为回调函数，主方法会在某一时刻调用回调函数。  
使用回调函数的好处是能够实现函数逻辑的解耦，主方法内可以定义通用的处理逻辑，部分特定的操作则交给回调函数来完成。例如 Java 中 Arrays 类的 sort(T[] a, Comparator<? super T> c) 方法允许我们传入一个比较器来自定义排序规则，这个比较器的 compare 方法就属于回调函数，sort 方法会在排序时调用 compare 方法。

---

### 拦截器（Interceptor）

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

### Filter 和 Interceptor 的区别

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

## Spring 框架中都用到了哪些设计模式
1. **简单工厂**：  
   BeanFactory：Spring的BeanFactory充当工厂，负责根据配置信息创建Bean实例。它是一种工厂模式的应用，根据指定的类名或ID创建Bean对象。
2. **工厂方法**：  
   FactoryBean：FactoryBean接口允许用户自定义Bean的创建逻辑，实现了工厂方法模式。开发人员可以使用FactoryBean来创建复杂的Bean实例。
3. **单例模式**：  
   Bean实例：Spring默认将Bean配置为单例，确保在容器中只有一个共享的实例，这有助于节省资源和提高性能。
4. **适配器模式**：  
   SpringMVC中的HandlerAdapter：SpringMVC的HandlerAdapter允许不同类型的处理器适配到处理器接口，以实现统一的处理器调用。这是适配器模式的应用。
5. **装饰器模式**：  
   BeanWrapper：Spring的BeanWrapper允许在不修改原始Bean类的情况下添加额外的功能，这是装饰器模式的实际应用。
6. **代理模式**：  
   AOP底层：Spring的AOP（面向切面编程）底层通过代理模式来实现切面功能，包括JDK动态代理和CGLIB代理。
7. **观察者模式**：  
   Spring的事件监听：Spring的事件监听机制是观察者模式的应用，它允许组件监听和响应特定类型的事件，实现了松耦合的组件通信。
8. **策略模式**：  
   excludeFilters、includeFilters：Spring允许使用策略模式来定义包扫描时的过滤策略，如在@ComponentScan注解中使用的excludeFilters和includeFilters。
9. **模板方法模式**：  
   Spring几乎所有的外接扩展：Spring框架的许多模块和外部扩展都采用模板方法模式，例如JdbcTemplate、HibernateTemplate等。
10. **责任链模式**：  
    AOP的方法调用：Spring AOP通过责任链模式实现通知（Advice）的调用，确保通知按顺序执行。