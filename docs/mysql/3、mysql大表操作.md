### 1、大数据表添加索引（修改字段）

**先创建一张和源表无关的新表，然后通过重命名和删表操作交换两张表；**

```sql
#操作步骤：
#1、创建一张和原表结构一样的空表，只是表名不一样
    create table tb_name_tmp like tb_name;
#2、把新建的空表非主键索引都删掉，因为这样在往新表导数据的时候效率会很快（因为除了必要的主键以外，不用再去建立其它索引数据了）
    alter tb_name_tmp drop index index_name;
##1、2合并  CREATE TABLE tb_name_tmp AS SELECT * FROM tb_name LIMIT N;
#3、从旧表往主表里导数据，如果数据太大，建议分批导入，只需确保无重复数据就行，因为导入数据太大，会很占用资源(内存，磁盘io, cpu等)，可能会影响旧表在线上的业务。我是每批次100万条数据导入，基本上每次都是在 20s左右
    insert into tb_name_tmp select * from tb_name where id between start_id and end_id;
#4、数据导完后，再对新表进行添加索引
     create index index_name on tb_name_tmp(column_name);
#5、当大部分数据导入后，索引也建立好了，但是旧表数据量还是会因业务的增长而增长，这时候为了确保新旧表的数据一至性和平滑切换，建议写一个脚本，判断当旧表的数据行数与新表一致时，就切换。我是以 max(id)来判断的。
    rename table tb_name to tb_name_tmp1;
    rename table tb_name_tmp to tb_name;
**<span style="color:red">推荐方案：使用在线DDL工具（首选）</span>**
1. Percona Toolkit 的 pt-online-schema-change
pt-online-schema-change \
  --alter "ADD INDEX idx_name (column_name)" \
  D=database_name,t=table_name \
  --user=username --password=password \
  --execute
操作原理：
1.创建影子表（包含新索引）
2.在原表上创建触发器同步增量数据
3.分批复制数据到影子表
4.原子切换表（重命名）
5.清理旧表
优势：
几乎零停机时间
自动处理增量数据
可暂停/恢复操作
实时显示进度
```

源sql：select * from product limit 1000000, 10

优化sql：SELECT * FROM product WHERE ID > =(select id from product limit 1000000, 1) limit 10

### **2、如何给字符串加索引（如：邮箱、身份证）**

①<span style="color:red">使用倒序存储</span>。如果你存储身份证号的时候把它倒过来存，每次查询的时候，你可以这么写：

```sql
mysql> select field_list from t where id_card = reverse('input_id_card_string');
由于身份证号的最后6位没有地址码这样的重复逻辑，所以最后这6位很可能就提供了足够的区分度。当然了，实践中你不要忘记使用
```

count(distinct)方法去做个验证。

②<span style="color:red">使用hash字段</span>。你可以在表上再创建一个整数字段，来保存身份证的校验码，同时在这个字段上创建索引。

```sql
alter table t add id_card_crc int unsigned, add index(id_card_crc);
//查询使用时
select field_list from t where id_card_crc=crc32('input_id_card_string') and id_card = 'input_id_card_string'
```

### **如何计算mysql单表中能存放多少条数据的数量？**
* mysql中系统是按页读取的，默认页：16kb
* b+树的高度就是索引的层数，高度越高，查询过程中就多一次IO操作。
* mysql的B+树中分为非叶子节点（存储索引指针）和叶子节点（存储数据）即：2层非叶子节点+1层叶子节点

```
一个非叶子节点可以装多少数据：
    默认页：16kb
    页头120b 页尾8b
    一页可以装索引的数据约为16kb - 128b = 约15kb可以用来存索引
    索引类型是bigint 占8b +页号4b = 占12b（一个索引）
    一页可以装多少个索引：15 * 1024 / 12 = 1280个

一个叶子节点可以装多少数据：
    默认页：16kb
    页头120b 页尾8b
    一行数据1kb（叶子节点）
    15kb / 1kb = 15条

3层索引的mysql表可以装：
1280 * 1280 * 15 = 24576000 (2.4kw)

```