---
title: oracle数据库和mysql数据库的区别 study
date: 2022-1-4 
tags:
 - mysql
 - oracle
---
## 在用友实习的时候用的是oracle数据库，与我平时用的mysql数据库有一些不同，所以今天来整理一下。
### 使用
1. 对事物的提交
mysql默认是自动提交，而oracle默认不自动提交，需要用户手动提交，需要在写commit；指令或者点击commit按钮。
2. 分页查询
mysql是直接在sql语句中写“select...from...where...limit x,y”，有limit就可以实现分页；而oracle则是需要用到伪列rownum和嵌套查询
3. 日期的比较(相似函数to_number()转换为数字)
mysql SELECT * FROM t_order WHERE create_time > DATE('2021-8-19');
oracle select * from  t_order WHERE  create_time  > to_date('2021-12-31','yy-MM-dd');
4. 注释的修改
mysql ALTER TABLE t_order MODIFY COLUMN t_order.`giao` VARCHAR(20) COMMENT 'giao';
oracle comment on column t_order.giao is 'giao';
5. ||
今天在做京东账户信息管理的时候想在数据库中的数据追加公司两个字，于是百度发现了oracle数据库的||和mysql数据库的+是一个意思。
6. varchar2
今天在做京东账户管理建表的时候有一个varchar2类型，没有见过这个类型于是百度了一下，varchar2和varchar的区别是varchar对于汉字占两个字节，对数字和英文字符占一个字节，占的内存小，varchar2一般把所有字符都占两个字节处理,varchar2是oracle独有的数据类型
CHAR的长度是固定的，而VARCHAR2的长度是可以变化的， 比如，存储字符串“abc"，对于CHAR (20)，表示你存储的字符将占20个字节(包括17个空字符)，而同样的VARCHAR2 (20)则只占用3个字节的长度，20只是最大值，当你存储的字符小于20时，按实际长度存储
7. 今天对数据库sql'语句的编写更加熟练了。
 SELECT S.DEALER_ID,  
                S.DEALER_CODE,  
                S.DEALER_NAME,  
                S.PRIORITY,  
                S.CREATE_DATE,  
                S.UPDATE_DATE,  
                S.SET_DEPT_ID,  
          (SELECT R.DEALER_CODE FROM TT_PART_ALLOT_SET_DEFINE D, TM_DEALER R WHERE D.SET_DEPT_ID = R.DEALER_ID AND D.ALLIT_ID=S.ALLIT_ID)R_DEALER_CODE, 
          //这里会返回多个数据，会报一行返回多行的错误，所以我在这个查询结果里面加入了一个判断，因为ALLIT_ID是唯一的，一个是里面的ALLIT_ID一个是外面的ALLIT_ID，就可以根据ALLIT_ID返回一个结果
          (SELECT R.DEALER_NAME FROM TT_PART_ALLOT_SET_DEFINE D, TM_DEALER R WHERE D.SET_DEPT_ID = R.DEALER_ID AND D.ALLIT_ID=S.ALLIT_ID)R_DEALER_NAME, 
                S.ALLIT_ID  
           FROM TT_PART_ALLOT_SET_DEFINE S , TM_DEALER R 
          WHERE S.STATE = 10011001
