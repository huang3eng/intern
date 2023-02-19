# SQL



## 刷题

- 按组同时统计两种状态出现的次数

  ```mysql
  select
      question_id,
      sum(if(action = 'answer', 1, 0)) as AnswerCnt,
      sum(if(action = 'show', 1, 0)) as ShowCnt
  from
      SurveyLog
  group by question_id
  ```

- 按组内个数进行逆序排序

  ```mysql
  select candidateId
  from Vote
  group by candidateId
  order by count(candidateId) desc # 用count(*)会慢很多
  ```

  

## 概念

- 事务：**是逻辑上的一组操作，要么都执行，要么都不执行。**

## 常用子句

### [窗口函数](https://zhuanlan.zhihu.com/p/92654574)

#### 基本用法

```sql
<窗口函数> over (partition by <用于分组的列名> order by <用于排序的列名>)
# 窗口函数的位置可以放专用窗口函数和聚合函数
# 窗口函数是对where或者group by子句处理后的结果进行操作，所以窗口函数原则上只能写在select子句中。
```

#### 专用窗口函数

```sql
# 专用窗口函数：rank, dense_rank, row_number
# 12 13 13 14
  # rank() over(condition) 1 2 2 4  注意rank() over排序空值最大
  # dense_rank(condition) over 1 2 2 3
  # row_number() over 1 2 3 4
# partition by用来对表分组,order by子句的功能是对分组后的结果进行排序
# partition by和rank函数不会减少原表中的行数。group by分组汇总后改变了表的行数，一行只有一个类别。
```

#### 聚合函数

```sql
# 聚合函数：sum. avg, count, max, min
# 聚合函数作为窗口函数，可以在每一行的数据里直观的看到，截止到本行数据，统计数据是多少（最大值、最小值等）
```

### limit

- 用于限制查询结果返回的数量，常用于分页查询

  ```sql
  limit len ->[0,len)
  limit i,len -> [i,i+len)
  limit len offset j [j,j+len)
  ```

### distinct

- distinct用来查询不重复记录的条数
- distinct 【查询字段】，必须放在要查询字段的**开头**，即放在第一个参数；
- 只能在**SELECT** 语句中使用，不能在 INSERT, DELETE, UPDATE 中使用；
- DISTINCT 表示对后面的**所有参数的拼接**取不重复的记录，即查出的参数拼接每行记录都是唯一的
- 不能与all同时使用，默认情况下，查询时返回的就是所有的结果

### join

- 连接关系图

<img src="https://www.runoob.com/wp-content/uploads/2019/01/sql-join.png" alt="img" style="zoom:50%;" />

### where/having区别

- where 子句	
  - 对查询结果进行分组前，将不符合 where 条件的行去掉，即在分组之前过滤数据，即先过滤再分组。
  - where 后面不可以使用聚合函数
  - 过滤行


- having 子句	
  - having 子句的作用是筛选满足条件的组，即在分组之后过滤数据，即先分组再过滤。
  - having 后面可以使用聚合函数
  - 过滤组
  - 支持所有WHERE操作符

### orderby

