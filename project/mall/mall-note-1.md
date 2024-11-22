---
title: 电商项目总结基础篇
layout: default
parent: 谷粒商城
nav_order: 1
---


1. TOC
{:toc}




# 【初始化】数据库和Docker

## Docker

**安装Docker**

```shell
sudo systemctl start docker
sudo systemctl enable docker # 启用开机自启
docker -v
sudo docker images # 检查当前下载的镜像
```

**下载Docker镜像**

从ducker-hub拉取镜像，这一步骤需要配置Docker Hub镜像加速器，但是国内很多加速器被关停了，需要仔细找一下。

另外，即便是可以用的镜像加速器（例如在阿里云ECS服务器上配置的阿里云的Docker Hub加速器），一些镜像可能也会下载失败。

```shell
# https://hub.docker.com/
docker pull mysql:5.7
docker pull redis

sudo docker images # 检查当前下载的镜像
```

**创建Docker容器**

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

-v是docker的**镜像卷**配置。docker容器默认会把容器的目录写入到**容器的可写层中**（位于主机文件系统中一个专用的存储区域），这些数据会随着容器的删除而被清除，因此不适合存储需要持久化的数据。

-v指定后会将指定目录的数据存储到主机的卷中，而不是写入文件系统。

```shell
docker ps # 查看当前容器

docker ps -a # 可以显示启动失败的实例
docker logs <containerid>
docker rm -f mysql # 删除容器

docker update redis --restart=always # 设置docker中的服务自动启动
```



## Git

更多SSH连接配置方式见博客吧。

```shell
git config --local user.name "Jasper"
git config --local user.email "JasperJin01@users.noreply.github.com"
# GitHub判定哪个作者提交代码（在GitHub主页的显示）是根据这个email来判断的
# git的每次提交记录（包括name和email）可以通过 git log 查看
# 修改这个email，就可以改变GitHub显示的提交作者了
# 如果Github设置了邮箱隐私保护，则需要使用上面的noreply形式的Github邮箱(否则无法pull)

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
# 表示连接成功 username是Github账户的名称
```



MySQL 配置文件

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



> 直接执行sql文件，会出现乱码（不太清楚原因）。所以我把sql文件内容粘贴到navicat创建的查询中再执行，就不会乱码。



# 【小结】Spring框架相关总结

## 常见的注释和使用方法

