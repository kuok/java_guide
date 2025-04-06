## JAVA基础

---

### 基础

---

#### Object对象方法有哪些

---

* getClass() - 获取运行是的Class对象，属于反射的内一套，获取运行是的一些数据。
* hashCode() - 返回对象的hash值。目的是为了更好的支持哈希表，比如基于Java中的HashMap使用
* equals() - 比较两个对象是否相等，默认 ==
* clone() - 创建对象的副本。深拷贝和浅拷贝的内容
  * 默认是浅拷贝，将当前对象复制一份，其中的基本数据类型直接复用值，引用数据类型是复用地址值。
  * 深拷贝，将当前对象复制一份，其中的基本数据类型直接复用值，引用数据类型会重新的创建一个，不会复制之前的地址。 深拷贝要自己编写克隆内部的引用类型对象。
* toString - 返回一个以字符串形式表示当前对象的信息。
* wait - 当某个线程持有当前对象锁时，可以执行对象锁.wait，将持有对象锁的线程挂起等待。
* notify - 当某个线程持有当前对象锁时，可以执行对象锁.notify，唤醒之前基于wait挂起的一个线程。
* notifyAll - 当某个线程持有当前对象锁时，可以执行对象锁.notifyAll方法，唤醒之前所有基于wait挂起的线程。
* finalize - 当触发垃圾回收时，如果当前对象无法基于可达性分析定位到，就会被垃圾回收器回收掉，在回收之前，如果这个对象重写了finalize，那就会触发finalize方法执行。可以执行一些其他的清理工作。（Finalize在JVM中，他不保证一定执行，他用的守护线程）

---

#### Java类的加载顺序

* 类加载过程
  * 加载： 先找到字节码文件（.class文件），加载到JVM内存中的方法区里。 然后在内存中的体现就是一个Class对象。
  * 验证： 验证加载到内存里的.class文件是否被篡改过，确认没有安全问题，以及符合JVM规范。
  * 准备： 为类中的一些变量分配内存空间，并且设置一下默认值。
  * 解析： 将常量池内的符号引用转为直接引用。
  * 符号引用：符号引用是一种泛指
  * 直接引用：直接指向的内库的具体位置，直接就是内存偏移量。后面调用会更快。
  * 初始化：对所有静态变量复制，执行静态代码块，初始化好父类~~
  * 前面走完，到这，这个.class就可以在Java程序中使用了，new一个对象，类名.静态方法都可以了

* 双亲委派
  * 他其实就是加载这个过程的细节，需要先掌握一下Java中默认的三种类加载器
    * BootstrapClassLoader：负责加载jdk/jre/lib/rt.jar
    * ExtensionClassLoader：负责加载jdk/jre/lib/ext目录下的jar文件
    * ApplicationClassLoader：负责加载classpath目录下的各种class文件。 所谓的classpath，其实就是编译后的classes目录。
    * 自定义的，你自己去继承ClassLoader，重新他的方法，指定你要加载的位置。
  * 双亲委派的过程。当需要用到某个class文件时，撇掉自定义类加载器，他会按照这个方式去加载
    * 先调度AppClassLoader，先查看AppClassLoader加载过么？没加载过，往上问。
    * 问到ExtClassLoader，先查看ExtClassLoader加载过么？没加载过，网上问。
    * 最终问到BootstrapClassLoader，先查看BootstrapClassLoader加载过么？没加载过，尝试加载！如果rt.jar里没有这个.class文件可以加载，往下分配。
    * 分配到ExtClassLoader，他去尝试在ext目录下去加载，如果也没加载到，往下分配。
    * 最终分配到AppClassLoader，他尝试去classpath目录下找这个.class文件加载。
    * 如果没找到，也没加载到，抛一个异常，ClassNotFoundException。
  * 双亲委派解决了什么问题，搞的这么麻烦？？
    * 防止类的重复加载……
    * 防止你破坏JDK的结构……

---

