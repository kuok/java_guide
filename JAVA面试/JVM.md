# JVM

---

## JVM架构

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-05-24/510065614364958.png?Expires=4901668292&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=RZGx%2BGgjrQVh%2BBTO7vZBsNDGBUM%3D)


---

## 线上内存溢出问题排查

1. top命令

   使用top命令查看机器内存占用情况，shift+m按内存排序
2. jstat命令

   使用jstat工具，对指定java进程（20886就是进程id，通过ps -aux | grep java命令就能找到），按照指定间隔，看一下统计信息，这里会每隔一段时间显示一下，包括新生代的两个S0、s1区、Eden区，以及老年代的内存使用率，还有young gc以及full gc的次数。
    ```bash
    jstat -gcutil 20886 1000 10
    ```

3. jmap命令
   
   执行jmap -histo pid可以打印出当前堆中所有每个类的实例数量和内存占用
   ```bash
   jmap -histo pid
   ```

4. jmap dump命令
   
   把当前堆内存的快照转储到dumpfile_jmap.hprof文件中，然后可以对内存快照进行分析
   ```bash
   jmap -dump:format=b,file=文件名 [pid]
   ```

线上jvm必须配置-XX:+HeapDumpOnOutOfMemoryError，-XX:HeapDumpPath=/path/heap/dump。因为这样就是说OOM的时候自动导出一份内存快照，你就可以分析发生OOM时的内存快照了，到底是哪里出现的问题。