有一篇总结的很好的文章了，请看[这里](https://javaguide.cn/system-design/framework/spring/spring-common-annotations.html)。

<p style="font-size: 1.3em;color:blue">Application</p>

```java
@SpringBootApplication // 这个注解是 Spring Boot 项目的基石，创建 SpringBoot 项目之后会默认在主类加上。
public class MainApplication {
      public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }
}
```





<p style="font-size: 1.3em;color:blue">Controller</p>



```java
@RestController // 等价于 @Controller + @ResponseBody
```






<p style="font-size: 1.3em;color:blue">Service</p>





<p style="font-size: 1.3em;color:blue">DAO</p>





<p style="font-size: 1.3em;color:blue">Entity</p>






## 配置文件

```
```







# 【项目】项目初始化

## 创建每个微服务模块

**<font color=blue>聚合服务（在pom.xml文件中）</font>**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.guigu.mall</groupId>
    <artifactId>mall</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>mall</name>
    <description>聚合服务</description>
    <packaging>pom</packaging> <!-- 打包类型为 pom -->
    
    <modules>
        <module>mall-coupon</module>
        <module>mall-member</module>
        <module>mall-order</module>
        <module>mall-product</module>
        <module>mall-ware</module>
        <module>renren-fast</module>
        <module>renren-generator</module>
        <module>mall-common</module>
        <module>mall-gateway</module>
    </modules>
</project>
```

**主要是添加了modules和packaging**



**<font color=blue>修改`.gitignore`文件</font>**

在idea git页面，标记是红色的表示没有纳入"要提交文件"的；绿色表示**新创建文件**（已纳入git）但未提交的；蓝色表示**修改**但暂未提交的</br>
屎黄色（很丑的黄色）的文件表示gitignore的文件

`**/mvnw` : 表示任意路径下的这个文件

`.gitignore` 中添加的路径只会对新添加的文件或未被跟踪的文件生效。对已提交的文件不起作用。一般情况下会把.gitignore提交到Git中。

**.gitignore格式要求：**

```
```





## 后台管理系统

相关链接：[后台管理系统后端](https://github.com/renrenio/renren-fast)，[后台管理系统前端](https://github.com/renrenio/renren-fast-vue)，[CURD代码生成器](https://github.com/renrenio/renren-generator)<br>需要声明：这些项目的版本可能会和本地的运行环境有不同，所以需要修改一下。

后台管理系统的作用就是修改商城中商品分类、商品信息等内容的。



> DEBUG：maven更新依赖失败，parent.relativePath报错
>
> 问题描述：更新maven依赖后，build中的package依然是红色的，显示一直飘红。
>
> 将依赖放到build外dependencies中，更新maven依赖后，飘红消失。也就是说maven并不会去找build中的依赖，只会去找dependencies中的。
>
> 参考：[maven更新依赖失败的原因](https://blog.csdn.net/loulanyue_/article/details/102524955)、[报错解决汇总](https://blog.csdn.net/qq_36765625/article/details/123622815)、[parent.relativePath报错](https://zhuanlan.zhihu.com/p/453547775)



>  DEBUG：p16 运行renren后台管理 循环报错（数据库Access denied for user）
>
>  报错信息：
>
>  ```
>  2024-10-31 22:04:48.177 ERROR 74692 --- [eate-1510129635] com.alibaba.druid.pool.DruidDataSource   : create connection SQLException, url: jdbc:mysql://120.26.83.211:3456/gulimall_admin?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai, errorCode 1045, state 28000
>  
>  java.sql.SQLException: Access denied for user 'root'@'211.161.157.209' (using password: YES)
>  	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:129)
>  	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:97)
>  	at com.mysql.cj.jdbc.exceptions.SQLExceptionsMapping.translateException(SQLExceptionsMapping.java:122)
>  	at com.mysql.cj.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:836)
>  	at com.mysql.cj.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:456)
>  	at com.mysql.cj.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:246)
>  	at com.mysql.cj.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:197)
>  	at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:156)
>  	at com.alibaba.druid.filter.FilterAdapter.connection_connect(FilterAdapter.java:786)
>  	at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:150)
>  	at com.alibaba.druid.filter.stat.StatFilter.connection_connect(StatFilter.java:218)
>  	at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:150)
>  	at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1572)
>  	at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1636)
>  	at com.alibaba.druid.pool.DruidDataSource$CreateConnectionThread.run(DruidDataSource.java:2550)
>  ```
>
>  很奇怪的一个问题，反正我是不理解，也没有在网上找到合理的解释。
>
>  运行renren后，会循环报错。从报错的内容中可以看到，主要的问题在于java.sql.SQLException: Access denied for user 'root'@'211.161.157.209' (using password: YES)。
>
>  这里报错的这个ip我不太清楚是什么ip，这个不是我要连接的数据库的ip。（可能是我本地的ip？）
>
>  排除了以下几个问题：
>
>  * 账号密码肯定是正确的
>  * `select host, user from mysql.user` 确保root用户可以远程登录。而且navicat、idea database都可以顺利连接数据库。不是数据库远程连接权限的问题。
>
>  在尝试更换数据库后，发现：
>
>  * 使用本地数据库，可以顺利连上
>  * 使用远程数据库，阿里ECS和虚拟机数据库，都连不上，报错内容相同，都是Access denied
>
>  **<font color=red>解决方法：MySQL权限设置有问题，不使用Root账户，新创建了一个别的用户，就好使了。</font>**
>
>  反思：既然报错中提示的是`Access denied for user 'root'@'211.161.157.209'`，权限问题，确实应该从权限入手。而root账户权限问题确实也是可能出现问题的一个地方。印象中之前配置数据库的时候也遇到过这种问题，使用root账号连接不上，换了一个新的账号就好使了。<br>这次的数据库是在Docker上配置的，也是出现了相同的情况。





## 管理系统前端

安装Nodejs

```shell
brew install node

node --version # v21.7.3
# npm是前端的包管理工具，类似maven
npm --version  # 10.5.0

npm config set registry https://registry.npmmirror.com

