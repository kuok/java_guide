# SHELL介绍
Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。

Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-13/65561163133916.png?Expires=4898150612&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=8OkN4t72n2tdoDhdVRbE9qV22%2FU%3D)

#!是一个特殊标记，说明这是一个可执行的脚本。除了第一行，其他以#开头的都不再生效，为注释。

#!后面是脚本的解释器程序路径。这个程序可以是shell，程序语言或者其他通用程序，常用的是bash、sh。

```shell
#!/bin/bash
 
#!/bin/sh
```

```shell
# 查看系统可使用的shell类型
cat /etc/shells
 
# 查看当前使用的SHELL
echo $SHELL
echo $0
```

---

## SH
第一个 Unix Shell 是Thompson Shell（简称sh ），由贝尔实验室的Ken Thompson编写，并随 Unix 1 至 6 版本一起发布，发行时间从 1971 年到 1975 年。虽然以现代标准来看还很简陋，但它引入了许多后来 Unix Shell 共有的基本特性，包括管道、使用和 的简单控制结构以及文件名通配符。虽然目前已不再使用，但它仍然是一些古老 UNIX系统的一部分。

它模仿了1965 年由美国软件工程师Glenda Schroeder开发的Multics shell。Schroeder的 Multics shell 本身又模仿了Louis Pouzin向 Multics 团队展示的RUNCOM程序。一些 Unix 配置文件（例如“.vimrc”）上的“rc”后缀，是 Unix shell 中 RUNCOM 的遗留。

PWB shell或 Mashey shell（sh）是 Thompson shell 的向上兼容版本，由John Mashey等人扩充，并随Programmer's Workbench UNIX一起分发，大约在 1975-1977 年。它专注于使 shell 编程实用，尤其是在大型共享计算中心。它添加了 shell 变量（环境变量的前身，包括演变为 $PATH 的搜索路径机制）、用户可执行的 shell 脚本和中断处理。控制结构从 if/goto 扩展到 if/then/else/endif、switch/breaksw/endsw 和 while/end/break/continue。随着 shell 编程的普及，这些外部命令被合并到 shell 本身中以提高性能。

---

## SH(Bourne shell)
Bourne shell，即sh ，是Stephen Bourne在贝尔实验室开发的一种新的 Unix shell 。 [ 6 ] 1979 年作为 UNIX 版本 7 的 shell 分发，它引入了后来所有 Unix shell 所共有的其余基本特性，包括here documents、命令替换、更通用的变量和更广泛的内建控制结构。该语言（包括使用反向关键字来标记块的结束）受到了ALGOL 68的影响。


---

## BASH(Bourne-Again shell)
Bash，Unix shell的一种，在1987年由Brian Fox为了GNU计划而编写。1989年发布第一个正式版本，原先是计划用在GNU操作系统上，但能运行于大多数类Unix系统的操作系统之上，包括Linux与Mac OS X v10.4起至macOS Mojave都将它作为默认shell，而自macOS Catalina，默认Shell以zsh取代。


sh 遵循POSIX规范：“当某行代码出错时，不继续往下解释”。bash 就算出错，也会继续向下执行。

---

## ZSH(Z shell)
Zsh 是一个扩展的Bourne shell，具有许多改进，包括Bash、ksh和tcsh的一些功能。

Zsh 由 Paul Falstad 于 1990 年在普林斯顿大学读书时创建。它融合了ksh和tcsh的功能，提供可编程命令行补全、扩展文件通配符、改进的变量/数组处理以及可主题化的提示符等功能。

2019年时，在MacOS上由于Bash的版本已经很旧(v3.2.57)，而新版本的Bash v5改采GPLv3授权，这是Apple公司无法接受的。于是自当时起，macOS的默认Shell已从Bash改为Zsh。

Kali Linux也使用zsh作为默认shell。

---

## ASH(Almquist shell)
它最初是Bourne Shell的System V.4变体的克隆；常用于资源受限的环境。FreeBSD 、NetBSD（及其衍生版本）的 sh 均基于 ash，并进行了增强以符合POSIX标准。

Ash（主要是 Dash 的分支）在嵌入式 Linux系统中也相当流行。Dash 0.3.8-5 版本被整合到BusyBox中，后者是该领域常用的全能可执行程序。现代 BusyBox 版本支持额外的Bash功能，这些功能已在Alpine Linux、Tiny Core Linux等现代发行版以及基于 Linux 的路由器固件（例如OpenWrt、Tomato和DD-WRT ）中启用。

---

## DASH(Debian Almquist shell)
1997 年，Herbert Xu 将Debian Linuxash从 NetBSD 移植到Debian Linux。2002 年 9 月，随着 0.4.1 版本的发布，该移植版本更名为Dash（Debian Almquist shell）。Xu 的主要工作重点是遵循 POSIX 规范并实现精简的实现。

---

## CSH(C SHELL)
C shell（即csh ）以 C 编程语言为蓝本，包括控制结构和表达式语法。它由加州大学伯克利分校的研究生Bill Joy编写，并随BSD Unix广泛发行。

