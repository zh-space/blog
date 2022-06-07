---
title: docker study
date: 2021-11-22 
tags:
 - docker
---
```sh
vagrant up
vagrant reload
vagrant ssh
```
强行删除容器
```sh
docker rm -f 容器id
```

切换到管理员模式
```sh
su root 
```
### 下载镜像文件
```sh
docker pull mysql:5.7
```
### 创建实例并启动
```sh
docker run \
--privileged=true \
-p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7
```
查看启动的容器
```sh
docker ps 
```
查看所有的容器
```sh
docker ps  -a
```
设置reids为自动启动
```sh
sudo docker update redis --restart=always
```
设置mysql为自动启动
```sh
sudo docker update mysql --restart=always
```
查看容器id
```sh
333e2828f035   
```
进入docker容器
```sh
docker exec -it 333e2828f035     /bin/bash
```
登录到 mysql 中
```sql
mysql -u root -p;
use mysql;
```

