# MySQL 优化

## 信息来源

[MySQL 优化常用的 19 种有效方法 (推荐!)](https://www.jb51.net/article/240990.htm)

[一文带你看懂 MySQL 存储引擎](https://blog.csdn.net/lzb348110175/article/details/106555504)

[MySQL 索引原理以及查询优化](https://www.cnblogs.com/bypp/p/7755307.html)

[Java 全栈知识体系](https://www.pdai.tech/md/db/sql-mysql/sql-mysql-b-tree.html)

## 创表优化

### 正确的数据类型

Mysql 是一种关系型数据库，可以很好地支持大数据量的存储，但是一般来说，数据库中的表越小，在它上面执行的查询也就越快。因此，在创建表的时候，为了获得更好的性能，我们可以将表中字段的宽度舍得尽可能小。

1. 如果存储的字符串长度几乎相等，使用 char 定长字符串类型。
2. varchar 是可变长字符串，不预先分配存储空间，长度不要超过 5000，如果存储长
度大于此值，定义字段类型为 text，独立出来一张表，用主键来对应，避免影响其它字段索
引效率。
3. 字段允许适当冗余，以提高查询性能，但必须考虑数据一致。冗余字段应遵循：
    - 不是频繁修改的字段。
    - 不是 varchar 超长字段，更不能是 text 字段。

    正例：商品类目名称使用频率高，字段长度短，名称基本一成不变，可在相关联的表中冗余存储类目名称，避免关联查询。
4. 任何字段如果为非负数，必须是 unsigned
5. 表达是与否概念的字段，必须使用 is_xxx 的方式命名，数据类型是 unsigned tinyint（1 表示是，0 表示否）。
6. 选择合适的字符存储长度，不但节约数据库表空间、节约索引存储，更重要的是提升检索速度。
    - 例如 人的年龄 150 岁之内，可以使用 `unsigned tinyint`  储存
7. 对于经常变更的数据来说，`CHAR` 比 `VARCHAR` 更好，因为 `CHAR` 不容易产生碎片

8. 尽量避免使用TEXT/BLOB类型，查询时会使用临时表，导致严重的性能开销。
9. 日期和时间类型，尽量使用 `timestamp`，空间效率高于 `datetime`，用整数保存时间戳通常不方便处理。如果需要存储微妙，可以使用 `bigint` 存储。

### 默认值

所有的字段都应该设置默认值，并且需要设置 not null
我们应该尽量将每个字段都设置默认值，且设置字段为 not null

下面是一些常用默认值

- 数值类型：包括 INT、BIGINT、DECIMAL 等）：默认值为 0

- 字符串类型（包括 CHAR、VARCHAR 等）：默认值为 ''（空字符串）。

- 日期类型（DATE、TIME、DATETIME、TIMESTAMP 等）我实验了 MySQL8.0，只有 DATE、DATETIME、TIMESTAMP 可以加上默认值，并且 DATE 类型加默认值需要加一个括号，本身是不支持默认值的，而且不支持 ON UPDATE，如下

    ```sql
    create table d(
        create_time   DATE      NOT NULL DEFAULT (curdate()),
        update_time   DATE      NOT NULL DEFAULT (curdate()),
        datetime_col2 DATETIME  NOT NULL DEFAULT CURRENT_TIMESTAMP,
        timestamp_col TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    )
    ```

### 必要时选择合适的引擎

MySQL 存储引擎是指定在表之上的，即一个库中的每一个表都可以指定选择存储引擎

MySQL 常用引擎有五种：`InnoDB`、`MyISAM`、`CSV`、`Archive`、`Memory`

- InnoDB：MySQL5.5 版本及以上默认存储引擎，支持事务、行级锁定、外键约束等高级功能。

- MyISAM：MySQL 默认存储引擎中最古老、最稳定的一种，适用于大量读取但少量写入的应用场景，不支持事务和外键。

- Memory：将数据存储在内存中，适用于缓存常用的临时数据。不支持持久化数据，重启服务器后数据会消失。

- Archive：适用于非常大的、几乎不会被更新的归档数据。压缩比非常高，但是不支持索引、删除和更新操作，只支持插入和查询操作，而且不支持事务和外键。

CSV：将数据以逗号分隔值格式存储在文本文件中。适用于数据交换和数据导入导出，但是不支持索引、事务和外键。

> 在 MySQL 5.5 之前默认的存储引擎是 MyISAM，5.5 版本及以后默认存储引擎修改为 InnoDB

在工作中，经常用到的还是 InnoDB 和 MyISAM 这两种

InnoDB 和 MyISAM 的区别：

- InnoDB 支持事务，MyISAM 不支持
- InnoDB 支持外键，而 MyISAM 不支持
- InnoDB 是聚集索引，使用 B+Tree 作为索引结构，数据文件是和（主键）索引绑在一起的；MyISAM 是非聚集索引，它也是使用 B+Tree 作为索引结构，但是索引和数据文件是分离的，索引保存的是数据文件的指针。
- InnoDB 必须要有主键，MyISAM 可以没有主键；InnoDB 如果我们没有明确去指定创建主键索引。它会帮我们隐藏的生成一个 6 byte 的 int 型的索引作为主键索引
- InnoDB 辅助索引和主键索引之间存在层级关系；MyISAM 辅助索引和主键索引则是平级关系。
- InnoDB 不保存表的具体行数，执行 select count(*) from table 时需要全表扫描。而 MyISAM 用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快（注意不能加有任何 WHERE 条件）；
- MySQL 5.7 之前 Innodb 不支持全文索引
- InnoDB 支持表级锁、行级锁，默认为行级锁；而 MyISAM 仅支持表级锁。InnoDB 的行锁是实现在索引上的，而不是锁在物理行上。如果访问未命中索引，也是无法使用行锁，将会退化为表锁
- MyISAM 崩溃后发生损坏的概率比 InnoDB 高很多，而且恢复的速度也更慢
- MyISAM 支持压缩表和空间数据索引

如何选择：

- **默认使用 InnoDB，只有在需要它不支持的特性时，才考虑使用其它存储引擎**
- 判断是否需要支持事务，如果要请选择 InnoDB，如果不需要可以考虑 MyISAM；
- 如果表中绝大多数都只是读查询，可以考虑 MyISAM，如果既有读也有写，那还是使用 InnoDB 吧。
- MySQL5.5 版本开始 InnoDB 已经成为 MysQL 的默认引擎 (之前是 MyISAM)，说明其优势是有目共睹的，如果你不知道用什么，那就用 InnoDB，至少不会差。

## SQL 优化

### Explain

MySQL 中的 EXPLAIN 是一个查询优化工具，它可以用来分析 SQL 语句的执行计划。通过 EXPLAIN 命令，可以查看 MySQL 数据库系统如何处理查询，并且获取到查询执行过程中的一些关键统计指标，如表扫描次数、索引使用情况、排序操作等等。

Explain 作用：

- 查看表的读取顺序
- 查看数据库读取操作的操作类型
- 查看哪些索引有可能被用到
- 查看哪些索引真正被用到
- 查看表之间的引用
- 查看表中有多少行记录被优化器查询

使用方法：

```sql
explain  sql语句
```

explain 中的列：

1. id: 每个 SELECT 子句或者操作的唯一标识符，id 相同执行顺序从上到下，Id 值不同时 id 值越大优先级越高

2. select_type：查询类型，有以下几种类型
    - SIMPLE：简单 SELECT 查询，不使用 UNION 或子查询等。
    - PRIMARY：最外层的 SELECT 查询，包括任何子查询
    - SUBQUERY：SELECT 中的子查询。
    - DERIVED：派生表的 SELECT，例如 FROM (SELECT ...) AS t
    - UNION：UNION 中的第二个或后面的 SELECT 语句
    - UNION RESULT：UNION 的结果集
3. table：显示了连接的表名字。
4. partitions：表的分区情况
5. type：访问类型，是衡量查询效率的一个重要指标，常见的值有（从好到坏）：
    - system：这是 type 类别最好的一种，系统表只有一行数据，可以快速返回，基本不会出现。
    - const：也称作“常量”，查询时优化器发现使用一个索引可以将表中只有一行数据的情况，这时使用 PRIMARY KEY 或 UNIQUE 索引进行等值匹配查询，一般都是在连表查询或者使用主键查询时出现。这种查询方式速度非常快，查询时间不随着表的大小而增加。
    - eq_ref：在连接查询中，此联接类型仅查询一次，对于每个所选行，联接将从前面的表中读取一行。这种联接类型在使用联合索引或唯一索引进行连接查询时出现。
    - ref：查询时所涉及的字段上建立了普通索引（即非唯一索引），查询时会根据索引进行扫描，并返回匹配的记录。当需要大量的数据时，采取这种方式相对比较快。
    - range：查询使用索引列范围的查询，例如：BETWEEN、IN() 函数，或者用 <、>、<=、>= 操作符的查询。这种联接类型由于仍然能够利用索引缩小搜索范围，因此相对效率还是比较高的。
    - index：类似于 ALL，但该查询仅扫描索引而非数据表。这种情况在查询数据非常少时效率会比较高，但数据量增加后效率会逐渐降低。index 和 all 的区别为 index 类型只遍历索引树。这通常比 all 快，因为索引文件通常比数据文件小，虽然 index 和 all 都是读全表，但是 index 是从索引中读取，而 all 是从硬盘中读取数据
    - all：表示全表扫描，这个类型一般很难避免，主要是由于没有合适的索引，MySQL 必须扫描全表匹配查询条件，性能非常差。
6. possible_keys：显示查询可能使用哪些索引来查找
    - explain 时可能出现 possible_keys 有列，而 key 显示 NULL 的情况，这种情况是因为表中数据不多，mysql 认为索引对此查询帮助不大，选择了全表查询
    - 如果该列是 NULL，则没有相关的索引。在这种情况下，可以通过检查 where 子句看是否可以创造一个适当的索引来提高查询性能，然后用 explain 查看效果。
7. key：查询过程中真正使用的索引，如果为 null，则表示没有使用索引
8. key_len：表示索引的长度，可通过该列计算查询中使用的索引的长度，长度越短越好
9. ref：表示哪个字段或常量与 key 一起被使用。常见的有：const（常量），func，NULL，字段名（例：t1.id）
10. rows：表示 MySQL 查询优化器预计查询需要读取的行数，越少越好。
11. filtered：表示查询结果的过滤程度。
12. Extra：其中有许多额外信息，比如使用了什么特定的算法、是否使用临时表等。常见值有
    - Using index：表示仅使用了索引而没有访问表中的实际数据行，这种情况出现在 SELECT 子句或者 WHERE 子句只访问索引中列的情况下。
    - Using where: 表示 MySQL 将在存储引擎检索行后再次进行过滤，这种情况出现是因为某些条件不包含在索引中，因此需要进行全表扫描或者回表操作来执行这些条件过滤。
    - Using temporary：表示 MySQL 在某个临时表中进行操作。MySQL 在执行查询时需要创建类似于临时表的数据结构，以便于排序、分组或者 JOIN 等操作，这种情况下 MySQL 会先将全表或部分表的数据加载到内存或者磁盘中，然后对这些数据进行排序或者聚合等运算，最后返回结果集给用户。
    - Using filesort：表示 MySQL 需要在内存或者磁盘上进行排序。如果查询涉及到 ORDER BY 子句，则 MySQL 会尝试使用索引进行排序，如果无法使用索引，则需要使用 filesort 算法进行排序。这种情况下需要注意，对大表进行排序可能会导致性能问题。
    - Using join buffer：表示查询使用了缓存（join buffer）来执行连接操作，这种操作符合“单行查询量少，连接量大”的特征。在连接查询时，MySQL 会先将其中一个表的记录全部读入内存或者磁盘缓存中，然后再进行连接操作。
    - Impossible where：表示查询条件中的某些子句或者逻辑不正确或者无法实现，如使用了类似 WHERE 1=0 的条件。
    - Select tables optimized away：表示 MySQL 可以通过优化查询语句，自动忽略某些不需要访问的表，从而减少查询时间和资源消耗。

### Select *

在 SELECT 语句中使用 SELECT \*（通配符）将会查询出表中所有的列，包括那些可能没有用处的列。无用字段增加网络带宽资源消耗，增加数据传输时间，尤其是大字段（如 varchar、blob、text）。

执行 SQL 时优化器需要将 * 转成具体的列；每次查询都要回表，不能走覆盖索引

建议改成 `SELECT <字段列表>` 可减少表结构变更带来的影响

### SQL 语句中 IN 包含的值不应过多

MySQL 对于 IN 做了相应的优化，即将 IN 中的常量全部存储在一个数组里面，而且这个数组是排好序的。但是如果数值较多，产生的消耗也是比较大的。再例如：select id from t where num in(1,2,3) 对于连续的数值，能用 between 就不要用 in 了；再或者使用连接来替换。

### 当只需要一条数据的时候，使用 limit 1

这是为了使 EXPLAIN 中 type 列达到 const 类型

### 尽量用 union all 代替 union

union 和 union all 的差异主要是前者需要将结果集合并后再进行唯一性过滤操作，这就会涉及到排序，增加大量的 CPU 运算，加大资源消耗及延迟。当然，union all 的前提条件是两个结果集没有重复数据。

### 不使用 ORDER BY RAND()

```sql
select id from `dynamic` order by rand() limit 1000;
```

可优化为

```sql
select id from `dynamic` t1 join (select rand() * (select max(id) from `dynamic`) as nid) t2 on t1.id > t2.nidlimit 1000;
```

### 区分 in 和 exists、not in 和 not exists

```sql
select * from 表A where id in (select id from 表B)
```

可优化为

```sql
select * from 表A where exists(select * from 表B where 表B.id=表A.id)
```

区分 in 和 exists 主要是造成了驱动顺序的改变（这是性能变化的关键），如果是 exists，那么以外层表为驱动表，先被访问，如果是 IN，那么先执行子查询。所以 IN 适合于外表大而内表小的情况；EXISTS 适合于外表小而内表大的情况。

关于 not in 和 not exists，推荐使用 not exists，不仅仅是效率问题，not in 可能存在逻辑问题。如何高效的写出一个替代 not exists 的 SQL 语句？

原 SQL 语句：

```sql
select colname … from A 表 where a.id not in (select b.id from B 表)
```

高效的 SQL 语句：

```sql
select colname … from A 表 Left join B 表 on where a.id = b.id where b.id is null
```

取出的结果集为，A 表不在 B 表中的数据

### 分页优化

普通的分页在数据量小的时候耗费时间还是比较短的。如果数据量变大，达到百万甚至是千万级别，普通的分页耗费的时间就非常长了。

如果 id 是自增的，那么可以取前一页的最大行数的 id，然后根据这个最大的 id 来限制下一页的起点。比如此列中，上一页最大的 id 是 866612。SQL 可以采用如下的写法：

```sql
select id,name from product where id> 866612 limit 20
```

如果不知道前面的 id 则使用子查询

```sql
select id,name from product where id >= (SELECT id FROM product LIMIT 866612, 1) limit 20
```

不过，子查询的结果会产生一张新表，会影响性能，应该尽量避免大量使用子查询

除了子查询之外，还以采用延迟查询的方式来优化。

```sql
select id,name from product a,(SELECT id FROM product LIMIT 866612, 20) b where a.id = b.id
```

### JOIN 优化

1. 尽量避免多表做 join，需要 join 的字段数据类型应该保持绝对一致，多表关联查询时，需要保证被关联的字段有索引

    实际业务场景避免多表 join 常见的做法有两种：

    - 单表查询后在内存中自己做关联：对数据库做单表查询，再根据查询结果进行二次查询，以此类推，最后再进行关联。
    - 数据冗余，把一些重要的数据在表中做冗余，尽可能地避免关联查询。很笨的一种做法，表结构比较稳定的情况下才会考虑这种做法。进行冗余设计之前，思考一下自己的表结构设计的是否有问题。

    一般使用第一种，如果并发量不高，数据量不大的话多表 join 也是没问题的

2. 尽量使用 inner join，避免 left join

    LEFT JOIN  左表为驱动表，INNER JOIN MySQL 会自动找出那个数据少的表作用驱动表，RIGHT JOIN 右表为驱动表。

3. 保证被关联的字段有索引
4. MySQL 中没有 full join，可以用以下方式来解决

    ```sql
    select * from A left join B on B.name = A.name where B.name is null union all select * from B;
    ```

5. 巧用 STRAIGHT_JOIN。inner join 是由 MySQL 选择驱动表，但是有些特殊情况需要选择另个表作为驱动表，比如有 group by、order by 等「Using filesort」、「Using temporary」时。STRAIGHT_JOIN 来强制连接顺序，在 STRAIGHT_JOIN 左边的表名就是驱动表，右边则是被驱动表。在使用 STRAIGHT_JOIN 有个前提条件是该查询是内连接，也就是 inner join。其他链接不推荐使用 STRAIGHT_JOIN，否则可能造成查询结果不准确。

### 避免在 where 子句中对字段进行 null 值判断

对于 null 的判断会导致引擎放弃使用索引而进行全表扫描

我们应该尽量将每个字段都设置默认值，且设置字段为 not null

下面是一些常用默认值

- 数值类型：包括 INT、BIGINT、DECIMAL 等）：默认值为 0

- 字符串类型（包括 CHAR、VARCHAR 等）：默认值为 ''（空字符串）。

- 日期类型（DATE、TIME、DATETIME、TIMESTAMP 等）我实验了 MySQL8.0，只有 DATE、DATETIME、TIMESTAMP 可以加上默认值，并且 DATE 类型加默认值需要加一个括号，本身是不支持默认值的，而且不支持 ON UPDATE，如下

    ```sql
    create table d(
        create_time   DATE      NOT NULL DEFAULT (curdate()),
        update_time   DATE      NOT NULL DEFAULT (curdate()),
        datetime_col2 DATETIME  NOT NULL DEFAULT CURRENT_TIMESTAMP,
        timestamp_col TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    )
    ```

### 不建议使用%前缀模糊查询

例如 `LIKE“%name”` 或者 `LIKE“%name%”`，这种查询会导致索引失效而进行全表扫描。但是可以使用 `LIKE“name%”`

如果需要查询 `%name%` ，可以使用全文索引

比如

```sql
elect id,fnum,fdst from dynamic_201606 where user_name like ‘%zhangsan%’
```

创建全文索引

```sql
ALTER TABLE `dynamic_201606` ADD FULLTEXT INDEX `idx_user_name` (`user_name`);
```

使用全文索引

```sql
select id,fnum,fdst from dynamic_201606 where match(user_name) against('zhangsan' in boolean mode);
```

### 避免在 where 子句中对字段进行表达式操作

```sql
select user_id,user_project from user_base where age*2=36;
```

中对字段就行了算术运算，这会造成引擎放弃使用索引，建议改成：

```sql
select user_id,user_project from user_base where age=36/2;
```

### 不要在`=`左边进行函数、算术运算或其他表达式运算

使用函数、算术运算或其他表达式运算符等复杂表达式作为条件筛选查询结果时（如在 WHERE 子句中），如果将这些表达式放在 = 左边，则可能会导致 MySQL 无法使用索引进行优化查询，从而导致查询效率降低。

这是因为 MySQL 查询优化器使用了索引的左前缀原则进行查询优化。当查询语句中的条件涉及到索引列时，MySQL 会尽可能地使用索引来加速查询操作。但是，如果在 = 左边使用了较为复杂的表达式，MySQL 将无法直接匹配索引列，而只能将整个索引列全部读入内存中，然后再对每一行数据计算表达式进行筛选，这样将会明显降低查询效率。

因此，为了保证查询效率，我们应该尽可能地在 `=` 左边边使用常量、字面量或者特定的表达式，而不是使用函数、算术运算或其他表达式运算符等复杂表达式。这样可以让 MySQL 更容易地使用索引进行优化查询，提高查询效率。

### 避免隐式类型转换

where 子句中出现 column 字段的类型和传入的参数类型不一致的时候发生的类型转换，建议先确定 where 中的参数类型

### force index 来强制查询走某个索引

有的时候 MySQL 优化器采取它认为合适的索引来检索 SQL 语句，但是可能它所采用的索引并不是我们想要的。这时就可以采用 forceindex 来强制优化器使用我们制定的索引。

### 批量插入使用单条语句

```sql
INSERT INTO t (id, name) VALUES(1,’Bea’);
INSERT INTO t (id, name) VALUES(2,’Belle’);
INSERT INTO t (id, name) VALUES(3,’Bernice’);
```

改为

```sql
INSERT INTO t (id, name) VALUES(1,’Bea’), (2,’Belle’),(3,’Bernice’);
```

## 索引优化

索引是在存储引擎层实现的，而不是在服务器层实现的，所以不同存储引擎具有不同的索引类型和实现

索引的目的在于提高查询效率，与我们查阅图书所用的目录是一个道理：先定位到章，然后定位到该章下的一个小节，然后找到页数。相似的例子还有：查字典，查火车车次，飞机航班等

本质都是：通过不断地缩小想要获取数据的范围来筛选出最终想要的结果，同时把随机的事件变成顺序的事件，也就是说，有了这种索引机制，我们可以总是用同一种查找方式来锁定数据。

**按照数据结构来分 MySQL 有：**

- B+树索引：B+树索引是最常见的一种索引类型，也是 MySQL 默认的索引类型。它是一种基于 B+树数据结构的索引，可以快速定位到索引列的值。B+树索引可以应用于主键索引、唯一索引、普通索引、前缀索引、组合索引、聚簇索引等功能。

- 倒排索引：倒排索引是将关键词与文档之间的对应关系以反向方式存储到磁盘中的一种索引类型。它可以快速定位包含关键词的文档，常用于搜索引擎和文本处理等领域。倒排索引使用了多个数据结构，包括哈希表、跳表、堆等。MySQL 中的全文索引就是使用了倒排索引的一种实现方式。

- 哈希索引：哈希索引使用了哈希表作为数据结构，可以快速定位索引列的值。由于哈希索引只适用于等值查询，因此仅适用于特定的查询场景。MySQL 中的自适应哈希索引就是使用了哈希表作为数据结构的一种索引类型。

- R-Tree（空间索引实现）：空间索引用于处理地理位置、地图等需要进行空间查询的数据。它可以支持基于距离的查询和几何图形运算。

**按照索引的功能和特点进行分类，MySQL 中有以下几种索引：**

主键索引：B+树索引。主键索引是一种特殊的唯一索引，用于快速查找表中的特定行。

唯一索引：B+树索引。唯一索引保证索引列中的值是唯一的，用于快速查找表中的特定行。

普通索引：B+树索引。普通索引是最基本的索引类型，用于加速查询和排序。它可以在索引列中存储重复的值，并支持多列组合索引。

全文索引：倒排索引。全文索引用于对文本内容进行全文检索。它可以根据关键字快速搜寻匹配的记录，常用于处理大量的文本信息。

空间索引：R-Tree 实现的索引。空间索引用于处理地理位置、地图等需要进行空间查询的数据。它可以支持基于距离的查询和几何图形运算。

前缀索引：B+树索引。前缀索引是将索引列的一部分作为索引值的一种索引类型。通过使用前缀索引，可以减小索引的存储空间和提高索引的查询效率。

哈希索引：哈希表。自适应哈希索引是一种动态更新的哈希索引，它能够自动地调整哈希表的大小和容量，提高索引的查询效率。

组合索引：B+树索引。组合索引是将多个列的值组合在一起创建索引的一种索引类型。它可以提高多列查询的效率，避免使用单独的索引时需要进行多次索引查找操作。

---

索引可以优化查询速度

### 独立的列

在进行查询时，索引列不能是表达式的一部分，也不能是函数的参数，否则无法使用索引。例如下面的查询不能使用 actor_id 列的索引：

```sql
SELECT actor_id FROM sakila.actor WHERE actor_id + 1 = 5;
```

### 多列索引

在需要使用多个列作为条件进行查询时，使用多列索引比使用多个单列索引性能更好。例如下面的语句中，最好把 actor_id 和 film_id 设置为多列索引。

```sql
SELECT film_id, actor_ id FROM sakila.film_actor WHERE actor_id = 1 AND film_id = 1;
```

### 前缀索引

对于 BLOB、TEXT 和 VARCHAR 类型的列，必须使用前缀索引，只索引开始的部分字符。

对于前缀长度的选取需要根据索引选择性来确定。

### 覆盖索引

索引包含所有需要查询的字段的值。具有以下优点：

- 索引通常远小于数据行的大小，只读取索引能大大减少数据访问量。
- 一些存储引擎 (例如 MyISAM) 在内存中只缓存索引，而数据依赖于操作系统来缓存。因此，只访问索引可以不使用系统调用 (通常比较费时)。
- 对于 InnoDB 引擎，若辅助索引能够覆盖查询，则无需访问主索引

### 索引的设计原则

1. 索引并非越多越好

    由于当表中的数据更改的同时，索引也会进行调整和更新，所以索引并非越多越好。
2. 避免对经常更新的表进行过多的索引

    避免对经常更新的表进行过多的索引，并且索引中的列尽可能少。而对于经常用于查询的字段应该创建索引，但要避免添加不必要的字段。
3. 数据量小的表最好不要使用索引，对于非常小的表、大部分情况下简单的全表扫描比建立索引更高效。
4. 在不同值较多的列上建立索引，不同值很少的列上不要建索引
5. 当唯一性是某种数据本身的特征时，指定唯一索引
6. 在频繁进行排序或分组的列上建立索引
7. 索引名尽量短些，索引创建之后也是使用硬盘来存储的，因此提升索引访问的 I/O 效率，也可以提升总体的访问效率
8. 尽量的扩展索引，不要新建索引。比如表中已经有 a 的索引，现在要加 (a,b) 的索引，那么只需要修改原来的索引即可

