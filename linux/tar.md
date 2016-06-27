---
title: Linux 的 tar 命令
categories:linux
---

tar是linux下常用的打包/解包程序，同时也可以压缩/解压。

command format:
  `tar [options] file_name`

options:
*   `-c` 创建新包
*   `-t` 查看tar包的内容
*   `-x` 解包/解压缩
*   `-f filename` 后面接tar包的文件名（通常单独一项出来）,如果有多个option， f要放在最后
*   `-v` 打包/解压过程中在终端输出所有文件
*   `-z` 以gzip压缩， 后缀名通常为tar.gz
*   `-j` 以bzip2压缩， 后缀名通常为tar.bz2
*   `-C dir_name`  解包/解压到特定目录

注意，如果不加`-z`,`-j`这些压缩选项， tar打包的过程是不会压缩的。

```bash
#打包,不压缩
tar -cv -f test.tar test/

#查看tar包内容，不解包
tar -tv -f test.tar

#解包
tar -xv -f test.tar
#解包到特定目录
tar -xv -f test.tar -C target/path

#打包并以gzip压缩
tar -czv -f test.tar.gz test/
#解压缩
tar -xzv -f test.tar.gz

```