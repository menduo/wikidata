---
title: 利用 SSH-KeyGen 为不同用户生成独立的 RSA
categories: ssh, rsa, git
...

## 生成默认

一般而言，生成 RSA 密钥，再复制公钥到服务端或 github 类的网站里。

注意：先用 `ls ~/.ssh` 看看你是不是已经生成过了，否则的话小心覆盖掉！

```shell
ssh-keygen -t rsa -C 'a@b.com'
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

## 生成多个

```shell
ssh-keygen -t rsa -f ~/.ssh/a1 -C "a1@b.com"
ssh-keygen -t rsa -f ~/.ssh/a2 -C "a2@b.com"
```

## 配置

`vi ~/.ssh/config`

```shell
# a1
host github.com-a1
    hostname github.com
    IdentityFile ~/.ssh/a1
    User git
```


## Git Repo

```shell
cd /project-dir
git init
git remote add origin git@github.com-a1:username/repo.git
# or:
# git remote set-url origin git@github.com-a1:username/repo.git
```

## 追加 rsa pub 到目标主机

如果你需要在服务器上追加 rsa pub key，可以参照下面的方法。如果只是要使用 github 之类的服务，那么请复制 a1.pub 文件的内容到网站指定地方。

### 手动上传，手动追加
1. 在本机：`scp ~/.ssh/a1.pub $remote_userr@$remote_host:~/`
2. 在服务端：`cat ~/a1.pub >> ~/.ssh/authorized_keys && rm ~/a1.pub`
     
### 自动追加:
1. `ssh-copy-id -i ~/.ssh/a1.pub $remote_user@$remote_host`


### 服务端设置相关文件权限

```
chmod go-w ~
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```


## 直接使用 key 
```
ssh-agent bash -c 'ssh-add ~/.ssh/my_private_key; git clone git@bitbucket.org:username/repo.git'
```


# links
* [多台电脑共用SSH Public/Private Key](http://www.douban.com/note/267252677/): [http://www.douban.com/note/267252677/](http://www.douban.com/note/267252677/)