---
title: Python 版本/环境管理工具 PyEnv 从安装到使用
categories: python, pyenv, virtualenv
...

# PyEnv 简介

`pyenv` 是一个 Python 版本管理工具。利用 `pyenv` 及其插件可以：

* 多版本共存，比如 `pyenv` 系统中同时存在 `2.7.6`、`2.7.8`、`3.3.1`、`3.4.2`、`pypy` 等多个版本
* 利用 `pyenv-virtualenv` 插件可基于指定版本的 Python 创建 env。
* 为指定项目（目录）指定 Python 版本；
* 等等。


# 安装 pyenv

推荐用 `pyenv-installer` 安装 pyenv。

```shell
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile # 指定 pyenv 的目录，未必要在家目录
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"'  >> ~/.bash_profile # 追加 PATH
$ echo 'eval "$(pyenv init -)"'  >> ~/.bash_profile
$ eval '$(pyenv virtualenv-init -)'  >> ~/.bash_profile
$ curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash # 使用 pyenv-installer 安装
$ exec $SHELL # 重启 shell
```

# 使用 Pyenv


## 检查环境

安装 python 版本前，需要检查一下你是否可以 build pyhton：

```
pyenv doctor
```

在 ubuntu/Debian 下，可能需要执行以下命令以安装依赖：

```
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm
```

## 安装指定版本的 Python

* `pyenv install --list` # 可看可用的 Python 版本
* `pyenv install 2.7.6` # 安装 Python 2.7.6

## 使用 Pyenv Virtualenv

* `pyenv virtualenv 2.7.6 py276 --no-site-packages` # 基于 python 2.7.6 创建一个名为 py276 的 env
* `pyenv virtualenv 3.3.4 py334 --no-site-packages` # 基于 python 3.3.4 创建一个名为 py334 的 env
* `pyenv activate py276` # 激活 Env 环境
* `pyenv deactivate` # 退出当前 Env 环境


## 常用命令
```shell
Some useful pyenv commands are:
   commands    List all available pyenv commands
   local       Set or show the local application-specific Python version
   global      Set or show the global Python version
   shell       Set or show the shell-specific Python version
   install     Install a Python version using python-build
   uninstall   Uninstall a specific Python version
   rehash      Rehash pyenv shims (run this after installing executables)
   version     Show the current Python version and its origin
   versions    List all Python versions available to pyenv
   which       Display the full path to an executable
   whence      List all Python versions that contain the given executable
```

# Links
* [yyuu/pyenv](https://github.com/yyuu/pyenv): [https://github.com/yyuu/pyenv](https://github.com/yyuu/pyenv)
* [pyenv-installer](https://github.com/yyuu/pyenv-installer): [https://github.com/yyuu/pyenv-installer](https://github.com/yyuu/pyenv-installer)
* [pyenv-doctor](https://github.com/yyuu/pyenv-doctor): [https://github.com/yyuu/pyenv-doctor](https://github.com/yyuu/pyenv-doctor)
