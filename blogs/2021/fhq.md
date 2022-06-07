---
title: 服务器的搭建
date: 2022-3-6 
tags:
 - 服务器
---
## 由于害怕自己电脑挂掉，然后毕业设计什么的都没了，于是自己弄了一台服务器，准备把毕业设计发布到上面，因为是新手，先弄windows的试一下。
### 搭建java环境
### 搭建mysql数据库
1. 首先用宝塔帮我们下载mysql数据库
2. 把mysql服务启动起来 net start mysql
3. 然后到环境变量里面添加mysql的环境变量，变量位置是MySQL数据库的bin目录的位置。这个时候你就可以在命令提示符输入mysql查看是否成功。
成功之后要想外面能访问数据库需要修改mysql表下面的user表的Host
4. 输入mysql -u root -p 登陆数据库，输入数据库密码；
5. 输入use mysql; 使用mysql数据库；
6. mysql> select host from user where user='root';
+-----------+
| host      |
+-----------+
| 127.0.0.1 |         |
| localhost |
+-----------+
2 rows in set (0.16 sec)

7. 可以看到host里面只允许本地访问，这个时候外面要修改  % 表示允许所有访问，这个不安全。
8. 输入update user set host='%' where user='root'
