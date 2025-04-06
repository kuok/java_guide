## MySQL

---

### 索引

---

#### 索引的种类
在MySQL中索引是在存储引擎层实现的，而不是在服务器层实现的，所以不同存储引擎具有不同的索引类型和实现。常见的索引分类如下：
* 按数据结构分类：B+tree索引、Hash索引、Full-text索引。
* 按物理存储分类：聚簇索引索引、非聚簇索引。
* 按字段特性分类：主键索引(PRIMARY KEY)、唯一索引(UNIQUE)、普通索引(INDEX)、全文索引(FULLTEXT)。
* 按字段个数分类：单列索引、联合索引（也叫复合索引、组合索引）。

---



---

#### MySQL 为什么选择B+树作为底层数据结构

---

##### 二叉树

二叉搜索树是遵守二分搜索法实现的一种数据结构，它具有下面特点：

* 任意节点的左节点不为空时，左节点值小于根节点值；
* 右节点不为空时，右节点值大于根节点值；

依次存入数据，如果数据是递增的，则原二叉树退化为链表结构。
![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-25/286332887249208.gif?Expires=4896514475&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=PhusmO%2FcIndC2Tpzt90yA6pign8%3D)

二叉树不适合作为索引结构的原因：

* 如果索引的字段值是按顺序增长的，二叉树会转变为链表结构，因此检索的过程和全表扫描无异。
* 每个节点中只存储一个数据，节点之间还是不连续的，每次磁盘IO只能读取一个数据。

---

##### 红黑树

相比于二叉树，红黑树则进一步做了优化，它是一种自适应的平衡树，会根据插入的节点数量以及节点信息，自动调整树结构来维持平衡。
![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-25/286441783346375.gif?Expires=4896514583&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=0AZPe8zsiAUbhihtSbp7YbIDPa8%3D)

由于树变矮了，其效果也很明显，在红黑树中只需要经过3次IO就可以找到目标数据，似乎看起来还不错对嘛？但MySQL为啥不用这颗名声远扬的红黑树呢？
红黑树不适合作为索引结构的原因：

* 每个节点中只存储一个数据，节点之间还是不连续的，每次磁盘IO只能读取一个数据。
* 数据量过大，节点个数就越多，树高度也会增高（也就是树的深度越深），增加磁盘I/O次数，影响查询效率。

---

##### B-Tree

B树和红黑树相比，其单节点可容纳多个数据，就能在很大程度上改善其性能，使B树的树高远远小于红黑树的高度。
![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-25/286555517405250.gif?Expires=4896514697&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=j4GENjoZTnd8%2F%2BK0M%2Bdy%2FeVMTiI%3D)
对比红黑树可以发现，每个节点上可以存储更多的数据，且树高固定，数据插入之后横向扩展。观察动画只需要2次IO就可以找到目标数据，搜索效率大大提高了。并且每个节点的元素我们可以自己控制。  

为什么MySQL没有采用B树结构了？

我们仔细观察可以知道B的叶子节点直接是没有指针的，但是日常查询中包含了大量的范围查找，所以当出现范围查找的时候，会出现多次的IO查找。

---

##### B+Tree

B+树是在B树的基础进一步优化，一方面节点分为了叶节点和叶子节点两类，另一方面叶子节点之间存在单向链表指针。
![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-25/286682856301583.gif?Expires=4896514824&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=BQHm9ZXEkxUFoM4kSqRNRQp6H9k%3D)

B+树相比于B树叶子节点之间多了个单项指针，当需要做范围查询时，只需要定位第一个节点，然后就可以直接根据各节点之间的指针，获取到对应范围之内的所有节点，也就是只需要发生一次IO，就能够确定所查范围之内的所有数据位置。

其实MySQL底层真正的索引结构还对叶子节点之间的指针进行了优化，B+树叶子节点的单向指针无法友好支持的倒叙查询，因此MySQL针对单向指针优化成了双向指针，也就是双向链表结构。即可以快速按正序进行范围查询，而可以快速按倒序进行范围操作，在某些业务场景下又能进一步提升整体性能！

节点分为了叶节点和叶子节点。为什么？
因为B+树的叶节点不存储数据，仅存储指向叶子节点的指针，这样在相同树高时，能存储更多的数据，需要注意的是叶节点数据与叶子结点数据是冗余的。
现在对于MySQL索引为何要选择B+树（变种）的原因大家应该懂了吧。

---

##### Hash索引
在存储引擎中，Memory引擎支持Hash索引  
Hash索引其实有点像Java中的HashMap底层的数据结构，他也有很多的槽，存的也是键值对，键值为索引列，值为数据的这条数据的行指针，通过行指针就可以找到数据
假设现在user表用Memory存储引擎，对name字段建立Hash索引，表中插入三条数据
![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/478796340196750.png?Expires=4896886283&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=qnGuxi1VlVDgOIzhG9x32%2FbgO7I%3D)

* hash索引只能用于等值比较，所以查询效率非常高
* 不支持范围查询，也不支持排序，因为索引列的分布是无序的

---

#### 聚簇索引与非聚簇索引索引
按物理存储分类：InnoDB的存储方式是聚集索引，MyISAM的存储方式是非聚集索引。
![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/478911637526291.png?Expires=4896886398&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=%2FjHV8LvtyiBPBM4UbojdB8xCOsg%3D)
![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/478926376348583.png?Expires=4896886413&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=fW%2BOCPi8IabR1t7zhEJzU0U6bok%3D)

聚簇索引（查询快）
* 聚簇索引将数据存储在索引树的叶子节点上。
* 聚簇索引可以减少一次查询，因为查询索引树的同时就能获取到数据。
* 聚簇索引的缺点是，对数据进行修改或删除操作时需要更新索引树，会增加系统的开销。
* 聚簇索引通常用于数据库系统中，主要用于提高查询效率。

非聚簇索引（更新快）
* 非聚簇索引不将数据存储在索引树的叶子节点上，而是存储在数据页中。
* 非聚簇索引在查询数据时需要两次查询，一次查询索引树，获取数据页的地址，再通过数据页的地址查询数据（通常情况下来说是的，但如果索引覆盖的话实际上是不用回表的）。
* 非聚簇索引的优点是，对数据进行修改或删除操作时不需要更新索引树，减少了系统的开销。
* 非聚簇索引通常用于数据库系统中，主要用于提高数据更新和删除操作的效率。

---

#### 二级索引
在MySQL中，创建一张表时会默认为主键创建聚簇索引，B+树将表中所有的数据组织起来，即数据就是索引主键所以在InnoDB里，主键索引也被称为聚簇索引，索引的叶子节点存的是整行数据。而除了聚簇索引以外的所有索引都称为二级索引，二级索引的叶子节点内容是主键的值。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/479193895784458.png?Expires=4896886680&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=2bwUYrFQs2alnLYdjEmRuLZSm0Y%3D)

在MySQL中主键索引的叶子节点存的是整行数据，而二级索引叶子节点内容是主键的值.

---

#### 回表
这里假设对name字段创建了一个索引，执行下面这条sql 则需要进行回表:
```sql
SELECT * FROM users WHERE age=35;
```

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/479289195365833.png?Expires=4896886776&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=DJlXl0bGciGn1NOm7R3QS7R6Y98%3D)

由于是select *，还要查其它字段，此时就会根据id=9到聚簇索引（主键索引）中查找其它字段数据，这个根据id=4到聚簇索引中查找数据的过程就被称为回表。

---

#### 覆盖索引
当执行select *  from `users` where age = 35;这条sql的时候，会先从索引页中查出来age = 35;对应的主键id，之后再回表，到聚簇索引中查询其它字段的值。
```sql
select id from `users` where age = 35;
```
这种需要查询的字段都在索引列中的情况就被称为覆盖索引，索引列覆盖了查询字段的意思。

---

#### 单列索引与联合索引
单列索引：
```sql
ALTER TABLE `test`.`user`  ADD INDEX(`name`);
```
联合索引
```sql
ALTER TABLE `test`.`user` ADD INDEX(`name`, `age`, `id`);
```
联合索引的劣势：
* **必须符合最左前缀原则**  
当创建(a,b,c)复合索引时，想要索引生效的话，只能使用 a和ab和abc三种组合！

联合索引的优势：
* **减少开销**  
建一个联合索引(a,b,c),实际相当于建了(a),(a,b),(a,b,c)三个索引.每多一个索引,都会增加写操作的开销和磁盘空间的开销.对于大量数据的表,使用联合索引会大大的减少开销!
* **覆盖索引**  
那么mysql可以直接通过遍历索引取得数据,而无需回表,这减少了很多的随机io操作.减少io操作,特别是随机io其实DBA主要的优化策略.所以,在真正的实际应用中,覆盖索引是主要的提升性能的优化手段之一.
* **效率高**  
索引列多,通过联合索引筛选出的数据越少.比如有1000w条数据的表,有如下sql:
select col1,col2,col3 from table where col1=1 and col2=2 and col3=3;
假设:假设每个条件可以筛选出10%的数据
A:如果只有单列索引,那么通过该索引能筛选出1000w*10%=100w条数据,然后再回表从100w调数据中找到符合col2=2 and col3=3的数据,然后再排序,再分页,以此类推(递归);
B:如果是(col1,col2,col3)联合索引,通过三列索引筛选出1000w*10%*10%*10%=1w,效率提升可想而知

