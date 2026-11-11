# Docker Registry

Docker Register作为Docker的核心组件之一负责镜像内容的存储与分发，客户端的docker pull以及push命令都将直接与register进行交互，最初版本的registry由python实现的，由于涉及初期在安全性、性能以及API的设计上有着诸多的缺陷，该版本在0.9之后停止了开发，由新的项目distribution（新的docker register被称为Distribution）来重新设计并开发了下一代的registry，新的项目由go语言开发，所有的api底层存储方式，系统架构都进行了全面的重新设计已解决上一代registry的问题。

官方文档地址：https://docs.docker.com/registry

官方github地址: https://github.com/docker/distribution

## 搭建镜像

- 下载docker registry镜像

```bash
[root@docker-server ~]# docker pull registry
```

- 创建授权使用目录

```bash
[root@docker-server ~]# mkdir /docker/auth -p
[root@docker-server ~]# cd /docker
```

- 创建用户

```bash
[root@docker-server docker]# yum install httpd-tools.x86_64 -y
[root@docker-server docker]# htpasswd -Bbn jack 123456 > auth/htpasswd
[root@docker-server docker]# cat auth/htpasswd 
jack:$2y$05$a2wtUYyoC8p/eXzoseT9Q.dhMDgQgwkUiKVfs1z6zijk6M4UIiUsq
[root@docker-server docker]# docker run -d -p 5000:5000 -v /docker/auth/:/auth -e \
 "REGISTRY_AUTH=htpasswd" \
 -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
 -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" registry
7be5539c6cbef114b33a083ef976836a44b97ea54a7c41beb1b568878794b22a
[root@docker-server docker]# docker ps -l
CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS         PORTS                                       NAMES
7be5539c6cbe   registry   "/entrypoint.sh /etc…"   7 seconds ago   Up 6 seconds   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   nostalgic_stonebraker
[root@docker-server docker]# 
[root@docker-server docker]# 
[root@docker-server docker]# 
[root@docker-server docker]# 
[root@docker-server docker]# curl http://jack:123456@127.0.0.1:5000/v2/_catalog
{"repositories":[]}
[root@docker-server docker]# docker login  127.0.0.1:5000
Username: jack
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[root@docker-server docker]# 
```

将本机的镜像push到仓库中

```bash
[root@localhost ~]# docker pull centos:7
[root@localhost ~]# docker tag centos:7 127.0.0.1:5000/test/centos:9
[root@localhost ~]# docker push 127.0.0.1:5000/test/centos:9
The push refers to repository [127.0.0.1:5000/test/centos]
74ddd0ec08fa: Pushed 
9: digest: sha256:a1801b843b1bfaf77c501e7a6d3f709401a1e0c83863037fa3aab063a7fdb9dc size: 529
[root@localhost ~]# curl http://jack:123456@127.0.0.1:5000/v2/_catalog
{"repositories":["test/centos"]}
```

添加本地信任，信任之后就可以从自己的仓库下载东西了，由于我们测试的是127.0.0.1的地址，不用添加

```bash
cat /etc/docker/daemon.json
{
    "registry-mirrors": ["远程的仓库地址"],
    "insecure-registries": ["远程的仓库地址"],
    "exec-opts": ["native.cgroupdriver=systemd"]
}
```

先删除本地的nginx镜像，然后尝试从远程下载一次

```bash
[root@localhost ~]# docker pull 127.0.0.1:5000/test/centos:9
9: Pulling from test/centos
a1d0c7532777: Pull complete 
Digest: sha256:a1801b843b1bfaf77c501e7a6d3f709401a1e0c83863037fa3aab063a7fdb9dc
Status: Downloaded newer image for 127.0.0.1:5000/test/centos:9
127.0.0.1:5000/test/centos:9
```

# Docker 仓库之分布式harbor

参考网址：https://goharbor.io/docs/2.15.0/install-config/

Harbor是一个用于存储和分发Docker镜像的企业级Registry服务器，由vmware开源，其通过添加一些企业必须的功能特性，例如安全、标识和管理等，扩展了开源的Docker Distribution。Harbor也提供了高级的安全特性，诸如用户管理，访问控制和活动审计等。

## 安装

- 下载镜像

