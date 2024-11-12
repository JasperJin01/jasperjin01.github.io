---
title: Mac上配置多个SSH连接
layout: default
nav_order: 3
parent: Git
---

# Git

在提交本地仓库到远程git服务器（例如GitHub）时，需要进行与远程服务器的连接。其中一种常用的连接方式是SSH连接。

简单来说，需要在本地SSH生成一个公钥和私钥，将公钥上传到GitHub平台。

由于我拥有两个GitHub账号，这两个账号在不同的项目中都需要使用。

同时GitHub平台中一个公钥只能与一个GitHub账号绑定（不可以两个GitHub账号公用一个公钥）。

因此，需要在本地创建两个SSH的公钥，并且配置连接。（[参考博客](https://www.quafoo.net/posts/Add-Multi-Github-Account-On-Same-Mac/)）



**步骤1：创建两个SSH密钥，并将公钥添加到相应的GitHub账号**

```shell
ssh-keygen -q -t rsa -C "your_email_1@example.com" -f ~/.ssh/id_rsa_github_1 -N ""

ssh-keygen -q -t rsa -C "your_email_2@example.com" -f ~/.ssh/id_rsa_github_2 -N ""

# -q: 以安静模式运行。不会显示生成过程中的详细输出信息，只输出结果
# -t rsa: 密钥加密算法
# -C "your_email_1@example.com": 为密钥添加一个注释，方便将密钥和用途关联
# -N "": 为私钥设置密码。这里是 ""，表示没有密码

ls ~/.ssh
# id_rsa_github_1 id_rsa_github_1.pub id_rsa_github_2 id_rsa_github_2.pub
```

其中`.pub`结尾的内容是公钥，需要将公钥上传给GitHub，在GitHub设置中就可以管理SSH key（别弄混了）。

```ba
cat ~/.ssh/id_rsa_github_1.pub
```



**步骤2：配置SSH config文件**

找到`~/.ssh/`目录下的config文件（没有就创建）

```shell
touch ~/.ssh/config   # 创建
open ~/.ssh/config
```

在文件中保存下面内容：

```
Host github.com
HostName github.com
IdentityFile ~/.ssh/id_rsa_github_1
PreferredAuthentications publickey

Host github_2.com 
HostName github.com
IdentityFile ~/.ssh/id_rsa_github_2
PreferredAuthentications publickey
```

**配置文件的含义说明：**

* `Host github.com`：命令行指定 `github.com` 时会匹配到这个配置项（==怎么在配置文件中匹配配置项啊？==）
* `HostName github.com`：远程服务器的域名，连接的目标
* `IdentityFile`：私钥文件路径
* `PreferredAuthentications publickey`：优先的认证方法

其中最主要的就是前三个配置项了，**Host类似于匹配规则**。HostName和IdentityFile分别按照远程服务器和私钥路径配置就好。



> 扩展一下加密算法（这个也是应该掌握的）
>
> 对称加密——AKA. 私钥加密——使用相同的密钥进行加密和
>
> 非对称加密——AKA. 公钥加密——<font color=red>有两把钥匙，公钥和私钥</font>。使用一把钥匙加锁，另一把钥匙解锁。即使用公钥加密，则需使用私钥解密；或者使用私钥加密，则需要使用公钥解密。
>
> HTTPS会话的过程既使用了对称加密又使用的非对称加密。<br>会话过程中，客户端发给服务端的报文是对称密钥加密
>
> 



**步骤3：把专用密钥添加到高速缓存中**

```shell
ssh-add --apple-use-keychain ~/.ssh/id_rsa_github_1
ssh-add --apple-use-keychain ~/.ssh/id_rsa_github_2
```

（操作即可，每台弄清有什么作用，并且是Mac专属的，其他系统上应该不需要）



**步骤4：测试连接**

```shell
ssh -T git@github.com
ssh -T git@github_2.com
# Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
```

返回的GitHub账号名应该和上传到GitHub服务器密钥是相对应的。



**步骤5：清除global config配置**

```shell
# 查看当前全局配置，看看是否配置了 user.name, user.email
git config --global user.name 
git config --global user.email

# 如果配置了全局 user.name, user.email，将他们清除
git config --global --unset user.name 
git config --global --unset user.email

# 为了方便，可以在cd到对应的仓库目录下，用下述命令配置user.name和user.email
git config --local user.name "Jasper"
git config --local user.email "JasperJin01@users.noreply.github.com"
```



```shell
# 这个是列出所有配置，包括本地配置和全局配置
git config -l
```





其他参考链接：

https://www.cnblogs.com/xiaoxi-jinchen/p/15440255.html

https://liubing.me/article/git/configuring-multiple-git-accounts-in-mac.html#gitlab

[Google搜索](https://www.google.com/search?q=mac%E8%AE%BE%E7%BD%AE%E4%B8%A4%E4%B8%AA%E8%B4%A6%E6%88%B7%E7%9A%84git+ssh&oq=mac%E8%AE%BE%E7%BD%AE%E4%B8%A4%E4%B8%AA%E8%B4%A6%E6%88%B7%E7%9A%84git+ssh&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIHCAEQIRigATIHCAIQIRigATIHCAMQIRigAdIBCDY5NjVqMGo3qAIAsAIA&sourceid=chrome&ie=UTF-8)







### **邮箱保护报错**

```
remote: https://github.com/settings/emails        
error: failed to push some refs to 'git-playground:TimaxThu/git-playground.git'
To git-playground:TimaxThu/git-playground.git
!	refs/heads/main:refs/heads/main	[remote rejected] (push declined due to email privacy restrictions)
```

应该就是在git提交时设置的`user.email`，如果是你的电子邮件（例如`timaxthu@gmail.com`），GitHub会觉得暴露隐私，拒绝推送代码。

这个是可以在GitHub设置中取消掉的。进入GitHub->Settings->Email，把这个Block command line pushes that expose my email取消勾选，就应该可以正常提交了。

<img src="https://raw.githubusercontent.com/JasperJin01/Photo/refs/heads/main/develop/2024-11-07_10-08-39.png">

同时，你也可以将`user.email`替换成GitHub 的 `noreply` 地址：

```shell
# 检查一下你设置的user.email
git config user.email

# 把 your-username 替换成GitHub账号的名字
git config user.email "your-username@users.noreply.github.com"
```





可以通过运行 `git push` 命令时增加 `--verbose` 参数来启用详细日志，这有助于诊断问题。