#### 判断一个对象是否可以被回收

* 引用计数法  
   引用计数算法（ReferenceCounting）比较简单，对每个对象保存一个整型的引用计数器属性。用于记录对象被引用的情况。  
   对于一个对象A，只要有任何一个对象引用了A，则A的引用计数器就加1；当引用失效时，引用计数器就减1。只要对象A的引用计数器的值为0，即表示对象A不可能再被使用，可进行回收
  * 优点：
    * 实现简单，垃圾对象便于辨识；
    * 判定效率高，回收没有延退性。
  * 缺点：
    * 它需要单独的字段存储计数器，这样的做法增加了存储空间的开销
    * 每次赋值都需要更新计数器，伴随着加法和减法操作，这增加了时间开销
    * 引用计数器有一个严重的问题，即无法处理循环引用的情况。这是一条致命缺陷，导致在Java的垃圾回收器中没有使用这类算法。
* 可达性分析算法  
   相对于引用计数算法而言，可达性分析算法不仅同样具备实现简单和执行高效等特点，更重要的是该算法可以有效地解决在引用计数算法中循环引用的问题，防止内存泄漏的发生，这里的可达性分析就是Java、c#选择的。这种类型的垃圾收集通常也叫作追踪性垃圾收集
  * 可达性分析算法是以根对象集合（GCRoots就是一组必须活跃的引用）为起始点，按照从上至下的方式搜索被根对象集合所连接的目标对象是否可达
  * 使用可达性分析算法后，内存中的存活对象都会被根对象集合直接或间接连接着，搜索所走过的路径称为引用链
  * 如果目标对象没有任何引用链相连，则是不可达的，就意味着该对象己经死亡，可以标记为垃圾对象
  * 在可达性分析算法中，只有能够被根对象集合直接或者问接连接的对象才是存活对象

> **GC Roots**
>
> * 虚拟机栈中引用的对象，比如：各个线程被调用方法中使用到的参数、局部变量等
> * 本地方法栈内JNI（通常说的本地方法）引用的对象
> * 方法区中类静态属性引用的对象，比如：Java类的引用类型静态变量
> * 方法区中常量引用的对象，比如：字符串常量池（StringTable）里的引用
> * 所有被同步锁synchronized持有的对象
> * Java虚拟机内部的引用，基本数据类型对应的class对象，一些常驻的异常对象（如： NullPointerException、OutofMemoryError），系统类加载器
> * 反映java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等
> * 除了这些固定的GCRoots集合以外，根据用户所选用的垃圾收集器以及当前回收的内存区域不同，还可以有其他对象“临时性”地加入，共同构成完 整GC Roots集合。比如：分代收集和局部回收（PartialGC）
> * 如果只针对了java堆中的某一块区域进行垃圾回收（比如：典型的只针对新生代），必须考虑到内存区域是虚拟机自己的实现细节，更不是孤立封闭的，这个区域的对象完全有可能被其他区域的对象所引用，这时候就需要一并将关联的区域对象也加入GCRoots集合中去考虑，才能保证可达性分析的准确性。

---

#### OOM问题

1. 堆内存OOM  
   不停增加对象。

   ```java
   @Test
   public void test01() {
       List<OOMTests> list = Lists.newArrayList();
       while (true) {
           list.add(new OOMTests());
       }
   }
   ```

   ```java
   java.lang.OutOfMemoryError: Java heap space
   ```

2. 栈内存OOM  
   不停创建线程。

   ```java
   public class StackOOMTest {
       public static void main(String[] args) {
           while (true) {
               new Thread().start();
           }
       }
   }
   ```

   ```java
   java.lang.OutOfMemoryError: unable to create new native thread
   ```

3. 栈内存溢出  
   方法递归调用

   ```java
   @Test
   public void test03() {
       recursiveMethod();
   }
   
   public static void recursiveMethod() {
       // 递归调用自身
       recursiveMethod();
   }
   ```

   ```java
   java.lang.StackOverflowError
   ```

