## 三节点部署MySQL group replication

1.安装MySQL, 已经测试过版本 mysql-8.0.20/mysql-5.7.37

2.创建工作目录

* s-1: mkdir /flash1/test/mysql/deploy/m1/ 
* s-2: mkdir /flash1/test/mysql/deploy/m2/
* s-3: mkdir /flash1/test/mysql/deploy/m3/ 

3.MySQL group replication service

* 启动s-1

``` bash 
rm -rf /flash1/test/mysql/deploy/m1/*
mysqld  --defaults-file=/home/ec2-user/gr/gr1.cnf --initialize-insecure
mysqld  --defaults-file=/home/ec2-user/gr/gr1.cnf &

mysql -uroot -S/flash1/test/mysql/deploy/m1/mysql.sock

```

* 设置s-1 replication 参数
``` bash

SET SQL_LOG_BIN=0;
CREATE USER 'root'@'%' IDENTIFIED BY 'password@polardb';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
CREATE USER rpl_user@'%' IDENTIFIED BY 'password@polardb';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
GRANT CONNECTION_ADMIN ON *.* TO rpl_user@'%';
GRANT BACKUP_ADMIN ON *.* TO rpl_user@'%';
GRANT GROUP_REPLICATION_STREAM ON *.* TO rpl_user@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;

```

* 启动s-1 replication 服务

```bash
mysql -uroot -h127.0.0.1 -P3750


CHANGE REPLICATION SOURCE TO SOURCE_USER='rpl_user', SOURCE_PASSWORD='password@polardb' FOR CHANNEL 'group_replication_recovery';

SET GLOBAL group_replication_bootstrap_group=ON;
START GROUP_REPLICATION;
SET GLOBAL group_replication_bootstrap_group=OFF;

SELECT * FROM performance_schema.replication_group_members;


```

* 添加其他s2,s3, 以s2为例

```bash
rm -rf /flash1/test/mysql/deploy/m2/*
mysqld  --defaults-file=/home/ec2-user/gr/gr2.cnf --initialize-insecure
mysqld  --defaults-file=/home/ec2-user/gr/gr2.cnf &

mysql -uroot -S/flash1/test/mysql/deploy/m2/mysql.sock
```

* 设置s-2 replication 参数

```bash

SET SQL_LOG_BIN=0;
CREATE USER 'root'@'%' IDENTIFIED BY 'password@polardb';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
CREATE USER rpl_user@'%' IDENTIFIED BY 'password@polardb';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
GRANT CONNECTION_ADMIN ON *.* TO rpl_user@'%';
GRANT BACKUP_ADMIN ON *.* TO rpl_user@'%';
GRANT GROUP_REPLICATION_STREAM ON *.* TO rpl_user@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;


CHANGE REPLICATION SOURCE TO SOURCE_USER='rpl_user', SOURCE_PASSWORD='password@polardb' FOR CHANNEL 'group_replication_recovery';

START GROUP_REPLICATION;

##verify
SELECT * FROM performance_schema.replication_group_members;
```

* 设置s-3 replication 参数

```bash
rm -rf /flash1/test/mysql/deploy/m3/*
mysqld  --defaults-file=/home/ec2-user/gr/gr3.cnf --initialize-insecure
mysqld  --defaults-file=/home/ec2-user/gr/gr3.cnf &

mysql -uroot -S/flash1/test/mysql/deploy/m3/mysql.sock
```

```bash

SET SQL_LOG_BIN=0;
CREATE USER 'root'@'%' IDENTIFIED BY 'password@polardb';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
CREATE USER rpl_user@'%' IDENTIFIED BY 'password@polardb';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
GRANT CONNECTION_ADMIN ON *.* TO rpl_user@'%';
GRANT BACKUP_ADMIN ON *.* TO rpl_user@'%';
GRANT GROUP_REPLICATION_STREAM ON *.* TO rpl_user@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;


CHANGE REPLICATION SOURCE TO SOURCE_USER='rpl_user', SOURCE_PASSWORD='password@polardb' FOR CHANNEL 'group_replication_recovery';

START GROUP_REPLICATION;

##verify
SELECT * FROM performance_schema.replication_group_members;
```

* single-primary 和 multi-primary 参数差异

```bash
SELECT group_replication_switch_to_multi_primary_mode()
group_replication_single_primary_mode=OFF;
group_replication_enforce_update_everywhere_checks=ON;

CREATE DATABASE test;
USE test;
CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL);
INSERT INTO t1 VALUES (1, 'Hello');

```




