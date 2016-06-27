---
title: Docker 的安装和使用
categories: develop, docker
...

# 安装 Docker
## Deibian/Ubuntu

1. `sudo apt-get update $ sudo apt-get install wget`
2. `wget -qO- https://get.docker.com/ | sh`（会提示输入`sudo`密码）
3. `sudo usermod -aG docker 用户名`

### docker 用户组

默认情况下，docker 要求用户使用 `sudo` 权限来运行使用 docker 服务，比如 `sudo docker ps`（查看正在运行的 container）。

如果使用普通的 `docker ps` 命令会报下面的错误：

```bash
FATA[0000] Get http:///var/run/docker.sock/v1.17/containers/json: 
dial unix /var/run/docker.sock: permission denied. 
Are you trying to connect to a TLS-enabled daemon without TLS?
```

### 防火墙

1. `sudo ufw status`
2. `sudo nano /etc/default/ufw`
3. 找到并将 `DEFAULT_FORWARD_POLICY="DROP"` 改为：`ACCEPT`
4. `sudo ufw reload`
5. `sudo ufw allow 2375/tcp`

### 升级 docker

`wget -N https://get.docker.com/ | sh`

### 更改 docker 镜像、容器的储存位置

1. 停止所有容器并停止 docker：

	`docker ps -q | xargs docker kill && service docker stop`
2. 创建新目录： `mkdir -p /data/libs/docker`
3. `sudo  mv /var/lib/docker/* /data/libs/docker/`
4. `sudo mount -o bind /data/libs/docker/ /var/lib/docker`
<!--4. 编辑 `/etc/default/docker` 中的 `DOCKER_OPTS` 并加入 `-g /data/libs/docker`-->
5. `sudo restart docker`


## Mac OS X
推荐使用 [Docker Toolbox | docker](https://www.docker.com/toolbox) 安装。

### Docker toolbox 包含:

* Docker Client
* Docker Machine
* Docker Compose (Mac only)
* Docker Kitematic
* VirtualBox

### Boot2Docker

如果已经安装 `docker-machine` 则不需要。`docker-machine` 支持从 `boot2docker` 导入配置。建议在安装 docker toolbox 时让安装程序帮忙导入。



### 增加端口转发

vm 关机并执行：

``` bash
for i in {5486..5487}; do
 VBoxManage modifyvm "boot2docker-vm" --natpf1 "tcp-port$i,tcp,,$i,,$i";
 VBoxManage modifyvm "boot2docker-vm" --natpf1 "udp-port$i,udp,,$i,,$i";
done
```

### 删除端口转发

vm 关机并执行：

```bash
for i in {27017..27017}; do
 VBoxManage modifyvm "boot2docker-vm" --natpf1 delete "tcp-port$i";
 VBoxManage modifyvm "boot2docker-vm" --natpf1 delete "udp-port$i";
done
```

### 目录共享

## Debian/Ubuntu


# 编译镜像
# 发布镜像

# 常用命令
## docker run
## docker exec
## docker logs

# 删除镜像/容器

# 相关工具
## fig/docker-compose

## Docker-machine

参见 [https://github.com/boot2docker/boot2docker/blob/master/doc/WORKAROUNDS.md]()


# Usefule

## 删除某些已经关闭的容器

```bash
docker ps -a | grep "Exited (0) 10 weeks ago" | grep -v "docker_" |grep -v "shajiquan" | awk '{print $1}'|xargs docker rm
```

## 删除 `<none>` 镜像

```bash
docker images -a |grep "<none>" | awk '{print $3}'|xargs docker rmi
```


# Links