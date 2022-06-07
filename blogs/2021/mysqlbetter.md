---
title: MySQL调优 study
date: 2022-5-24 
categories:
 - sql调优
tags:
 - sql
 - sql调优
sticky: 1
---
## 在家玩太久了，不能当一条咸鱼了，开始浅学一下Mysql调优。
索引是帮助MySQL高效获取数据的排好序的数据结构。
索引的数据结构
 1. 二叉树
 使用二叉树的话，如果查询id递增的数据，其实就是退化成了链表。
 2. 红黑树
 使用红黑树，当数据量太大，树的高度不可控，当查询叶子节点的时候就需要很多次IO。
 3. Hash表
 4. B+Tree
## sql之sql注入
防止 sql 注入的方式：
1. 预编译语句：如，select * from user where username = ？，sql 语句语义不会发生改
变，sql 语句中变量用？表示，即使传递参数时为“admin or ‘a’= ‘a’”，也会把这整体当
做一个字符创去查询。
2. Mybatis 框架中的 mapper 方式中的 # 也能很大程度的防止 sql 注入（$无法防止 sql 注
入）。
## mysql性能优化
1. 当只要一行数据时使用 limit 1
查询时如果已知会得到一条数据，这种情况下加上 limit 1 会增加性能。因为 mysql 数据库引
擎会在找到一条结果停止搜索，而不是继续查询下一条是否符合标准直到所有记录查询完毕。
2. 选择正确的数据库引擎
Mysql 中有两个引擎 MyISAM 和 InnoDB，每个引擎有利有弊。
MyISAM 适用于一些大量查询的应用，但对于有大量写功能的应用不是很好。甚至你只需要
update 一个字段整个表都会被锁起来。而别的进程就算是读操作也不行要等到当前 update 操作完
成之后才能继续进行。另外，MyISAM 对于 select count(*)这类操作是超级快的。
InnoDB 的趋势会是一个非常复杂的存储引擎，对于一些小的应用会比 MyISAM 还慢，但是支
持“行锁”，所以在写操作比较多的时候会比较优秀。并且，它支持很多的高级应用，例如：事物。
3. 用 not exists 代替 not in
Not exists 用到了连接能够发挥已经建立好的索引的作用，not in 不能使用索引。Not in 是最
慢的方式要同每条记录比较，在数据量比较大的操作红不建议使用这种方式。
4. 对操作符的优化，尽量不采用不利于索引的操作符
如：in not in is null is not null <> 等
某个字段总要拿来搜索，为其建立索引：
Mysql 中可以利用 alter table 语句来为表中的字段添加索引，语法为：alter table 表明
add index (字段名)；
## mysql语句优化
5. where 子句中可以对字段进行 null 值判断吗？ 
可以，比如 select id from t where num is null 这样的 sql 也是可以的。但是最好不要给数据库留 NULL，尽可
能的使用 NOT NULL 填充数据库。不要以为 NULL 不需要空间，比如：char(100) 型，在字段建立时，空间就固定了，
不管是否插入值（NULL 也包含在内），都是占用 100 个字符的空间的，如果是 varchar 这样的变长字段，null 不占
用空间。可以在 num 上设置默认值 0，确保表中 num 列没有 null 值，然后这样查询：select id from t where num 
= 0。
6.  select * from admin left join log on admin.admin_id = log.admin_id where 
log.admin_id>10 如何优化? 
优化为： select * from (select * from admin where admin_id>10) T1 lef join log on T1.admin_id = 
log.admin_id。
使用 JOIN 时候，应该用小的结果驱动大的结果（left join 左边表结果尽量小如果有条件应该放到左边先处理，
right join 同理反向），同时尽量把牵涉到多表联合的查询拆分多个 query（多个连表查询效率低，容易到之后锁表和
阻塞）
7. limit 的基数比较大时使用 between 
例如：select * from admin order by admin_id limit 100000,10 
优化为：select * from admin where admin_id between 100000 and 100010 order by admin_id。
8. 尽量避免在列上做运算，这样导致索引失效 
例如：select * from admin where year(admin_time)>2014 
优化为： select * from admin where admin_time> '2014-01-01
## 在千万级的数据库查询中，如何提高效率？
1. 数据库设计方面
2. 对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。
3. 应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描，
如： select id from t where num is null 可以在 num 上设置默认值 0，确保表中 num 列没有 null 值，然后这样查
询： select id from t where num=0
4. 并不是所有索引对查询都有效，SQL 是根据表中数据来进行查询优化的，当索引列有大量数据重复时,查询
可能不会去利用索引，如一表中有字段 sex，male、female 几乎各一半，那么即使在 sex 上建了索引也对查询效
率起不了作用。
5. 索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的
效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表
的索引数最好不要超过 6 个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。
6. 应尽可能的避免更新索引数据列，因为索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将
导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新索引数据列，那么需要考虑是否
应将该索引建为索引。
7. 尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会
增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比
较一次就够了。
8. 尽可能的使用 varchar/nvarchar 代替 char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，
其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。
9. 尽量使用表变量来代替临时表。如果表变量包含大量数据，请注意索引非常有限（只有主键索引）。
10. 避免频繁创建和删除临时表，以减少系统表资源的消耗。
11. 临时表并不是不可使用，适当地使用它们可以使某些例程更有效，例如，当需要重复引用大型表或常用表
中的某个数据集时。但是，对于一次性事件，最好使用导出表。
12. 在新建临时表时，如果一次性插入数据量很大，那么可以使用 select into 代替 create table，避免造成
大量 log ，以提高速度；如果数据量不大，为了缓和系统表的资源，应先 create table，然后 insert。
13. 如果使用到了临时表，在存储过程的最后务必将所有的临时表显式删除，先 truncate table ，然后 drop 
table ，这样可以避免系统表的较长时间锁定。
14. SQL 语句方面
15. 应尽量避免在 where 子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描。
16. 应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，如：
select id from t where num=10 or num=20 可以这样查询： select id from t where num=10 union all 
select id from t where num=20
17. in 和 not in 也要慎用，否则会导致全表扫描，如： select id from t where num in(1,2,3) 对于连续的
数值，能用 between 就不要用 in 了： select id from t where num between 1 and 3
18. 下面的查询也将导致全表扫描： select id from t where name like ‘%abc%’
19. 如果在 where 子句中使用参数，也会导致全表扫描。因为 SQL 只有在运行时才会解析局部变量，但优化
程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然而，如果在编译时建立访问计划，变量的
值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描： select id from t where 
num=@num 可以改为强制查询使用索引： select id from t with(index(索引名)) where num=@num
20. 应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。如：
select id from t where num/2=100 应改为: select id from t where num=100*2
21. 应尽量避免在where子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。如：select 
id from t where substring(name,1,3)= ‘ abc ’ – name 以 abc 开 头 的 id select id from t where 
datediff(day,createdate,’2005-11-30′)=0–‘2005-11-30’生成的 id 应改为: select id from t where name 
like ‘abc%’ select id from t where createdate>=’2005-11-30′ and createdate<’2005-12-1′
22. 不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用
索引。
23. 不要写一些没有意义的查询，如需要生成一个空表结构： select col1,col2 into #t from t where 1=0 这
类代码不会返回任何结果集，但是会消耗系统资源的，应改成这样： create table #t(…)
24. 很多时候用 exists 代替 in 是一个好的选择： select num from a where num in(select num from b) 
用下面的语句替换： select num from a where exists(select 1 from b where num=a.num)
25. 任何地方都不要使用 select * from t ，用具体的字段列表代替“*”，不要返回用不到的任何字段。
26. 尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过 1 万行，那么就应该考虑改写。
27. 尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。
28. 尽量避免大事务操作，提高系统并发能力。
29. java 方面：重点内容
30. 尽可能的少造对象。
31. 合理摆正系统设计的位置。大量数据操作，和少量数据操作一定是分开的。大量的数据操作，肯定不是 ORM
框架搞定的。，
32. 使用 jDBC 链接数据库操作数据
33. 控制好内存，让数据流起来，而不是全部读到内存再处理，而是边读取边处理；
34. 合理利用内存，有的数据要缓存


