[TOC]



# 一 安装PostgreSQL

## Mac OS上安装PostgreSQL

参考一下教程：

下载地址：https://www.enterprisedb.com/downloads/postgres-postgresql-downloads。

## Centos7服务器安装

### 1. 安装存储库

```
sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm 
```

### 2. 安装客户端

```
sudo yum install postgresql11
```

### 3. 安装服务端

```
sudo yum install postgresql11-server
```

### 4. 数据库初始化

```
sudo /usr/pgsql-11/bin/postgresql-11-setup initdb
```

### 5. 设置启用开机自动启动

```
systemctl enable postgresql-11
systemctl start postgresql-11
```

### 6. 配置防火墙

```
firewall-cmd --permanent --add-port=5432/tcp  
firewall-cmd --permanent --add-port=80/tcp  
firewall-cmd --reload  
```

### 7. 修改用户密码

```
su - postgres  切换用户，执行后提示符会变为 '-bash-4.2$'
psql -U postgres 登录数据库，执行后提示符变为 'postgres=#'
ALTER USER postgres WITH PASSWORD 'postgres'  设置postgres用户密码为postgres
\q  退出数据库
```

### 8. 开启远程访问

```
sudo vim /var/lib/pgsql/11/data/postgresql.conf
    
修改#listen_addresses = 'localhost'  为  listen_addresses='*'
当然，此处‘*’也可以改为任何你想开放的服务器IP
```

### 9. 信任远程连接

```
sudo vim /var/lib/pgsql/11/data/pg_hba.conf

修改如下内容，信任指定服务器连接
# IPv4 local connections:
host    all            all      127.0.0.1/32      trust
host    all            all      (需要连接的服务器IP或0.0.0.0/0）  trust
```

### 10. 重启服务

```
systemctl restart postgresql-11
```

# 二 PostgreSQL基本操作

## 基本语句

```
\l 查看所有数据库
\c xxx 进入xxx数据库
\d 查看当前数据库中所有table
\d xxx 查看具体table xxx
```

## 创建数据库

```
CREATE DATABASE xxx;
```

## 导入数据

方法一：

```
CREATE TABLE xxx(
	update_time date,
	id text,
	title text,
	price numeric,
	sale_count int,
	comment_count int
	);#创建表格并保证列名与数据一致且类型相同

insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
```

方法二：

```
\COPY table FROM '本地路径' WITH CSV HEADER; #\COPY用于导入客户端数据，COPY用于直接导入数据库数据, with csv header是为了不导入表中第一行的列名。
```

方法三：

```
CREATE TABLE xxx AS SELECT id,MAX(price),MIN(price) FROM xxx GROUP BY id;
```

## 查看数据

```
SELECT * FROM table; #查看table中所有数据
SELECT id,title,price FROM table 
	WHERE title = 'xx' AND price > 1000 
	ORDER bY id 
	LIMIT 10; 
	#查看table中id,title,price,title为xx且price>1000的前10条数据,根据id排序
SELECT DISTINCT title FROM data; #查看所有不同的title
```

## 清洗数据

```
SELECT * FROM table WHERE price IS NULL; #查看price为缺失值的数据
DELETE FROM table WHERE price IS NULL; #删除价格为空的那一行数据
```

## 整理数据及计算

```
ALTER TABLE table RENAME COLUMN xxx TO yyy; #修改列名xxx为yyy
ALTER TABLE table ADD xxx numeric; #增加一列列名为xxx的小数
UPDATE table SET xxx=yyy*zzz; #更新计算后的数据
ALTER TABLE table DROP xxx; #删除table中xxx一整列
```

## 删除表

```
DROP TABLE xxx;
```

# 三 PostgreSQL备份与恢复

## 本地通过直接执行语句实现数据库的备份与恢复

### 1. 备份过程 

第一步 备份所有公共对象，包括编码用户，权限等

```
pg_dumpall –h xx.xx.xx.xx –U ADMIN_USER –p 5432 –g –f /xxx/global.sql
```

第二步 备份某一个数据库

- 方式一: 导出自动建数据语句 + 备份

```
# “-C”选项，可以将建库的语句也输出到文件中；如果手动建库，则需要去除该选项
pg_dump –h xx.xx.xx.xx –U ADMIN_USER–p 5432 –d xxxdb  –C  –f /xxx/xxxdb.sql
```

