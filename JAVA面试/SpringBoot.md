# SpringBoot

---

## 自动配置原理

---

### 自动配置流程

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/129651993742875.png?Expires=4897869980&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=WQOfg6tTdk7CzrvZx49RUTWaEQw%3D)

1. 导入starter
2. 依赖导入autoconfigure
3. 寻找类路径下 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`文件

   ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/130277704737291.png?Expires=4897870605&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=vAKkhZd2WNUDj6eRu5gi%2FhKeH5E%3D)

4. 启动，加载所有 自动配置类 xxxAutoConfiguration
   * 给容器中配置功能组件
   * 组件参数绑定到 属性类中。xxxProperties
   * 属性类和配置文件前缀项绑定
   * @Contional派生的条件注解进行判断是否组件生效

   ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/130809484255041.png?Expires=4897871137&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=F6EcPDWG8P3nnPA1hWUh9ruALoM%3D)

   ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/130828339367125.png?Expires=4897871156&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=r4oE5RM9%2FTGmtogLXwG2STSbT3Q%3D)

5. 效果：
   * 修改配置文件，修改底层参数
   * 所有场景自动配置好直接使用
   * 可以注入SpringBoot配置好的组件随时使用

---

### SPI机制
Java中的SPI（Service Provider Interface）是一种软件设计模式，用于在应用程序中动态地发现和加载组件。SPI的思想是，定义一个接口或抽象类，然后通过在classpath中定义实现该接口的类来实现对组件的动态发现和加载。

在SpringBoot中，`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`就是这个机制的一种实现。

---

### @SpringBootApplication

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/131425217448958.png?Expires=4897873274&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=DKBfTEyD8HPWsPu%2BYm47Qu%2BdyDA%3D)

---

#### @SpringBootConfiguration
@SpringBootConfiguration继承自@Configuration，二者功能也一致，标注当前类是配置类， 并会将当前类内声明的一个或多个以@Bean注解标记的方法的实例纳入到spring容器中，并且实例名就是方法名。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/131577186553166.png?Expires=4897873426&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=%2FkV0Abi8ZD5utqPNSyXNjhjeJMc%3D)

---

#### @EnableAutoConfiguration

---

##### @AutoConfigurationPackage
AutoConfigurationPackage注解的作用是将 添加该注解的类所在的package 作为 自动配置package 进行管理。

也就是只要代码当中添加@AutoConfigurationPackage注解，就会将注解所在的包名添加到basePackages集合当中。

通过@Import机制，@AutoConfigurationPackage注解将AutoConfigurationPackageRegistrar类导入到Spring容器中，从而实现自动配置的包扫描功能。

---

##### @Import(AutoConfigurationImportSelector.class)
AutoConfigurationImportSelector这个类这个类的核心方法是selectImports方法，实现ImportSelector接口。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/136734244335875.png?Expires=4897878583&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=n5atqNbPaYFyVtMBl3VTks8Tsw0%3D)

selectImports里面可一步步获取到各配置类的路径注入到容器中。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/136876611877375.png?Expires=4897878725&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=X3iGokeBFFjKs6cbySqol9C1%2FoU%3D)

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/137050984287458.png?Expires=4897878900&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=qOtqXGnRAWpA2kPLlBqM4D2kTFo%3D)

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/137184418808916.png?Expires=4897879033&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=zqeg8e6jmjF3JEc0qRRcHEyOIZg%3D)

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/137256709304875.png?Expires=4897879105&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=Hqq7XohoXsfet3N1vg4KyM37MJY%3D)

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/137330697356666.png?Expires=4897879179&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=pyDsfrTo%2FxCUYOWEkaB7G6x8%2Bt4%3D)

方法基于我们在pom.xml文件中配置的jar包和组件进行导入。所以方法返回的是一个Class全路径的String数组，返回的Class会被Spring容器管理。

由此将`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`文件中的自动配置类导入进容器中。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/137379520710916.png?Expires=4897879228&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=U4T1teTQm2sUI7kM2SxFxNyu62I%3D)

---

#### @ComponentScan
@ComponentScan的功能其实就是自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IoC容器中。
我们可以通过basePackages等属性来细粒度的定制@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描。

---

## 自定义starter
场景：抽取聊天机器人场景，它可以打招呼。  
效果：任何项目导入此starter都具有打招呼功能，并且问候语中的人名需要可以在配置文件中修改

1. 创建自定义starter项目，引入spring-boot-starter基础依赖
   ```xml
   <dependencies>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter</artifactId>
      </dependency>
      
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
      
      <dependency>
         <groupId>org.projectlombok</groupId>
         <artifactId>lombok</artifactId>
         <optional>true</optional>
      </dependency>
      
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-test</artifactId>
         <scope>test</scope>
      </dependency>
      
      <!--        导入配置处理器，配置文件自定义的properties配置都会有提示-->
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-configuration-processor</artifactId>
         <optional>true</optional>
      </dependency>
   </dependencies>
   ```
2. 编写模块功能，引入模块所有需要的依赖。
    ```java
    @RestController
    public class RobotController {
    
        @Autowired
        RobotService robotService;
    
        @GetMapping("/robot/hello")
        public String sayHello() {
            String s = robotService.sayHello();
            return s;
        }
    }
    ```
    ```java
    @Service
    public class RobotService {
    
        @Autowired
        RobotProperties robotProperties;
    
        public String sayHello(){
            return "你好：名字：【"+robotProperties.getName()+"】;年龄：【"+robotProperties.getAge()+"】";
        }
    }
    ```
    ```java
    @ConfigurationProperties(prefix = "robot")  //此属性类和配置文件指定前缀绑定
    @Component
    @Data
    public class RobotProperties {
    
        private String name;
        private String age;
        private String email;
    }
    ```
3. 编写xxxAutoConfiguration自动配置类，帮其他项目导入这个模块需要的所有组件
    ```java
    //给容器中导入Robot功能要用的所有组件
    @Import({RobotProperties.class, RobotService.class})
    @Configuration
    public class RobotAutoConfiguration {
        @Bean //把组件导入到容器中
        public RobotController robotController() {
            return new RobotController();
        }
    }
    ```
4. 使用SpringBoot的SPI机制，编写配置文件`reources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`指定启动需要加载的自动配置
    ```text
    com.atguigu.boot3.starter.robot.RobotAutoConfiguration
    ```

    或者使用@EnableXxx机制

    ```java
    @Retention(RetentionPolicy.RUNTIME)
    @Target({ElementType.TYPE})
    @Documented
    @Import(RobotAutoConfiguration.class)
    public @interface EnableRobot {
    
    }
    ```
5. 其他项目引入即可使用

---

## SpringBoot的配置优先级
Spring Boot 允许将配置外部化，以便可以在不同的环境中使用相同的应用程序代码。

我们可以使用各种外部配置源，包括Java Properties文件、YAML文件、环境变量和命令行参数。
@Value可以获取值，也可以用@ConfigurationProperties将所有属性绑定到java object中

以下是 SpringBoot 属性源加载顺序。(后面覆盖前面)

1. `默认属性`（通过SpringApplication.setDefaultProperties指定的） 
2. @PropertySource指定加载的配置（需要写在@Configuration类上才可生效）
3. `配置文件`（application.properties/yml等）
    1. jar 包内的application.properties/yml
    2. jar 包内的application-{profile}.properties/yml
    3. jar 包外的application.properties/yml
    4. jar 包外的application-{profile}.properties/yml
4. RandomValuePropertySource支持的random.*配置（如：@Value("${random.int}")）
5. OS 环境变量
6. Java 系统属性（System.getProperties()）
7. JNDI 属性（来自java:comp/env）
8. ServletContext 初始化参数
9. ServletConfig 初始化参数
10. SPRING_APPLICATION_JSON属性（内置在环境变量或系统属性中的 JSON）
11. `命令行参数`
12. 测试属性。(@SpringBootTest进行测试时指定的属性)
13. 测试类@TestPropertySource注解
14. Devtools 设置的全局属性。($HOME/.config/spring-boot)

> `命令行` > `配置文件` > `springapplication配置`

> 配置文件：`包外` > `包内`； 同级：`profile配置` > `application配置`

> 所有参数均可由命令行传入，使用--参数项=参数值，将会被添加到环境变量中，并优先于配置文件。
> 比如java -jar app.jar --name="Spring",可以使用@Value("${name}")获取

---

## spring.factories文件的作用

spring.factories 文件是 Spring 框架中的一个机制，用于实现在类路径下自动发现和加载扩展点的功能。这个文件的作用类似于 Java 的服务提供者接口（Service Provider Interface，SPI）机制，它允许第三方库或模块在应用程序中注册自己的实现，从而扩展或定制 Spring 框架的行为。

1. 扩展点的自动发现： Spring 框架内部会在类路径下搜索并加载 spring.factories 文件，并根据文件中的配置来自动发现扩展点。这些扩展点可以是自定义的类、接口或配置类，用于在 Spring 应用程序中提供额外的功能、特性或行为。
2. 解耦和可插拔性： spring.factories 文件的作用类似于插件机制，它允许你将自定义的实现或功能集成到 Spring 框架中，而不需要显式地修改 Spring 的源代码。这提高了代码的解耦性和可维护性，使得应用程序更具有可插拔性。
3. 多模块项目的模块发现： 如果你的项目是一个多模块项目，每个模块都可以有自己的 spring.factories 文件。这样，每个模块都可以在应用程序中注册自己的扩展点，实现模块之间的松耦合和灵活性。
4. 实现框架的定制和扩展： Spring 框架本身也使用了 spring.factories 文件来实现一些定制和扩展。这意味着你可以通过自定义的 spring.factories 文件来覆盖或替代 Spring 框架默认的行为，以满足特定的需求。
