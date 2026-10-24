# Docker容器管理

# 创建容器

## docker create

docker create命令新建的容器处于停滞状态，可以使用docker start命令来启动它

```Plain Text
docker create -it --name mycontainer mycentos bash
docker start mycontainer
```

## docker run

除了创建容器后通过start命令来启动也可以通过docker run直接新建并启动容器。

- 启动一个容器

```Bash
[root@docker-server ~]# docker run -it centos:latest bash
[root@4abaf8a399fe /]# 
```

- 显示正在运行的容器

```Bash
[root@docker-server ~]# docker ps 
CONTAINER ID   IMAGE           COMMAND   CREATED          STATUS          PORTS     NAMES
4abaf8a399fe   centos:latest   "bash"    47 seconds ago   Up 46 seconds             hardcore_perlman
```

- 显示所有容器，包括停止的所有容器

```Bash
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE           COMMAND   CREATED         STATUS                     PORTS     NAMES
4abaf8a399fe   centos:latest   "bash"    2 minutes ago   Exited (0) 9 seconds ago             hardcore_perlman
```

## 端口映射

- 前台启动随机映射端口

```Bash
[root@docker-server ~]# docker pull nginx
[root@docker-server ~]# docker run -P nginx
*# 随机映射端口，其实是从32768开始映射*
[root@docker-server ~]# ss -tanl
State       Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN      0      128        *:22                     *:*                  
LISTEN      0      100    127.0.0.1:25                     *:*                  
LISTEN      0      128        *:49153                  *:*                  
LISTEN      0      128       :::22                    :::*                  
LISTEN      0      100      ::1:25                    :::*                  
LISTEN      0      128       :::49153                 :::*    
```

![image\-20210609140116589\.png](图片和附件/image-20210609140116589.png)

- 指定端口映射

```Bash
*# 方式1，本地端口80映射到容器80端口*
[root@docker-server ~]# docker run -p 80:80 --name nginx-1 nginx:latest 
*# 方式2，本地ip：本地端口：容器端口*
[root@docker-server ~]# docker run -p 192.168.204.135:80:80 --name nginx-1 nginx:latest 
*# 方式3，本地ip：本地随机端口：容器端口*
[root@docker-server ~]# docker run -p 192.168.175.10::80 --name nginx-1 nginx:latest 
*# 方式4，本地ip：本地端口：容器端口/协议默认为tcp协议*
[root@docker-server ~]# docker run -p 192.168.175.10:80:80/tcp --name nginx-1 nginx:latest 
```

- 查看容器已经映射的端口

```Bash
[root@docker-server ~]# docker port nginx-1 
80/tcp -> 0.0.0.0:80
80/tcp -> :::80
```

## 后台启动容器

- 当容器前台启动时，前台进程退出容器也就退出，更多时候需要容器在后台启动

```Bash
[root@docker-server ~]# docker run -d -P --name nginx-2 nginx
c75333168c0dad9094d94828c33998294f2809ae8c5b60881707d9cc33ea4893
```

- 传递运行命令

容器需要由一个前台运行的进程才能保持容器的运行，通过传递运行参数是一种方式，另外也可以在构建镜像的时候指定容器启动时运行的前台命令

```Bash
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@docker-server ~]# docker run -d centos
9ef312d30f7a396ecb5c93b7b70e70a742f333bbe01e9112d6f22fc52aeb71b8
[root@docker-server ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                      PORTS     NAMES
9ef312d30f7a   centos    "/bin/bash"   12 seconds ago   Exited (0) 11 seconds ago             upbeat_noether
[root@docker-server ~]# docker run -d centos tail -f /etc/hosts
7b0700c01f9516f49e70ad92e7256d965e0fe4eb8ccc7b30676a03c1d8046c64
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                CREATED         STATUS         PORTS     NAMES
7b0700c01f95   centos    "tail -f /etc/hosts"   3 seconds ago   Up 2 seconds             charming_brahmagupta
```

- 单次运行，容器退出后自动删除