- 方式二: 手动建数据库 + 备份

```
# 手动建库
create database BACKUP_DB;

# 备份某数据库
pg_dump –h xx.xx.xx.xx –U ADMIN_USER –p 5432 –d BACKUP_DB –f /xxx/BACKUP_DB.sql
```

> 注意：
>
> xx.xx.xx.xx  # 数据库所在服务器IP
>
> ADMIN_USER # 具有管理员权限的账号，比如: postres
>
> BACKUP_DB # 数据库名称
>
> /xxx/ # 备份文件存储目录
>
> global.sql # 数据库全局信息-备份文件名称
>
> BACKUP_DB.sql # 备份文件名称



### 2. 恢复过程

> 注意：
>
> 还原数据的时候，根据备份的过程，先还原全局对象，再还原数据库

第一步 恢复全局的信息，包括用户，编码等

    psql –h 192.168.xx.xx –U ADMIN_USER –p 5432 –f /xxx/global.sql

第二步 恢复某数据库

- 方式一: 恢复某数据库数据-用之前生成的自动建库的文件

```
psql –h 192.168.xx.xx –U ADMIN_USER –p 5432 –f /xxx/BACKUP_DB.sql
```

- 方式二: 恢复某数据库数据-用之前生成的手动建库的文件

```
# 手动建库
create database BACKUP_DB;

# 恢复某数据库数据
psql –h 192.168.xx.xx –U ADMIN_USER –p 5432 -d BACKUP_DB –f /xxx/BACKUP_DB.sql
```

> 注意：
>
> 上面备份pg_dump中写了”-C”，它会自动建库，如果没有写这个选项，要在psql中写-d xxxdb，就是方式二的写法

### 3. 操作实例

```
# 进入psql对数据库进行操作
[talent@talent ~]$ sudo -i -u postgres psql

以下是输出结果：
[sudo] password for talent:
psql (11.7)
Type "help" for help.

# 查看现有数据库
postgres=# \l

以下是输出结果：
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 company   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 exampledb | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +
           |          |          |             |             | postgres=CTc/postgres
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(5 rows)

# 创建要备份的数据库
postgres=# create database backup_db;

以下是输出结果：
CREATE DATABASE

# 在要备份的数据库中创建表
postgres=# CREATE TABLE test_table(
postgres(# update_time date,
postgres(# id text,
postgres(# title text,
postgres(# price numeric,
postgres(# sale_count int,
postgres(# comment_count int
postgres(# );

以下是输出结果：
CREATE TABLE

# 查看创建的表信息
postgres=# \d test_table

以下是输出结果：
                Table "public.test_table"
    Column     |  Type   | Collation | Nullable | Default
---------------+---------+-----------+----------+---------
 update_time   | date    |           |          |
 id            | text    |           |          |
 title         | text    |           |          |
 price         | numeric |           |          |
 sale_count    | integer |           |          |
 comment_count | integer |           |          |

# 往表中插入数据
postgres=# insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
INSERT 0 1
postgres=# insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
INSERT 0 1
postgres=# insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
INSERT 0 1
postgres=# insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
INSERT 0 1
postgres=# insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
INSERT 0 1
postgres=# insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
INSERT 0 1
postgres=# insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
INSERT 0 1
postgres=# insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
INSERT 0 1
postgres=# insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
INSERT 0 1
postgres=# insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
INSERT 0 1
postgres=# insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
INSERT 0 1
postgres=# insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
INSERT 0 1
postgres=# insert into test_table values ('2020-03-01', 'testid','test_title',45.3, 10, 55);
INSERT 0 1

# 查看表中的数据
postgres=# select * from test_table;
 update_time |   id   |   title    | price | sale_count | comment_count
-------------+--------+------------+-------+------------+---------------
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
(13 rows)

# 创建账号
postgres=# create user test superuser;

以下是输出结果：
CREATE ROLE

# 修改密码
postgres=# alter user test password 'test';

以下是输出结果：
ALTER ROLE

# 将刚创建的用户授权给要备份的数据库
postgres=# grant all privileges on database backup_db to test;

以下是输出结果：
GRANT

# 将刚创建的用户授权给要备份的数据库中的表
postgres=# grant all privileges on table test_table to test;

以下是输出结果：
GRANT

# 退出psql
postgres=# \q

# 执行备份操作
[talent@talent ~]$ pg_dump -U postgres -f /home/talent/backup_db.sql;
Password:

# 查看是否备份到指定目录下
[talent@talent ~]$ cd /home/talent/
[talent@talent ~]$ ls

以下是输出结果：
body.text  body.txt  data  backup_db.sql  u-w


# 再次登录psql，执行恢复操作
[talent@talent ~]$ sudo -i -u postgres psql
以下是输出结果：
Password for user postgres:
psql (11.7)
Type "help" for help.

# 手动创建要恢复的数据库
postgres=# create database retore_db;

以下是输出结果：
CREATE DATABASE

# 查看现有的数据库
postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 company   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 exampledb | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +
           |          |          |             |             | postgres=CTc/postgres
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 backup_db    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
(6 rows)

# 退出psql
postgres=#\q

# 执行恢复操作
[talent@talent ~]$ psql -d retore_db -U postgres -f backup_db.sql;
以下是输出结果：
Password for user postgres:
SET
SET
SET
SET
SET

##  set_config

(1 row)

SET
SET
SET
SET
SET
SET
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
COPY 0
COPY 13
GRANT

# 进入psql执行操作
[talent@talent ~]$ sudo -i -u postgres psql

以下是输出结果：
Password for user postgres:
psql (11.7)
Type "help" for help.

# 查看现有数据库，restore_db是否存在
postgres=# \l

以下是输出结果：
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 company   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 exampledb | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +
           |          |          |             |             | postgres=CTc/postgres
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 backup_db    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 restore_db    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
(6 rows)

# 查看restore_db里面的表信息是否和backup_db一致
postgres=# \d test_table

以下是输出结果：
                Table "public.test_table"
    Column     |  Type   | Collation | Nullable | Default
---------------+---------+-----------+----------+---------
 update_time   | date    |           |          |
 id            | text    |           |          |
 title         | text    |           |          |
 price         | numeric |           |          |
 sale_count    | integer |           |          |
 comment_count | integer |           |          |
 
# 查看restore_db数据库里面的test_table里面的数据信息是否和backup_db数据库里面的test_table一致
postgres=# select * from test_table;

以下是输出结果：
 update_time |   id   |   title    | price | sale_count | comment_count
-------------+--------+------------+-------+------------+---------------
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
(13 rows)

# 退出psql
postgres=#\q


```