---

#### 索引下推
索引下推（INDEX CONDITION PUSHDOWN，简称 ICP）是在 MySQL 5.6 针对扫描二级索引的一项优化改进。总的来说是通过把索引过滤条件下推到存储引擎，来减少 MySQL 存储引擎访问基表的次数以及 MySQL 服务层访问存储引擎的次数。  
减少回表次数，直接在存储层判断，而不用回表去取得数据判断，相当于是范围版的覆盖索引。

我们来具体看一下，在`没有使用ICP`的情况下，MySQL的查询：
* 存储引擎读取索引记录；
* 根据索引中的主键值，定位并读取完整的行记录；
* 存储引擎把记录**交给Server层去检测该记录是否满足WHERE条件**。
  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-25/287668925071708.png?Expires=4896515811&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=BG1IBUjPMu6OdwcwgokI5MQLGRw%3D)
> 上述情况中，like“A%”是在联合索引中即存储引擎判断的，然后回表取得数据，而“age=40”是要回到服务层去判断的。

`使用ICP`的情况下，查询过程：
* 存储引擎读取索引记录（不是完整的行记录）；
* **判断WHERE条件部分能否用索引中的列来做检查**，条件不满足，则处理下一行索引记录；
* 条件满足，使用索引中的主键去定位并读取完整的行记录（就是所谓的回表）；
* 存储引擎把记录交给Server层，Server层检测该记录是否满足WHERE条件的其余部分。
  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-25/287680116033375.png?Expires=4896515822&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=eslvywttLzTiIweaRGz77VSS5M8%3D)
> 上述情况中，like“A%”是在联合索引中存储引擎中判断的，同时“age=40”也是在联合索引中判断的，再回表取得数据给服务层。

索引下推的使用条件
* 只能用于range、 ref、 eq_ref、ref_or_null访问方法；
* where 条件中是用 and 而非 or 的时候。
* 只能用于InnoDB和 MyISAM存储引擎及其分区表。
* 对InnoDB存储引擎来说，索引下推只适用于二级索引（也叫辅助索引），ICP目标是减少全行记录读取，从而减少IO 操作，只能用于非聚簇索引。聚簇索引本身包含的表数据，也就不存在下推一说。
* ICP不支持基于虚拟列上建立的索引，比如说函数索引。
* ICP不支持引用子查询作为条件。
* ICP不支持存储函数作为条件，因为存储引擎无法调用存储函数。

相关系统参数  
索引条件下推默认是开启的，可以使用系统参数optimizer_switch来控制器是否开启。
```sql
select @@optimizer_switch;
```
切换状态
```sql
set optimizer_switch="index_condition_pushdown=off";
set optimizer_switch="index_condition_pushdown=on";
```
---

#### 索引合并
索引合并（index merge）是从MySQL5.1开始引入的索引优化机制，在之前的MySQL版本中，一条sql多个查询条件只能使用一个索引，但是引入了索引合并机制之后，MySQL在某些特殊的情况下会扫描多个索引，然后将扫描结果进行合并

结果合并会为下面三种情况：
* 取交集（intersect）
```sql
select * from `user` where name = '张三' and age= 22;
```
* 取并集（union）
```sql
select * from `user` where name = '张三' or age= 22;
```
* 排序后取并集（sort-union）
```sql
select * from `user` where name = '张三' and age= 22;
```

大概过程：  
分别根据idx_name和idx_age取出对应的主键id，之后将主键id取交集，那么这部分交集的id一定同时满足查询name = '张三' and age= 22的查询条件，之后再根据交集的id回表
不过要想使用取交集的联合索引，需要满足各自索引查出来的主键id是排好序的，这是为了方便可以快速的取交集

---

#### MySQL索引失效场景
1. 对索引使用左或者左右模糊匹配  
    在使用LIKE关键字进行模糊匹配时，如`LIKE '%xx'`或`LIKE '%xx%'`，都会导致索引失效，从而引发全表扫描。  
    ```sql
    SELECT * FROM t_user WHERE name LIKE '%林';
    ```
    这是因为B+树索引是根据索引值有序存储的，仅能支持前缀比较。
2. 对索引使用函数  
    如果在查询条件中对索引字段使用了函数，这通常会导致索引失效，从而导致全表扫描。  
    比如：`length(name)`函数被应用于name字段。  
    ```sql
    SELECT * FROM t_user WHERE LENGTH(name) = 2;
    ```
    不过，从MySQL 8.0版本开始，数据库引入了函数索引这一特性，允许我们针对函数计算结果建立索引。也就是说，索引的值可以是某个函数计算后的结果，这样就可以通过索引来快速查询所需数据。
    ```sql
    ALTER TABLE t_user ADD KEY idx_name_length ((LENGTH(name)));
    ```
3. 对索引进行表达式计算  
    在查询条件中进行表达式计算通常会导致无法使用索引。
    ```sql
    SELECT * FROM t_user WHERE id + 1 = 10;
    ```
4. 对索引隐式类型转换  
    如果索引字段是字符串类型，但是在条件查询中，输入的参数是整型的话，查看执行计划结果可以发现这条语句会走全表扫描。
    ```sql
    SELECT * FROM t_user WHERE phone = 18800000001;
    ```
    然而，如果索引字段是整型，即使查询条件中使用字符串类型的参数，索引依然能够正常生效。例如，执行以下SQL：
    ```sql
    SELECT * FROM t_user WHERE id = '1';
    ```
    **为何第一个例子导致索引失效，而第二个例子却没有？**  
    要理解这一点，我们首先需要掌握MySQL的数据类型转换规则，它决定了在比较字符串和数字时，哪个会被转换。
    用一个简单的方法来测试这一规则：选择 `SELECT "10" > 9`。
    * 如果MySQL会自动将字符串转换为数字，那么这相当于执行 SELECT 10 > 9，结果应该是1，因为数字10确实大于9。
    * 如果MySQL会将数字转换为字符串，那么这相当于执行 SELECT "10" > "9"。在这种情况下，字符串比较是逐位进行的，按照ASCII码进行比较。首先比较字符“1”和“9”，由于“1”的ASCII码小于“9”，所以结果应该是0。
    ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/477439922076375.png?Expires=4896884926&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=%2F7AidbK1VwRI%2BedW8xBEzaT2yw4%3D)

    根据测试结果，可以知道MySQL在比较时，会将字符串转换为数字。  
    因此
    ```sql
    SELECT * FROM t_user WHERE phone = 18800000001;
    等同于:
    SELECT * FROM t_user WHERE CAST(phone AS SIGNED) = 18800000001;
    ```
    ```sql
    SELECT * FROM t_user WHERE id = '1';
    等同于:
    SELECT * FROM t_user WHERE id = CAST('1' AS SIGNED);
    ```
5. 联合索引非最左匹配  
    当多个普通字段组合在一起创建索引时，称为联合索引（或组合索引）。  
    创建联合索引时，字段的顺序至关重要。例如，联合索引 (a, b, c) 与 (c, b, a) 在使用时会有显著不同。为了有效利用联合索引，必须遵循最左匹配原则，即查询条件必须从最左边的字段开始匹配。  
    例如，对于 (a, b, c) 的联合索引 ，以下查询能够成功匹配联合索引：  
    * `WHERE a=3`
    * `WHERE a=3 AND b=5 AND c=4`
    * `WHERE a=3 AND b=5`

    然而，若查询条件不符合最左匹配原则，索引将失效，如以下查询：
    * `WHERE b=5`
    * `WHERE c=4`
    * `WHERE b=5 AND c=4`
6. WHERE 子句中的 OR  
    如果在 WHERE 子句中，OR 前的条件列是索引列，而 OR 后的条件列不是索引列，将会面临全表扫描的问题。
    ```sql
    // id 是主键且为索引列，age 是普通列
    SELECT * FROM t_user WHERE id = 1 OR age = 18;
    ```
    因为 OR 的逻辑是，只需满足任一条件即可。因此，当有一个条件不使用索引时，索引的存在毫无意义。  
    解决这个问题很简单：将 age 字段设置为索引。这样，查询将会充分利用索引，避免全表扫描。
    ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/478020055197416.png?Expires=4896885506&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=GyMvCn3vSX5i8Jmlnf4cy9E3Ue4%3D)

    执行计划变为“type=index merge”，则意味着数据库分别对 id 和 age 条件进行了索引扫描，并将结果合并，从而提高了查询效率。
    通过这种方式，可以显著提升查询性能，减少数据库的负载。


---

### InnoDB和MyISAM
* **索引（聚簇索引和非聚簇索引）**：  
InnoDB 是聚集索引，数据文件是和索引绑在一起的，必须要有主键，通过主键索引效率很高，但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，否则其他索引也会很大。而 MyISAM 是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针，主键索引和辅助索引是独立的。