```Bash
[root@docker-server ~]# docker run --name hello_world_test --rm hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@docker-server ~]# 
```

# 停止容器

## 暂停容器

- 挂起容器

```Bash
[root@docker-server ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
4c6344c46c80   nginx     "/docker-entrypoint.…"   8 seconds ago   Up 8 seconds   80/tcp    wizardly_hofstadter
[root@docker-server ~]# docker pause wizardly_hofstadter 
wizardly_hofstadter
[root@docker-server ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                   PORTS     NAMES
4c6344c46c80   nginx     "/docker-entrypoint.…"   17 seconds ago   Up 16 seconds (Paused)   80/tcp    wizardly_hofstadter
```

- 取消挂起容器

```Bash
[root@docker-server ~]# docker unpause wizardly_hofstadter 
wizardly_hofstadter
[root@docker-server ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
4c6344c46c80   nginx     "/docker-entrypoint.…"   33 seconds ago   Up 33 seconds   80/tcp    wizardly_hofstadter
```

## 终止容器

```Bash
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS     NAMES
4c6344c46c80   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp    wizardly_hofstadter

[root@docker-server ~]# docker stop wizardly_hofstadter 
wizardly_hofstadter
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS                     PORTS     NAMES
4c6344c46c80   nginx     "/docker-entrypoint.…"   2 minutes ago   Exited (0) 5 seconds ago             wizardly_hofstadter

[root@docker-server ~]# docker start wizardly_hofstadter 
wizardly_hofstadter
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
4c6344c46c80   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 seconds   80/tcp    wizardly_hofstadter
```

# 删除容器

## docker rm

- 删除正在运行的容器

```Bash
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE           COMMAND   CREATED         STATUS         PORTS     NAMES
dfd5ba20c3c6   centos:latest   "bash"    8 seconds ago   Up 6 seconds             frosty_elbakyan
[root@docker-server ~]# docker rm -f dfd5ba20c3c6
dfd5ba20c3c6
```

- 批量删除容器

```Bash
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
8e3cb314c9ad   nginx     "/docker-entrypoint.…"   4 seconds ago   Up 3 seconds   80/tcp    pedantic_lovelace
4ab46864c8a3   nginx     "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   80/tcp    beautiful_spence
26a154528469   nginx     "/docker-entrypoint.…"   5 seconds ago   Up 5 seconds   80/tcp    serene_booth
2ecbf60d817a   nginx     "/docker-entrypoint.…"   6 seconds ago   Up 6 seconds   80/tcp    dreamy_bassi
d73faf8c2f7d   nginx     "/docker-entrypoint.…"   8 seconds ago   Up 8 seconds   80/tcp    beautiful_solomon
[root@docker-server ~]# docker ps -a -q
8e3cb314c9ad
4ab46864c8a3
26a154528469
2ecbf60d817a
d73faf8c2f7d
[root@docker-server ~]# docker rm -f `docker ps -a -q`
8e3cb314c9ad
4ab46864c8a3
26a154528469
2ecbf60d817a
d73faf8c2f7d
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

# 进入容器

## attach

所有使用此方式进入容器的操作都是同步显示的且exit容器将被关闭，且使用exit退出后容器关闭，不推荐使用

![image\-20210609143957975\.png](图片和附件/image-20210609143957975.png)

当有一个容器执行exit退出后会导致容器退出

## exec

执行单次命令与进入容器，退出容器后容器还在运行

```Bash
[root@docker-server ~]# docker run -d -it centos
129d518869d550e579bcff38608bae38209923dcbfab49c823d5e1473d38214a
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS        PORTS     NAMES
129d518869d5   centos    "/bin/bash"   2 seconds ago   Up 1 second             jovial_haibt
[root@docker-server ~]# docker exec -it jovial_haibt /bin/bash
[root@129d518869d5 /]# echo hello
hello
[root@129d518869d5 /]# exitexit
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
129d518869d5   centos    "/bin/bash"   46 seconds ago   Up 45 seconds             jovial_haibt
```

```Plain Text
在Docker中，docker exec命令用于在正在运行的容器中执行命令。docker exec命令的一般语法如下：

docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
其中，参数的含义如下：

OPTIONS：可选参数，用于指定一些附加选项。常用的选项包括：

-i, --interactive：保持标准输入（stdin）打开，允许用户与命令交互。
-t, --tty：分配一个伪终端（pseudo-TTY），以便于命令的交互和输出格式化。
-u, --user：指定执行命令的用户或用户组。
-d, --detach：在后台运行命令。
-e, --env：设置环境变量。
-w, --workdir：指定命令执行的工作目录。
等等。
CONTAINER：必需参数，指定要执行命令的容器名称或容器ID。

COMMAND：必需参数，指定要在容器中执行的命令。

ARG：可选参数，传递给命令的参数
```

## nsenter

nsenter命令需要通过pid进入到容器内部，不过可以使用docker inspect获取到容器的pid

- 可以通过docker inspect获取到某个容器的进程id

```Bash
[root@docker-server ~]# docker inspect -f ".State.Pid" 129d518869d5   *# .State.Pid 两边需要加上两个花括号*
7949
```

- 通过nsenter进入到容器内部

```Bash
[root@docker-server ~]# nsenter -t 7949 -m -u -i -n -p
[root@129d518869d5 /]# 
```

- 使用脚本方式进入

```Bash
[root@docker-server ~]# cat docker_in.sh 
*#!/bin/bash*docker_in(){
DOCKER_ID=$1
PID=`docker inspect -f ".State.Pid" ${DOCKER_ID}`
nsenter -t ${PID} -m -u -i -n -p
}

docker_in $1
[root@docker-server ~]# chmod +x docker_in.sh 
[root@docker-server ~]# ./docker_in.sh 129d518869d5
[root@129d518869d5 /]# exitlogout
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
129d518869d5   centos    "/bin/bash"   14 minutes ago   Up 14 minutes             jovial_haibt
[root@docker-server ~]# 
```

# 查看容器日志

- 查看日志可以在不进入容器的前提下查看，即使退出容器也不会停止

- `docker logs 容器名`

    - 可以查看容器的现有日志

- `docker logs -f 容器名`

    - 持续追踪，查看容器日志

# 指定容器DNS

dns服务，默认采用dns地址

一是通过将dns地址配置在宿主机上

二是将参数配置在docker启动脚本里面

```Bash
[root@docker-server ~]# docker run -it --rm --dns 8.8.8.8 centos bash
[root@a6ce80126e75 /]# cat /etc/resolv.conf 
nameserver 8.8.8.8
[root@a6ce80126e75 /]# ping www.baidu.com -c 1
PING www.a.shifen.com (180.101.49.11) 56(84) bytes of data.
64 bytes from 180.101.49.11 (180.101.49.11): icmp_seq=1 ttl=127 time=9.35 ms