4. 直接内存OOM  
   操作大文件

   ```java
   private static final int BUFFER = 1024 * 1024 * 20;
   
   @Test
   public void test04() {
       ArrayList<ByteBuffer> list = new ArrayList<>();
       int count = 0;
       try {
           while (true) {
               ByteBuffer byteBuffer = ByteBuffer.allocateDirect(BUFFER);
               list.add(byteBuffer);
               count++;
               try {
                   Thread.sleep(100);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
           }
       } finally {
           System.out.println(count);
       }
   }
   ```

   ```java
   java.lang.OutOfMemoryError: Direct buffer memory
   ```

5. GC OOM  
   JVM在GC时，对象过多，导致内存溢出

   ```java
   public class GCOverheadOOM {
       public static void main(String[] args) {
           ExecutorService executor = Executors.newFixedThreadPool(5);
           for (int i = 0; i < Integer.MAX_VALUE; i++) {
               executor.execute(() -> {
                   try {
                       Thread.sleep(10000);
                   } catch (InterruptedException e) {
                   }
               });
           }
       }
   }
   ```

   ```java
   java.lang.OutOfMemoryError: GC overhead limit exceeded
   ```

6. 元空间OOM  
   Metaspace是方法区在HotSpot中的实现，这个问题一般是由于加载到内存中的类太多，或者类的体积太大导致的。

   ```java
   public class MetaspaceOOMTest {
       static class OOM {
       }
   
       public static void main(String[] args) {
           int i = 0;
           try {
               while (true) {
                   i++;
                   Enhancer enhancer = new Enhancer();
                   enhancer.setSuperclass(OOM.class);
                   enhancer.setUseCache(false);
                   enhancer.setCallback(new MethodInterceptor() {
                       @Override
                       public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                           return methodProxy.invokeSuper(o, args);
                       }
                   });
                   enhancer.create();
               }
           } catch (Throwable e) {
               e.printStackTrace();
           }
       }
   }
   ```

   ```java
   java.lang.OutOfMemoryError: Metaspace
   ```

---

#### BIO、NIO、AIO

* BIO（Blocking I/O）是最传统的I/O模型，它采用同步阻塞方式处理I/O操作。在BIO模型中，当一个I/O操作发生时，线程会被阻塞，直到I/O操作完成才能继续执行。这种模型的缺点是性能较低，因为大量的线程会被阻塞，导致资源浪费。
* NIO（Non-blocking I/O）是一种更高效的I/O模型，它采用事件驱动的方式处理I/O操作。在NIO模型中，线程可以注册对I/O事件的关注，并在事件发生时得到通知，从而可以继续执行其他任务。这种模型可以大大减少线程的阻塞，提高系统的并发性能。
* AIO（Asynchronous I/O）是在NIO的基础上进一步发展的一种I/O模型，它采用异步非阻塞方式处理I/O操作。在AIO模型中，I/O操作会在后台进行，当操作完成时会触发回调通知。这种模型可以进一步提高系统的并发性能，特别适用于处理大量并发的I/O操作。

---

#### 反射

---


### Lambda表达式

1. 集合遍历

   ```java
   List list = Arrays.asList("apple", "banana", "orange");
   for (String fruit : list) {
       System.out.println(fruit);
   }
   ```

   ```java
   List list = Arrays.asList("apple", "banana", "orange");
   list.forEach(fruit -> System.out.println(fruit));
   ```

2. 排序

   ```java
   List list = Arrays.asList("apple", "banana", "orange");
   Collections.sort(list, new Comparator() {
       public int compare(String s1, String s2) {
           return s1.compareTo(s2);
       }
   });
   ```

   ```java
   List list = Arrays.asList("apple", "banana", "orange");
   Collections.sort(list, (s1, s2) -> s1.compareTo(s2));
   ```