## 服务器之间通过shell脚本实现数据库的备份与恢复

### 1. 编写脚本

备份脚本 backup.sh 

```
#!/bin/sh

# 备份机的登录密码
export PGPASSWORD="xxx"

# pg_dump等命令所在的bin目录
export POSTGRESPATH=/usr/pgsql-11/bin/

# 要备份的数据源IP
HOST_NAME="xx.xx.xx.xx"

# 备份机的登录用户名
ADMIN_USER="xxx"

# 要备份的数据库
BACKUP_DB="xxx"

echo "backup database start......"

# 备份全局对象
$POSTGRESPATH/pg_dumpall -h $HOST_NAME -U $ADMIN_USER -p 5432 -g -f /xxx/global.sql

# 备份某一个数据库+导出自动建数据库语句
$POSTGRESPATH/pg_dump -h $HOST_NAME -U $ADMIN_USER -p 5432 -d $BACKUP_DB -C -f /xxx/$BACKUP_DB.sql

echo "backup database end....."
```

> 注意：
>
> BACKUP_DB # 数据库名称
>
> /xxx/ # 备份文件存储目录
>
> global.sql # 数据库全局信息-备份文件名称
>
> BACKUP_DB.sql # 备份文件名称



恢复脚本restore.sh 

```
#!/bin/sh

# 恢复机的登录密码
export PGPASSWORD="xxx"

# pg_dump等命令所在的bin目录
export POSTGRESPATH=/usr/pgsql-11/bin/

# 恢复机IP
HOST_NAME="xx.xx.xx.xx"

# 恢复机用户名
ADMIN_USER="xxx"

# 要恢复的数据库
RESTORE_DB="xxx"

echo "restore database start......"

#还原全局对象
$POSTGRESPATH/psql -h $HOST_NAME -U $ADMIN_USER -p 5432 -d postgres -f /xxx/global.sql

#还原数据库
$POSTGRESPATH/psql -h $HOST_NAME -U $ADMIN_USER -p 5432 -d postgres -f /xxx/$RESTORE_DB.sql

echo "restore database end......"
```



