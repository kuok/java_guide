## SpringBoot

---

### 自动配置原理

---

#### 自动配置流程

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

#### SPI机制
Java中的SPI（Service Provider Interface）是一种软件设计模式，用于在应用程序中动态地发现和加载组件。SPI的思想是，定义一个接口或抽象类，然后通过在classpath中定义实现该接口的类来实现对组件的动态发现和加载。

在SpringBoot中，`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`就是这个机制的一种实现。

---

#### @SpringBootApplication

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/131425217448958.png?Expires=4897873274&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=DKBfTEyD8HPWsPu%2BYm47Qu%2BdyDA%3D)

---

##### @SpringBootConfiguration
@SpringBootConfiguration继承自@Configuration，二者功能也一致，标注当前类是配置类， 并会将当前类内声明的一个或多个以@Bean注解标记的方法的实例纳入到spring容器中，并且实例名就是方法名。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-10/131577186553166.png?Expires=4897873426&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=%2FkV0Abi8ZD5utqPNSyXNjhjeJMc%3D)

---

##### @EnableAutoConfiguration

---

###### @AutoConfigurationPackage
AutoConfigurationPackage注解的作用是将 添加该注解的类所在的package 作为 自动配置package 进行管理。

也就是只要代码当中添加@AutoConfigurationPackage注解，就会将注解所在的包名添加到basePackages集合当中。

通过@Import机制，@AutoConfigurationPackage注解将AutoConfigurationPackageRegistrar类导入到Spring容器中，从而实现自动配置的包扫描功能。

---

###### @Import(AutoConfigurationImportSelector.class)
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

##### @ComponentScan
@ComponentScan的功能其实就是自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IoC容器中。
我们可以通过basePackages等属性来细粒度的定制@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描。

---

### 自定义starter