* 外键（支持与不支持）  
InnoDB 支持外键，而 MyISAM 不支持。对一个包含外键的 InnoDB 表转为 MYISAM 会失败。
* 全文索引（效率低与效率高）  
InnoDB 在 MySQL 5.6 之前不支持全文索引，而 MyISAM 一直都支持，如果你用的是老版本，查询效率上 MyISAM 要高。
* **锁粒度（行锁与表锁）**  
InnoDB 锁粒度是行锁，而 MyISAM 是表锁。
* **事务（支持与不支持）**  
InnoDB 支持事务，MyISAM 不支持，对于 InnoDB 每一条 SQL 语言都默认封装成事务，自动提交，这样会影响速度，所以最好把多条 SQL 语言放在 begin 和 commit 之间，组成一个事务。
* 行数（没保存与保存了）
InnoDB 不保存表的具体行数，执行 select count(*) from table 时需要全表扫描。而 MyISAM 用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快，但如果上述语句还包含了 where 子句，那么两者执行效率是一样的。

---

### Explain

---

#### Explain作用
* 表的读取顺序
* SQL执行时查询操作类型
* 可以使用哪些索引
* 实际使用哪些索引
* 每张表有多少行记录被扫描
* SQL语句性能分析

---

#### Explain用法
![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/481407017200625.png?Expires=4896888893&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=ANesPulWNSmREZ6Oop8rnYiD80M%3D)

| 列名            | 含义                            |
|---------------|-------------------------------|
| id            | 每个select都有一个对应的id号，并且是从1开始自增的 |
| select_type   | 查询语句执行的查询操作类型                 |
| table         | 表名                            |
| partitions    | 表分区情况                         |
| type          | 查询所用的访问类型                     |
| possible_keys | 可能用到的索引                       |
| key           | 实际查询用到的索引                     |
| key_len       | 所使用到的索引长度                     |
| ref           | 使用到索引时，与索引进行等值匹配的列或者常量        |
| rows          | 预计扫描的行数（索引行数或者表记录行数）          |
| filtered      | 表示符合查询条件的数据百分比                |
| Extra         | SQL执行的额外信息                    |

---

##### id列：每个select都有一个对应的id号，并且是从1开始自增的。
* 如果id序号相同，从上往下执行。
* 如果id序号不同，序号大先执行。
* 如果两种都存在，先执行序号大，在同级从上往下执行。
* 如果显示NULL，最后执行。表示结果集，并且不需要使用它来进行查询。

---

##### select_type列：表示查询语句执行的查询操作类型
1. simple：简单select，不包括union与子查询
    ```sql
    Explain select * from users inner join orders on users.id = orders.user_id;
    ```

    ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/481983130578166.png?Expires=4896889470&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=BT8XXf3sBKzt4bPIv%2FUtmN8Pd9k%3D)

2. primary：复杂查询中最外层查询，比如使用union或union all时，id为1的记录select_type通常是primary
    ```sql
    explain
    select id from users
    union
    select id from products;
    ```

    ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/482073661949458.png?Expires=4896889560&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=LX%2BkTH6vG85WLjjog%2Fr%2Bx2MnBTc%3D)

3. subquery：指在 select 语句中出现的子查询语句,结果不依赖于外部查询（不在from语句中）
    ```sql
    explain
    select orders.*,(select name from products where id = 1) from orders;
    ```

    ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/482114882761458.png?Expires=4896889601&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=W0vcAtDO3PdOi9l7JsVuRrFSORE%3D)

4. dependent subquery：指在 select 语句中出现的查询语句，结果依赖于外部查询
    ```sql
    explain
    select orders.*,(select name from products where products.id = orders.user_id) from orders;
    ```

    ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/482166470159333.png?Expires=4896889653&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=%2FbVwVtZYqNTEjmzV1dLt5USViRs%3D)

5. derived：派生表，在FROM子句的查询语句，表示从外部数据源中推导出来的，而不是从 SELECT 语句中的其他列中选择出来的。
    ```sql
    set session optimizer_switch='derived_merge=off'; #关闭MySQL5.7对衍生表合并优化
    
    explain
    select * from (select user_id from orders where id = 1) as temp;
    
    set session optimizer_switch='derived_merge=on'; #还原配置
    ```

    ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/482232577715791.png?Expires=4896889719&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=gXcfYgmpAj8i5A9c2rEhPHuNfu0%3D)

6. union：分union与union all两种，若第二个select出现在union之后，则被标记为union；如果union被from子句的子查询包含，那么第一个select会被标记为derived；union会针对相同的结果集进行去重，union all不会进行去重处理。
    ```sql
    explain 
    select * from (
    select id from products where price = 10
    union
    select id from orders where user_id in (1,2)
    union 
    select id from users where name = '张三' ) as temp;
    ```

    ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/482355836195666.png?Expires=4896889842&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=V97fAwL8lqS8UEqautjFbI7MD9o%3D)

7. dependent union：当union作为子查询时，其中第一个union为dependent subquery，第二个union为dependent union。
    ```sql
    explain 
    select * from orders where id in (
    select id from products where price = 10
    union
    select id from orders where user_id = 2
    union 
    select id from users where name = '张三' );
    ```

    ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/482398088534750.png?Expires=4896889885&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=pNMjo2cf%2FZD7xCRgJz9FiZ5iLjk%3D)

8. union result：如果两个查询中有相同的列，则会对这些列进行重复删除，只保留一个表中的列。
    ```sql
    explain
    select id from users
    union
    select id from products;
    ```

    ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/482424032061750.png?Expires=4896889910&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=AfCdR3Hm0XmSx%2BkR660CAG2n4iY%3D)

---

##### table列：查询所涉及的表名。如果有多个表，将显示多行记录

---

##### partitions列：表分区情况
查询语句所涉及的表的分区情况。具体来说，它会显示出查询语句在哪些分区上执行，以及是否使用了分区裁剪等信息。如果没有分区，该项为NULL。

---

##### type列：查询所使用的访问类型
效率从高到低分别为：system > const > eq_ref > ref > fulltext > ref_or_null > range > index > ALL，一般来说保证range级别，最好能达到ref级别。

* NULL：MySQL在优化过程中分解语句就已经可以获取到结果，执行时甚至不用访问表或索引。
  ```sql
  explain 
  select min(id) from users;
  ```

  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/482741065191791.png?Expires=4896890227&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=GgwzDw4J4HnpQavCyjyJVEIrPQA%3D)

* system：const类型的一种特殊场景，查询的表只有一行记录的情况，并且该表使用的存储引擎的统计数据是精确的  
  InnoDb存储引擎的统计数据不是精确的，虽然只有一条数据但是type类型为ALL；
  ```sql
  DROP TABLE t;
  CREATE TABLE t(i INT) ENGINE=InnoDb;
  INSERT INTO t VALUES(1);
  explain select * from t;
  ```
  
  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/482834047646875.png?Expires=4896890320&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=yfqWue1uoqVsgW4PGuqfxYJxXSM%3D)

  Memory存储引擎的统计数据是精确的，所以当只有一条记录的时候type类型为system。
  ```sql
  DROP TABLE tt;
  CREATE TABLE tt(i INT) ENGINE=memory;
  INSERT INTO tt VALUES(1);
  explain select * from tt;
  ```

  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/482854978998166.png?Expires=4896890341&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=ROZHxTwiJBmCpBnbH6izveFW198%3D)

* const：基于主键或唯一索引查看一行，当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问转换成常量查询，效率高
  ```sql
  explain
  select * from orders where id = 1;
  ```

  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/482896491693958.png?Expires=4896890383&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=apuXNcDtNz0FWA9kqxQ2W%2Fc0Nlo%3D)

* eq_ref：基于主键或唯一索引连接两个表，对于每个索引键值，只有一条匹配记录，被驱动表的类型为'eq_ref'
  ```sql
  explain
  select users.* from users inner join orders on users.id = orders.id;
  ```

  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/482938716128833.png?Expires=4896890425&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=HBEsr9YRTiy%2BWplvPygHsPoA7gQ%3D)

* ref：基于非唯一索引连接两个表或通过二级索引列与常量进行等值匹配，可能会存在多条匹配记录
  ```sql
  //关联查询，使用非唯一索引进行匹配。
  explain
  select users.* from users inner join orders on users.id = orders.user_id;
  ```

  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/483248671794791.png?Expires=4896890735&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=MbXPU9s51QjMo7bcNxYbG99x%2BMk%3D)

  ```sql
  //简单查询，使用二级索引列匹配。
  explain
  select * from orders where user_id = 1;
  ```

  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/483279233191666.png?Expires=4896890766&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=HgHgbhVa%2BjIxKrVn449Q3O5GF08%3D)

* range：使用非唯一索引扫描部分索引，比如使用索引获取某些范围区间的记录
  ```sql
  explain
  select * from orders where user_id > 3;
  ```
  
  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/483314353236416.png?Expires=4896890801&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=2ieD65JQnT3jkD2DVDL%2BLpJv%2BUg%3D)

