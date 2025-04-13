# 我为什么买了Jetbrains的全家桶

晚饭后打开idea准备搞点小东西，于是idea启动，突然出现一个弹窗：idea崩溃！

奇怪，我也没做什么啊，为什么idea会崩溃？

于是打开崩溃报告看看，看不出什么特殊的，最关键的大概是这个

```text
# No core dump will be written. Core dumps have been disabled. To enable core dumping, try "ulimit -c unlimited" before starting Java again
#
# If you would like to submit a bug report, please visit:
#   https://youtrack.jetbrains.com/issues/JBR
# The crash happened outside the Java Virtual Machine in native code.
# See problematic frame for where to report the bug.
#
```

可能是内存不够的问题，好，先清理一下Mac，打开------继续崩溃。

拿着关键字去搜索一下，发现都是说JVM的问题，确实日志也是JAVA_开头的，是不是我的JAVA版本太新了，换成1.8的试试，打开------继续崩溃。

再去晚上继续搜索，每一篇都仔细看一下，找不到跟我这个情况相似的啊。

那就在回去看看崩溃报告

```text
Periodic native trim disabled

JNI global refs:
JNI global refs: 17, weak refs: 65

JNI global refs memory usage: 835, weak refs: 1465

Process memory usage:
Resident Set Size: 184288K (1% of 16777216K total physical memory with 412176K free physical memory)

OOME stack traces (most recent first):
Classloader memory used:
Loader com.intellij.util.lang.PathClassLoader                                          : 9757K
Loader bootstrap                                                                       : 5760K
Loader jdk.internal.loader.ClassLoaders$AppClassLoader                                 : 309K
Loader jdk.internal.loader.ClassLoaders$PlatformClassLoader                            : 47352B
```

大概是内存的问题，无法分配内存给idea，但是我的可用内存还有好几G啊，不至于打不开idea吧。

突然想起这个idea大概下载了一个月左右，之前用的盗版激活码，后来才买了正版激活码，正好是一个月的试用期，会不会跟这个有什么关系？

于是打开了Jetbrains家的另一个不需要许可证的WebStorm，发现也崩溃。那就了解了不是我Mac的问题，大概率是Jetbrain的问题了，可能是试用期到了，他执行了什么验证，发现了盗版验证码，然后发生了什么冲突？

再去google和StackOverFlow上面搜索看下，还是找不到什么关键文章。

这时已经三个小时过去了，有点绝望的想到了重置系统。

没办法，上次想到在youtrack上面用中文提了个issue，也是有人回答的，说明JetBrain是有中文的客服人员的，那就去上面再提个issue，明天再看看吧。

> https://youtrack.jetbrains.com/issue/IDEA-370754/macbook-airidea

好，issue提完，就看看Jetbrains的人看看有没有什么解决办法吧。

---

**然后，两秒钟后，网页上弹出消息这个issue已解决？？？**

有点生气，难道我用中文提的issue你们根本就不管的？

然后点进去，是这样子回复的

```text
YouTrack Workflow Bot
Commented about 2 second ago

@kamphirundo thank you for this report. Because JBR-8422 looks very similar, we've marked this ticket as a duplicate and you'll automatically receive updates in that one from now on. If you think your issue is different from the one we've linked, please leave a comment below.
```

有点疑惑，为什么Workflow Bot把我的issue标记为重复了，好，点进去看看。

> https://youtrack.jetbrains.com/issue/JBR-8422/IntelliJ-based-apps-and-Fleet-crashes-on-macOS-15.4-Beta-3-developer-beta

狂喜啊，一模一样的崩溃报告，有救了！

> 大概意思就是MacOS升级到15.4后，idea的jre有个权限问题，不能执行了，所以启动不了。

然后解决办法大概就是把idea升级到2025.1 Beta version或者在2024的版本上面下个jar，加入到idea.jdk文件夹里面。

那我就直接升级了，然后，打开了，完美解决。。。

---

然后我再回来看这个Bot，就是震惊了，这个bot能精准分析我用中文写的issue详情，上传的log文件，然后准确找到已有的issue并关联，以前一直对AI的多模态不了解，现在算是知道了这个完美的应用场景。

Jetbrains的AI已经应用到了这个地步了吗？

2秒钟，Jetbrains的AI就能分析一篇文章的中心意思，然后找出一篇相似的文章，直接回复给客户。要是换成人类呢，大概明天早上需要有个客服人员看看，然后转交给技术人员，然后技术人员再看看，要是见过这个问题还好，能直接回复，要是没见过，岂不是要再跟我来回的沟通，复现这个问题再解决？
这样一来，耗费的时间起码要以周来计算吧。如果这个AI能广泛应用，全世界的客服人员都可以解放了。

本来有点犹豫要不要买全家桶，现在好了，直接下单吧！

现在就让我先在`用AI的生产力解放全人类`的道路上贡献出微不足道的一点力量吧。