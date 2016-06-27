---
categories: postgresql,sql
...
1.数据库信息
```
postgres安装目录：   /data/pgsql
表空间 ：   /data/db/pgsql/thebitsea
数据库名：  thebitsea
模式名：      radius   wifipro
用户：         thebigwifi

备注： 数据库才有表空间概念，模式，表都没有
```

2.表空间
```
# mkdir -p /data/db/pgsql/thebitsea
# chown -R postgres:postgres /data/db/pgsql/thebitsea
postgres=# create tablespace "thebitsea" location '/data/db/pgsql/thebitsea'; 

带用户的创建方式
postgres=# create tablespace "thebitsea" owner thebigwifi location '/data/db/pgsql/thebitsea'; 

创建是不带owner，授权表空间给用户
postgres=# GRANT ALL PRIVILEGES ON TABLESPACE thebitsea TO thebigwifi;
```
```
备注：
        pg_global 是系统全局表空间，存储了关键的共享系统目录。pg_default 是系统默认表空间，可通过set default tablespace=tablespacename来指定为其他表空间，在建立数据库、表、索引等数据库对象时，若不指定表空间参数，则系统自动将对象创建到默认表空间中。
        如create table tt(id int) tablespace space1.该语句等价于set default tablespace=space1;create table tt(id int);
```

查询：
```
postgres=# \db
               List of tablespaces
    Name    |   Owner    |       Location       
------------+------------+-----------------------
pg_default | postgres   |
pg_global  | postgres   |
thebitsea  | thebigwifi | /data/pgsql/thebitsea
(3 rows)

postgres=# select * from pg_tablespace;
  spcname   | spcowner | spcacl | spcoptions
------------+----------+--------+------------
pg_default |       10 |        |
pg_global  |       10 |        |
thebitsea  |    17172 |        |
(3 rows)
```
移动数据目录到其他位置：  需要修改pg_tablspace这个表内对应的表空间的Location字段为新的位置后，重启数据库

3.数据库
```
CREATE DATABASE name
    [ [ WITH ] [ OWNER [=] user_name ]
           [ TEMPLATE [=] template ]
           [ ENCODING [=] encoding ]
           [ LC_COLLATE [=] lc_collate ]
           [ LC_CTYPE [=] lc_ctype ]
           [ TABLESPACE [=] tablespace_name ]
           [ CONNECTION LIMIT [=] connlimit ] ]  

postgres=# CREATE DATABASE thebitsea OWNER=thebigwifi ENCODING='UTF8' TABLESPACE=thebitsea;

将数据库授权给用户
postgres=# GRANT ALL PRIVILEGES ON DATABASE thebitsea TO thebigwifi; 
```

4.模式
```
postgres=# \c thebitsea;

thebitsea=# CREATE SCHEMA radius AUTHORIZATION thebigwifi;

thebitsea=# CREATE SCHEMA wifipro AUTHORIZATION thebigwifi;

thebitsea=# \dn
   List of schemas
  Name   |   Owner    
---------+------------
 public  | postgres
 radius  | thebigwifi
 wifipro | thebigwifi
(3 rows)
```
指定模式给用户授权
```
GRANT { { SELECT | INSERT | UPDATE | DELETE | TRUNCATE | REFERENCES | TRIGGER }
    [, ...] | ALL [ PRIVILEGES ] }
    ON { [ TABLE ] table_name [, ...]
         | ALL TABLES IN SCHEMA schema_name [, ...] }
    TO { [ GROUP ] role_name | PUBLIC } [, ...] [ WITH GRANT OPTION ]
```

```
postgres=# \c thebitsea;

thebitsea=# GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA radius to thebigwifi;

thebitsea=# GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA wifipro to thebigwifi;
```

5.用户
thebigwifi
```
$ createuser thebigwifi --no-superuser --no-createdb --no-createrole -P
两次 thebitsea 密码，一次 postgres密码

postgres=# CREATE USER thebigwifi WITH PASSWORD 'test';
CREATE ROLE
```
6.创建表
```
在public下：
SET default_tablespace = thebitsea;

CREATE TABLE foo(i int);

thebitsea=# insert into foo values(1)；

INSERT 0 1

thebitsea-# \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | foo  | table | postgres
(1 row)
```