* index：扫描整个索引就能拿到结果，一般是二级索引，这种查询一般为使用覆盖索引（需优化，缩小数据范围）
  ```sql
  explain
  select user_id from orders;
  ```

  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/483344553174666.png?Expires=4896890831&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=a4w5QPKlTQyiUrHjV6zaWUxegys%3D)

* all：扫描整个表进行匹配，即扫描聚簇索引树（需优化，添加索引优化）
  ```sql
  explain
  select * from users;
  ```

  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/483366154538500.png?Expires=4896890853&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=Yxra75M3AX3VoDvQMsDG02gEZdM%3D)


---

##### possible_keys列：表示在查询中可能使用到某个索引或多个索引；如果没有选择索引，显示NULL

---

##### key列：表示在查询中实际使用的索引，如果没有使用索引，显示NULL。

---

##### key_len列：表示当优化器决定使用某个索引执行查询时，该索引记录的最大长度（主要使用在联合索引）
联合索引可以通过这个值算出具体使用了索引中的哪些列。
```sql
explain 
select * from users where name = '张三' and email = 'zhangsan@example.com';
```

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/483604175926875.png?Expires=4896891091&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=J9gsNUnsBSbNIYvo6qtAVzrTEbU%3D)

计算规则：
* 字符串：  
char(n)：n个字节  
varchar(n)：如果是uft-8：3n+2字节，加的2个字节存储字符串长度。如果是utf8mb4：4n+2字节。  
* 数值类型：  
tinyint：1字节  
smaillint：2字节  
int：4字节  
bigint：8字节  
* 时间类型：  
date：3字节  
timestamp：4字节  
datetime：8字节  

字段如果为NULL，需要1个字节记录是否为NULL

---

##### ref列：表示将哪个字段或常量和key列所使用的字段进行比较。
当使用索引列等值查询时，与索引列进行等值匹配的对象信息。
```sql
//常量
explain 
select * from users where name = '张三' and email = 'zhangsan@example.com';
```

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/483769969706041.png?Expires=4896891256&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=zUMQ5f4WPe0tgTYei%2BW%2FgmNTUfY%3D)

```sql
//字段
explain
select users.* from users inner join orders on users.id = orders.id;
```

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/483797128251791.png?Expires=4896891284&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=Ol6TAjr0xbvJ27yWHmBctkp5ia8%3D)

```sql
//函数
explain
select users.* from users inner join orders on users.id = trim(orders.id);
```

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/483833429088500.png?Expires=4896891320&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=rElfBcZYdz2PySAeVm%2FW3IzFcDU%3D)

---

##### rows列：全表扫描时表示需要扫描表的行数估计值；索引扫描时表示扫描索引的行数估计值；值越小越好（不是结果集中的行数）

---

##### filtered列：表示符合查询条件的数据百分比。可以使用rows * filtered/100计算出与explain前一个表进行连接的行数。
前一个表指 explain 中的id值比当前表id值小的表，id相同的时候指后执行的表。
```sql
explain
select users.* from users inner join orders on users.id = orders.id;
```

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/483940073726500.png?Expires=4896891426&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=euu38nC6pYK7FwEcMiK2ePrfAZM%3D)

---

##### Extra列：SQL执行查询的一些额外信息
* Using Index：使用非主键索引树就可以查询所需要的数据。一般是覆盖索引，即查询列都包含在辅助索引树叶子节点中，不需要回表查询。
  ```sql
  explain
  select user_id,id from orders where user_id = 1;
  ```
  
  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/484077895820833.png?Expires=4896891564&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=6X0t2gSU27haeSXQWPMbRur1kxs%3D)

* Using where：不通过索引查询所需要的数据
  ```sql
  explain
  select * from orders where total_price = 100;
  
  explain
  select * from orders where user_id = 1 and total_price = 100;
  ```
  
  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/484110831195916.png?Expires=4896891597&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=auaEpGRsky5RBVzc1rxvL7zUDLo%3D)

* Using index condition：表示查询列不被索引覆盖，where 条件中是一个索引范围查找，过滤完索引后回表找到所有符合条件的数据行。
  ```sql
  explain
  select * from orders where user_id > 3;
  ```
  
  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/484148759029500.png?Expires=4896891635&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=9GGwnLudIJ50%2FzX2WiCCo1rQSF8%3D)

* Using temporary：表示需要使用临时表来处理查询；
  ```sql
  //total_price列无索引，需要创建一张临时表进行去重
  explain
  select distinct total_price from orders;
  ```
  
  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/484186511796208.png?Expires=4896891673&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=ofTJy2r61oVUvKYLIAE%2BYLnD2WQ%3D)

* Using filesort：当查询中包含 order by 操作而且无法利用索引完成的排序操作，数据较少时从内存排序，如果数据较多需要在磁盘中排序。	需优化成索引排序。
  ```sql
  explain
  select total_price from orders order by total_price;
  ```
  
  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/484240616588958.png?Expires=4896891727&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=KQTsrCUeR52sw5VY4XAgJBOeVTw%3D)

* Select tables optimized away：使用某些聚合函数（min,max）来访问某个索引值。
  ```sql
  
  explain 
  select min(id) from users;
  
  explain 
  select min(password) from users;
  ```
  
  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-30/484286455011916.png?Expires=4896891773&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=SWi5RMDSkHJ7uz%2BCLuoJslXeTtE%3D)

---

### 慢SQL优化
1. **避免使用`select *`**  
    查看执行计划，`select *` 走全表扫描，没有用到任何索引，查询效率非常低；查询列都是索引列那么这些列被称为覆盖索引。
    * 查询时需要先将星号解析成表的所有字段然后在查询，增加查询解析器的成本；
    * `select *` 查询一般不走覆盖索引会产生大量的回表查询；
    * 在实际应用中我们通常只需要使用某几个字段，其他不需要使用的字段也查出来浪费CPU、内存资源；
    * 文本数据、大字段数据数据传输增加网络消耗。
2. **使用limit**
    * 提高查询效率：一个查询返回成千上万的数据行，不仅占用了大量的系统资源，也会占用更多的网络带宽，影响查询效率。使用LIMIT可以限制返回的数据行数，减轻了系统负担，提高了查询效率。
    * 优化分页查询：分页查询需要查询所有的数据才能进行分页处理，这会浪费大量的系统资源和时间。使用LIMIT优化分页查询可以只查询需要的数据行，缩短查询时间，减少资源的浪费。
    * 简化查询结果：有时我们只需要一小部分数据来得出决策，而不是整个数据集。使用LIMIT可以使结果集更加精简和易于阅读和理解。
3. **小表驱动大表**  
    `Join Buffer（连接缓冲区）`是优化器用于处理连接查询操作时的临时缓冲区。简单来说当我们需要比较两个或多个表的数据进行Join操作时，Join Buffer可以帮助MySQL临时存储结果，以减少磁盘读取和CPU负担，提高查询效率。需要注意的是每个join都有一个单独的缓冲区。
    
    `Block nested-loop join（BNL算法）`会将驱动表数据加载到join buffer里面，然后再批量与非驱动表进行匹配；如果驱动表数据量较大，join buffer无法一次性装载驱动表的结果集，将会分阶段与被驱动表进行批量数据匹配，会增加被驱动表的扫描次数，从而降低查询效率。所以开发中要遵守小表驱动大表的原则。
4. **用连接查询代替子查询**  
    因为子查询需要执行两次数据库查询，一次是外部查询，一次是嵌套子查询。因此，使用连接查询可以减少数据库查询的次数，提高查询的效率。
   
    连接查询可以更好地利用数据库索引，提高查询的性能。子查询通常会使用临时表或内存表，而连接查询可以直接利用表上的索引。这意味着连接查询可以更快地访问表中的数据，减少查询的资源消耗。
5. **提升group by的效率**  
    创建索引：如果你使用group by的列没有索引，那么查询可能会变得很慢。因此，可以创建一个或多个适当的索引来加速查询。
6. **批量操作**  
    批量插入或批量删除数据，比如说现在需要将1w+数据插入到数据库，大家是一条一条处理还是批量操作呢？建议是批量操作，逐个处理会频繁的与数据库交互，损耗性能。
    ```java
    for(Order order: list){   
         orderMapper.insert(order):
    }
    ```
    ```sql
    insert into order(id,code,user_id)  values(123,'001',100)
    ```
   该操作需要多次请求数据库，才能完成这批数据的插入。可优化为：
   ```sql
   insert into order(id,code,user_id)  values(123,'001',100),(124,'002',100),(125,'003',101);
   ```
7. **用`union all`代替`union`**  
    如果当然它业务数据容许出现重复的记录，我们更推荐使用union all，因为union去重数据需要遍历、排序和比较，它更耗时，更消耗cpu资源，但是数据结果最完整。
   * `union all`：获取所有数据但是数据不去重，包含重复数据；
   * `union`：获取所有数据且数据去重，不包含重复数据；
8. **最左前缀法则**  
   如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引中的列.
9. **不在索引列上做任何操作（计算、函数、（自动or手动）类型转换），会导致索引失效而转向全表扫描**  
10. **不等空值还有or，索引失效要少用**  
    mysql在使用不等于（`！=`或者<>），`not in` ，`not exists`, `is null`,`is not null`,少用`or`或`in`, 此时无法使用索引会导致全表扫描