### 2. 根据实际环境修改脚本中的连接参数



### 3. 将backup.sh和restore.sh放进linux下的某一目录当中



### 4. 增加脚本backup.sh和restore.sh的运行权限

```
chmod u+x backup.sh

chmod u+x restore.sh
```

### 5. 运行备份脚本

```
# 进入备份脚本backup.sh所在目录
cd /xxx/

# 运行备份脚本
./backup.sh

# 进入备份文件所在目录
cd  /xxx/

# 查看备份文件是否生成
ls -l
```

### 6. 运行恢复脚本

```
# 进入恢复脚本restore.sh所在目录
cd /xxx/

# 运行恢复脚本
./restore.sh
```

### 7. 对照生成的库/用户/角色/schema/表/表的数据完成验证

### 8. crontab设定定时备份

```
crontab -e

0 0 * * * /[脚本所在路径]/backup.sh #设置每天0点备份

crontab -l #查看当前计划
crontab -r #删除所有计划
cat /var/mail/[脚本所在路径]#查看记录
```

### 9. 操作实例

#### 1.执行备份脚本backup.sh

> 注意：BACKUP_DB在数据库要先存在，才能备份

```
[talent@talent ~]$ ./backup.sh
```

以下是执行结果：

```
[talent@talent ~]$ ./backup.sh
backup database start......
backup database end.....
```

backup.sh脚本具体内容:

```
#!/bin/sh

# 备份机的登录密码
export PGPASSWORD="*******"

# pg_dump等命令所在的bin目录
export POSTGRESPATH=/usr/pgsql-11/bin/

# 要备份的数据源IP
HOST_NAME="********"

# 备份机的登录用户名
ADMIN_USER="postgres"

# 要备份的数据库
BACKUP_DB="testdb"

echo "backup database start......"

# 备份全局对象
$POSTGRESPATH/pg_dumpall -h $HOST_NAME -U $ADMIN_USER -p 5432 -g -f /home/talent/global.sql

# 备份某一个数据库+导出自动建数据库语句
$POSTGRESPATH/pg_dump -h $HOST_NAME -U $ADMIN_USER -p 5432 -d $BACKUP_DB -C -f /home/talent/$BACKUP_DB.sql

echo "backup database end....."
```



#### 2.删除备份好的数据库

> 注意：演示恢复数据库的操作之前，我们先故意删除备份好的数据库

```
[talent@talent ~]$ sudo -i -u postgres psql

以下是输出结果：
Password for user postgres:
psql (11.7)
Type "help" for help.

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 company   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 exampledb | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +
           |          |          |             |             | postgres=CTc/postgres
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
(6 rows)

postgres=# drop database testdb;

以下是输出结果：
DROP DATABASE

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 company   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 exampledb | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +
           |          |          |             |             | postgres=CTc/postgres
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(5 rows)

postgres=# \q

```



#### 3.执行恢复脚本restore.sh

```
[talent@talent ~]$ ./restore.sh

以下是输出结果：
restore database start......
SET
SET
SET
psql:/home/talent/global.sql:14: ERROR:  role "postgres" already exists
ALTER ROLE
psql:/home/talent/global.sql:16: ERROR:  role "test" already exists
ALTER ROLE
SET
SET
SET
SET
SET

 set_config
------------

(1 row)

SET
SET
SET
SET
CREATE DATABASE
ALTER DATABASE
You are now connected to database "testdb" as user "postgres".
SET
SET
SET
SET
SET

 set_config
------------

(1 row)

SET
SET
SET
SET
SET
SET
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
COPY 0
COPY 13
GRANT
restore database end......
```



#### 4.查看数据库是否恢复

```
[talent@talent ~]$ sudo -i -u postgres psql

以下是输出结果：
Password for user postgres:
psql (11.7)
Type "help" for help.

postgres=# \l

以下是输出结果：
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 company   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 exampledb | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +
           |          |          |             |             | postgres=CTc/postgres
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
(6 rows)

postgres=# \d test_table

以下是输出结果：
                Table "public.test_table"
    Column     |  Type   | Collation | Nullable | Default
---------------+---------+-----------+----------+---------
 update_time   | date    |           |          |
 id            | text    |           |          |
 title         | text    |           |          |
 price         | numeric |           |          |
 sale_count    | integer |           |          |
 comment_count | integer |           |          |

postgres=# select * from test_table;

以下是输出结果：
 update_time |   id   |   title    | price | sale_count | comment_count
-------------+--------+------------+-------+------------+---------------
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
 2020-03-01  | testid | test_title |  45.3 |         10 |            55
(13 rows)

postgres=#\q
```