# 在项目中执行命令（想当于让maven下载依赖）
npm install
# package.json 文件中记录了依赖和版本
```



> DEBUG 前端报错（🔴 暂时保留的问题：chromedriver）
>
> ```
> npm ERR! code 1
> npm ERR! path /Users/jmjin/Developer/frontend/renren-fast-vue/node_modules/chromedriver
> npm ERR! command failed
> npm ERR! command sh -c node install.js
> npm ERR! Only Mac 64 bits supported.
> 
> npm ERR! A complete log of this run can be found in: /Users/jmjin/.npm/_logs/2024-10-31T15_47_30_511Z-debug-0.log
> ```
>
> 原因似乎是arm64芯片兼容性问题。
>
> 在package.json中，发现chromedriver的版本是` "chromedriver": "2.27.2"`，但是[官网](https://www.npmjs.com/package/chromedriver?activeTab=versions)中查看到的版本都已经100多了。我很怀疑这两个到底是不是一个东西。
>
> 运行`npm run dev`是用来启动前端项目的。
>
> ```shell
> npm run dev
> 
> npm install node-sass --unsafe-perm --save-dev
> ```
>
> 启动的时候如果报错还可能需要执行下面这行指令。然后他又提示我了一些python相关的报错（可能是找不到python版本，这和我本地的python环境有关）。
>
> ```shell
> alias python=/usr/bin/python3
> export PYTHON=/Users/jmjin/opt/anaconda3/bin/python
> ```
>
> 我在.zshrc添加这两条代码并编译后，再执行`npm run dev`就莫名其妙的跑起来了。






> DEBUG Git pull失败
>
> 失败描述：每次git pull提交时，会要求登录github。但是登录github后，依旧无法pull成功。
>
> ```shell
> git remote -v
> # origin  https://github.com/JasperJin01/mall.git (fetch)
> # origin  https://github.com/JasperJin01/mall.git (push)
> ```
>
> 输出表示远程仓库 URL 使用的是 **HTTPS**，而不是 **SSH**。
>
> 使用SSH方式进行认证和操作，需要将远程仓库 URL 设置为 SSH 格式。
>
> ```shell
> # 将当前的 HTTPS URL 更改为 SSH URL
> git remote set-url origin git@github.com:JasperJin01/mall.git
> 
> # 再次确认是否更新成功：
> git remote -v
> # origin  git@github.com:JasperJin01/mall.git (fetch)
> # origin  git@github.com:JasperJin01/mall.git (push)
> ```
>
> 这样就可以用SSH成功git pull了。







## generator 快速生成CRUD代码



生成的代码有很多依赖

创建mall-common，用来放一些公用的包，让每个微服务都依赖它



在product项目的pom.xml添加一个依赖，依赖common项目



### 测试项目报错

```java
package com.guigu.mall.product;

import com.guigu.mall.product.entity.BrandEntity;
import com.guigu.mall.product.service.BrandService;
import org.junit.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

// FIXME 这里必须有 RunWith
@RunWith(SpringRunner.class)
@SpringBootTest
public class MallProductApplicationTests {

    @Autowired
    BrandService brandService;

    // NOTE 这个test导包要注意一下，`import org.junit.Test;`, 不要导错了
    @Test
    public void contextLoads() {
        BrandEntity brandEntity = new BrandEntity();
        brandEntity.setLogo("华为");
        brandService.save(brandEntity);
        System.out.println("保存成功");
    }
}
```

**报错信息：**

```
java.lang.IllegalArgumentException: Property 'sqlSessionFactory' or 'sqlSessionTemplate' are required
```



<font color=red>代码是没有问题的！错误是版本问题。在pom.xml(mall-product)中设置的Springboot的版本，最开始为3就会报错，设为2就不会报错！</font>





生成过程中遇到的问题：

* Junit版本和对应的import @Test包不对。可能是sb项目默认sb3的原因吧，Tests模版中的@Test是`org.junit.jupiter.api.Test`。一定要改成`org.junit.Test`。
* 还有一些版本的问题，例如在pom.xml中springboot的版本，java的版本都改正确，避免出问题。（之前的报错bug就是配置文件中版本为sb3导致的）
* 🔴 半保留的问题。在UndoLogEntity中rollbackInfo类型由*Longblob*改成了byte[]，同时添加了@Lob注解，引入了import jakarta.persistence.Lob;包，引入了jakarta.persistence-api 依赖。
* `<dependencyManagement>` 中的依赖不会自动引入，只有在子模块中显式添加 `<dependency>` 引用时，才会生效。
* 想把junit依赖放到common里面，但是发现mybatisplus会报错。最终发现是因为自己的mybatisplus没有声明版本。（这是一种语法错误，在maven clean的时候都会报错）





# 【项目】数据库表设计

所有表之间不建立外键！外键关联消费性能





# 【项目】注册服务与远程调用

## 安装Nacos

Nacos用来做注册中心和配置中心

<font color=red>TODO：老师找的nacos的文档是从哪里找到的啊？好多年之前的链接了现在好像都没有了，有空查找一下！</font>

首先要在pom.xml中导入相关依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <version>2.1.0.RELEASE</version>
</dependency>
```