11. **Like百分写最右**  
    如果一定有`LIKE '%suffix'`的需求，可以使用反向索引：
    ```sql
    ALTER TABLE users ADD reversed_username VARCHAR(255);  
    UPDATE users SET reversed_username = REVERSE(username);  
    CREATE INDEX idx_reversed_username ON users(reversed_username);
    ```

12. **范围查询优化**  
    ```sql
    explain select * from employees where age >=1 and age <=2000; 
    ```
    没走索引原因：mysql内部优化器会根据检索比例、表大小等多个因素整体评估是否使用索引。比如这个例子，可能是由于单次数据量查询过大导致优化器最终选择不走索引  
    优化方法：可以将大的范围拆分成多个小范围
    ```sql
    explain select * from employees where age >=1 and age <=1000;
    explain select * from employees where age >=1001 and age <=2000;
    ```
    
13. 

---

### 深分页优化方案
MySQL通过Limit关键字实现分页查询，语法如下：
```sql
SELECT column_name(s) FROM table_name Limit offset, row_count;
```
其中，offset 表示起始偏移量，row_count 表示要返回的行数。在执行 SELECT 查询时，MySQL首先会先扫描整个表或使用索引，找到所有符合 WHERE 条件的记录。这个过程**需要将所有记录都读入内存**，然后根据 LIMIT 子句的指定返回查询结果集中的一部分。
* 按需查询字段，避免使用`select *`，减少网络IO消耗
* 查询的字段尽量保证索引覆盖
* 借助nosql缓存数据缓解MySQL数据库的压力
* 将偏移量改为使用Id限定的方式提升查询效率


```sql
SELECT * FROM user_login_log where id > 1000000 LIMIT 100;
```

说起数据库查询优化，第一时间想到的就是索引，所以便有了第二次优化：先查找出需要数据的索引列（假设为 id），再通过索引列查找出需要的数据。

```sql
Select * From table_name Where id in (Select id From table_name where ( user = xxx )) limit 10000, 10;

select * from table_name where( user = xxx ) limit 10000,10
```

在数据量大的时候 in 操作的效率就不怎么样了，我们需要把 in 操作替换掉，使用 join 就是一个不错的选择。
```sql
select * from table_name inner join ( select id from table_name where (user = xxx) limit 10000,10) b using (id)
```

---

### NULL 值的 5 大致命陷阱及解决方案

---

#### COUNT/DISTINCT 数据丢失
`COUNT` 是 MySQL 中常用的统计函数，但当字段值为 NULL 时，`COUNT(column)` 会忽略这些记录，导致统计结果不准确。
```sql
SELECT COUNT(name) AS name_count FROM person;
```
> 使用 COUNT(*)：统计所有记录，包括字段值为 NULL 的行。
> ```sql
> SELECT COUNT(*) AS name_count FROM person;
> ```

当使用 COUNT(DISTINCT column1, column2) 时，如果任意一个字段值为 NULL，即使另一列有不同的值，查询结果也会忽略这些记录。
```sql
SELECT COUNT(DISTINCT name, mobile) AS distinct_count FROM person;
```
> 使用`IFNULL`函数：通过 IFNULL(column, default_value) 将 NULL 转换为默认值，
> ```sql
> SELECT COUNT(DISTINCT IFNULL(name, ''), IFNULL(mobile, '')) AS distinct_count FROM person;
> ```

---

#### NULL 对比结果为未知（false）
在使用非等于查询（`<>` 或 `!=`）时，NULL 值的记录会被忽略。
```sql
SELECT * FROM person WHERE name != 'Alex';
```
> 显式处理 NULL
> ```sql
> SELECT * FROM person WHERE name != 'Alex' OR name IS NULL;
> ```

---

#### NULL 值运算都为 NULL
在使用 NULL 值进行运算时，比如加减乘除，拼接等等 最终的结果都为 NULL
```sql
SELECT id,concat(name,'name') FROM person;
```
> 使用`IFNULL`函数：通过 IFNULL(column, default_value) 将 NULL 转换为默认值，
> ```sql
> SELECT id,concat(IFNULL(name, ''),'name') FROM person;
> ```

---

#### 聚合空指针异常
在使用聚合函数（如 SUM、AVG）时，如果字段值为 NULL，查询结果也会为 NULL，而不是预期的 0。这可能导致程序在处理结果时出现空指针异常。
```sql
SELECT SUM(salary) AS total_salary FROM employee WHERE id >= 3;
```
> 使用`IFNULL`函数：通过 IFNULL(column, default_value) 将 NULL 转换为默认值，
> ```sql
> SELECT SUM(IFNULL(salary, 0)) AS total_salary FROM employee WHERE id >= 3;
> ```

---

#### Group By Order By 会统计 NULL 值
在使用 group by 与 order by 时，不会剔除 NULL，会将 NULL 作为最小值
```sql
SELECT * FROM person order by name desc;
```
> 使用 IS NOT NULL：剔除掉 NULL 记录，
> ```sql
> SELECT * FROM person where name is not null order by name desc;
> ```

---

### Binlog有几种录入格式与区别
* **Statement**格式  
  将SQL语句本身记录到Binlog中。记录的是在主库上执行的SQL语句，从库通过解析并执行相同的SQL来达到复制的目的。  
  在某些情况下，由于执行计划或函数等因素的影响，相同的SQL语句在主从库上执行结果可能不一致，导致复制错误。简单、易读，节省存储空间。
* **Row**格式  
  记录被修改的每一行数据的变化。不记录具体的SQL语句，而是记录每行数据的变动情况，如插入、删除、更新操作前后的值。  
  保证了复制的准确性，不受SQL语句执行结果的差异影响，适用于任何情况。相比Statement格式，Row格式会占用更多的存储空间。
* **Mixed**格式  
  Statement格式和Row格式的结合，MySQL自动选择适合的格式。大多数情况下使用Statement格式进行记录，但对于无法保证安全复制的情况，如使用非确定性函数、触发器等，会自动切换到Row格式进行记录。  
  结合了两种格式的优势，既减少了存储空间的占用，又保证了复制的准确性。

---

### NOT IN 为什么不会返回任何匹配 NULL 值的行
这个问题涉及到 SQL 中的三值逻辑，即真（TRUE）、假（FALSE）和未知（UNKNOWN）。

当你使用 NOT IN 条件时，如果其中包含 NULL 值，这会导致整个条件的结果不确定。这是因为 SQL 中的比较操作符（如 IN、NOT IN、=, !=等）对于 NULL 的处理方式是特殊的。具体来说：
> 如果一个值与 NULL 进行比较，结果是未知（UNKNOWN）。  
> 如果一个条件的结果是未知（UNKNOWN），那么整个条件的结果也是未知（UNKNOWN）。

因此，当你使用 NOT IN 条件时，如果其中包含 NULL 值，它会导致整个条件结果为未知（UNKNOWN）。在 SQL 中，任何未知（UNKNOWN）的条件都被视为不符合条件，因此相关的行将被过滤掉，不会包含在结果中。

这就是为什么在处理包含 NULL 值的情况时，需要谨慎地使用条件，以确保你的查询逻辑正确。
在这种情况下，使用 IS NULL 条件或者将 NULL 视为一个单独的选项。

---

### MySQL事务

---

#### 事务特性
* **原子性（Atomicity）**：事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么全不执行，不会出现部分执行的情况。
* **一致性（Consistency）**：执行事务前后，数据保持一致，多个事务对同一个数据读取的结果是相同的。
* **隔离性（Isolation）**：并发访问数据库时，事务的执行不会受到其他事务的干扰，即每个事务都应该像独立运行一样。
* **持久性（Durability）**：事务一旦提交，其结果就应该永久保存在数据库中，即使系统崩溃也不应该丢失。

---

#### 事务隔离级别
* **读未提交（read-uncommitted）**：最低的隔离级别，允许读取尚未提交的数据变更。可能会导致脏读、幻读或不可重复读。
* **读已提交（read-committed）**：允许读取并发事务已经提交的数据。可能会导致幻读或不可重复读。
* **可重复读（repeatable-read）**：同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改。可能会导致幻读。
* **串行化（serializable）**：最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰。

---

#### 事务隔离问题
* **脏读（Dirty Reads）**：事务A读取到了事务B已经修改但尚未提交的数据。
  * 事务A修改balance并且不提交事务，事务B读取balance值为900；
  * 如果此时事务A回滚数据，事务B读取balance值为1000（脏读）；
* **不可重读（Non-Repeatable Reads）**：事务A内部的相同查询语句在不同时刻读出的结果不一致。
  * 事务A修改balance并且不提交事务，事务B读取balance为1000；当事务A提交后，事务B读取balance值为900；
  * 再重新开启事务A修改balance并提交事务，事务B中在读取balance值为800(整个过程事务B都不提交)（不可重复读）；
