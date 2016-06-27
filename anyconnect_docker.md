---
title: 使用 Dokcer 创建一个 Any Connect 服务
categories: proxy, anyconnect,docker
...


# 下载相关配置

```bash
cd ~; git clone https://github.com/wppurking/ocserv-docker.git
```

# 删除默认用户

```bash
rm ~/ocserv-docker/ocserv/ocpasswd && touch ~/ocserv-docker/ocserv/ocpasswd
```

# 更改防火墙

```bash
sudo ufw allow 443/tcp && sudo ufw allow 443/udp
```

# 启动服务

```bash
docker run -d --name shajiquan_anyconnect --privileged -v ~/ocserv-docker/ocserv:/etc/ocserv -p 443:443/tcp wppurking/ocserv
```

# 自定义用户名密码

## 得到当前 container 的 id

```bash
docker ps | grep shajiquan_anyconnect | awk '{print $1}'
```

## 生成密码

```bash
docker exec -it CONTAINER_ID ocpasswd USERNAME
```

* `CONTAINER_ID`: 上文据说的 container 的 id
* `USERNAME`: 用于连接 any connect 用户名
 
运行如上命令后，终端会提示我们输入 2 次密码。

## 查看密码文件

```bash
docker exec -it CONTAINER_ID cat ocpasswd
```

或者

```
cat ~/ocserv-docker/ocserv/ocpasswd
```


##  查看运行日志

```bash
docker ps | grep shajiquan_anyconnect | awk '{print $1}' | xargs docker logs
```

# Links
* [wppurking/ocserv-docker](https://hub.docker.com/r/wppurking/ocserv/): [https://hub.docker.com/r/wppurking/ocserv/](https://hub.docker.com/r/wppurking/ocserv/)
* [wppurking/ocserv-docker](https://github.com/wppurking/ocserv-docker): [https://github.com/wppurking/ocserv-docker](https://github.com/wppurking/ocserv-docker)