--- www.a.shifen.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 9.346/9.346/9.346/0.000 ms
[root@a6ce80126e75 /]# exitexit
```

# 导入导出容器

## docker export

导出容器是指，导出一个已经创建的容器到一个文件，不管此时这个容器是否处于运行状态，**导出一个容器快照**

```Bash
[root@docker-server ~]# docker run -d -it centos
43f2397b9456d27a3b84dba0d79ae9a1dd8dddf40440d7d73fca71cddea0e10d
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
43f2397b9456   centos    "/bin/bash"   2 seconds ago   Up 2 seconds             awesome_rubin
[root@docker-server ~]# docker export -o /opt/centos.tar 43f
[root@docker-server ~]# ll /opt/centos.tar
-rw------- 1 root root 216525312 6月   9 13:28 /opt/centos.tar
```

## docker import

导出的文件可以使用docker import命令导入变成镜像，**导入一个容器快照到本地镜像库**

```Bash
[root@docker-server ~]# docker import /opt/centos.tar mycentos:v1
sha256:acf250a6cabb56e0464102dabedb0a562f933facd3cd7b387e665459da46bf29
[root@docker-server ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
mycentos      v1        acf250a6cabb   9 seconds ago   209MB
nginx         latest    d1a364dc548d   2 weeks ago     133MB
hello-world   latest    d1165f221234   3 months ago    13.3kB
centos        latest    300e315adb2f   6 months ago    209MB
```

适用场景：主要用来制作基础镜像，比如从一个ubuntu镜像启动一个容器，然后安装一些软件和进行一些设置后，使用docker export保存为一个基础镜像。然后把这个镜像分发给其他人使用，作为基础的开发环境。\(因为export导出的镜像只会保留从镜像运行到export之间对文件系统的修改，所以只适合做基础镜像\)

# LXC和Docker的区别

- LXC：带完整 init、systemd，容器里面是一套完整 Linux 环境，可以 ssh 进去、跑多个服务，像低配虚拟机；迁移麻烦，没有标准镜像分层，适合长期运行整套环境（Proxmox 里的 LXC 就是这个）

    - 有systemd服务，是在宿主机的systemd的服务下的一个小分支

    - 臃肿

- Docker：设计理念**一个容器一个应用**，PID1 就是业务进程，不带完整 init；有分层镜像、Dockerfile、仓库，主打一次构建到处运行，容器设计为可销毁、弹性扩缩，适合微服务、CI/CD

    - 无systemd服务

    - 轻便

# 在容器中开启SSH

## 下载并开启SSH

- 下载ssh的服务`openssh-server`

- 编辑对应的内容`/etc/ssh/sshd-config`

- 查看ssh启动的相关命令`systemctl cat sshd`

- 启动ssh`/usr/sbin/sshd`

- 检查是否启动成功`ps aux`

## 容器远程登录

sshd前台启动

- `docker run -d -p 8022:22 myubuntu:26.04 /usr/sbin/sshd -D`

- 可以通过cmd登录容器

![image\.png](图片和附件/image.png)

## 容器连接失败分析

- 主要罪魁祸首：容器ssh密钥动态变化

    一句话： **每次新建这个 ssh 容器，容器内部会自动重新生成一套 ssh 主机密钥，指纹就变了，ssh 客户端就会报「主机身份变了」。**

    拆开讲：

    1. ssh 服务（sshd）需要一套**主机密钥**（`ssh_host_ed25519_key`这类），用来给服务器做身份标识、加密握手。

    2. 你的 ssh 镜像里**没有预装固定的主机密钥**；容器第一次启动时，脚本自动生成这组密钥。

    3. Docker 容器是隔离文件系统： 

        - 容器删了（`docker rm`），容器里面生成的这组密钥跟着一起消失

        - 再 `docker run` 重新拉起，又重新生成全新密钥 → **指纹不一样**

    4. Windows 这边 ssh 客户端记住了上一次的指纹，这次收到新指纹，直接拦截，就是你看到的警告。

    > 对比：虚拟机（Rocky/Ubuntu 宿主机）装好系统只生成一次主机密钥，只要不重装系统，密钥不变； 而临时 ssh 容器默认每次重建都换密钥。
    > 
    > 

    两种解决思路对照：

    - 不想折腾：连接时临时跳过指纹校验（适合本地测试容器）

    - 想要稳定：把提前生成好的密钥用 `-v` 挂载进容器`/etc/ssh/`，密钥固定不变

# 指令示例

```Bash
[root@localhost ~]# cat /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://service-crproxy.onrender.com"],
  "insecure-registries": ["service-crproxy.onrender.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
[root@localhost ~]# systemctl restart docker

docker images
docker pull python:3.12.0
docker run -d --name web-server -p 8000:8000 -v /root/web:/root/web python:3.12.0 python3 -m http.server -d /root/web

docker ps -a
docker ps -l

docker logs -f web-server


docker run --name myubuntu -it ubuntu 
apt update
apt install -y vim net-tools openssh-server iputils-ping
vim /etc/ssh/sshd_config
省略，开启root登录
passwd
exit

docker export -o myubuntu.tar myubuntu
gzip myubuntu.tar
docker import myubuntu.tar.gz myubuntu:26.04
```