* **幻读（Phantom Reads）**：事务A读取到了事务B提交的新增数据。
  * 事务A修改balance并且不提交事务，事务B读取balance为1000；当事务A提交后，事务B读取balance值为1000；
  * 开启事务A修改balance并提交事务，事务B中在读取balance值为1000（可重复读）(整个过程事务B都不提交)；
  * 开启事务A插入为2的记录，事务B**无法读取到2的记录**，此时修改id为2balance+1000，可以修改成功，**重新读取为2的记录balance为3000**（幻读）(整个过程事务B都不提交)

---

### MVCC机制的核心原理
MVCC全称（多版本并发控制），本质就是通过一种乐观锁的思想，维护数据的**多个版本**，以减少数据读写操锁的冲突，做到即使有读写冲突时也能做到不加锁，非阻塞并发读而这个读指的就是**快照读** , 而非**当前读**，这样就可以提高了 MySQL 的事务并发性能。

首先MySQL InnoDB存储引擎需要支持一条数据可以保留多个历史版本。  
对于使用 InnoDB 存储引擎的数据库表，它的聚簇索引记录中都包含下面两个隐藏列：
* trx_id，当一个事务对某条聚簇索引记录进行改动时，就会把该事务的事务 id 记录在 trx_id 隐藏列里；
* roll_pointer，每次对某条聚簇索引记录进行改动时，都会把旧版本的记录写入到 undo 日志中，然后这个隐藏列是个指针，指向每一个旧版本记录，于是就可以通过它找到修改前的记录。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-26/291040212581083.png?Expires=4896519182&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=D0V%2BcmNY2LjZ2xV%2Br4LrwyDsaD8%3D)

如上图所示，针对id=10001的这条数据，都会将旧值放到一条undo日志中，就算是该记录的一个旧版本，随着更新次数的增多，所有的版本都会被 roll_pointer 属性连接成一个链表，我们把这个链表称之为版本链，根据版本链就可以找到这条数据历史的版本。
![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-26/291060896145916.png?Expires=4896519202&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=5buVuHbskBsDslV4fdPkINnuNTw%3D)

利用undo log日志我们已经保留下了数据的各个版本，那么现在关键的问题是要读取哪个版本的数据呢？

这时就需要用到ReadView了，ReadView就是事务在使用MVCC机制进行快照读操作时产生的一致性视图, 比如默认隔离级别可重复读隔离级别（RR），在第一次查询的时候(注意这个细节，RC和RR区别关键在此)，创建一个ReadView, 那ReadView种都有哪些关键信息呢？

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-26/291133568259750.png?Expires=4896519275&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=7QPd6mTAd9xwP6R7FJw9YYTl4yk%3D)
* trx_ids: 指的是在创建 ReadView 时，当前数据库中「活跃事务」的事务 id 列表，注意是一个列表， “活跃事务”指的就是，启动了但还没提交的事务。
* min_trx_id: 指的是在创建 ReadView 时，当前数据库中「活跃事务」中事务 id 最小的事务，也就是 m_ids 的最小值。
* max_trx_id：这个并不是 m_ids 的最大值，而是创建 ReadView 时当前数据库中应该给下一个事务的 id 值，也就是全局事务中最大的事务 id 值 + 1；
* creator_trx_id ：指的是创建该 ReadView 的事务的事务 id, 只有在对表中的记录做改动时（执行INSERT、DELETE、UPDATE这些语句时）才会为 事务分配事务id，否则在一个只读事务中的事务id值都默认为0。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-26/291393648348250.png?Expires=4896519535&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=uIZJbExtYKIAIXhe5GRKIjYyWJY%3D)

* 如果被访问版本的trx_id属性值与ReadView中的 creator_trx_id 值相同，意味着当前事务在访问它自己修改过的记录，所以该版本可以被当前事务访问。
* 如果落在绿色部分，表示这个版本是已提交的事务或者是当前事务自己生成的，这个数据是可见的；
* 如果落在红色部分，表示这个版本是由将来启动的事务生成的，是肯定不可见的；
* 如果落在黄色部分，那就包括两种情况
  * 若 数据的trx_id在trx_ids数组中，表示这个版本是由还没提交的事务生成的，不可见, 去读取这条数据的历史版本，这条数据的历史版本中都包含了事务id信息，去查找第一个不在活跃事务数组的版本记录。
  * 若 数据的trx_id不在trx_ids数组中，表示这个版本是已经提交了的事务生成的，可见。

这种通过版本链 + 一致性视图 来控制并发事务访问同一个记录时的行为就叫 MVCC（多版本并发控制）。

---

### 为什么MySQL要默认使用RR隔离级别
Oracle的默认隔离级别是RC，而MySQL的默认隔离级别是RR。

主要是因为MySQL在主从复制的过程是通过bin log 进行数据同步的，而MySQL早期只有statement这种bin log格式，这种格式下，bin log记录的是SQL语句的原文。

当出现**事务乱序**的时候，就会导致备库在 SQL 回放之后，结果和主库内容不一致。

为了解决这个问题，MySQL默认采用了Repetable Read这种隔离级别，因为在 RR 中，会在更新数据的时候增加记录锁的同时增加间隙锁。可以避免这种情况的发生。

---

### MySQL 自增主键值一定是连续的吗

#### 自增值存储机制
1. MyISAM 引擎的自增值保存在数据文件中。
2. Innodb 引擎在 MySQL 5.7 及之前的版本，自增值保存在内存里。每次重启后，第一次打开表的时候，都会去找自增值的最大值 max(id)，然后将 max(id) + 1 作为这个表当前的自增值。
3. 在 MySQL 8.0 版本，将自增值的变更记录在了 redo log 中，重启的时候依靠 redo log 恢复重启之前的值。

---

#### 自增值修改机制
* 如果ID字段未指定具体的值，则将当前AUTO_INCREMENT值并将其填入自增字段，并生成新的自增值
* 如果ID字段已指定具体的值，则直接使用指定的值作为 ID 字段的值，而不会生成新的 AUTO_INCREMENT 值。
  * 如果插入值小于当前自增值，那么直接使用插入值填入ID字段，自增值不变；
  * 如果插入值大于当前自增值，那么除了直接使用插入值填入ID字段外，自增值需修改为插入值+1；
    > 上述”插入值+1‘不是直接使用”插入值“+1，是auto_increment_offset（自增初始值）以 auto_increment_increment（自增步长）为步长，持续累加，直到找到大于插入值的值，作为新的自增值。

---

#### 导致自增值不连续的原因
1. 唯一键冲突
    > 比如increnment_test中已经存在了col1为3的记录，我们继续插入col1为3的记录，此时会出现唯一键冲突插入报错，但是没有将自增值再改回去。重新插入col1为4的值，此时对应的id为5；
2. 事务回滚
    > 开启一个事务插入col1为6的数据，然后进行回滚。回滚后重新插入col1为6的记录，此时col1为6对应的id值为7。
3. 批量插入数据
   > 对于批量插入数据的语句，MySQL有一个批量申请自增 id 的策略：  
   > SQL语句执行过程中，第1次申请自增 id，会分配 1 个；  
   > 1 个用完以后，第2次申请自增 id，会分配 2 个；  
   > 2 个用完以后，第3次申请自增 id，会分配 4 个；  
   > 依此类推，同一个语句去申请自增 id，每次申请到的自增id个数都是上一次的两倍。

MySQL 8.0中的改进  
> 到了MySQL 8.0，对AUTO_INCREMENT的行为进行了优化，旨在减少上述提到的自增ID浪费的问题。具体来说：
> 
> 更高效的ID分配算法：MySQL 8.0引入了一个更加智能的ID分配策略，它试图根据实际需求来分配自增ID，而不是简单地按照固定模式增加。这有助于减少不必要的ID预留，从而降低ID浪费的程度。
> 
> 减少事务回滚导致的ID浪费：在MySQL 8.0中，通过改进事务管理和自增ID的分配逻辑，尽量减少了由于事务回滚而导致的自增ID浪费现象。这意味着即使在一个批量插入操作中只有部分记录成功插入，也不会像以前那样大量浪费未使用的自增ID。
---

### MySQL表设计经验
* 命名规范
  > 数据库表名、字段名、索引名等都需要命名规范。命名可读性要高，尽量使用英文，采用驼峰或者下划线分割的方式，让人见名知意。
* 选择合适的字段类型
  > * 根据数据类型选择字段类型：不同的数据类型应该使用不同的字段类型。  
  > * 考虑数据长度：字段类型应该根据所需存储的数据长度来选择。  
  > * 注意精度和小数位数：对于需要精确数值计算的字段（如货币和百分比），应该选择带有精度和小数位数的字段类型（如 DECIMAL ）。
  > * 考虑数据完整性：字段类型也应该考虑到数据完整性。
* 主键设计要合理
  > 应当使用自增的 id，并且保持主键的连续性。
* 选择合适的字段长度
  > 需要注意字段长度一般设置为2的n次方。
* 优先考虑逻辑删除，而不是物理删除
  > 物理删除数据恢复困难。会导致索引树重构。
