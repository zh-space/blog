---
title: docker study(搭建集群先不学了，用不到)
date: 2022-6-23 
categories:
 - Docker
tags:
 - docker
sticky: 2
---
## 思考:为什么会有Docker的出现
假定您在开发一个尚硅谷的谷粒商城，您使用的是一台笔记本电脑而且您的开发环境具有特定的配置。其他开发人员身处的环境配置也各有不同。您正在开发的应用依赖于您当前的配置且还要依赖于某些配置文件。此外，您的企业还拥有标准化的测试和生产环境，且具有自身的配置和一系列支持文件。您希望尽可能多在本地模拟这些环境而不产生重新创建服务器环境的开销。请问？  

您要如何确保应用能够在这些环境中运行和通过质量检测？并且在部署过程中不出现令人头疼的版本、配置问题，也无需重新编写代码和进行故障修复？  

答案就是使用容器。Docker之所以发展如此迅速，也是因为它对此给出了一个标准化的解决方案-----系统平滑移植，容器虚拟化技术。  

环境配置相当麻烦，换一台机器，就要重来一次，费力费时。很多人想到，能不能从根本上解决问题，软件可以带环境安装？也就是说，安装的时候，把原始环境一模一样地复制过来。开发人员利用 Docker 可以消除协作编码时“在我的机器上可正常工作”的问题。  

之前在服务器配置一个应用的运行环境，要安装各种软件，就拿尚硅谷电商项目的环境来说，Java/RabbitMQ/MySQL/JDBC驱动包等。安装和配置这些东西有多麻烦就不说了，它还不能跨平台。假如我们是在 Windows 上安装的这些环境，到了 Linux 又得重新装。况且就算不跨操作系统，换另一台同样操作系统的服务器，要移植应用也是非常麻烦的。  

传统上认为，软件编码开发/测试结束后，所产出的成果即是程序或是能够编译执行的二进制字节码等(java为例)。而为了让这些程序可以顺利执行，开发团队也得准备完整的部署文件，让维运团队得以部署应用程式，开发需要清楚的告诉运维部署团队，用的全部配置文件+所有软件环境。不过，即便如此，仍然常常发生部署失败的状况。Docker的出现使得Docker得以打破过去「程序即应用」的观念。透过镜像(images)将作业系统核心除外，运作应用程式所需要的系统环境，由下而上打包，达到应用程式跨平台间的无缝接轨运作。  
## Docker理念
Docker是基于Go语言实现的云开源项目。  

Docker的主要目标是“Build，Ship and Run Any App,Anywhere”，也就是通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户的APP（可以是一个WEB应用或数据库应用等等）及其运行环境能够做到“一次镜像，处处运行”。  

Linux容器技术的出现就解决了这样一个问题，而 Docker 就是在它的基础上发展过来的。将应用打成镜像，通过镜像成为运行在Docker容器上面的实例，而 Docker容器在任何操作系统上都是一致的，这就实现了跨平台、跨服务器。只需要一次配置好环境，换到别的机子上就可以一键部署好，大大简化了操作。  
## 一句话总结
解决了运行环境和配置问题的软件容器， 方便做持续集成并有助于整体发布的容器虚拟化技术。  
想要运行Linux虚拟机需要Centos镜像文件和VMware软件。我们的代码，运行环境等，相当于Centos镜像文件而Docker相当于VMware软件。  
## 容器与虚拟机比较
### 传统虚拟机技术
虚拟机（virtual machine）就是带环境安装的一种解决方案。
它可以在一种操作系统里面运行另一种操作系统，比如在Windows10系统里面运行Linux系统CentOS7。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。这类虚拟机完美的运行了另一套系统，能够使应用程序，操作系统和硬件三者之间的逻辑不变。  
### 容器虚拟化技术
由于前面虚拟机存在某些缺点，Linux发展出了另一种虚拟化技术：  
Linux容器(Linux Containers，缩写为 LXC)  
Linux容器是与系统其他部分隔离开的一系列进程，从另一个镜像运行，并由该镜像提供支持进程所需的全部文件。容器提供的镜像包含了应用的所有依赖项，因而在从开发到测试再到生产的整个过程中，它都具有可移植性和一致性。  
 
Linux 容器不是模拟一个完整的操作系统而是对进程进行隔离。有了容器，就可以将软件运行所需的所有资源打包到一个隔离的容器中。容器与虚拟机不同，不需要捆绑一整套操作系统，只需要软件工作所需的库资源和设置。系统因此而变得高效轻量并保证部署在任何环境中的软件都能始终如一地运行。  
## 对比
比较了 Docker 和传统虚拟化方式的不同之处：  
* 传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；  
* 容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。  
* 每个容器之间互相隔离，每个容器有自己的文件系统 ，容器之间进程不会相互影响，能区分计算资源。  

## 安装

### 首先参考官方文档，如果之前已经安装过docker那么需要先卸载之前的依赖文件

```sh

旧版本的 Docker 被称为docker或docker-engine. 如果安装了这些，请卸载它们以及相关的依赖项。

  yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
yum如果报告没有安装这些软件包，那也没关系。

的内容/var/lib/docker/，包括图像、容器、卷和网络，都被保留。Docker 引擎包现在称为docker-ce.
```

### yum安装gcc
1. 需要Centos7能上外网
2. yum -y install gcc
3. yum -y install gcc c++

### 设置储蓄库
```sh
设置存储库
安装yum-utils包（提供yum-config-manager 实用程序）并设置存储库。

 yum install -y yum-utils
 yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo   这里是国外的网站，下载都会很慢，所以要换成国内镜像
```