3. 过滤

   ```java
   List list = Arrays.asList("apple", "banana", "orange");
   List filteredList = new ArrayList();
   for (String fruit : list) {
       if (fruit.startsWith("a")) {
           filteredList.add(fruit);
       }
   }
   ```

   ```java
   List list = Arrays.asList("apple", "banana", "orange");
   List filteredList = list.stream().filter(fruit -> fruit.startsWith("a")).collect(Collectors.toList());
   ```

4. 映射

   ```java
   List list = Arrays.asList("apple", "banana", "orange");
   List lengths = new ArrayList();
   for (String fruit : list) {
       lengths.add(fruit.length());
   }
   ```

   ```java
   List list = Arrays.asList("apple", "banana", "orange");
   List lengths = list.stream().map(fruit -> fruit.length())
                .collect(Collectors.toList());
   ```

5. 归约

   ```java
   List list = Arrays.asList(1, 2, 3, 4, 5);
   int sum = 0;
   for (int i : list) {
       sum += i;
   }
   ```

   ```java
   List list = Arrays.asList(1, 2, 3, 4, 5);
   int sum = list.stream().reduce(0, (a, b) -> a + b);
   ```

6. 分组

   ```java
   List list = Arrays.asList("apple", "banana", "orange");
   Map<Integer, List> grouped = new HashMap<Integer, List>();
   for (String fruit : list) {
       int length = fruit.length();
       if (!grouped.containsKey(length)) {
           grouped.put(length, new ArrayList());
       }
       grouped.get(length).add(fruit);
   }
   ```

   ```java
   List list = Arrays.asList("apple", "banana", "orange");
   Map<Integer, List> grouped = list.stream()
                   .collect(Collectors.groupingBy(fruit -> fruit.length()));
   ```

7. 函数式接口

   ```java
   public interface MyInterface {
       public void doSomething(String input); 
   }
   
   MyInterface myObject = new MyInterface() {
       public void doSomething(String input) {
           System.out.println(input);
       }
   };
   
   myObject.doSomething("Hello World");
   ```

   ```java
   MyInterface myObject = input -> System.out.println(input);
   myObject.doSomething("Hello World");
   ```

8. 线程的创建

   ```java
   Thread thread = new Thread(new Runnable() {
       public void run() {
         System.out.println("Thread is running.");
       }
   });
   thread.start();
   ```

   ```java
   Thread thread = new Thread(() -> System.out.println("Thread is running."));
   thread.start();
   ```

9. Optional

   ```java
   String str = "Hello World";
   if (str != null) {
       System.out.println(str.toUpperCase());
   ```

   ```java
   Optional str = Optional.ofNullable("Hello World");
   str.map(String::toUpperCase)
       .ifPresent(System.out::println);
   ```

10. Stream操作

   ```java
   List list = Arrays.asList("apple", "banana", "orange");
   List filteredList = new ArrayList();
   for (String fruit : list) {
       if (fruit.startsWith("a")) {
           filteredList.add(fruit.toUpperCase());
       }
   }
   Collections.sort(filteredList);
   ```

   ```java
   List list = Arrays.asList("apple", "banana", "orange");
   List filteredList = list.stream().filter(fruit -> fruit.startsWith("a"))
                           .map(String::toUpperCase)
                           .sorted()
                           .collect(Collectors.toList());
   ```

---

### List

---

#### CopyOnWriteArrayList

Copy-On-Write它是一种在计算机科学中常见的优化技术，主要应用于需要频繁读取但很少修改的数据结构上。  
简单的说就是在计算机中就是当你想要对一块内存进行修改时，我们不在原有内存块中进行写操作，而是将内存拷贝一份，在新的内存中进行写操作，写完之后呢，就将指向原来内存指针指向新的内存，原来的内存就可以被回收掉了！  
从JDK1.5开始Java并发包里提供了两个使用CopyOnWrite机制实现的并发容器，它们是CopyOnWriteArrayList和CopyOnWriteArraySet。CopyOnWrite容器非常有用，可以在非常多的并发场景中使用到。