* 每个表都需要添加通用字段
  > id： 主键，一个表必须得有主键，必须  
  > create_time： 创建时间  
  > creator ：创建人  
  > update_time: 修改时间，必须，更新记录时，需要更新它  
  > update_by :修改人，非必须  
  > remark ：数据记录备注，非必须  
* 一张表的字段不宜过多
  > 建表的时候一张表的字段不要太多了。尽量不超过 20 个。超出的话优先考虑拆分，也就是通常的查询表，详情表。
* 定义字段尽可能not null
  > 首先，NOT NULL 可以防止出现空指针问题。  
  > 其次，NULL值存储也需要额外的空间的，它也会导致比较运算更为复杂，使优化器难以优化SQL。  
  > NULL值有可能会导致索引失效。
* 合理添加索引
  > * 根据查询条件进行选择（高频使用）：如果在查询中使用了某个字段作为查询条件，那么这个字段就应该建立索引。  
  > * 区分度高的字段优先：如果一个字段的取值范围非常小，例如性别只有男女两种可能，那么这个字段就不适合建立索引。  
  > * 不要建立过多的索引：每个表所建立的索引数量应该控制在一个合理的范围内，一般不要超过5个。  
  > * 联合索引优化：在某些情况下，可以通过联合索引的方式来优化查询速度，减少所需的索引数量。
* 不需要严格遵守 3NF，通过业务字段冗余来减少表关联
  > 常见形式是在第三范式(3NF)的基础上，进一步进行冗余，从而减少表关联。  
  > 假设需要设计一个产品订单表，包含以下字段：订单ID、用户ID、订单日期、产品名称、产品价格、产品数量以及订单总价。正常情况下，可能会分别设计订单表和产品表，并使用外键进行关联  
  > 为了提高查询效率，我们可以使用反范式的设计方式，将订单表中的产品名称、产品价格和订单总价冗余存储到订单表中，从而避免关联查询。
* 避免使用MySQL保留字
  > 如果库名、表名、字段名等属性含有保留字时，SQL语句必须用反引号来引用属性名称，这将使得SQL语句书写、SHELL脚本中变量的转义等变得非常复杂。
  > ```text
  > ADD
  > ALL
  > ALTER
  > AND
  > AS
  > BETWEEN
  > BY
  > CASE
  > DELETE
  > FROM
  > GROUP
  > HAVING
  > INSERT
  > INTO
  > JOIN
  > LIKE
  > NOT
  > OR
  > SELECT
  > UPDATE
  > WHERE
  > ```
* 不搞外键关联，一般都在代码维护
  > * 可能会导致性能问题，尤其是在对大型数据集进行操作时。这是因为每次插入、更新或删除操作都需要进行约束检查，这可能会导致额外的开销和延迟。  
  > * 可能会限制数据库的灵活性和可扩展性。例如，如果需要对数据库进行分区或垂直分割，外键关联可能会导致额外的复杂性和限制。  
  > * 可能会导致死锁和死循环，特别是在进行并发操作时。这可能会导致数据库出现不稳定的状态，从而影响系统的性能和可用性。  
  > * 可能会导致数据库的维护和管理成本的增加。这是因为外键关联需要额外的管理和维护工作，例如添加、修改或删除外键约束时需要额外的测试和验证。
* 字段注释
  > 设计表时每个字段的含义要注释清楚，包括枚举类型。
* 时间的类型选择
  > * datetime：表示的日期时间值，格式yyyy-mm-dd hh:mm:ss，范围1000-01-01 00:00:00到9999-12-31 23:59:59，8字节，跟时区无关
  > * timestamp：表示的时间戳值，格式为yyyymmddhhmmss，范围1970-01-01 00:00:01到2038-01-19 03:14:07，4字节，跟时区有关

---

### 分库分表下如何实现精准分页
互联网中许多业务都需要进行分页拉取数据，比如电商商城系统的订单列表、贴吧社区系统的帖子回复、以及手机APP消息列表等。  
这些业务场景通常具有以下共性：数据量大、使用业务主键ID、分页排序通常不是按照主键排序，而是按照创建时间排序。  
在数据量较小的情况下，可以通过在排序字段时间上建立索引，利用SQL提供的offset/limit功能来满足分页查询需求。  
![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-27/353005963713000.png?Expires=4896618143&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=o0YSK%2BGUWCnq9kCRPaIkk0m2GZ8%3D)

服务层通过 id 取模将数据分布到两个库上去之后，每个数据库都失去了全局视野，数据按照 time 局部排序之后，不管哪个分库的第1000-1010条数据，都不一定是全局排序的第1000-1010条数据。  
假设现在需要根据订单的时间进行排序分页查询，查询第1000-1010条数据，单库内查询方法为：
```sql
select * from t_order order by time asc limit 1000,10;
```
分库后，具体查询方法有以下几种：

---

#### 全局查询法

要在每个表中将前两页的数据全部查询出来，即每个库找出第0-1010条数据，然后在内存中再次重新排序，最后从中取出数据，这就是全局查询法。

```sql
select * from t_order_1 order by time asc limit 0,1010;

select * from t_order_2 order by time asc limit 0,1010;
```

> 该方案的缺点非常明显：  
> 随着页码的增加，每个节点返回的数据会增多，性能非常低  
> 服务层需要进行二次排序，增加了服务层的计算量，如果数据过大，对内存和CPU的要求也非常高

---

#### 禁止跳页法

数据量很大时，可以禁止跳页查询，只提供下一页的查询方法。  
查询第二页的时候可以将上一页的最大值10010作为查询条件，此时的两个表中的SQL改写：

```sql
select * from t_order_1 where time>10010 order by time asc limit 10;

select * from t_order_2 where time>10010 order by time asc limit 10;
```

> 同样是需要在内存中再次进行重新排序，最后取出前10条数据。  
> 
> 但是好处就是不用返回前面的全部数据了，只需要返回一页数据，在页数很大的情况下也是一样，在性能上的提升非常大  
> 
> 此种方案的缺点也是非常明显：不能跳页查询，只能一页一页地查询，比如说从第一页直接跳到第五页，因为无法获取到第四页的最大值，所以这种跳页查询肯定是不行的。

---

#### 允许数据精度损失  
数据库分库-数据均衡原理
> 使用 partition key 进行分库，在数据量较大，数据分布足够随机的情况下，各分库所有非 partition key 属性，在各个分库上的数据分布，统计概率情况是一致的。
> 
> 性别属性，如果 db0 库上的男性用户占比 70%，则 db1 上男性用户占比也应为 70% ;  
> 年龄属性，如果 db0 库上 18-28 岁少女用户比例占比 15%，则 db1 上少女用户比例也应为 15% ;  
> 时间属性，如果 db0 库上每天 10:00 之前登录的用户占比为 20%，则 db1 上应该是相同的统计规律 ;

利用这一原理，要查询全局 100 页数据，offset 9900 limit 100 改写为 offset 4950 limit 50，每个分库偏移 4950（一半），获取 50 条数据（半页），得到的数据集的并集，基本能够认为，是全局数据的offset 9900 limit 100 的数据，当然，这一页数据的精度，并不是精准的。

```sql
select * from t_order_1 order by time asc limit 500,5;

select * from t_order_2 order by time asc limit 500,5;
```

---

#### 二次查询法
> 这是网上大部份教程的方法，但实际上这是错误的。  
> 同步于 https://zhuanlan.zhihu.com/p/1888471384864309385
> 
> 以下sql示例以四个库的情况来编写
1. **查询 SQL 改写**  
    将  
    `select * from t_order order by time asc limit 1000,10;`  
    改写为  
    `select * from t_order order by time asc limit 250,10;`  
    并投递给所有的分库。  
   > 注意，这个 offset 的 250，来自于全局offset的总偏移量 1000，除以水平切分数据库个数 4
2. **返回数据的最小值**  
  `t_order_1`：10条数据中最小值为：`10000`；  
  `t_order_2`：10条数据中最小值为：`20000`；  
  `t_order_3`：10条数据中最小值为：`30000`；  
  `t_order_4`：10条数据中最小值为：`40000`；  
  那么四张表中的最小值为`10000`，记为`time_min`，来自`t_order_1`这张表
3. **查询二次改写**  
   > 第二次的SQL改写也是非常简单，使用between语句，起点就是第2步返回的最小值`time_min`，终点就是每个表中在`第一次查询时的最小值`。

   `t_order_1`：`time_min` 位于第一个分库，直接不用查询了。  
   `t_order_2`：改写为 `select * from t_order order by time where time between time_min and 20000`；假设返回11000；  
   `t_order_3`：改写为 `select * from t_order order by time where time between time_min and 30000`；假设返回22000，23000；  
   `t_order_4`：改写为 `select * from t_order order by time where time between time_min and 40000`；假设返回空；  
