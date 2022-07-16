---
title: RabbitMQ的学习(持续学习中)
date: 2022-06-11 
categories:
 - RabbitMQ
tags:
 - 消息队列
 - RabbitMQ
sticky: 1
---
## 首先安装RabbitMQ
刚刚学完docker，如果使用docker安装RabbitMQ的话就很快，一个命令就可以了，然后需要自己配置一下允许远程访问就可以了。但是我们刚开始学习的话还是要自己手动安装一下。  
```sh
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.10-management
```
1. 首先我们需要安装Erlang 环境。注意Erlang 和 rabbitmq的版本对应关系喔。还要安装socat软件包。最后安装rabbitmq就好。我的安装包都是从老师哪里拿来的，大家可以自己去官网下载。
以下是软件的安装地址，一定记得版本需要对应喔:  
```sh
【erlang下载地址】：https://hub.fastgit.org/rabbitmq/erlang-rpm/releases  

【socat下载地址】：http://www.rpmfind.net/linux/rpm2html/search.php?query=socat(x86-64)  

【rabbitmq下载地址】：https://github.com/rabbitmq/rabbitmq-server/releases  
```
2. 然后将这三个安装包都上传到linux服务器上，要记得上传到那个目录了哟。  
三个安装包的安装命令,需要按照顺序执行:
```sh
rpm -ivh erlang-21.3-1.el7.x86_64.rpm  
yum install socat -y  
rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm  
```
3. 安装好后就可以启动啦，RabbitMQ其实就是一个中间件，和大家经常使用的redis，nginx，nacos都差不多，有过这些中间件的使用经验就上手很快。  
添加开机启动 RabbitMQ 服务  
```sh
chkconfig rabbitmq-server on  
启动服务  
/sbin/service rabbitmq-server start   
查看服务状态  
/sbin/service rabbitmq-server status  
停止服务(选择执行)  
/sbin/service rabbitmq-server stop   
开启 web 管理插件  
rabbitmq-plugins enable rabbitmq_management  
```
4. 添加配置文件，解决只能localhost访问的问题
```sh
进入【/etc/rabbitmq】文件夹下  
cd /etc/rabbitmq  
编辑【rabbitmq.config】文件  
vim rabbitmq.config  
在新建的配置文件下添加如下，不要忘记后面的点:
[{rabbit,[{loopback_users,[]}]}].
```
5. 开放两个端口，一个是图形化界面需要的端口，一个是RabbitMQ服务的端口
```sh
开放5672端口命令  
/sbin/iptables -I INPUT -p tcp --dport 5672 -j ACCEPT

开放15672端口命令
/sbin/iptables -I INPUT -p tcp --dport 15672 -j ACCEPT
```
## 新增用户
```sh
创建账号  
rabbitmqctl add_user admin 123  
设置用户角色  
rabbitmqctl set_user_tags admin administrator  
设置用户权限  
set_permissions [-p <vhostpath>] <user> <conf> <write> <read>  

rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"  
用户 user_admin 具有/vhost1 这个 virtual host 中所有资源的配置、写、读权限  
当前用户和角色  
rabbitmqctl list_users 
``` 
## 卸载RabbitMQ
```sh
卸载前先停止rabbitmq服务
systemctl stop rabbitmq-server
查看rabbitmq安装的相关列表
yum list | grep rabbitmq
卸载rabbitmq-server.noarch
yum -y remove rabbitmq-server.noarch
```
## 卸载erlang
```sh
查看erlang安装的相关列表
yum list | grep erlang
卸载erlang已安装的相关内容
yum -y remove erlang-*
删除有关的所有文件
rm -rf /usr/lib64/erlang 
rm -rf /var/lib/rabbitmq
rm -rf /usr/local/erlang
rm -rf /usr/local/rabbitmq
```
