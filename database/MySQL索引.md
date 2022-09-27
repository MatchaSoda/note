# MySQL索引
## 索引介绍
索引用于**快速找出在某个列中有一特定值的行**。不使用索引，MySQL必须从第一条记录开始读完整个表，直到找出相关的行，表越大查询数据所花费的时间就越多。如果表中查询的列有索引，MySQL能够快速到达一个位置去搜索数据文件，而不必查看所有数据，那么将会节省很大一部分时间。

例如：有一张person表，其中有2W条记录，记录着2W个人的信息。有一个Phone的字段记录每个人的电话号码，现在想要查询出电话号码为xxxx的人的信息。
* 如果没有索引，那么将从表中第一条记录一条条往下遍历，直到找到该条信息为止。
* 如果有了索引，那么会将 Phone 字段，通过一定的方法进行存储，好让查询该字段上的信息时，能够快速找到对应的数据，而不必在遍历2W条数据了。
## 索引优缺点
优点：
* 所有的MySQL列类型（字段类型）都可以被索引，也就是可以给任意字段设置索引。
* 大大加快数据的查询速度。

缺点：
* 创建索引和维护索引（对表中的数据进行增加、删除、修改时）要耗费时间，并且随着数据量的增加所耗费的时间也会增加。
* 索引也需要占空间，我们知道数据表中的数据也会有最大上线设置的，如果我们有大量的索引，索引文件可能会比数据文件更快达到上线值。

使用原则：
* 对经常更新的表就避免对其设置过多的索引，对经常用于查询的字段应该创建索引。
* 数据量小的表最好不要使用索引，因为由于数据较少，可能查询全部数据花费的时间比遍历索引的时间还要短，索引就可能不会产生优化效果。
* 在一个列上(字段上)不同值较少的不要建立索引，比如在学生表的"性别"字段上只有男，女两个不同值。相反的，在一个字段上不同值较多的可是建立索引。
* 对主键和unique key这样的约束列是自带索引的，不需要创建索引这些列的查询速度本身就很快
## 索引存储
B树、B+树、哈希(待补充)  
https://blog.csdn.net/a519640026/article/details/106940115  
https://blog.csdn.net/Lee_SmallNorth/article/details/115626136
## 索引分类
* 单列索引：一个索引只包含单个列，但一个表中可以有多个单列索引。
  * 普通索引：MySQL中基本索引类型，没有什么限制，允许在定义索引的列中插入重复值和空值，纯粹为了查询数据更快一点。
  * 唯一索引：索引列中的值必须是唯一的，但是允许为空值，
  * 主键索引：是一种特殊的唯一索引，不允许有空值。
* 组合索引：一个的索引包含多个列，只有在查询条件中使用了这些字段的左边字段时，索引才会被使用，使用组合索引时遵循最左前缀。
* 全文索引：要求只有在MyISAM引擎上才能使用，只能在CHAR、VARCHAR、TEXT类型字段上使用全文索引。就是在一堆文字中，通过其中的某个关键字等，就能找到该字段所属的记录行.
* 空间索引：空间索引是对空间数据类型的字段建立的索引，MySQL中的空间数据类型有四种，GEOMETRY、POINT、LINESTRING、POLYGON。
## 索引使用
### 索引相关语句
创建索引
```sql
# 为某表中的某个字段创建索引
create index 索引名 on 表名(字段名);
```
查看某表索引
```sql
# 查看表结构即可获知索引信息
show create table 表名;
```
删除索引
```sql
# 删除某表中某个索引
drop index 索引名 on 表名;
```
### 测试索引效果
1. 准备数据
   1. 准备表  
        ```sql
        CREATE TABLE s1 (
            id int,
            name varchar(20),
            gender char(6),
            email varchar(50)
        );
        ```
   2. 创建存储过程
        ```sql
        #1. 声明存储过程的结束符号为$$
        delimiter $$ 

        #2. 创建存储过程，实现批量插入记录
        create procedure auto_insert1()
        BEGIN
            declare i int default 1;
            while(i<3000000)do
                insert into s1 values(i,'mlg','male',concat('mlg',i,'@qq'));
                set i=i+1;
            end while;
        END$$ #$$结束

        #3. 重新声明分号为结束符号
        delimiter ; 
        ```
   3. 查看存储过程
        ```sql
        mysql> show create procedure auto_insert1\G
        *************************** 1. row ***************************
        Procedure: auto_insert1
        sql_mode: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
        Create Procedure: CREATE DEFINER=`root`@`%` PROCEDURE `auto_insert1`()
        BEGIN
            declare i int default 1;
            while(i<3000000)do
                insert into s1 values(i,'mlg','male',concat('mlg',i,'@qq'));
                set i=i+1;
            end while;
        END
        character_set_client: utf8
        collation_connection: utf8_general_ci
        Database Collation: utf8_general_ci
        1 row in set (0.00 sec)
        ```
   4. 调用存储过程
        ```sql
        mysql> call auto_insert1();
        Query OK, 1 row affected (7 min 21.30 sec)
        mysql> select count(*) from s1;
        +----------+
        | count(*) |
        +----------+
        |  2999999 |
        +----------+
        1 row in set (0.97 sec)
        ```
