---
title: MySQL配置和连接
layout: default
parent: MySQL
nav_order: 1
---

<h1>MySQL配置和连接</h1>




- TOC
{:toc}

# 安装MySQL

在Ubuntu18版本非常容易安装MySQL5.7版本，新版本的ubuntu好像不太好安装老版本MySQL。

**1. 移除可能已存在的 MySQL 版本**

```shell
sudo apt-get remove --purge mysql*
sudo apt-get autoremove
sudo apt-get autoclean
```

**<font color=red>2.更新包列表并安装 MySQL 5.7</font>**

```shell
sudo apt-get update
sudo apt-get install mysql-server=5.7.*
mysql --version               # 验证MySQL安装
```

**3. 启动 MySQL**

```shell
sudo systemctl start mysql
sudo systemctl restart mysql  # 重启
sudo systemctl enable mysql   # 将 MySQL 配置为在系统启动时自动启动
service mysql start           # 更推荐用 systemctl 启动
```

**<font color=red>4.进行初步安全配置（一定要运行，配置root账户密码等操作）</font>**

```shell
sudo mysql_secure_installation
```

这里注意一下，输入密码后，让配置很多信息，感觉不懂的话都选no比较稳妥。

其中有一个设置是设置的 "Disallow root login remotely"（禁止 root 远程登录），选了好像ssh系统然后登录MySQL登不上去。

**<span style="color: #ff6200;">5.配置远程登录</span>**

上述配置后是无法远程连接MySQL数据库的（2002 - Can't connect to server on '120.26.83.211' (36)）。

* 检查云服务器，开放3306端口。（MySQL默认端口**3306**，Redis默认端口**6379**）

* 修改配置文件`/etc/mysql/mysql.conf.d/mysqld.cnf`，查找bind-address，修改为`bind-address = 0.0.0.0`（默认可能是`bind-address = 127.0.0.1`）

* 修改MySQL数据库中的user表

    ```sql
    select user, host from mysql.user; -- 此时应该看到user为root对应的host是localhost
    update mysql.user set host = '%' where user ='root';
    ```

**修改之后还是报错1698错误。经过网上查询，说是root的plugin是auth_socket的问题。**

```sql
select user, plugin from mysql.user; -- root的plugin的确是auth_socket
update mysql.user set plugin='mysql_native_password' where user='root';
```

**把他改成`mysql_native_password`，然后重新`mysql_secure_installation`，就可以连上了。**

{: .warning}

> 1045 - Access denied for user 'root'@'xxx.xxx.xxx.xxx' (using password: YES)：这个报错表示输入账号密码错了

**3. 命令登录 MySQL**

```shell
# -u 用于指定用户名。
# -p 表示需要密码（密码在提示后输入，不直接在命令中写出）。
# -h 用于指定远程主机的 IP 地址或域名。
# -P 用于指定端口（默认 3306）。
mysql -u root -p
mysql -h xxx.xxx.xxx.xxx -u jasper -p
# 为什么不直接在命令中给出密码？有安全隐患，担心密码泄露！
```



# MySQL 连接与连接池

{: .note-title}
> <p style="font-size: 1.5em;">MySQL短连接和长连接</p>
>
> **短连接**（Short Connection）和**长连接**（Long Connection）是两种不同的连接管理方式。
>
> **短连接**是指每次客户端与 MySQL 交互时，都会重新建立一个连接，完成查询或操作后立即关闭该连接。建立连接的过程是比较复杂的，所以短连接适用于轻量级的、偶尔的查询，避免长时间占用数据库资源。（建议使用长连接）
>
> **长连接**是指客户端与 MySQL 建立连接后，保持该连接持续一段时间，不会立即断开。多个查询或操作会复用这个连接。不过，**MySQL在执行sql过程时临时内存是在连接对象中管理的**。长时间不释放长连接可能导致MySQL占用内存太大。解决问题的方法有两种：
>
> 1. 定期断开长连接。使用一段时间，或者程序里面判断执行过一个占用内存的大查询后，断开连接，之后要查询再重连。
> 2. 如果你用的是MySQL 5.7或更新版本，可以在每次执行一个比较大的操作后，通过执行 `mysql_reset_connection`来重新初始化连接资源。这个过程不需要重连和重新做权限验证，但是会将连接恢复到刚刚创建完时的状态。
>
> **如何分辨短连接和长连接？**
>
> 在程序中，每次执行sql后都需要关闭连接，下次查询再打开连接的就是短连接（也就是说这是看我们怎么实现的）。目前的web框架中，我们会使用连接池进行数据库的连接，例如Springboot使用的HikariCP，这样与数据库连接的事宜就由连接池负责了。
>
> **在MySQL中，可以通过`show processlist` 命令查看当前的连接状态。**

<br>

{: .note-title}

> <p style="font-size: 1.5em;">连接池（aka. 缓冲池）</p>
>
> 连接池是**可共享和重复使用的数据库连接的缓存**。通过复用连接和高效管理连接资源，连接池使得长连接的管理更加高效，适用于高并发和频繁数据库操作的场景。
>
> 想象在你的服务器程序中可能要频繁进行数据库查询操作，每次查询建立短连接肯定是效率低的，但是长连接可能不太好管理和协调。这时我们就可以使用连接池。连接池中会有几条已经建立好的连接，当需要访问数据库时可以请求连接池中的连接进行数据库的访问，这样就可以复用连接池中的连接。
>
> **懒加载机制**：在Springboot HikariCP中，Hikari连接池会在第一次sql查询请求数据库时初始化，而不是服务启动时就立刻初始化。<br>（如果你希望在服务初始化时就初始化连接池，可以通过配置文件修改默认行为，强制连接池在应用启动时就预热。）



