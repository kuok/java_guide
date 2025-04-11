# MAC命令

---

## 基础操作

---

### 最小化所有窗口

> Command⌘+Option⌥+M+H

---

## 刷新DNS缓存

```bash
sudo dscacheutil -flushcache;
sudo killall -HUP mDNSResponder
```

## MAC免密登陆SSH

1. 检查是否已存在公私钥对

    ```bash
    cd ~/.ssh
    ```

    ```bash
    ls
    ```

2. 生成公私钥对  
   根据交互，输入你想要的名字（默认id_rsa）  
   然后是passphrase，设置为空即可。这样就生成了一对公私钥

    ```bash
    ssh-keygen
    ```

    这时候当前目录下会多了一对公私钥对。

    ```bash
    ls
    ```

3. 上传公钥到服务器

    ```bash
    # user是你的ssh的用户，host是服务器地址，这时候还要输入密码。
    # 例子：ssh-copy-id -i id_rsa.pub root@111.111.111.111
    # ssh-copy-id -i tencent.pub root@tencent.server
    ssh-copy-id -i [公钥文件] user@host
    ```

4. ssh-add（mac的坑点）  
每次重启后要重新添加一遍
    ```bash
    # 例如，ssh-add -K id_rsa
    #ssh-add --apple-use-keychain tencent
    #ssh-add --apple-use-keychain ali
    ssh-add -K [你的私钥文件，就是那个不加.pub结尾的文件] 
    ```

    -K命令过时了，根据提示使用下面这个命令

    ```bash
    ssh-add --apple-use-keychain ali
    ```

5. 如此就可以使用ssh命令直接免密登陆

    ```bash
    ssh root@111.111.111.111
    # ssh root@111.230.113.52
    # ssh root@tencent.server
    ```

## iTerm命令

> 清除当前行：ctrl + u
>
> 到行首：ctrl + a
>
> 到行尾：ctrl + e
>
> 前进后退：ctrl + f/b (相当于左右方向键)
>
> 上一条命令：ctrl + p
>
> 搜索命令历史：ctrl + r
>
> 删除当前光标的字符：ctrl + d
>
> 删除光标之前的字符：ctrl + h
>
> 删除光标之前的单词：ctrl + w
>
> 删除到文本末尾：ctrl + k
>
> 交换光标处文本：ctrl + t
>
> 清屏1：command + r
>
> 清屏2：ctrl + l
>
> 自带有哪些很实用的功能/快捷键
>
> ⌘ + 数字在各 tab 标签直接来回切换
>
> 选择即复制 + 鼠标中键粘贴，这个很实用
>
> ⌘ + f 所查找的内容会被自动复制
>
> ⌘ + d 横着分屏 / ⌘ + shift + d 竖着分屏
>
> ⌘ + r = clear，而且只是换到新一屏，不会想 clear 一样创建一个空屏
>
> ctrl + u 清空当前行，无论光标在什么位置
>
> 输入开头命令后 按 ⌘ + ; 会自动列出输入过的命令
>
> ⌘ + shift + h 会列出剪切板历史
>
> 可以在 Preferences > keys 设置全局快捷键调出 iterm，这个也可以用过 Alfred 实现

---

## Redis
使用brew安装Redis
```bash
brew install redis
```

查看Redis信息
```bash
brew info redis
```

查看redis是否启动
```bash
ps -ef | grep redis
```

启动Redis
```bash
brew services start redis
```

停止Redis
```
brew services stop redis
```

卸载Redis
```bash
brew uninstall redis
```

---

## MySQL
启动
```bash
sudo /usr/local/mysql/support-files/mysql.server start
```
停止
```bash
sudo /usr/local/mysql/support-files/mysql.server stop
```
检查状态
```bash
sudo /usr/local/mysql/support-files/mysql.server status
```
