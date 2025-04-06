# 记一次Nacos内存不够退出问题排查

---

## Nacos掉线且启动不了

安装Nacos运行几天后无故掉线，且docker再启动马上退出。  
重新启动

```bash
docker restart nacos-standalone
```

启动成功了，但是查看容器又马上退出了。

```bash
docker -ps -a
```

Nacos显示又立即退出了

```bash
Exited (1) 9 seconds ago
```

---

## 检查日志

查看Nacos日志

```bash
docker logs nacos-standalone
```

发现最后几行提示

```bash
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 536870912 bytes for committing reserved memory.
OpenJDK 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000c0000000, 536870912, 0) failed; error='Out of memory' (errno=12)
# An error report file with more information is saved as:
# /tmp/hs_err_pid1.log
```

提示分配内存失败。转换一下 536870912 bytes=512 MB，奇怪了，Nacos单机版运行要这么吞内存吗，512MB还不够？  
这个服务器是2核2G的规格，`top`检查一下

```bash
KiB Mem :  2046500 total,    96704 free,  1567704 used,   382092 buff/cache
```

96704 kb=94MB
好像确实内存不够了,`M`按内存排一下序，显示

```bash
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
25913 root      20   0 3129916 571884   5292 S   0.0 27.9   5:25.94 java
20701 mysql     20   0 1861900 468756   3868 S   0.3 22.9  12:32.12 mysqld
20338 rabbitmq  20   0 2281148  81980   2980 S   0.0  4.0  17:17.37 beam.smp
 3625 root      20   0 1052896  81288   8040 S   1.0  4.0 101:14.29 YDService
 9792 root      20   0 1321232  52692   1020 S   0.3  2.6   0:17.54 node
24216 root      20   0 1999620  42096   4856 S   0.0  2.1   2:45.89 dockerd
26448 root      20   0  363104  29792   2920 S   0.0  1.5   0:16.27 firewalld
 2236 root      20   0 1216640  19820   2540 S   1.0  1.0 112:55.85 barad_agent
```

java是Halo博客，还有MySQL两个就占了一半了。。  
MySQL不知道在干嘛，Halo是可以下了的。  

---

## 解决思路

关了Halo之后再启动还是报错。

```bash
2025-03-19 15:00:36,652 INFO Tomcat initialized with port(s): 8848 (http)

2025-03-19 15:00:36,837 INFO Root WebApplicationContext: initialization completed in 6037 ms

2025-03-19 15:00:47,375 INFO Will secure any request with [org.springframework.security.web.session.DisableEncodeUrlFilter@260a3a5e, org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@49206065, org.springframework.security.web.context.SecurityContextPersistenceFilter@7efe7b87, org.springframework.security.web.header.HeaderWriterFilter@3a543f31, org.springframework.security.web.csrf.CsrfFilter@3dd818e8, org.springframework.security.web.authentication.logout.LogoutFilter@66420549, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@5a2bd7c8, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@7187bac9, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@3c0bbc9f, org.springframework.security.web.session.SessionManagementFilter@2b9f74d0, org.springframework.security.web.access.ExceptionTranslationFilter@41b1f51e]
```

明明已经启动成功了，还是退出。  
这个日志看不出什么问题，Docker的日志查看感觉是一个坑，果断放弃Docker改为本地部署。

---
