---
title: 软件安装与配置
layout: default
parent: 电商
nav_order: 1
---



1. TOC
{:toc}



## Docker

### 安装Docker





```shell
sudo systemctl start docker
sudo systemctl enable docker # 启用开机自启
docker -v
sudo docker images # 检查当前下载的镜像

```







### Docker介绍





### Docker安装镜像

下载镜像

```shell
# https://hub.docker.com/
docker pull mysql:5.7
docker pull redis

sudo docker images # 检查当前下载的镜像
```





### Docker启动容器

**启动MySQL容器**

```shell
# 主机:容器
```



```shell
sudo docker run -p 3456:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=011013 \
-d mysql:5.7
```

-v是docker的镜像卷配置。docker容器默认会把容器的目录写入到**容器的可写层中**（位于主机文件系统中一个专用的存储区域），这些数据会随着容器的删除而被清除，因此不适合存储需要持久化的数据。

-v指定后会将指定目录的数据存储到主机的卷中，而不是写入文件系统。



```shell
docker ps # 查看当前实例

docker ps -a # 可以显示启动失败的实例
docker logs <containerid>
docker rm -f mysql # 删除容器

docker update redis --restart=always # 设置docker中的服务自动启动
```







# Git



```shell
git config --global user.name "jasper"

git config --global user.email "2426069686@qq.com"
# GitHub判定哪个作者提交代码（在GitHub主页的显示）是根据这个email来判断的
# git的每次提交记录（包括name和email）可以通过git log查看
# 修改这个email，就可以改变GitHub显示的提交作者了

# 执行命令，按三次回车
ssh-keygen -t rsa -C "2426069686@qq.com"
# Your identification has been saved in /Users/jmjin/.ssh/id_rsa
# 这里的邮箱是一个注释，没有实际作用。。。

# 具体的文件路径会给出
cat /User/jmjin/.ssh/id_rsa.pub

# 到git平台设置中找到ssh密钥，把这一串字符复制粘贴进去

# 测试是否成功
ssh git@github.com
# Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.
# 表示连接成功
```



```shell
git log # 在项目目录中使用该命令，查看git提交的记录
```





## MySQL 配置文件

```
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
init_connect='set collation_connection = utf8_unicode_ci'
init_connect='set NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve

```



我直接执行sql文件，会出现乱码（不太清楚原因）。所以我把sql文件内容粘贴到navicat创建的查询中再执行，就不会乱码。