在Ubuntu系统安装nacos server

```shell
java -version
# 没有java，按照命令提示安装java（我安装的是jdk8）

# 下载nacos
wget https://github.com/alibaba/nacos/releases/download/2.2.0/nacos-server-2.2.0.zip

# 解压nacos
unzip nacos-server-2.2.0.zip
cd nacos/bin
# readlink: missing operand
# Try 'readlink --help' for more information.
# dirname: missing operand
# Try 'dirname --help' for more information.
# ERROR: Please set the JAVA_HOME variable in your environment, We need java(x64)! jdk8 or later is better! !!

sh startup.sh 
###### 最后启动失败了，不知道
```



于是使用docker

```shell
docker pull nacos/nacos-server

docker run -d --name nacos -p 8848:8848 -e MODE=standalone nacos/nacos-server
```



## ⭐️ 远程调用方法

> 概括：配置注册服务中心（引入nacos和openfeign依赖），添加nacos server配置，启用Nacos客户端和Feign客户端，创建客户端接口并调用接口

配置注册服务中心

```xml
<!--        服务注册/发现-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<!--        远程调用     -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

添加nacos server注册服务配置

```properties
# 注册服务 相关配置
spring.cloud.nacos.discovery.server-addr=xxx.xxx.xxx.xxx:8848
```

不要忘记声明spring.application.name，他是nacos查找微服务的标识。



启用Nacos客户端

```java
// 注册服务的注解，添加这个就可以在 xxx.xxx.xxx.xxx:8848/nacos 中查看到服务了
@EnableDiscoveryClient
// 远程调用注解，basePackages指定扫描 Feign 客户端接口的包路径
@EnableFeignClients(basePackages = "com.guigu.mall.member.feign")
```

```properties
# 应该是等价于这样配置
feign.client.base-packages=com.guigu.mall.member.feign
```



声明式远程调用

```java
@FeignClient("mall-coupon")
public interface AlphaFeignService {

    @RequestMapping("/coupon/coupon/member/list")
    public R membercoupons();

}
```







## 将一个模块服务注册到Nacos-server

又是一个debug了很长时间的问题。实际上还是版本不匹配的问题。

简单说来，springboot必须选择等于或高于2.6的版本，要不然会抛出各种错误。

同时，Spring Boot 在2.4.0 *版本*之后，便不再兼容Junit4。所以还要修改junit版本

[关于nacos、spring boot和spring cloud版本引起报错问题](https://blog.csdn.net/qq_38364794/article/details/124665603)

https://blog.csdn.net/weixin_44457062/article/details/122950232

[SpringBoot版本与Spring、java、maven、gradle版本对应汇总](https://blog.csdn.net/weixin_72244810/article/details/134713656)





报错：Class not found exception
`org.springframework.cloud.client.loadbalancer.AsyncLoadBalancerAutoConfiguration` 类未找到

是因为springcloud版本和SpringBoot版本不匹配导致。





## Feign远程调用

给一个模块引入open-feign依赖，他就有了调用其他服务的能力。

编写一个接口，告诉



**编写Feign代码后member无法运行服务，报错：**

错误提示 `No Feign Client for loadBalancing defined. Did you forget to include spring-cloud-starter-loadbalancer?` 表明在使用 Feign 客户端进行负载均衡时，缺少 `spring-cloud-starter-loadbalancer` 依赖。

在 Spring Cloud 2020 及更高版本中，OpenFeign 默认依赖 `spring-cloud-starter-loadbalancer` 作为负载均衡组件。如果没有添加这个依赖，Feign 在请求服务时无法启用负载均衡，从而导致上述错误。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```



**启动member服务，调用远程服务报错（netflix,ribbon相关报错）：**

java.lang.AbstractMethodError: Receiver class org.springframework.cloud.netflix.ribbon.RibbonLoadBalancerClient does not define or inherit an implementation of the resolved method 'abstract org.springframework.cloud.client.ServiceInstance choose(java.lang.String, org.springframework.cloud.client.loadbalancer.Request)' of interface org.springframework.cloud.client.loadbalancer.ServiceInstanceChooser.