8. decode
需要将一个字段写入表格，但是是数字，所以需要在查询的时候或者写入表格的时候把数字转换为指定意思的汉字。我这里选择了查数据库的时候就返回汉字。方法可以用if判断，或者使用decode函数DECODE(T.ALLOCATION_TYPE,50791001,'工厂调拨','供应商调拨') 第一个表达式是否等于第二个，等于就返回第三个，不等于返回第四个。
9. 数据库脚本
今天自己写了一个数据库脚本 select 'comment on column TT_PART_DEFINE_HISTORY_TAMM.'||t1.column_name || ' is '''||t.comments||''';' from user_col_comments t,user_col_comments t1 where t.table_name='TT_PART_DEFINE' and t1.table_name='TT_PART_DEFINE_HISTORY_TAMM' and t.column_name=t1.column_name
10. 限定返回行数
今天在oracle数据库想要只查询一条结果的时候使用limit函数进行限制查询条数，发现不行。就猜测可能oracle数据库没有limit这个函数，用mysql数据库测试发现可以，所有得出结论oracle数据库没有limit函数。那么oracle数据库怎么只返回一条数据呢，可以使用 rownum=1或者rownum<2
11. nvl函数
oracle数据库 NVL(1，2) 如果第一个参数为空就取第二个
 NVL2(1，2，3)如果第一个参数为空就取第二个第二个为空返回第三个
NULLIF(1，2)如果第一个参数等于第二个返回空，否则返回第一个
12. 函数和存储过程
今天让我写一个 定时任务加工用品信息 的功能，需要每隔一段时间就执行sql更新数据库，那么这个sql语句怎么写呢。这就要用到存储过程了。
```sql
CREATE OR REPLACE PROCEDURE P_INSERT_JD_PARTS
--参数
(USERID IN NUMBER) --修改人id
AS
BEGIN
  --查询京东商品信息表中名称不为空且加工状态为未加工的数据
  FOR DATAS IN(
    SELECT * FROM TT_PART_JD_SKU T WHERE T.SKU_NAME is not null AND T.MACHINE_FLAG=0
    )LOOP
    INSERT INTO TT_PART_DEFINE TP(
    --插入到TT_PART_DEFINE的字段
    TP.PART_ID,       --用品ID
    TP.PART_OLDCODE,  --用品编码
    TP.PART_CNAME,    --用品名称
    TP.PART_CODE,     --用品件号
    TP.ARTICLES_TYPE, --用品还是备件
    TP.CREATE_DATE,   --创建时间
    TP.CREATE_BY      --创建人
    )values(
    SEQ_OWNID.NEXTVAL,--通过序列设置PART_ID
    DATAS.JD_SKU_ID,
    DATAS.SKU_NAME,
    DATAS.SKU_ID,
    92931001,         --默认为用品
    SYSDATE,          --当前时间
    USERID            
    );
    --将TT_PART_JD_SKU表中的加工状态改为已加工状态，并且设置加工时间
    UPDATE TT_PART_JD_SKU TK SET TK.MACHINE_FLAG=1,TK.MACHINE_TIME=SYSDATE WHERE TK.JD_SKU_ID=DATAS.JD_SKU_ID AND TK.SKU_NAME=DATAS.SKU_NAME AND TK.SKU_ID=DATAS.SKU_ID;
    PKG_PART_UTIL.P_INSERT_HIS(SEQ_OWNID.NEXTVAL,USERID);
    END LOOP;
END P_INSERT_JD_PARTS;
```
13. SYSDATE函数
获取当前时间

14. 获取当前最大id
```sql
max(id) 么 ，不一定对。
因为如果是按字符串排列，999会大于 100000。
用max(to_number(id)) 比较好。
select max(to_number(id)) from t_user;
还可以使用 SELECT SEQ_OWNID.NEXTVAL FROM DUAL  SEQ_OWNID是需要自己创建的序列 NEXTVAL是函数 DUAL是oracle 数据库中的虚拟表，并不是真实存在的
序列如何创建，如下：
create sequence ZH
increment by 1 --自增1
start with 1   --从多少开始
minvalue 1     --最小值
maxvalue 99    --最大值
order          --是否按顺序给值
cache 20         --缓存20个序列值
cycle;         --达到最大值后循环

使用
select ZH.NEXTVAL from dual  怎么样，序列得创建很简单得，因为oracle数据库没有自增（auto_increament）所以用序列很方便.
```
15.  with as 用法
语法：
```sql
with tempName as (select ....)
select ...
```
例：现在要从1-19中得到11-14。一般的sql如下：
```sql
select * from
(
            --模拟生一个20行的数据
             SELECT LEVEL AS lv
               FROM DUAL
         CONNECT BY LEVEL < 20
) tt
 WHERE tt.lv > 10 AND tt.lv < 15
 ```
 使用With as 的SQL为：
 ```sql
 with TT as(
                --模拟生一个20行的数据
                 SELECT LEVEL AS lv
                 FROM DUAL
                CONNECT BY LEVEL < 20
             ) 
select lv from TT
WHERE lv > 10 AND lv < 15
 ```
With查询语句不是以select开始的，而是以“WITH”关键字开头
    可认为在真正进行查询之前预先构造了一个临时表TT，之后便可多次使用它做进一步的分析和处理

WITH Clause方法的优点
     增加了SQL的易读性，如果构造了多个子查询，结构会更清晰；更重要的是：“一次分析，多次使用”，这也是为什么会提供性能的地方，达到了“少读”的目标。

     第一种使用子查询的方法表被扫描了两次，而使用WITH Clause方法，表仅被扫描一次。这样可以大大的提高数据分析和查询的效率。

     另外，观察WITH Clause方法执行计划，其中“SYS_TEMP_XXXX”便是在运行过程中构造的中间统计结果临时表。

16. round函数使用 （就是精确到小数点后多少位的意思）
Round函数用法：

截取数字 
格式如下：ROUND（number[,decimals]）
其中：number 待做截取处理的数值
decimals 指明需保留小数点后面的位数。可选项，忽略它则截去所有的小数部分，并四舍五入。如果为负数则表示从小数点开始左边的位数，相应整数数字用0填充，小数被去掉。需要注意的是，和trunc函数不同，对截取的数字要四舍五入。
举例如下:

```sql
Sql代码：

SQL>   select   round(1234.5678,4)   from   dual;

ROUND(1234.5678,4)
——————
1234.5678

SQL>   select   round(1234.5678,3)   from   dual;

ROUND(1234.5678,3)
——————
1234.568

SQL>   select   round(1234.5678,2)   from   dual;

ROUND(1234.5678,2)
——————
1234.57

SQL>   select   round(1234.5678,1)   from   dual;

ROUND(1234.5678,1)
——————
1234.6

SQL>   select   round(1234.5678,0)   from   dual;

ROUND(1234.5678,0)
——————
1235

SQL>   select   round(1234.5678,-1)   from   dual;

ROUND(1234.5678,-1)
——————-
1230

SQL>   select   round(1234.5678,-2)   from   dual;

ROUND(1234.5678,-2)
——————-
1200

SQL>   select   round(1234.5678,-3)   from   dual;

ROUND(1234.5678,-3)
——————-
1000

附加：

SQL>   select   round(45.923,-1)   from   dual;

ROUND(45.923,-1)
——————-
50

```
17.  对于外连接，Oracle中可以使用“(+)”来表示
18. 集合操作
Union：对两个结果集进行并集操作，不包括重复行，同时进行默认规则的排序；

Union All：对两个结果集进行并集操作，包括重复行，不进行排序；

Intersect：对两个结果集进行交集操作，不包括重复行，同时进行默认规则的排序；

Minus：对两个结果集进行差操作，不包括重复行，同时进行默认规则的排序。
```sql
SELECT DEALER_NAME,
       ERP_CODE,
       SUM(ACCOUNT_SUM) ACCOUNT_SUM,
       DECODE(IS_REL,'','否','是') IS_REL 
       FROM (SELECT DISTINCT 
                       N.RE_DEALER_NAME z, 
                       R.ERP_CODE      h,
                       T.DEALER_NAME    DEALER_NAME,
                       T.ERP_CODE      ERP_CODE,
                       A.ACCOUNT_SUM   ACCOUNT_SUM,
                       1 AS IS_REL
                  FROM TT_PART_DEALER_ACC_RELATION N, 
                       TT_PART_ACCOUNT_DEFINE      A, 
                       TM_DEALER                   R,
                       TT_PART_DEALER_ACC_TOTAL    T
                 WHERE N.RE_DEALER_ID = A.CHILDORG_ID 
                   AND R.DEALER_ID = N.RE_DEALER_ID 
                   AND A.PARENTORG_ID = N.SELLER_ID 
                   AND A.ACCOUNT_PURPOSE  = '95631001'                    
                   AND A.PARENTORG_ID = 2010010100070674
                   AND T.ACC_ID=N.ACC_ID
                   
                   UNION ALL                
                       
              SELECT   N.RE_DEALER_NAME z,
                       R.ERP_CODE      h,
                       R.DEALER_NAME,
                       R.ERP_CODE,
                       A.ACCOUNT_SUM, 
                       NULL REL_ID 
                 FROM TT_PART_ACCOUNT_DEFINE A,
                      TM_DEALER R ,
                      TT_PART_DEALER_ACC_RELATION N
                WHERE R.DEALER_ID = A.CHILDORG_ID 
                  AND A.ACCOUNT_PURPOSE  = '95631001'            
               --AND NOT EXISTS (SELECT 1 
               --  FROM TT_PART_DEALER_ACC_RELATION N ,
                -- TT_PART_DEALER_ACC_TOTAL    T
               --  WHERE N.DEALER_ID = A.CHILDORG_ID 
               --    AND N.SELLER_ID = A.PARENTORG_ID
                --   AND N.ACC_ID=T.ACC_ID) 
                 AND NOT EXISTS (
                 SELECT 1 
                   FROM TT_PART_DEALER_ACC_RELATION N ,
                   TT_PART_DEALER_ACC_TOTAL    T
                 WHERE N.RE_DEALER_ID = A.CHILDORG_ID 
                   AND N.SELLER_ID = A.PARENTORG_ID
                   AND N.ACC_ID=T.ACC_ID) 
                   AND A.PARENTORG_ID = 2010010100070674 
                   AND R.ERP_CODE IS NOT NULL             
                   )GROUP BY DEALER_NAME,ERP_CODE ,IS_REL          
```
19. 函数的使用，现在需要查询税率，如果大于0小于0.15就返回XX-  如果大于0.15就返回JP- 注意，没有小于 不然sql直接where后面跟 FL_START<参数<FL_END 这个时候要用函数就很方便。
```sql
CREATE OR REPLACE FUNCTION F_GET_PART_QZ(ARG_PRA_VALUE IN NUMBER)RETURN VARCHAR2 AS
BEGIN
  --得到所有的编码前缀
  FOR DATAS IN (
    SELECT * 
    FROM 
        TT_PART_JD_SKU_QZ D 
    WHERE 
        D.STATE=10011001
    ORDER BY 
        NVL(D.DATA_ID,0) ASC
    )LOOP
    
    --判断是否是已经循环到了最后一个
    IF DATAS.FL_END IS NULL THEN
      IF ARG_PRA_VALUE>DATAS.FL_START THEN
      RETURN DATAS.PART_QZ;
      END IF;
    END IF;
    
    --不是最后一条的时候
    IF DATAS.FL_START<=ARG_PRA_VALUE AND ARG_PRA_VALUE<DATAS.FL_END THEN 
      RETURN DATAS.PART_QZ;
    END IF;
  END LOOP;
  RETURN NULL;
END F_GET_PART_QZ;
```
20. 关于oracle时间如何比较
```sql
to_date() 将字符串转换为日期
select * from tt_part_jd_user t where to_date('2022-03-31','yyyy-MM-dd')>t.create_date
to_char()将日期，数字装换为字符串
select * from tt_part_jd_user t where to_char(t.create_date-2,'yyyy-MM-dd')<'2022-03-15'
select * from tt_part_jd_user t where to_char(t.create_date-2,'yyyyMMdd')<'20220315'
to_number()将字符串转换为数字 yyyy-MM-dd类型减一就是日期减一 比如今天是2022/3/24日 那么to_char(sysdate-2,'yyyyMMdd')就是2022-03-22
select * from tt_part_jd_user t where to_number(to_char(t.create_date-2,'yyyyMMdd'))<20220315
select to_char(sysdate,'yyyy-MM-dd')from dual
ADD_MONTHS(sysdate,-1)就是月份减一，比如今天是2022/3/24日那么ADD_MONTHS(sysdate,-1)就是2022-02-24
select to_char(ADD_MONTHS(sysdate,-1),'yyyy-mm-dd') from dual
```
21.  关于union all的使用
```sql
在oracle sql中，要求order by是select语句的最后一个语句，而且一个select语句中只允许出现一个order by语句，而且order by必须位于整个select语句的最后。

union操作实际上做了两部分动作：结果集合并 + 排序，

union all只进行结果集简单合并，不做排序，效率比union高 。

例子：　　　 表一：table1  查询语句 ： select  * from table1 t1  order by t1. c1  ;

　　　　        表二：table2  查询语句 ： select  * from table1 t2  order by  t2.c1  . 

　　需求：合并表一表二结果集，使用union  或者 union all 都会报错：ORA-00933 sql命令未正确结束。

　　原因：oracle 认为第一个order by结束后整个select语句就该结束了，但是发现后面没有逗号（；）或斜线（/）结束符，反而后边有 union all 或者 union，即sql语句并未结束，所以报错。

　　解决：使用  with ... as ... select ...

　　　　　with s1 as (select  * from table1 t1  order by t1. c1 ),

　　　　　s2 as ( select  * from table1 t2  order by  t2.c1 )

　　　　　select  *  from s1 union all (此处可以换为 union ) select * from s2
1.  truncate  截断
TRUNCATE 语句具有以下特征：

1.   不能加 WHERE 条件，清除整表数据；

2.   不需要 COMMIT 提交，不支持事务回滚，并且会结束 SAVEPOINT（回滚点）；

3.   效率高于 DELETE 语句（速度较快）；

4.   不记录日志，并清除所占用的空间；

5.   不会触发 DELETE 出发器等特点。

 

而 DELETE 语句的特征：

1.   可以根据条件删除数据；

2.   需要显示 COMMIT 提交，支持事务回滚；

3.   会记录更新日志，删除后仍然占用物理空间；

4.   会触发 DELETE 触发器等。
23. sql语句执行顺序  https://www.cnblogs.com/warehouse/p/9409303.html
FROM
ON
JOIN
WHERE
GROUP BY
WITH CUBE or WITH ROLLUP
HAVING
SELECT
DISTINCT
ORDER BY
TOP
```
22. sql题
```sql
select t.name  "姓名",max(case when t.subject=1 then t.result end)  "机试",max(case when t.subject=2 then t.result end)  "笔试",max(case when t.subject=3 then t.result end)  "答辩"
 from h_l t group by t.name
```
### 大体
事务隔离级别
MySQL是read commited的隔离级别，而Oracle是repeatable read的隔离级别，同时二者都支持serializable串行化事务隔离级别，可以实现最高级别的
读一致性。每个session提交后其他session才能看到提交的更改。Oracle通过在undo表空间中构造多版本数据块来实现读一致性，每个session
查询时，如果对应的数据块发生变化，Oracle会在undo表空间中为这个session构造它查询时的旧的数据块
MySQL没有类似Oracle的构造多版本数据块的机制，只支持read commited的隔离级别。一个session读取数据时，其他session不能更改数据，但可以在表最后插入数据。session更新数据时，要加上排它锁，其他session无法访问数据
对事务的支持
MySQL在innodb存储引擎的行级锁的情况下才可支持事务，而Oracle则完全支持事务
保存数据的持久性
MySQL是在数据库更新或者重启，则会丢失数据，Oracle把提交的sql操作线写入了在线联机日志文件中，保持到了磁盘上，可以随时恢复
并发性
MySQL以表级锁为主，对资源锁定的粒度很大，如果一个session对一个表加锁时间过长，会让其他session无法更新此表中的数据。
虽然InnoDB引擎的表可以用行级锁，但这个行级锁的机制依赖于表的索引，如果表没有索引，或者sql语句没有使用索引，那么仍然使用表级锁。
Oracle使用行级锁，对资源锁定的粒度要小很多，只是锁定sql需要的资源，并且加锁是在数据库中的数据行上，不依赖与索引。所以Oracle对并
发性的支持要好很多。
逻辑备份
MySQL逻辑备份时要锁定数据，才能保证备份的数据是一致的，影响业务正常的dml使用,Oracle逻辑备份时不锁定数据，且备份的数据是一致
复制
MySQL:复制服务器配置简单，但主库出问题时，丛库有可能丢失一定的数据。且需要手工切换丛库到主库。
Oracle:既有推或拉式的传统数据复制，也有dataguard的双机或多机容灾机制，主库出现问题是，可以自动切换备库到主库，但配置管理较复杂。
性能诊断
MySQL的诊断调优方法较少，主要有慢查询日志。
Oracle有各种成熟的性能诊断调优工具，能实现很多自动分析、诊断功能。比如awr、addm、sqltrace、tkproof等
权限与安全
MySQL的用户与主机有关，感觉没有什么意义，另外更容易被仿冒主机及ip有可乘之机。
Oracle的权限与安全概念比较传统，中规中矩。
分区表和分区索引
MySQL的分区表还不太成熟稳定。
Oracle的分区表和分区索引功能很成熟，可以提高用户访问db的体验。
管理工具
MySQL管理工具较少，在linux下的管理工具的安装有时要安装额外的包（phpmyadmin， etc)，有一定复杂性。
Oracle有多种成熟的命令行、图形界面、web管理工具，还有很多第三方的管理工具，管理极其方便高效。
最最重要的区别
MySQL是轻量型数据库，并且免费，没有服务恢复数据。
Oracle是重量型数据库，收费，Oracle公司对Oracle数据库有任何服务。