```sh
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  用这个
```
### 更新yum软件索引
```sh
yum makecache fast
```
### 下载docker
```sh
安装最新版本的 Docker Engine、containerd 和 Docker Compose 或进入下一步安装特定版本：

  yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
### 启动docker
```sh
启动 Docker。

 systemctl start docker
```
### 验证docker是否启动成功
```sh
hello-world 通过运行映像来验证 Docker 引擎是否已正确安装。

docker run hello-world
```
### 卸载docker
```sh
卸载 Docker Engine、CLI、Containerd 和 Docker Compose 软件包：

 yum remove docker-ce docker-ce-cli containerd.io docker-compose-plugin
主机上的映像、容器、卷或自定义配置文件不会自动删除。要删除所有映像、容器和卷：

 rm -rf /var/lib/docker
 rm -rf /var/lib/containerd
```
### 配置镜像加速
需要注册阿里云账户，使用容器镜像服务，然后点击镜像加速，可以获得一个镜像加速地址，通过这个可以提高我们docker下载镜像的速度
```sh
 mkdir -p /etc/docker
 #运行这个命令后会创建daemon.json配置文件，并且在里面追加这个json串
 tee /etc/docker/daemon.json <<-'EOF'                                     
{
  "registry-mirrors": ["https://vv339w0w.mirror.aliyuncs.com"]
}
EOF
 systemctl daemon-reload
 systemctl restart docker
```
### docker常用命令
```sh
帮助启动类命令
启动docker： systemctl start docker
停止docker： systemctl stop docker
重启docker： systemctl restart docker
查看docker状态： systemctl status docker
开机启动： systemctl enable docker
查看docker概要信息： docker info
查看docker总体帮助文档： docker --help
查看docker命令帮助文档： docker 具体命令 --help
```
```sh
镜像命令
docker images
REPOSITORY：表示镜像的仓库源
TAG：镜像的标签版本号
IMAGE ID：镜像ID
CREATED：镜像创建时间
SIZE：镜像大小
 同一仓库源可以有多个 TAG版本，代表这个仓库源的不同个版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。
如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用 ubuntu:latest 镜像
OPTIONS说明：
-a :列出本地所有的镜像（含历史映像层）
-q :只显示镜像ID。

docker search 某个XXX镜像名字
命令
docker search [OPTIONS] 镜像名字
OPTIONS说明：
--limit : 只列出N个镜像，默认25个
docker search --limit 5 redis
docker pull 某个XXX镜像名字
下载镜像
docker pull 镜像名字[:TAG]  
docker pull 镜像名字  
没有TAG就是最新版  
等价于  
docker pull 镜像名字:latest  
docker pull ubuntu  
 
docker system df 查看镜像/容器/数据卷所占的空间  
 
docker rmi 某个XXX镜像名字ID  
删除镜像  
删除单个  
docker rmi  -f 镜像ID  
删除多个  
docker rmi -f 镜像名1:TAG 镜像名2:TAG  
删除全部  
docker rmi -f $(docker images -qa)  

面试题：谈谈docker虚悬镜像是什么？  
是什么  
仓库名、标签都是<none>的镜像，俗称虚悬镜像dangling image
```
### 启动容器
```sh
docker run -it centos /bin/bash 
```
参数说明：  
-i: 交互式操作。  
-t: 终端。  
centos : centos 镜像。  
/bin/bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。  
要退出终端，直接输入 exit:  
### 列出正在运行的容器
```sh
docker ps [OPTTIIONS]
OPTIONS说明（常用）：

-a :列出当前所有正在运行的容器+历史上运行过的
-l :显示最近创建的容器。
-n：显示最近n个创建的容器。
-q :静默模式，只显示容器编号。
```
### 退出容器
```sh
exit  run进容器，退出，容器停止
ctrl+p+q run进容器，退出，容器不停止
```
### 启动yj停止的容器
```sh
docker restart 容器ID或容器名字
```
### 停止容器
```sh
docker stop 容器ID或容器名字
```
### 强制停止容器
```sh
docker kill 容器ID或容器名字
```
### 删除已经停止的容器
```sh
docker rm 容器ID
删除多个
docker rm -f $(docker ps -a -q)
或
docker ps -a -q | xargs docker rm
```
### 启动守护式容器(后台服务器)
在大部分的场景下，我们希望 docker 的服务是在后台运行的，我们可以过 -d 指定容器的后台运行模式。  
```sh
docker run -d 容器名

redis 前后台启动演示case
前台交互式启动   docker run -it redis:6.0.8
后台交互式启动   docker run -d redis:6.0.8
```
### 查看容器日志
```sh
docker logs 容器ID
```
### 查看容器内运行的进程
```sh
docker top 容器ID
```
### 查看容器内部细节
```sh
docker inspect 容器ID
```
### 进入正在运行的容器并以命令行交互
```sh
docker exec -it 容器ID /bin/bash   进入后使用exit退出不会导致容器停止
重新进入docker attach 容器ID   进入后使用exit退出会导致容器停止
```
### 从容器内拷贝文件到主机上
```sh
export 导出容器的内容留作为一个tar归档文件[对应import命令]
import 从tar包中的内容创建一个新的文件系统再导入为镜像[对应export]
案例
docker export 容器ID > 文件名.tar
cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号
```
### 将镜像发布到阿里云，自行参考官方文档



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
 docker update redis --restart=always
```
设置mysql为自动启动
```sh
 docker update mysql --restart=always
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

