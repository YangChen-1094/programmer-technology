### **一、mysql是如何排序的**

现有如下表数据：

```mysql
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `city` varchar(16) NOT NULL,
  `name` varchar(16) NOT NULL,
  `age` int(11) NOT NULL,
  `addr` varchar(128) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `city` (`city`)
) ENGINE=InnoDB;
//假设你要查询城市是"杭州"的所有人名字，并且按照姓名排序返回前1000个人的姓名、年龄
select city,name,age from t where city='杭州' order by name limit 1000;
```

Mysql是如何排序的呢？

#### **1、全字段排序**

顾名思义，就是把所有查询的字段放在一起，按某一个字段排序。通常情况下，这个语句执行流程如下所示 ：

1. 初始化sort_buffer，确定放入name、city、age这三个字段；
2. 从索引city找到第一个满足city='杭州'条件的主键id，也就是图中的ID_X；
3. 到主键id索引取出整行，取name、city、age三个字段的值，存入sort_buffer中；
4. 从索引city取下一个记录的主键id；
5. 重复步骤3、4直到city的值不满足查询条件为止，对应的主键id也就是图中的ID_Y；
6. 对sort_buffer中的数据按照字段name做快速排序；
7. 按照排序结果取前1000行返回给客户端。

![图片](images/14、mysql是如何排序的/img1.png)

**<span style='color:red'>图中"按name排序"的动作，可能在内存也可能在磁盘进行，取决于排序所需的内存和参数sort_buffer_size。sort_buffer_size，就是MySQL为排序开辟的内存（sort_buffer）的大小。如果要排序的数据量小于sort_buffer_size，排序就在内存中完成。但如果排序数据量太大，内存放不下，则不得不利用磁盘临时文件辅助排序。 内存放不下时，就需要使用外部排序，外部排序一般使用归并排序算法。MySQL将需要排序的数据分成12份，每一份单独排序后存在这些临时文件中。然后把这12个有序文件再合并成一个有序的大文件。</span>**

#### **2、rowid排序**

对于全字段排序，如果查询要返回的字段很多的话，那么sort_buffer里面要放的字段数太多，这样内存里能够同时放下的行数很少，要分成很多个临时文件，排序的性能会很差

```mysql
SET max_length_for_sort_data = 16;
//如果单行的长度超过这个值，MySQL就认为单行太大，要换一个算法
//city、name、age 这三个字段的定义总长度是36，max_length_for_sort_data设置为16
```

rowid排序，只把排序字段（name）和主键id，放入排序内存sort_buffer

上面的select语句整个执行流程就变成：
1. 初始化sort_buffer，确定放入两个字段，即name和id；
2. 从索引city找到第一个满足city='杭州'条件的主键id；
3. 到主键id索引取出整行，取name、id这两个字段，存入sort_buffer中；
4. 从索引city取下一个记录的主键id；
5. 重复步骤3、4直到不满足city='杭州'条件为止；
6. 对sort_buffer中的数据按照字段name进行排序；
7. 遍历排序结果，取前1000行，并按照id的值回到原表（磁盘操作）中取出city、name和age三个字段返回给客户端。

![图片](images/14、mysql是如何排序的/img2.png)

**如果内存够用就使用全字段排序，否则使用rowid排序**

不使用排序达到排序效果的情况，索引，联合索引