```
在 radius下

thebitsea=> \c thebitsea thebigwifi
Password for user thebigwifi: 

You are now connected to database "thebitsea" as user "thebigwifi".

thebitsea=> 

thebitsea=> create table radius.foo(id int);
CREATE TABLE
thebitsea=> insert into radius.foo values(2);
INSERT 0 1

thebitsea=> select * from radius.foo;
 id 
----
  2
(1 row)

wifipro 使用和他一样
```


7.导入
```
# 在 mysql 转化的 postgres 备份文件前面加入一行命令

# 如果是 schema 是 radius 的，需要加入，加入到注释语句完成的下一行即可

SET search_path = radius;

# 密码环境变量

export PGPASSWORD=test123

/usr/bin/psql -h localhost -U thebigwifi -p 5432 -d thebitsea < radius_postgres.sql

/usr/bin/psql -h localhost -U thebigwifi -p 5432 -d thebitsea < wifipro_postgres.sql
```


8.备份
```
# 导出数据库，不需要模式名

# 密码环境变量

export PGPASSWORD=test123

/usr/bin/pg_dump -h localhost -U thebigwifi -p 5432 -E UTF8 -o -Fp -b thebitsea -n radius | gzip > radius_postgres.sql.gz

/usr/bin/pg_dump -h localhost -U thebigwifi -p 5432 -E UTF8 -o -Fp -b thebitsea -n wifipro | gzip > wifipro_postgres.sql.gz
```
数据库 thebitsea 备份

涉及到 数据库 thebitsea 的权限问题，所以用postgres 用户来备份，因为public有用户postgres的表，如果没有，则可以用下面的语句：
```
pg_dump -h localhost -U postgres -p 5432 -E UTF8 -o -Fp -b thebitsea | gzip > thebitsea_postgres.sql.gz

pg_dump -h localhost -U thebigwifi -p 5432 -E UTF8 -o -Fp -b thebitsea | gzip > thebitsea_postgres.sql.gz
```
备份成文本文件格式
```
$ pg_dump -h 192.168.56.101 -U postgres -c -f thebitsea_`date +%F`.sql thebitsea
```


整个postgres全备
```
/usr/bin/pg_dumpall -h 192.168.56.101 -U postgres -p 5432 | gzip > postgres_backup.sql.gz
```



9.备份管理工具[[BR]]
pg_rman   [[BR]]

这个版本的postgres没有这个工具，暂时不考虑这种方式管理备份。  [[BR]]



10.恢复[[BR]]

数据库恢复
```
# pg_restore -U thebigwifi -p 5432 -d thebitsea thebitsea_postgres.sql
$ psql -U postgres -p 5432 -d thebitsea thebitsea_2014-09-21.sql
```
分模式的恢复
```
# pg_restore -U thebigwifi -p 5432 -d thebitsea radius_postgres.sql
# pg_restore -U thebigwifi -p 5432 -d thebitsea wifipro_postgres.sql
```

11.radius 索引
```
CREATE INDEX radacct_active_user_idx ON radacct (UserName, NASIPAddress, AcctSessionId) WHERE AcctStopTime IS NULL;
CREATE INDEX radacct_start_user_idx ON radacct (AcctStartTime, UserName);
CREATE INDEX radacct_stop_user_idx ON radacct (acctStopTime, UserName);

create index radacct_UserName on radacct (UserName);
create index radacct_AcctSessionId on radacct (AcctSessionId);
create index radacct_AcctUniqueId on radacct (AcctUniqueId);
create index radacct_FramedIPAddress on radacct (FramedIPAddress);
create index radacct_NASIPAddress on radacct (NASIPAddress);
create index radacct_AcctStartTime on radacct (AcctStartTime);
create index radacct_AcctStopTime on radacct (AcctStopTime);
create index radcheck_UserName on radcheck (UserName,Attribute);

create index radcheck_UserName_lower on radcheck (lower(UserName),Attribute);
create index radgroupcheck_GroupName on radgroupcheck (GroupName,Attribute);
create index radgroupreply_GroupName on radgroupreply (GroupName,Attribute);
create index radreply_UserName on radreply (UserName,Attribute);
create index radreply_UserName_lower on radreply (lower(UserName),Attribute);
create index radusergroup_UserName on radusergroup (UserName);
create index radusergroup_UserName_lower on radusergroup (lower(UserName));
create index realmgroup_RealmName on realmgroup (RealmName);
```