以下是脚本restore.sh的具体内容:

```
#!/bin/sh

# 恢复机的登录密码
export PGPASSWORD="*********"

# pg_dump等命令所在的bin目录
export POSTGRESPATH=/usr/pgsql-11/bin/

# 恢复机IP
HOST_NAME="********"

# 恢复机用户名
ADMIN_USER="postgres"

# 要恢复的数据库
RESTORE_DB="testdb"

echo "restore database start......"

#还原全局对象
$POSTGRESPATH/psql -h $HOST_NAME -U $ADMIN_USER -p 5432 -d postgres -f /home/talent/global.sql

#还原数据库
$POSTGRESPATH/psql -h $HOST_NAME -U $ADMIN_USER -p 5432 -d postgres -f /home/talent/$RESTORE_DB.sql

echo "restore database end......"
```

#### 5. crontab设定定时备份

```
crontab -e

#设置每天0点备份
0 0 * * * /home/talent/backup.sh 

#查看当前计划
crontab -l 
#查看记录
cat /var/mail/home/talent
#删除所有计划
crontab -r 
```

# 四 可能异常处理

## -bash: vim: 未找到命令

如果报错-bash: vim: 未找到命令，可能是Linux环境没有安装vim编辑器。

vim編輯器需要安裝三個包： 
vim-enhanced-7.0.109-7.el5 
vim-minimal-7.0.109-7.el5 
vim-common-7.0.109-7.el5 
### 1. 查看一下你本機已經存在的包，確認一下你的VIM是否已經安裝： 

```
rpm -qa|grep vim
```

如何vim已經正確安裝，則會顯示上面三個包的名稱 

### 2. 若缺少其中某个：

```
yum -y install "package_name" 
```

比如說： vim-enhanced這個包少了，執行：yum -y install vim-enhanced 命令，它會自動下載安裝。 

### 3. 若三个都缺少：

```
yum -y install vim* 
```

即可自動安裝，完畢後，即可使用vim編輯器。

## PostgreSQL 与 pg_dump版本不匹配

```
sudo ln -s /usr/lib/postgresql/11.7/bin/pg_dump /usr/bin/pg_dump --force
```

## -bash: pg_dump: command not found

```
sudo yum install postgresql10.x86_64#服务器上
#Mac终端输入报错-bash: pg_dump: command not found
cd ~
vim ~/.bash_profile
PATH="/Library/PostgreSQL/12/bin:${PATH}"#添加路径
export PATH
```

### 服务器上报错

```
sudo yum install postgresql10.x86_64#服务器上
```

### Mac终端输入报错

```
cd ~
vim ~/.bash_profile
PATH="/Library/PostgreSQL/12/bin:${PATH}"#添加路径
export PATH
```

## 异常：FATAL:  Peer authentication failed for user "postgres"

### 异常信息

```
# 执行备份的操作
[talent@talent ~]$ pg_dump -U postgres backup_db -f \home\talent\backup_db.sql

出现异常信息：
pg_dump: [archiver (db)] connection to database "backup_db" failed: FATAL:  Peer authentication failed for user "postgres"
```

### 解决方法

1. 修改pg_hba.conf配置文件

```
[talent@talent ~]$ sudo vim /var/lib/pgsql/11/data/pg_hba.conf

找到下面的一行：
local   all             postgres                                peer
改成
local   all             postgres                                md5
```



>  说明：
>
>  - Peer authentication 是默认的配置，如果你执行操作的计算机用户名和postgres数据库名是一样的话，那么就不会出现此错误, 也不需要为你的数据库设置密码。
>
>  - md5 authentication,它需要密码。如果你执行操作的计算机用户名和数据库名不一致，就需要把Peer authentication改成md5 authentication，然后给数据库设置密码。



2. 重启postgresql

```
[talent@talent ~]$ systemctl restart postgresql-11

以下是输出结果：
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to manage system services or units.
Authenticating as: talent
Password:
```

## E212: Can't open file for writing

```
sudo chmod u+w filename
```