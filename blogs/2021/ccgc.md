---
title: 存储过程 study
date: 2022-01-20
tags:
 - sql
---
## 在用友实习的时候做一个 定时任务加工用品信息 的功能，需要每隔一段时间就执行sql更新数据库，那么这个sql语句怎么写呢。这就要用到存储过程了。
## 存储过程
### 示例一：无参无返
```sql
create or replace procedure p1
--or replace代表创建该存储过程时，若存储名存在，则替换原存储过程，重新创建
--无参数列表时，不需要写()
as
begin
  dbms_output.put_line('hello world');
end;

--执行存储过程方式1
set serveroutput on;
begin
  p1();
end;
--执行存储过程方式2
set serveroutput on;
execute p1();
```
### 示例二：有参有返
```sql
create or replace procedure p2
(name in varchar2,age int,msg out varchar2)
--参数列表中，声明变量类型时切记不能写大小，只写类型名即可，例如参数列表中的name变量的声明
--参数列表中，输入参数用in表示，输出参数用out表示，不写时默认为输入参数。
------------输入参数不能携带值出去，输出参数不能携带值进来，当既想携带值进来，又想携带值出去，可以用in out
as
begin
  msg:='姓名'||name||',年龄'||age;
  --赋值时除了可以使用：=，还可以用into来实现
  --上面子句等价于select '姓名'||name||',年龄'||age into msg from dual;
end;
--执行存储过程
set serveroutput on;
declare
  msg varchar2(100);
begin
  p2('张三',23,msg);
  dbms_output.put_line(msg);
end;
```
### 示例四：参数列表中有in out参数
```sql
create or replace procedure p3
(msg in out varchar2)
--当既想携带值进来，又想携带值出去，可以用in out
as
begin
  dbms_output.put_line(msg); --输出的为携带进来的值
  msg:='我是从存储过程中携带出来的值';
end;


--执行存储过程
set serveroutput on;
declare
  msg varchar2(100):='我是从携带进去的值';
begin
  p3(msg);
  dbms_output.put_line(msg);
end;
```
### 示例四：存储过程中定义参数
```sql
create or replace procedure p4
as
  --存储过程中定义的参数列表
  name varchar(50);
begin
  name := 'hello world';
  dbms_output.put_line(name);
end;
---执行存储过程
set serveroutput on;
execute p4();
```
## 函数存储
```sql
create or replace function f1
return varchar--必须有返回值，且声明返回值类型时不需要加大小
as
  msg varchar(50);
begin
   msg := 'hello world';
   return msg;
end;

--执行函数方式1
select f1() from dual;
--执行函数方式2
set serveroutput on;
begin 
  dbms_output.put_line(f1());
end;

```
### 再看看我写的存储过程
```sql
CREATE OR REPLACE PROCEDURE P_INSERT_JD_PARTS
--参数
(USERID IN NUMBER) --修改人id
AS
PARTID NUMBER(16);
BEGIN
  --查询京东商品信息表中名称不为空且加工状态为未加工的数据
  FOR DATAS IN(
    SELECT * FROM TT_PART_JD_SKU T WHERE T.SKU_NAME is not null AND T.MACHINE_FLAG=0
    )LOOP
    SELECT F_GETID INTO PARTID FROM DUAL;
    INSERT INTO TT_PART_DEFINE TP(
    --插入到TT_PART_DEFINE的字段
    TP.PART_ID,       --用品ID
    TP.PART_OLDCODE,  --用品编码
    TP.PART_CNAME,    --用品名称
    TP.PART_CODE,     --用品件号
    TP.ARTICLES_TYPE, --用品还是备件
    TP.FRAME_ID,      --备件大类 京东
    TP.CREATE_DATE,   --创建时间
    TP.CREATE_BY      --创建人
    )values(
    PARTID,
    DATAS.JD_SKU_ID,
    DATAS.SKU_NAME,
    DATAS.SKU_ID,
    92931001,
    92861010,
    SYSDATE,
    USERID
    );
    UPDATE TT_PART_JD_SKU TK SET TK.MACHINE_FLAG=1,TK.MACHINE_TIME=SYSDATE,TK.PART_ID=PARTID WHERE TK.JD_SKU_ID=DATAS.JD_SKU_ID AND TK.SKU_NAME=DATAS.SKU_NAME AND TK.SKU_ID=DATAS.SKU_ID;
    PKG_PART_UTIL.P_INSERT_HIS(PARTID,USERID);
    END LOOP;
END P_INSERT_JD_PARTS;


```
我这个存储过程首先是for in 循环取出数据并且赋值给DATAS 然后再把每个数据插入到表TT_PART_DEFINE,LOOP开始循环 END LOOP 结束循环.
PKG_PART_UTIL.P_INSERT_HIS(PARTID,USERID);这一行是调用另一个别人写好的存储过程,调用方式是 包.存储过程然后传入参数.
给存储过程传参在as前 定义变量在as后.
TT_PART_DEFINE的PART_ID是必须字段,所以我们插入的时候要插入id进去,那怎么插入不重复的id呢.我首先想到的是用max(PART_ID),但是不行,这里不允许使用组函数.所以使用SEQ_OWNID.NEXTVAL获取id  SEQ_OWNID是自己写的序列,有最小值最大值,增量等.调用NEXTVAL取下一个.但是有函数可以获取id，底层也是用的这个序列但是进行了格式化处理，所以这里用SELECT F_GETID INTO PARTID FROM DUAL;赋值给变量，然后使用变量。
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
select ZH.NEXTVAL from dual  怎么样，序列的创建很简单得，因为oracle数据库没有自增（auto_increament）所以用序列很方便.
语句结束需要;
### 补充
使用set serveroutput on 命令设置环境变量
serveroutput为打开状态，从而使得pl/sql程序能够再SQL*plus中输出结果
declare 就是用来定义变量用的（貌似输出参数一定要用declare定义，并且调用的时候要传入 msg）
比如
```sql
create or replace procedure p2
(name in varchar2,age int,msg out varchar2)
--参数列表中，声明变量类型时切记不能写大小，只写类型名即可，例如参数列表中的name变量的声明
--参数列表中，输入参数用in表示，输出参数用out表示，不写时默认为输入参数。
------------输入参数不能携带值出去，输出参数不能携带值进来，当既想携带值进来，又想携带值出去，可以用in out
as
begin
  msg:='姓名';
  --赋值时除了可以使用：=，还可以用into来实现
  --上面子句等价于select '姓名'||name||',年龄'||age into msg from dual;
end;
--执行存储过程
declare
  msg varchar2(100);
begin
  p2('张三',23,msg);
  dbms_output.put_line(msg);
end;
```
另外还有很多语句可以使用,如:
```sql
create or replace procedure test_p is
begin
  if 1=2 then
  null;
else if 2 = 3 then
  null;
  end if;
end if;

end test_p;
```
```sql
CREATE OR REPLACE PROCEDURE GIAO
(i in NUMBER)
as
BEGIN
  for datas in(
    select * from Z_H)LOOP
    BEGIN
      IF i<= 2 then
      UPDATE Z_H T SET T.NAME='小于2';
      else
      UPDATE Z_H T SET T.NAME='大于2';
      end if;
      end;
      end loop;
      end;
      
      
      begin 
        GIAO(3);
        end;
```
还有commit  exception Rollback等等 