解决方法：参考[文章](https://blog.csdn.net/cainiao805/article/details/132444510)（感谢作者！）

出现这个问题是因为导入的 spring-cloud-loadbalancer后nacos中 pring-cloud-starter-netflix-ribbon会与它冲突，造成loadbalanc包失效。

给注册中心依赖加入不适用ribbon进行负载均衡。

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <!--不使用Ribbon进行客户端负载均衡-->
    <exclusions>
        <exclusion>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```





# 【项目】配置中心

使用nacos作为配置中心

配置中心报错NacosConfigProperties : create config service error!properties=NacosConfigPropertie 

引入依赖：

```xml
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-bootstrap</artifactId>
     <version>3.0.3</version>
</dependency>
```

参考：[CSDN谷粒商城整合配置中心报错](https://blog.csdn.net/weixin_42198690/article/details/132574167)，[Spring nacos config 启动无效原因分析](https://blog.csdn.net/jonhy_love/article/details/120316550)



## ⭐️ 统一管理配置方法

> 概括：在后端引入依赖，创建bootstrap.properties，
> 在nacos-server端

**<font color=blue>引入依赖</font>**

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

**<font color=blue>在`bootstrap.properties`中添加配置</font>**（bootstrap.properties是在SpringBoot中的设定，会比application.properties更早的进行载入）

```properties
spring.application.name=mall-coupon
spring.cloud.nacos.config.server-addr=172.16.251.130:8848
spring.cloud.nacos.config.namespace=<xx-xxx-xxxx-xxx...xxx>
```

**命名空间 namespace**：用于 **环境隔离**，将不同的项目、租户或环境的配置完全隔离。是 Nacos 中最顶层的隔离维度。在项目中，把每个微服务的配置放在了单独的命名空间中管理。

* <font color=red>配置中必须指定命名空间！</font>
* 启动后，会在配置的命名空间中寻找**<font color=red>组(Group)</font>**。默认的组为DEFAULT_GROUP，也可以通过`spring.cloud.nacos.config.group`指定组。

**<font color=blue>添加额外的配置文件</font>**

```properties
spring.cloud.nacos.config.ext-config[0].data-id=datasource.yml
spring.cloud.nacos.config.ext-config[0].group=dev
spring.cloud.nacos.config.ext-config[0].refresh=true

spring.cloud.nacos.config.ext-config[1].data-id=mybatis.yml
spring.cloud.nacos.config.ext-config[1].group=dev
spring.cloud.nacos.config.ext-config[1].refresh=true

spring.cloud.nacos.config.ext-config[2].data-id=others.yml
spring.cloud.nacos.config.ext-config[2].group=dev
spring.cloud.nacos.config.ext-config[2].refresh=true
```

这种方式指定了配置命名空间下的<font color=red>配置文件(Data Id)和组(Group)</font>，Data Id在nacos服务端设置，相当于配置的文件名。

**<font color=blue>动态获取配置</font>**





**<font color=blue>赋值</font>**

| 特性         | `@ConfigurationProperties`           | `@Value`       |
| ------------ | ------------------------------------ | -------------- |
| **绑定机制** | 基于类的字段绑定，支持嵌套、批量绑定 | 单个属性注入   |
| **类型安全** | 支持强类型和复杂对象                 | 仅支持简单值   |
| **校验**     | 支持使用校验注解进行参数校验         | 不支持         |
| **使用场景** | 绑定配置文件的一组相关属性到一个类   | 注入单个配置值 |







```java
/**
 * 1、如何使用Nacos作为配置中心统一管理配置
 *
 * 1）、引入依赖，
 *         <dependency>
 *             <groupId>com.alibaba.cloud</groupId>
 *             <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
 *         </dependency>
 * 2）、创建一个 bootstrap.properties。
 *      bootstrap.properties是在SpringBoot中的设定，会比application.properties更早的进行载入
 *      spring.application.name=mall-coupon
 *      spring.cloud.nacos.config.server-addr=127.0.0.1:8848
 * 3）、需要给配置中心默认添加一个叫 数据集（Data Id）mall-coupon.properties。
 *     默认规则，应用名.properties
 * 4）、给 应用名.properties 添加任何配置
 * 5）、动态获取配置。
 *      @RefreshScope：动态获取并刷新配置
 *      @Value("${配置项的名}")：获取到配置。
 *      如果配置中心和当前应用的配置文件中都配置了相同的项，优先使用配置中心的配置。
 *
 * 2、细节
 *  1）、命名空间：配置隔离；
 *      默认：public(保留空间)；默认新增的所有配置都在public空间。
 *      1、开发，测试，生产：利用命名空间来做环境隔离。
 *         注意：在bootstrap.properties；配置上，需要使用哪个命名空间下的配置，
 *         spring.cloud.nacos.config.namespace=9de62e44-cd2a-4a82-bf5c-95878bd5e871
 *      2、每一个微服务之间互相隔离配置，每一个微服务都创建自己的命名空间，只加载自己命名空间下的所有配置
 *
 *  2）、配置集：所有的配置的集合
 *
 *  3）、配置集ID：类似文件名。
 *      Data ID：类似文件名
 *
 *  4）、配置分组：
 *      默认所有的配置集都属于：DEFAULT_GROUP；
 *      1111，618，1212
 *
 * 项目中的使用：每个微服务创建自己的命名空间，使用配置分组区分环境，dev，test，prod
 *
 * 3、同时加载多个配置集
 * 1)、微服务任何配置信息，任何配置文件都可以放在配置中心中
 * 2）、只需要在bootstrap.properties说明加载配置中心中哪些配置文件即可
 * 3）、@Value，@ConfigurationProperties。。。
 * 以前SpringBoot任何方法从配置文件中获取值，都能使用。
 * 配置中心有的优先使用配置中心中的，
 *
 *
 */
```





# 【项目】API 网关

## API 创建网关









## 实现网关功能的跳转测试





报错Unable to load io.netty.resolver.dns.macos.MacOSDnsServerAddressStreamProvider

```
2024-11-08 00:18:39.501 ERROR 20841 --- [ctor-http-nio-2] i.n.r.d.DnsServerAddressStreamProviders  : Unable to load io.netty.resolver.dns.macos.MacOSDnsServerAddressStreamProvider, fallback to system defaults. This may result in incorrect DNS resolutions on MacOS.
```

这个报错和mac系统arm有关，解决办法添加一个依赖就好了：

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-resolver-dns-native-macos</artifactId>
    <version>4.1.75.Final</version>
    <classifier>osx-aarch_64</classifier>
    <scope>runtime</scope>
</dependency>
```

参考：https://likefacai.com/archives/macos-m1-yun-xing-unable-to-load-io.netty.resolver.dns.macoscuo-wu-chu-li





# 【入门】前端和Vue2

[Vue2教程](https://v2.cn.vuejs.org/v2/guide/index.html)

js语言的版本：es6、es7、es8

框架：jQuery、Vue、React



VSCode live server插件



这里我们遇到了一点新东西。你看到的 `v-bind` attribute 被称为**指令**。指令带有前缀 `v-`，以表示它们是 Vue 提供的特殊 attribute。可能你已经猜到了，它们会在渲染的 DOM 上应用特殊的响应式行为。在这里，该指令的意思是：“将这个元素节点的 `title` attribute 和 Vue 实例的 `message` property 保持一致”。



```html
<p v-if="seen">现在你看到我了</p>
```

这里，`v-if` 指令将根据表达式 `seen` 的值的真假来插入/移除 `<p>` 元素。







---

DOM操作是什么？

vue主要讲解的就是一个“双绑定”，也就是数据发生了变化，页面可以跟着变；或者页面发生了变化（用户填入的数据）数据也可以跟着变（不需要向之前jQuery写很多DOM操作）。



```shell
npm init -y # 在当前文件夹执行，让npm初始化项目
npm install vue
```



```html
<div id="app">
    <h1> {{name}}, 非常帅</h1> <!-- 从数据区找到name -->
</div>


<script src="./node_modules/vue/dist/vue.js"></script>
<script>
    let vm =  new Vue({
        el: "#app", // id 是元素，#是元素选择器
        data: {
            name: "张三"
        }
    });
</script>
```




```
vm.name="李四"
```







# 【入门】Mybatis-Plus







# 【项目】三级分类功能







# 【项目】后台管理系统类别前端



```shell
# 启动vue前端工程
npm run dev
```













[Page Helper报错参考博客](https://blog.csdn.net/qq_44413835/article/details/125602095)
