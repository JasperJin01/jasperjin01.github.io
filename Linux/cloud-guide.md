---
title: 阿里云服务器指导
layout: default
parent: Linux
---







## 重置系统

阿里云服务器重置为初始状态：实例 更多操作 云盘与镜像 重新初始化云盘













<span style="color: #ff6200;">5.配置远程登录</span>



```
<span style="color: #ff6200;">5.配置远程登录</span>
```





```shell
# 查看内存大小
free
free -h

# 创建新用户
sudo adduser username
sudo usermod -aG sudo usernamew
```





```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://[xxxxxx].mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
docker pull mysql:5.7
```



```
{
  "registry-mirrors": [
        "https://dockerproxy.cn",
        "https://docker.m.daocloud.io/",
        "https://huecker.io/",
        "https://dockerhub.timeweb.cloud",
        "https://noohub.ru/",
        "https://dockerproxy.net/",
        "https://docker.1panel.dev/",
        "https://docker.gh-proxy.com/"
  ]
}
```





docker的安装：安装的过程参考官网，安装过程中有一个http认证的过程，才能更新成功docker 的源好像。这个用阿里云ecs的话，要换一下认证的url，否则官网的源是不行的。

docker必须配置加速器才能用，现在的加速器好像特别少不知道为什么了。
