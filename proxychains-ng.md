---
title: 使用 ProxyChains-NG 在 Mac OS X 的 Terminal（终端）里使用代理
categories: mac,osx,proxy,proxychains-ng, linux, ubuntu, debian
...

# 安装与配置
## Mac OS X
* 安装：`brew install proxychains-ng`
* 编辑配置文件: `echo "socks5 	127.0.0.1 9090" >> /usr/local/etc/proxychains.conf`

## Ubuntu/Debian

* 安装：`sudo apt-get install proxychains`
* 配置：`echo "socks5 	127.0.0.1 9090" >> /etc/proxychains.conf`

# 使用
1. 创建一个别名

    `echo "alias proxy=proxychains4" >> ~/.bash_profile && source ~/.bash_profile`
    
2. 使用代理 curl twitter.com

    `proxy curl twitter.com`

3. 用代理打开 mail.app，Gmail 无碍啦（Mac OS X）
    
    `proxy open -a mail`

# links
* [Homebrew](http://brew.sh/): [http://brew.sh/](http://brew.sh/)
* [rofl0r/proxychains-ng](https://github.com/rofl0r/proxychains-ng): [https://github.com/rofl0r/proxychains-ng](https://github.com/rofl0r/proxychains-ng)