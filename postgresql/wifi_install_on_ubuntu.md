---
categories:postgresql,sql
...

1.Ubuntu 版本(产线环境) [[BR]]

Ubuntu 13.04

2.安装过程
```
$ wget http://ftp.postgresql.org/pub/source/v9.3.5/postgresql-9.3.5.tar.gz
$ tar zxvf postgresql-9.3.5.tar.gz
$ cd postgresql-9.3.5

$ ./configure --prefix=/usr/local/pgsql-9.3.5 --with-pgport=5432 --with-perl --with-python --with-openssl --with-pam --without-ldap --with-libxml --with-libxslt --enable-thread-safety --with-wal-blocksize=8 --with-blocksize=8 --without-readline
$ make all
$ make install

# cp contrib/start-scripts/linux /etc/init.d/postgresql

$ cd contrib/pg_stat_statements/
$ make
$ sudo make install

$ sudo ln -s /usr/local/pgsql-9.3.5 /usr/local/pgsql

# 升级
# mkdir -p /data/db /data/db/archivelog /data/db/pg_log  /data/db/thebitsea /data/db/pgsql
# chown -R postgres:postgres /data/db

# 全备
/usr/bin/pg_dumpall -U postgres -p 5432 > postgres_backup_all.sql

# inittab
# cp contrib/start-scripts/linux /etc/init.d/postgresql
ulimit -s 81920


# 安装的库 /data/db/pgsql
# vi /home/postgres/.profile
export PGPORT=5432
export PGDATA=/data/db/pgsql
export LANG=en_US.utf8
export PGHOME=/usr/local/pgsql
export LD_LIBRARY_PATH=$PGHOME/lib:/lib64:/usr/lib64:/usr/local/lib64:/lib:/usr/lib:/usr/local/lib:$LD_LIBRARY_PATH
export DATE=`date +"%Y%m%d%H%M"`
export PATH=$PGHOME/bin:$PATH:.
export MANPATH=$PGHOME/share/man:$MANPATH
export PGUSER=postgres
export PGHOST=$PGDATA
export PGDATABASE=postgres
alias rm='rm -i'
alias ll='ls -lh'

# su - postgres
$ initdb -A md5 -D $PGDATA -E UTF8 --locale=C -U postgres -W
```

3.postgresql 运行
```
# /etc/init.d/postgresql start

# su - postgresql
$ psql

postgres=# \i postgres_backup_all.sql
安装完成之后，用户名和密码即为全备的数据库的信息了。

```