2. 无索引测试
   1. 表大小
        ```sql
        [root@centos-linux-6 blog01]# pwd
        /var/lib/mysql/blog01
        [root@centos-linux-6 blog01]# du -sh *
        12K     book.frm
        96K     book.ibd
        4.0K    db.opt
        12K     s1.frm
        177M    s1.ibd
        [root@centos-linux-6 blog01]# 
        ```
        s1表的ibd数据文件(由于InnoDB用的是聚集索引，所以数据和索引都存在ibd文件中)大小为**177M**
   2. 查询速度
        ```sql
        # 查找一个在范围内，但是id比较大大值
        mysql> select * from s1 where id = 2999990;
        +---------+------+--------+---------------+
        | id      | name | gender | email         |
        +---------+------+--------+---------------+
        | 2999990 | mlg  | male   | mlg2999990@qq |
        +---------+------+--------+---------------+
        1 row in set (1.10 sec)

        # 查找一个不在表内大数据
        mysql> select * from s1 where id = 3999999; 
        Empty set (1.10 sec)
        ```
        在无索引状况下，以id为条件进行数据查询话费约**1.10 sec**  
        即便id=3999999不在我们表中，但mysql并不知道，因此他会遍历整个表才能反馈出结果  
        无索引状态下查询数据，都需要从头开始遍历整张表
1. 有索引测试
   1. 添加索引
        ```sql
        mysql> create index ind_id on s1(id);
        Query OK, 0 rows affected (3.31 sec)
        Records: 0  Duplicates: 0  Warnings: 0
        ```
        2. 表大小
        ```sql
        [root@centos-linux-6 blog01]# du -sh *
        12K     book.frm
        96K     book.ibd
        4.0K    db.opt
        12K     s1.frm
        233M    s1.ibd
        ```
        由于为s1表中的id字段创建了索引，因此s1.ibd文件大小为**233M**，较之前当**177M**明显增大
   3. 查询速度
        ```sql
        mysql> select * from s1 where id = 2999990;
        +---------+------+--------+---------------+
        | id      | name | gender | email         |
        +---------+------+--------+---------------+
        | 2999990 | mlg  | male   | mlg2999990@qq |
        +---------+------+--------+---------------+
        1 row in set (0.00 sec)

        mysql> select * from s1 where id = 3999999;
        Empty set (0.00 sec)
        ```
        在为id字段创建索引状况下，以id为条件进行数据查询话费约0.00 sec  
        mysql先到索引表中根据b+树的搜索原理搜到了2999990的结果，然后返回（B+树的作用就是可以大大降低IO操作节省查询时间）  
        同理，3999999也能迅速得出其不存在的结论
   4. 未添加索引字段速度
        ```sql
        mysql> select * from s1 where email = 'mlg2999990@qq';
        +---------+------+--------+---------------+
        | id      | name | gender | email         |
        +---------+------+--------+---------------+
        | 2999990 | mlg  | male   | mlg2999990@qq |
        +---------+------+--------+---------------+
        1 row in set (1.34 sec)
        ```
        由对比结果可知，由于没有给email字段加索引，所以当以其为条件进行查询时，速度依旧很慢
## 参考
https://www.cnblogs.com/nananana/p/10387720.html  
https://blog.csdn.net/qq_31851107/article/details/103467869  
https://blog.csdn.net/fly_zhaohy/article/details/104014391  
http://javainterview.gitee.io/luffy/2021/08/19/08-MySQL/01.%20MySQL%E7%B4%A2%E5%BC%95/  
https://blog.csdn.net/a519640026/article/details/106940115