4. **分析这二次查询结果**  
   * 我们最初的需求是要`select * from t_order order by time asc limit 1000,10;`找出第`1000-1010`条数据；
   * 然后因为分库的原因，对四个库`select * from t_order order by time asc limit 250,10;`，找出了每个库中第`250-260`条数据；
   * 此时我们得到最小值 `time_min`，所以可以确定`time_min`的offset一定大于等于250；
   * 第二次查询`t_order_2`有 `1` 条数据，说明`t_order_2`有 `248` 条数据在 `time_min` 之下；
   * 第二次查询`t_order_3`有 `2` 条数据，说明`t_order_3`有 `247` 条数据在 `time_min` 之下；
   * 第二次查询`t_order_4`没有数据；说明`t_order_4`有 `249` 条数据在 `time_min` 之下；
   * 所以`time_min`的offset确定为250+248+247+249=`994`
   > `t_order_1`有249条数据比`time_min`小；  
   > `t_order_2`的第250条数据都比`time_min`大，第二次查询只有`1`条数据比`time_min`大，所以`t_order_2`有248条数据比`time_min`小；  
   > `t_order_3`的第250条数据都比`time_min`大，第二次查询只有`2`条数据比`time_min`大，所以`t_order_3`有247条数据比`time_min`小；  
   > `t_order_4`的第250条数据都比`time_min`大，第二次查询没有数据比`time_min`大，所以`t_order_4`有249条数据比`time_min`小；  
   > 总共249+248+247+249=993条数据比`time_min`小，所以`time_min`的offset为994。
   * 结合两次查询的数据， `t_order_1`的`time_min`的offset为`994`；
   * 然后所有的教程都是说把这43个数据结合在一起排序得出1000-1010次的数据；
   * 但其实这里是错误的。
   > `t_order_1`的`time_min`的offset是`994`，那么`995`呢，一定是`t_order_2`的`11000`吗？  
   > 为什么不能是`t_order_1`的`10001`呢？  
   > 如果`t_order_1`的10条数据是`10000-10010`，那么我们只找到了`994-1004`的数据，
   > offset为1005的数据是`t_order_1`的10011，并不在这43个数据中！！！那就别提第`1000-1010`条了。

| 查询次数 | t_order_1 | t_order_2 | t_order_3 | t_order_4 | 本地offset | 全局offset |
|------|-----------|-----------|-----------|-----------|----------|----------|
| 第二次  |           |           | 22000     |           | <250     |          |
| 第二次  |           | 11000     | 23000     |           | <250     |          |
| 第一次  | 10000     | 20000     | 30000     | 40000     | 250      |          |
| 第一次  | 10001     | 20001     | 30001     | 40001     |          |          |
| 第一次  | 10002     | 20002     | 30002     | 40002     |          |          |
| 第一次  | 10003     | 20003     | 30003     | 40003     |          |          |
| 第一次  | 10004     | 20004     | 30004     | 40004     |          |          |
| 第一次  | 。。。       | 。。。       | 。。。       | 。。。       |          |          |
| 第一次  | 10010     | 20010     | 30010     | 40010     |          |          |

> 所以问题的关键在于第二次查询只能确定一个`time_min`的offset，这个offset并不能满足我们的需求，最后一步需要改进再查一次。


---

#### 三次查询法
> 第一次查询，大概确定offset为1000的数据；  
> 第二次查询，精确定位`time_min`的offset；  
> 第三次查询，根据`time_min`找到数据；  
1. **查询 SQL 改写**  
   将  
   `select * from t_order order by time asc limit 1000,10;`  
   改写为  
   `select * from t_order order by time asc limit 250,1;`  
   并投递给所有的分库。
   > 注意，这个 offset 的 250，来自于全局offset的总偏移量 1000，除以水平切分数据库个数 4  

2. **返回数据的最小值**  
   同二次查询法
3. **查询二次改写**  
   同二次查询法
4. **分析这二次查询结果**  
    同上
    * 结合两次查询的数据， `t_order_1`的`time_min`的offset为`994`；
    * ~~然后所有的教程都是说把这43个数据结合在一起排序得出1000-1010次的数据~~；
    * `time_min`的offset为994离我们的目标1010还有16条数据，要确保这16条数据**不会分布在一个表中**；
    * 所以要进行第三次查询。
5. **第三次查询**  
    每个库中进行查询
```sql
select * from t_order order by time where time > time_min limit 16;
```
这里再把数据结合起来，以`time_min`的offset为`994`为基石进行排序，找出1000-1010条数据。

---

### 加密后的数据该如何支持模糊查询
在日常工作中，我们经常会有一些模糊查询的条件，比如说按照手机号模糊查询，或者是身份证号码。正常情况下我们可以使用模糊查询
```sql
select * from user where mobile like %123%
```
但是这种方式是在你的手机号码没有加密的前提下，但是对于一些用户私密数据，我们在数据库都会进行加密保存。加密后的数据就不支持模糊查询了。
1. 内存解密(数据量少可用，数据量多不推荐)  
   将所有数据都查询出来，然后在内存中将所有的手机号进行解密，然后在做一个匹配
2. 分片加密
   密文检索的功能实现是根据4位英文字符（半角），2个中文字符（全角）为一个检索条件。将一个字段拆分为多个。  
   比如：taobao123  
   使用4个字符为一组的加密方式。  
   第一组 taob ，第二组aoba ，第三组obao ，第四组 bao1 … 依次类推  
   如果需要检索 所有包含 检索条件4个字符的数据 比如：aoba ，加密字符后通过key like “%partial%” 查库。  
   因为密文检索开启后 密文长度会膨胀几倍以上，如果没有强需求建议不开启。  

---

### count（*）很慢，具体如何提升性能
1. **查询系统表**
   如果业务需要的统计结果不需要特别精确，可以查询系统表 information_schema.tables，这个存在误差但查询非常快。
    ```sql
    EXPLAIN SELECT COUNT(*) FROM user_innodb where id>0;
    
    SELECT *
    FROM information_schema.tables
    WHERE Table_schema = 'demo'
      AND table_name = 'user_innodb';
    ```

    ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-31/516337055330166.png?Expires=4896970438&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=2YuVLexHNbVbeqCLiELtVA2o0xI%3D)

2. explain分析sql看是否有优化的余地，最好能走主键索引  
   通过上面explain可以看到 key走的不是primary ，加上过滤条件id>0速度能提升一截

   ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-03-31/516384173157208.png?Expires=4896970485&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=bGRS4rGXL9VwB1MOQRfU1mCrD0c%3D)

3. 分批查询或者数据分片汇总统计  
   如果查询的结果集很大，可以考虑将查询分批进行，每次查询一部分数据，然后累加结果。这样可以减少单次查询的数据量，提高查询速度。  
   如果数据量非常大，可以考虑将数据进行分片存储，将数据分散到多个表或数据库中。这样可以将查询的数据量分散到多个节点上，提高查询性能。

4. 使用缓存或者直接新建统计表  
   如果查询的结果不需要实时更新，可以将结果缓存在缓存中，避免每次查询都执行COUNT(*)操作。可以使用缓存技术如Redis或Memcached来实现。  
   如果对缓存的统计还不满意，可以新建一张统计表  
   如果是想精确的获取表的记录总数，我们可以将这个计数值保存到单独的一张计数表中。  
   当我们在数据表插入一条记录的同时，将计数表中的计数字段 + 1。也就是说，在新增和删除操作时，我们需要额外维护这个计数表。

---

### MySQL 8的索引跳跃扫描是什么
MySQL中的索引跳跃扫描（Skip Scan）是一种优化策略，旨在提高在特定条件下对复合索引的利用效率。尽管MySQL并没有明确标记这项技术为"Skip Scan"功能，但通过其优化器的改进，特别是在MySQL 8.0.13及其后的版本中，MySQL可以在某些条件下实现类似Skip Scan的效果。让我们通过一个具体示例来说明其概念和潜在应用。

假设有一个包含以下数据结构的表employees：

| ID | Department | Position  |
|----|------------|-----------|
| 1  | Sales      | Manager   |
| 2  | Sales      | Executive |
| 3  | HR         | Manager   |
| 4  | HR         | Executive |
| 5  | IT         | Manager   |
| 6  | IT         | Executive |

假设我们希望查找所有Manager职位的员工：

* 传统索引处理  
  如果正常按索引(Department, Position)的顺序使用，理想情况下需要首先给出Department条件，以充分利用索引优势，但本查询并没有提供对Department的条件。
* 跳跃扫描的优化实现（概念性）
  1. **分区扫描：**  
  将索引按Department的唯一值进行分割。每个分区相当于不同的部门，比如Sales、HR、IT。
  2. **应用条件：**  
  对每个Department分区中的Position行进行扫描，即跳跃扫描中仅搜索Position = 'Manager'的员工。
  3. **跳过无关分区：**  
  由于Position = 'Manager'限定，我们只在每个Department的分区中寻找匹配的Position，而不是对整个表进行扫描。


> 虽然MySQL索引跳跃扫描概念上并不是一个独立标识的特性，但此类索引优化策略在新版MySQL中可能通过改进的索引条件推送和查询优化器来实现。它潜在地减少读取和滤除不必要数据的开销。
> 这种策略在数据有较低基数（即Department的唯一值少）的场景中特别有效，因为跳跃的次数减少，提高了效率。可通过 EXPLAIN 检查特定查询的实际执行计划以确认优化的应用。

---


