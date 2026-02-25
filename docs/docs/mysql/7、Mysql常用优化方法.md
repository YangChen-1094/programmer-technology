### <span style="color:red">1、</span>**选取最合适的字段属性**

将表中字段的宽度设得尽可能小

邮政编码 CHAR(6)就可以, 如："5200" 存储的字节还是6字节

邮政编码 VARCHAR(6)就可以, 如："5200" 存储的字节是4字节

年龄 tinyint unsigned就可以了0-255

男女  enum枚举类型“0”，“1”，“2”，插入时需要是字符串

tinyint(4), tinyint(80), tinyint(0) 三者没有任何区别，该类型最多还是只能存储1字节大小（即-127-128）；类似于注释，并不起任何作用

下面是MySQL数据库常见的基本类型及其长度：

1.**<span style="color:red">1、</span>**<span style="color:red">整数类型</span>：

TINYINT：1个字节，范围为-128到127。

SMALLINT：2个字节，范围为-32,768到32,767。

MEDIUMINT：3个字节，范围为-8,388,608到8,388,607。

INT：4个字节，范围为-2,147,483,648到2,147,483,647。

BIGINT：8个字节，范围为-9,223,372,036,854,775,808到9,223,372,036,854,775,807。

1.2、<span style="color:red">浮点数类型</span>：

FLOAT：4个字节，单精度浮点数。

DOUBLE：8个字节，双精度浮点数。

1.3、<span style="color:red">字符串类型</span>：

CHAR：0-255个字符，固定长度。

VARCHAR：0-65,535个字符，可变长度。

TINYTEXT：最大长度为255个字符。

TEXT：最大长度为65,535个字符。

MEDIUMTEXT：最大长度为16,777,215个字符。

LONGTEXT：最大长度为4,294,967,295个字符。

1.4、<span style="color:red">日期与时间类型</span>：

DATE：3个字节，表示日期，范围为'1000-01-01'到'9999-12-31'。

TIME：3个字节，表示时间，范围为'-838:59:59'到'838:59:59'。

DATETIME：8个字节，表示日期和时间，范围为'1000-01-01 00:00:00'到'9999-12-31 23:59:59'。

TIMESTAMP：4个字节，自1970年1月1日以来的秒数，范围为'1970-01-01 00:00:01'到'2038-01-19 03:14:07'。

除了以上列举的类型外，MySQL还提供了其他一些复杂的数据类型，如枚举类型（ENUM）、集合类型（SET）、二进制类型（BLOB）等。

**1.5、TIMESTAMP与DATETIME的区别**

**<span style="color:red">1、</span>**两者的存储方式不一样

TIMESTAMP:把客户端插入的时间从当前时区转化为UTC(世界标准时间)进行存储，查询时，将其又转化为客户端当前时区进行返回。

DATETIME:不做任何改变，基本上是原样输入和输出

2、两者所能存储的时间范围不一样

timestamp存储的时间范围为:'1970-01-01 00:00:01.000000'到“2038-01-19 03:14:07.999999'

datetime存储的时间范围为:'1000-01-01 0000:00.000000'到'9999-12-31 23:59:59.999999’

3、timestamp支持default current timestamp 来设置默认自动当前时间

4、timestamp支持on update current timestamp 来设置更新时自动当前时间

5、timestamp时区相关，存储时以UTC时间保持，查询时转换为当前时区，即如果在东8区的08.00:0分保存的数据，在东9区看到的是09:0.00，datetime与时区无关

6、timestamp 4个字节存储(实际上就是int)，datetime 8个字节

如果timestamp的值超出范围，mysql不会报错7

8、如果是自动更新模式，手动修改数据导致timestamp字段更新

9、同时有两个timestamp字段默认值为current timestamp会报错



### **2、Mysql 高可用集群**

Mysql互为主从-且读写分离  MySQL-MMM实现MySQL高可用群集  4台机器（2主，2从， 其中2主互为主从）

服务器读写采有VIP地址进行读写，出现故障时VIP会漂移到其它节点，由其它节点提供服务

### **3、如果优化大数据分页**

```mysql
深分页的根本原因有2个：
1、LIMIT 10000000,10 导致 MySQL 先读取 10000010 行再抛弃前 1000 万行，效率极低
2、select * form table 回标开销极大

SELECT * FROM mytable LIMIT 10000000,10;##全表扫描
   id   select_type  table           partitions  type    possible_keys  key     key_len  ref       rows  filtered  Extra   
------  -----------  --------------  ----------  ------  -------------  ------  -------  ------  ------  --------  --------
     1  SIMPLE       incoming_order  (NULL)      ALL     (NULL)         (NULL)  (NULL)   (NULL)     631    100.00  (NULL)  

//查询性能问题：数据库需要跳过前面的 10000000 行数据，然后返回接下来的 10 行数据。这种跳跃操作可能会消耗较多的时间和资源，特别是在没有适当的索引支持的情况下。
//内存消耗：尽管只返回 10 行数据，但是在执行查询时，数据库通常需要将满足条件的所有行加载到内存中，然后再进行偏移和限制操作。如果表中的每行数据较大，或者结果集较大，可能会导致较大的内存消耗。
//查询结果不稳定：由于没有指定明确的排序规则，查询结果的顺序可能会不稳定。


EXPLAIN SELECT * FROM mytable  WHERE  id >= (SELECT id FROM mytable ORDER BY id LIMIT 10000000,1) LIMIT 10;
id  select_type  table           partitions  type    possible_keys  key      key_len  ref       rows  filtered  Extra        
------  -----------  --------------  ----------  ------  -------------  -------  -------  ------  ------  --------  -------------
     1  PRIMARY      incoming_order  (NULL)      range   PRIMARY        PRIMARY  4        (NULL)     315    100.00  Using where  
     2  SUBQUERY     incoming_order  (NULL)      index   (NULL)         PRIMARY  4        (NULL)     101    100.00  Using index  
//基于游标的分页查询：使用子查询来定位起始位置---使用主键索引定位 再通过主键id定位查询数据


EXPLAIN SELECT b.* FROM (SELECT id FROM mytable ORDER BY id ASC LIMIT 10000000,10) a LEFT JOIN mytable b ON a.id=b.id;
//预先计算分页查询：首先在子查询中计算出指定偏移量和行数的 id 值，再根据id连表查询

 最优解法为：
1、需要前端配合，传参“上一页的最大id”给后端
2、后端拿到“上一页最大id”后，作用到sql语句中
SELECT * FROM mytable  WHERE  id in (SELECT id FROM mytable where id > xxx LIMIT 10000000,10)

```