1. 原理：  
在写操作(add、remove等)时，不直接对原数据进行修改，而是先将原数据复制一份，然后在新复制的数据上执行写操作，最后将原数据引用指向新数据。这样做的好处是读操作(get、iterator等)可以不加锁，因为读取的数据始终是不变的。  
这种写时复制的机制保证了读操作的线程安全性，但是会牺牲一些写操作的性能，因为每次修改都需要复制一份数组。因此，适合读远多于写的场合。
2. 优点：
   * **线程安全**。CopyOnWriteArrayList是线程安全的，由于写操作对原数据进行复制，因此写操作不会影响读操作，读操作可以不加锁，降低了并发冲突的概率。
   * **不会抛出ConcurrentModificationException异常**。由于读操作遍历的是不变的数组副本，因此不会抛出ConcurrentModificationException异常。
3. 缺点：
   * **写操作性能较低**。由于每一次写操作都需要将元素复制一份，因此写操作的性能较低。
   * **内存占用增加**。由于每次写操作都需要创建一个新的数组副本，因此内存占用会增加，特别是当集合中有大量数据时，内存占用较高。
   * **数据一致性问题**。由于读操作遍历的是不变的数组副本，因此在对数组执行写操作期间，读操作可能读取到旧的数组数据，这就涉及到数据一致性问题。
4. 适用场景
   * **读多写少**。因为写的时候会复制新集合
   * **集合不大**。因为写的时候会复制新集合
   * **实时性要求不高**。因为有可能会读取到旧的集合数据

---

### HashMap

---

#### HashMap 的底层数据结构

在 JDK 1.7 中 HashMap 是以「数组加链表」的形式组成的，JDK 1.8 之后新增了「红黑树」的组成结构，「**当链表长度大于 8 并且 hash 桶的容量大于 64 时，链表结构会转换成红黑树结构**」。所以，它的组成结构如下图所示：
![map数据结构](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-13/16760483157899.png?Expires=4895452013&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=9KyoYe%2F0gr6P3nOErU7%2BD%2BrTMeQ%3D)
HashMap 中数组的每一个元素又称为哈希桶，也就是 key-value 这样的实例。在 Java7 中叫 Entry，Java8 中叫 Node。

---

#### HashMap 有哪些属性

```java
// HashMap 初始化长度
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

// HashMap 最大长度
static final int MAXIMUM_CAPACITY = 1 << 30; // 1073741824

// 默认的加载因子 (扩容因子)
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 当链表长度大于此值且数组长度大于 64 时，会从链表转成红黑树
static final int TREEIFY_THRESHOLD = 8;

// 转换链表的临界值，当元素小于此值时，会将红黑树结构转换成链表结构
static final int UNTREEIFY_THRESHOLD = 6;

// 最小树容量
static final int MIN_TREEIFY_CAPACITY = 64;
```

##### 树化是 8，退树化是 6

红黑树平均查找长度为 log (n)，长度为 8 时，查找长度为 3，而链表平均查找长度为 n/2；也就是 8 除以 2；查找长度链表大于树，转化为树，效率更高。  
当为 6 时，树：2.6；链表：3。链表 > 树。这时理应也还是树化，但是树化需要时间，为了这点效率牺牲时间是不划算的。

---

### BigDecimal

---

#### 浮点数

注意浮点数转BigDecimal

```java
@Test
public void bigDecimalDemo2(){
    BigDecimal bigDecimal1 = new BigDecimal(0.01);
    BigDecimal bigDecimal2 = BigDecimal.valueOf(0.01);
    System.out.println("bigDecimal1 = " + bigDecimal1);
    System.out.println("bigDecimal2 = " + bigDecimal2);
}
```

结果

```java
bigDecimal1 = 0.01000000000000000020816681711721685132943093776702880859375
bigDecimal2 = 0.01
```

---


