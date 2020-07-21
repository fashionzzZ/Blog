## 索引的基本知识

### 索引的优点

1. 大大减少了服务器需要扫描的数据量

2. 帮助服务器避免排序和临时表

3. 将随机IO变成顺序IO

### 索引的用处

1. 快速查找匹配where子句的行

2. 从consideration中消除行，如果可以再多个索引之间进行选择，MySQL通常会使用找到最少行的索引

3. 如果表具有多列索引，则优化器可以使用索引的任何最左前缀来查找行

4. 当有表连接的时候，从其它表检索行数据

5. 查找特定索引列的min或max值

6. 如果排序或分组时在可用索引的最左前缀上完成的，则对表进行排序和分组

7. 在某些情况下，可以优化查询以检索值而无需查询数据行

### 索引的分类

1. 主键索引
2. 唯一索引
3. 普通索引
4. 全文索引
5. 组合索引

### 技术名词

1. 回表

   > InnoDB**普通索引**的叶子节点存储主键值。
   >
   > 先定位主键，再通过主键定位数据行
   >
   > staffs表 id为主键 name, age 为组合索引
   >
   > ` select name, age, pos from staffs where name = 'July' and age = 25; `
   >
   > 可以看到查询列为name, age ,pos 由于pos 列没有建立索引,此时需要根据已经检索出的id，再去查找对应的数据行，这个过程叫做回表。

2. 覆盖索引

   > 基本介绍
   > > 1. 如果一个索引包含所有需要查询的字段的值，我们称之为覆盖索引
   > > 2. 不是所有类型的索引都可以称为覆盖索引，覆盖索引必须要存储索引列的值
   > > 3. 不同的存储实现覆盖索引的方式不同，不是所有的引擎都支持覆盖索引，memory不支持覆盖索引
   >
   > 优势
   >
   > > 1. 索引条目通常远小于数据行大小，如果只需要读取索引，那么mysql就会极大的较少数据访问量
   > > 2. 因为索引是按照列值顺序存储的，所以对于IO密集型的范围查询会比随机从磁盘读取每一行数据的IO要少的多
   > > 3. 一些存储引擎如MYISAM在内存中只缓存索引，数据则依赖于操作系统来缓存，因此要访问数据需要一次系统调用，这可能会导致严重的性能问题
   > > 4. 由于INNODB的聚簇索引，覆盖索引对INNODB表特别有用

3. 最左匹配

4. 索引下推

   > staffs表	组合索引(name, age)
   >
   > ` select * from staffs where name like 'J%' and age > 25; `
   >
   > > 执行该语句有两种可能
   > >
   > > 1. 根据组合索引(name, age) 查询所有满足name以J开头的索引，然后回表查询出相应的全行数据，然后再筛选出age大于25的数据。
   > >
   > > 2. 根据组合索引(name, age) 查询所有满足name以J开头的索引，然后直接在筛选出age大于25的索引，之后再回表查询出相应的全行数据。
   > >
   > > 很明显第二种需要回表查询的全行数据比较少，这就是索引下推。MySQL默认启用索引下推。

### 索引采用的数据结构

1. 哈希表
2. B+树

### 索引匹配方式

1. 全值匹配

   > 全值匹配指的是和索引中的所有列进行匹配
   >
   > ` explain select * from staffs where name = 'July' and age = 23 and pos = 'dev'; `

2. 匹配最左前缀

   > 只匹配前面的几列
   >
   > `explain select * from staffs where name = 'July' and age = 23; `
   >
   > `explain select * from staffs where name = 'July'; `

3. 匹配列前缀

   > 可以匹配某一列的值的开头部分
   >
   > `explain select * from staffs where name like 'J%'; `

4. 匹配范围值

   > 可以查找某一个范围的数据
   >
   > ` explain select * from staffs where name > 'Mary';`

5. 精确匹配某一列并范围匹配另外一列

   > 可以查询第一列的全部和第二列的部分
   >
   > `explain select * from staffs where name = 'July' and age > 25;`

6. 只访问索引的查询

   > 查询的时候只需要访问索引，不需要访问数据行本质上就是覆盖索引
   >
   > `explain select name, age,pos from staffs where name = 'July' and age = 25 and post = 'dev';`