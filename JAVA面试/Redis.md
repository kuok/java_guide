# Redis

---

## RDB与AOF持久化机制

---

### RDB(Redis Data Base) 
在指定的时间间隔内将内存中的数据集快照写入磁盘，RDB是内存快照（内存数据的二进制序列化形式）的方式持久化，每次都是从Redis中生成一个快照进行数据的全量备份。
---

### AOF(Append Only File)

---

### 混合持久化