- 后面可以用聚合函数
  - [第三种做法](https://leetcode.cn/problems/get-highest-answer-rate-question/solution/cha-xun-hui-da-lu-zui-gao-de-wen-ti-by-leetcode-so/)

## 其他

- 派生表

  ```sql
  select max(num) as num
  from (
      select num
      from MyNumbers
      group by num
      having count(num) = 1
  ) t
  
  -- 这里t就是派生表，可避免使用临时表
  ```

- 顺序：where > group by > having > order by

- 联合键子查询

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/1651745557-JqFxKG-image.png" alt="image.png" style="zoom: 67%;" />

### 慢查询优化经验

- 在实际生产中，面对千万上亿级别的数据，连接的效率往往最高，因为用到索引的概率较高

### MVCC

多版本并发控制 todo

### JPA

Java Persistence API todo

### MyBatis



## docs

- https://cstack.github.io/db_tutorial/

## 八股文

### 引擎

- MySQL 5.5 之前，MyISAM 引擎是 MySQL 的默认存储引擎，5.5.5 版本之后，InnoDB 是 MySQL 的默认存储引擎。
- 对比
  - MyISAM 只有表级锁(table-level locking)，而 InnoDB 支持行级锁(row-level locking)和表级锁,默认为行级锁。
  - MyISAM 不提供事务支持。InnoDB 提供事务支持，实现了 SQL 标准定义了四个隔离级别，具有提交(commit)和回滚(rollback)事务的能力。并且，InnoDB 默认使用的 REPEATABLE-READ（可重读）隔离级别是可以解决幻读问题发生的（基于 MVCC 和 Next-Key Lock）。
  - MyISAM 不支持**外键**，而 InnoDB 支持。外键对于维护数据一致性非常有帮助，但是对性能有一定的损耗。[阿里的《Java 开发手册》也是明确规定禁止使用外键的。](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/mysql/image-20220510090309427.png)
  - MyISAM 不支持**数据库异常崩溃后的安全恢复**，而 InnoDB 支持。
  - MyISAM 不支持**MVCC**，而 InnoDB 支持。
  - MyISAM，不管主键还是非主键，都是非聚簇索引。InnoDB主键索引就属于聚簇索引，非主键索引为辅助索引或者二级索引。

### 索引

#### 数据结构

- Hash 索引不支持顺序和范围查询
- B树只适合随机检索,而B+树同时支持随机检索和顺序检索

#### 引擎索引结构

- MyISAM，不管主键还是非主键，都是非聚簇索引。InnoDB主键索引就属于聚簇索引，非主键索引为辅助索引或者二级索引。

#### 索引类型

- 主键索引：索引的叶子节点存储的是数据或者数据的地址

- 辅助索引：辅助索引的叶子节点存储的数据是主键。

- 聚簇索引：索引的叶子节点包含完整的行数据

- 非聚簇索引：非聚集索引的叶子节点存储的是每行数据的辅助索引键 + 该行数据对应的聚集索引键（主键值）

- 联合索引：对表上的多个列合起来做一个索引。联合索引的键值得数量不是1，而是>=2。

  ```sql
  # idx_a_b
  mysql> create table t(
      -> a int,
      -> b int,
      -> primary key(a),
      -> key idx_a_b(a,b)
      -> );
  Query OK, 0 rows affected (0.11 sec)
  
  ```

  

- 覆盖索引：从辅助索引中就可以得到查询记录，而不需要查询聚集索引中的记录。通常的[实现手段](https://zhuanlan.zhihu.com/p/107125866)是将被查询的字段，建立到**联合索引**里去。适合使用索引覆盖的常见场景：全表count查询优化、列查询回表优化、分页查询。

### ACID

#### 原子性（Atomicity）

- 一个事务必须是一系列操作的最小单元，这系列操作的过程中，要么整个执行，要么整个回滚，不存在只执行了其中某一个或者某几个步骤。

#### 隔离性（Isolation）

- 隔离性是说两个事务的执行都是独立隔离开来的，事务之前不会相互影响，多个事务操作一个对象时会以串行等待的方式保证事务相互之间是隔离的

#### 一致性（Consistency）

- 事务要保证数据库整体数据的完整性和业务数据的一致性，事务成功提交整体数据修改，事务错误则回滚到数据回到原来的状态。
- 原子性其实并不能保证一致性的。再多个事务并行进行的情况下，即使保证每一个事务的原子性，任然可能导致数据不一致的结果。为了保证并发情况下的一致性，引入了隔离性，即保证每一个事务能够看到的数据总是一致的，就好象其它并发事务并不存在一样。**用术语来说，就是多个事务并发执行后的状态，和它们串行执行后的状态是等价的。**
  - [利用两种锁实现隔离性](https://www.zhihu.com/question/30272728/answer/132403859)
    - 一种是悲观锁，即当前事务将所有涉及操作的对象加锁，操作完成后释放给其它对象使用。为了尽可能提高性能，发明了各种粒度(数据库级/表级/行级... /各种性质(共享锁/排他锁/共享意向锁/排他意向锁/共享排他意向锁...)的锁。为了解决死锁问题，又发明了两阶段锁协议死锁检测等一系列的技术。
    - 一种是乐观锁，**即不同的事务可以同时看到同一对象（一般是数据行）的不同历史版本。**如果有两个事务同时修改了同一数据行，那么在较晚的事务提交时进行冲突检测。

#### D(Durability）持久性

- 持久性是指一旦事务成功提交后，只要修改的数据都会进行持久化（通常是指数据成功保存到磁盘），不会因为异常、宕机而造成数据错误或丢失。

### 事务隔离

#### 事务隔离级别

- **READ-UNCOMMITTED(读取未提交)** ： 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。
- **READ-COMMITTED(读取已提交)** ： 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。
- **REPEATABLE-READ(可重复读)** ： 对同一字段的多次读取结果都是一致的，**除非数据是被本身事务自己所修改**，可以阻止脏读和不可重复读，但幻读仍有可能发生。MySQL InnoDB 存储引擎的**默认支持的隔离级别**。
- **SERIALIZABLE(可串行化)** ： 最高的隔离级别，完全服从 ACID 的隔离级别。**所有的事务依次逐个执行**，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

#### 脏读

- 定义：在事务1中，读取到了事务2未提交的数据
- 解决：**READ-COMMITTED(读取已提交)**

#### 不可重复读

- 定义：在事务1中，读取两次，但是结果不同。因为事务2在两次读取中间更新了事务并提交。
- 解决：**REPEATABLE-READ(可重复读)**

#### 幻读

- 定义：并不是说事务中多次读取获取的结果集不同，幻读更重要的是某次的 select 操作得到的结果集所表征的数据状态无法支撑后续的业务操作。
- 解决
  - **SERIALIZABLE(可串行化)**
  -  通过对select操作手动加行锁(SELECT ... FOR UPDATE )。原因是InnoDB中行锁锁定的
    是索引，纵然当前记录不存在，当前事务也会获得一把记录锁（记录存在就加行锁，不
    存在就加next-key lock间隙锁），这样其他事务则无法插入此索引的记录，杜绝幻
    读。
- 区别：所以“不可重复读”和“幻读”都是读的过程中数据前后不一致，只是前者侧重于修改，后者侧重于增删。个人认为，严格来讲“幻读”可以被称为“不可重复读”的一种特殊情况，没错的。但是从数据库管理的角度来看二者是有区别的。解决“不可重复读”只要加行级锁就可以了。而解决“幻读”则需要加表级锁，或者采用其他更复杂的技术，总之代价要大许多。这是搞数据库的那帮家伙非要把这两者区分开的动机吧。

### [三个日志](https://www.cnblogs.com/xiaolincoding/p/16396502.html)

#### undo log（回滚日志）

> 是 Innodb 存储引擎层生成的日志，实现了事务中的**原子性**，主要**用于事务回滚和 MVCC**

- 实现
  - 在事务**执行过程中**，都记录下回滚时需要的信息到一个日志里，那么在事务执行中途发生了 MySQL 崩溃后，就不用担心无法回滚到事务之前的数据，我们可以通过这个日志回滚到事务之前的数据。
- 「读提交」和「可重复读」隔离级别的事务来说，它们的快照读（普通 select 语句）是通过 Read View + undo log 来实现的
  - 「读提交」隔离级别是在**每个 select 都会生成一个新的 Read View**，也意味着，事务期间的多次读取同一条数据，前后两次读的数据可能会出现不一致，因为可能这期间另外一个事务修改了该记录，并提交了事务。
  - 「可重复读」隔离级别是启动事务时生成一个 Read View，然后**整个事务期间都在用这个 Read View**，这样就保证了在事务期间读到的数据都是事务启动前的记录。

#### redo log（重做日志）

> 是 Innodb 存储引擎层生成的日志，实现了事务中的**持久性**，主要**用于掉电等故障恢复**；

- 实现
  - redo log 是物理日志，**记录了某个数据页做了什么修改**，对 XXX 表空间中的 YYY 数据页 ZZZ 偏移量的地方做了AAA 更新，每当执行一个事务就会产生这样的一条物理日志。
  - **在事务提交时**，只要先将 redo log 持久化到磁盘即可，可以不需要将缓存在 Buffer Pool 里的脏页数据持久化到磁盘。当系统崩溃时，虽然脏页数据没有持久化，但是 redo log 已经持久化，接着 MySQL 重启后，可以根据 redo log 的内容，将所有数据恢复到最新的状态。

- **undo log和redo log**

  - redo log 记录了此次事务**「完成后」**的数据状态，记录的是更新之**「后」**的值

  - undo log 记录了此次事务**「开始前」**的数据状态，记录的是更新之**「前」**的值

  - 事务提交之前发生了崩溃，重启后会通过 undo log 回滚事务，事务提交之后发生了崩溃，重启后会通过 redo log 恢复事务

#### binlog （归档日志）

> 是 Server 层生成的日志，主要**用于数据备份和主从复制**；

- 实现
  - MySQL 在完成一条更新操作后，Server 层还会生成一条 binlog，**等之后事务提交的时候**，会将该事物执行过程中产生的所有 binlog 统一写 入 binlog 文件。

- **bin log和redo log**

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221106164244739.png" alt="image-20221106164244739" style="zoom:30%;" />

- 两阶段提交

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221106164522250.png" alt="image-20221106164522250" style="zoom:30%;" />

- **两阶段提交是以 binlog 写成功为事务提交成功的标识**
  - **如果 binlog 中没有当前内部 XA 事务的 XID，说明 redolog 完成刷盘，但是 binlog 还没有刷盘，则回滚事务**。对应时刻 A 崩溃恢复的情况。
  - **如果 binlog 中有当前内部 XA 事务的 XID，说明 redolog 和 binlog 都已经完成了刷盘，则提交事务**。对应时刻 B 崩溃恢复的情况。

### MVCC

#### 隐藏字段

> `InnoDB` 存储引擎为**每行数据**添加了三个隐藏字段

- `DB_TRX_ID（6字节）`：表示最后一次插入或更新该行的事务 id。此外，`delete` 操作在内部被视为更新，只不过会在记录头 `Record header` 中的 `deleted_flag` 字段将其标记为已删除
- `DB_ROLL_PTR（7字节）` 回滚指针，指向该行的 `undo log` 。如果该行未被更新，则为空
- `DB_ROW_ID（6字节）`：如果没有设置主键且该表没有唯一非空索引时，`InnoDB` 会使用该 id 来生成聚簇索引

#### ReadView

- `m_low_limit_id`：目前出现过的最大的事务 ID+1，即下一个将被分配的事务 ID。大于等于这个 ID 的数据版本均不可见
- `m_up_limit_id`：活跃事务列表 `m_ids` 中最小的事务 ID，如果 `m_ids` 为空，则 `m_up_limit_id` 为 `m_low_limit_id`。小于这个 ID 的数据版本均可见
- `m_ids`：`Read View` 创建时其他未提交的活跃事务 ID 列表。创建 `Read View`时，将当前未提交事务 ID 记录下来，后续即使它们修改了记录行的值，对于当前事务也是不可见的。`m_ids` 不包括当前事务自己和已提交的事务（正在内存中）
- `m_creator_trx_id`：创建该 `Read View` 的事务 ID

#### undo log

> 当前查找的 Record 不满足事务的可见性，我们需要通过 Undo Log 回溯该 Record 在 MVCC 中满足可见性的数据版本。

#### 比较规则

- 将`DB_TRX_ID`和`ReadView`中的字段进行比较，比较规则如下。

  <img src="https://leviathan.vip/2019/03/20/InnoDB的事务分析-MVCC/trans_visible.jpg" alt="trans_visible" style="zoom:50%;" />

- 如果不满足可见性，在该记录行的 `DB_ROLL_PTR` 指针所指向的 `undo log` 取出快照记录，用快照记录的 `DB_TRX_ID` 再利用上诉规则判断，直到找到满足的快照版本或返回空。

#### RC和RR的MVCC实现

- 在 RC 隔离级别下的 **`每次select`** 查询前都生成一个`Read View` (m_ids 列表)
- 在 RR 隔离级别下只在事务开始后 **`第一次select`** 数据前生成一个`Read View`（m_ids 列表）

#### 防止幻读

- **执行普通 `select`，此时会以 `MVCC` 快照读的方式读取数据**

  在快照读的情况下，RR 隔离级别只会在事务开启后的第一次查询生成 `Read View` ，并使用至事务提交。所以在生成 `Read View` 之后其它事务所做的更新、插入记录版本对当前事务并不可见，实现了可重复读和防止快照读下的 “幻读”

- **执行 select...for update/lock in share mode、insert、update、delete 等当前读**

  `InnoDB` 使用 [Next-key Lockopen](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html#innodb-next-key-locks) 来防止这种情况。当执行当前读时，会锁定读取到的记录的同时，锁定它们的间隙，防止其它事务在查询范围内插入数据。只要我不让你插入，就不会发生幻读

### 锁

#### 表级锁

-  S锁 、 X锁
-  IS锁 、 IX锁
- AUTO-INC锁：设计InnoDB的大叔提供了一个称之为innodb_autoinc_lock_mode的系统变量来控制到底使用上 述两种方式中的哪种来为AUTO_INCREMENT修饰的列进行赋值，当innodb_autoinc_lock_mode值 为0时，一律采用AUTO-INC锁;当innodb_autoinc_lock_mode值为2时，一律采用轻量级锁;当 innodb_autoinc_lock_mode值为1时，两种方式混着来(也就是在插入记录数量确定时采用轻 量级锁，不确定时使用AUTO-INC锁)。不过当innodb_autoinc_lock_mode值为2时，可能会造 成不同事务中的插入语句为AUTO_INCREMENT修饰的列生成的值是交叉的，在有主从复制的场景 中是不安全的。

#### 行级锁

- record locks：仅仅把一条记录锁上，是有 S锁 和 X锁 之分。
- gap locks：事务在第一次执行读取操作时，那些幻影记录尚不存在，我们无法给这些幻影记录加上record locks 。gap locks仅仅是为了防止插入幻影记录而提出的。
- next-key locks：既锁住某条记录，又阻止其他事务在该记录前边的间隙插入新记录。
- insert intention locks：设计 InnoDB 的大叔规定事务在等待的时候也需要在内存中生成一个 锁结构 ，表明有事务想在某 个 间隙 中插入新记录，但是现在在等待。
- 隐式锁（todo）