```bash
[root@docker-server1 ~]# wget https://github.com/goharbor/harbor/releases/download/v2.15.2/harbor-offline-installer-v2.3.1.tgz
```

- 解压缩

```bash
[root@localhost ~]# cd /opt
[root@localhost opt]# tar xzvf harbor-offline-installer-v2.15.2.tgz 
# 创建软连接，便于所有用户都能访问harbor
[root@localhost opt]# ln -sv /root/harbor /usr/local/	
# "/usr/local/harbor" -> "/root/harbor"

# 使用dockge管理harbor
[root@localhost opt]# tar xzvf harbor-offline-installer-v2.15.2.tgz 
# 转移harbor到stacks文件夹中
[root@localhost opt]# mv harbor stacks/
```

- 安装harbor

```bash
[root@localhost harbor]# cp harbor.yml.tmpl harbor.yml
# 去掉所有注释
[root@localhost harbor]# grep -Ev '#|^$' harbor.yml.tmpl > harbor.yml
[root@localhost harbor]# cat harbor.yml
hostname: 192.168.80.10	# 改成IP地址
http:
  port: 80
harbor_admin_password: Harbor12345
database:
  password: root123
  max_idle_conns: 100
  max_open_conns: 900
data_volume: /data
trivy:
  ignore_unfixed: false
  skip_update: false
  insecure: false
jobservice:
  max_job_workers: 10
notification:
  webhook_job_max_retry: 10
chart:
  absolute_url: disabled
log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /var/log/harbor
_version: 2.3.0
proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - trivy
# 安装docker-compose，但是现在的docker自带docker-cpmpose，所以不用安装
[root@localhost harbor]## curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
[root@localhost harbor]# chmod +x /usr/local/bin/docker-compose
[root@localhost harbor]# docker-compose --version
docker-compose version 1.29.2, build 5becea4c
# 现在的docker直接从这里开始
[root@localhost harbor]# ./prepare 
[root@localhost harbor]# ./install.sh 

[root@docker-server1 harbor]# ss -tnl
State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN     0      128             *:80                          *:*                  
LISTEN     0      128             *:22                          *:*                  
LISTEN     0      100     127.0.0.1:25                          *:*                  
LISTEN     0      128     127.0.0.1:1514                        *:*                  
LISTEN     0      128            :::80                         :::*                  
LISTEN     0      128            :::22                         :::*                  
LISTEN     0      100           ::1:25                         :::*  

---
# 之后的启动关闭可以通过docker-compose管理，自动生成docker-compose.yml文件
[root@docker-server1 harbor]# ls
common     docker-compose.yml    harbor.yml       install.sh  prepare
common.sh  harbor.v2.3.1.tar.gz  harbor.yml.tmpl  LICENSE
```

* 通过dockge连接的docker

![image-20261029172327558](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20261029172327558.png)

## web访问

默认用户名和密码在harbor.yml中设置为：harbor_admin_password: Harbor12345

![image-20261029172903910](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20261029172903910.png)

![image-20261029173110947](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20261029173110947.png)

## 推送镜像

一、创建项目

![image-20261029173139394](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20261029173139394.png)

二、docker 登录仓库

- 登录后推送镜像

```bash
[root@localhost ~]# cat /etc/docker/daemon.json 
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://docker.1ms.run",
    "http://192.168.198.135/"
  ],
  "insecure-registries": ["192.168.198.135"]
}


[root@localhost ~]# systemctl restart docker

# 私有仓库需要在此登录
[root@localhost ~]# docker login 192.168.198.135
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

- 打标签推送镜像

```bash
[root@localhost harbor]# docker tag tank:v1 192.168.198.135/test01/tank:v1
[root@localhost harbor]# docker push 192.168.198.135/test01/tank:v1
```

![image-20220922115258105](09.docker%E4%BB%93%E5%BA%93%E7%AE%A1%E7%90%86/image-20220922115258105.png)

# 阿里云容器镜像服务

* 登录阿里云，搜索acr，进入容器镜像服务

![image-20261029151700115](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20261029151700115.png)

* 选择个人仓库之后按照步骤来即可
* 点击进去创建好的仓库，可以通过提示，拉取或推送镜像

![image-20261029152936084](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20261029152936084.png)

