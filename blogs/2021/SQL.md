---
title: SQL高级查询(层次化查询，递归查询) study
date: 2022-4-8 
categories:
 - sql
tags:
 - sql
---
## 层次化查询
在oracle中遇到了Start With查询语句,经过查询资料了解现在总结一下.

参考文章:https://www.cnblogs.com/zyzdisciple/p/7873584.html

首先我们可以新建一个demo表做测试
```sql

create table DEMO ( ID varchar2(10) primary key, DSC varchar2(100), PID varchar2(10) )

--插入几条数据

Insert Into DEMO values ('00001', '中国', '-1');

Insert Into DEMO values ('00011', '陕西', '00001');

Insert Into DEMO values ('00012', '贵州', '00001');

Insert Into DEMO values ('00013', '河南', '00001');

Insert Into DEMO values ('00111', '西安', '00011'); I

nsert Into DEMO values ('00112', '咸阳', '00011');

Insert Into DEMO values ('00113', '延安', '00011');

start with的基本语法如下

SELECT ... FROM +

表名 WHERE + 条件3

START WITH + 条件1

CONNECT BY PRIOR + 条件2
```
条件1表示我数据的切入点,也就是我第一条数据从哪里开始.

条件2是连接条件，其中用PRIOR表示上一条记录，例如CONNECT BY PRIOR ID = PID，意思就是上一条记录的ID是本条记录的PID

条件3表示条件12执行遍历结果之后再进行条件约束.

首先 我们查询所有表示这样的,
![Image text](http://101.42.150.127:8081/blog/startwith1.png)
```sql
SELECT * FROM start_demo start with id = '00001' Connect By Prior id = pid
```

start with id = '00001' 表示切入点,也就是我的第一条数据

Connect By Prior id = pid 表示我的上一条数据的id是我当前数据的pid(如果不是就表明不是当前节点)
![Image text](http://101.42.150.127:8081/blog/startwith2.png)
反之

SELECT * FROM start_demo start with id = '00113' Connect By Prior PID = id  ---上一条记录的PID是本条记录的ID
![Image text](http://101.42.150.127:8081/blog/startwith3.png)
## 递归查询
首先建表
```sql
CREATE TABLE DISTRICT
(
  ID         NUMBER(10)                  NOT NULL,
  PARENT_ID  NUMBER(10),
  NAME       VARCHAR2(255 BYTE)          NOT NULL
);

ALTER TABLE DISTRICT ADD (
  CONSTRAINT DISTRICT_PK
 PRIMARY KEY
 (ID));

ALTER TABLE DISTRICT ADD (
  CONSTRAINT DISTRICT_R01 
 FOREIGN KEY (PARENT_ID) 
 REFERENCES DISTRICT (ID));
 
 
 insert into DISTRICT (id, parent_id, name)
values (1, null, '四川省');
insert into DISTRICT (id, parent_id, name)
values (2, 1, '巴中市');
insert into DISTRICT (id, parent_id, name)
values (3, 1, '达州市');
insert into DISTRICT (id, parent_id, name)
values (4, 2, '巴州区');
insert into DISTRICT (id, parent_id, name)
values (5, 2, '通江县');
insert into DISTRICT (id, parent_id, name)
values (6, 2, '平昌县');
insert into DISTRICT (id, parent_id, name)
values (7, 3, '通川区');
insert into DISTRICT (id, parent_id, name)
values (8, 3, '宣汉县');
insert into DISTRICT (id, parent_id, name)
values (9, 8, '塔河乡');
insert into DISTRICT (id, parent_id, name)
values (10, 8, '三河乡');
insert into DISTRICT (id, parent_id, name)
values (11, 8, '胡家镇');
insert into DISTRICT (id, parent_id, name)
values (12, 8, '南坝镇');
insert into DISTRICT (id, parent_id, name)
values (13, 6, '大寨乡');
insert into DISTRICT (id, parent_id, name)
values (14, 6, '响滩镇');
insert into DISTRICT (id, parent_id, name)
values (15, 6, '龙岗镇');
insert into DISTRICT (id, parent_id, name)
values (16, 6, '白衣镇');
commit;
```
表中数据如图：
```sql
select * from DISTRICT
```
![Image text](http://101.42.150.127:8081/blog/startwith4.png)
在10g中 Oracle提供了新的操作： CONNNECT_BY_ROOT，通过这个操作，可以获取树形查询根记录的字段。
```sql
SELECT d.*,
connect_by_root(d.id),
connect_by_root(NAME)
FROM district d
WHERE NAME='平昌县'
START WITH d.parent_id=1    --d.parent_id is null 结果为四川省
CONNECT BY PRIOR d.ID=d.parent_id
```
![Image text](http://101.42.150.127:8081/blog/startwith5.png)
这样就是查询平昌县的根节点的id和根节点的名称
然后
查询巴中市下行政组织递归路径
```sql
SELECT ID,parent_id,NAME,
sys_connect_by_path(NAME,'->') namepath,
LEVEL
FROM district 
START WITH NAME='巴中市'
CONNECT BY PRIOR ID=parent_id
```
![Image text](http://101.42.150.127:8081/blog/startwith6.png)

### with递归
with递归子类
```sql
WITH t (ID ,parent_id,NAME) --要有列名
AS(
SELECT ID ,parent_id,NAME FROM district WHERE NAME='巴中市'
UNION ALL
SELECT d.ID ,d.parent_id,d.NAME FROM t,district d --要指定表和列表，
WHERE t.id=d.parent_id
)
SELECT * FROM t;
```
![Image text](http://101.42.150.127:8081/blog/startwith7.png)

递归父类
```sql
WITH t (ID ,parent_id,NAME) --要有表
AS(
SELECT ID ,parent_id,NAME FROM district WHERE NAME='通江县'
UNION ALL
SELECT d.ID ,d.parent_id,d.NAME FROM t,district d --要指定表和列表，
WHERE t.parent_id=d.id
)
SELECT * FROM t;
```
![Image text](http://101.42.150.127:8081/blog/startwith8.png)


