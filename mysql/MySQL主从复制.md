# MySQL主从复制

MySQL主从复制是实现读写分离的前提，读写分离技术可以将系统读写的压力分开，一主多从的架构可以使得读的能力大大增加。

## 环境

**OS**: MacOS Big Sur 11.2.2

**brew**: 

```
HOMEBREW_VERSION: 3.0.3
ORIGIN: https://github.com/Homebrew/brew
HEAD: 9d2ee344f6a0f5e032618802e624777fc7812ae3
Last commit: 29 hours ago
Core tap ORIGIN: https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
Core tap HEAD: cde0aa962a918e54c2d769fa516d299c4b6444fa
Core tap last commit: 30 hours ago
Core tap branch: master
HOMEBREW_PREFIX: /usr/local
HOMEBREW_BOTTLE_DOMAIN: https://mirrors.ustc.edu.cn/homebrew-bottles
HOMEBREW_CASK_OPTS: []
HOMEBREW_MAKE_JOBS: 4
Homebrew Ruby: 2.6.3 => /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/bin/ruby
CPU: quad-core 64-bit broadwell
Clang: 12.0 build 1200
Git: 2.30.1 => /usr/local/bin/git
Curl: 7.64.1 => /usr/bin/curl
macOS: 11.2.2-x86_64
CLT: 12.4.0.0.1.1610135815
Xcode: 12.2
```



**MySQL**: 5.7



## 安装

```shell
brew install mysql
```



## 主从环境准备

```bash
mkdir -p ~/mysql/server1/data
mkdir -p ~/mysql/server2/data
```



#### 准备两个配置文件

server1-> `~/mysql/server1/my.cnf`

```properties
[mysqld]
bind-address = 127.0.0.1
port = 3316
server_id = 1
datadir = /Users/{YOUR_USERNAME}/mysql/server1/data
socket = /tmp/mysql3316.sock

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
log_bin=mysql-bin
binlog-format=Row
```



server2 ->`~/mysql/server2/my.cnf`

```properties
[mysqld]
bind-address = 127.0.0.1
port = 3326
server_id = 2
datadir = /Users/{YOUR_USERNAME}/mysql/server2/data
socket = /tmp/mysql3326.sock

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
log_bin=mysql-bin
binlog-format=Row
```



#### 初始化

```bash
mysqld --defaults-file=~/mysql/server1/my.cnf --initialize-insecure
mysqld --defaults-file=~/mysql/server2/my.cnf --initialize-insecure
```



#### 启动

```bash
mysqld --defaults-file=~/mysql/server1/my.cnf
mysqld --defaults-file=~/mysql/server2/my.cnf
```



## 开始主从同步过程

#### 1. 连接主库

```bash
mysql -uroot -P3316 -h127.0.0.1
```

#### 2. 创建用户

```mysql
mysql> CREATE USER 'repl'@'%' IDENTIFIED BY '123456';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
mysql> flush privileges;
mysql> show master status;
+------------------+------------+----------------+--------------------+---------------------+
| File             | Position   | Binlog_Do_DB   | Binlog_Ignore_DB   | Executed_Gtid_Set   |
|------------------+------------+----------------+--------------------+---------------------|
| mysql-bin.000002 | 747      |                |                    |                     |
+------------------+------------+----------------+--------------------+---------------------+
```



#### 3. 登录从库操作

```bash
mysql -uroot -P3326 -h127.0.0.1

# 填上主库的信息
mysql> CHANGE MASTER TO
    MASTER_HOST='localhost',  
    MASTER_PORT = 3316,
    MASTER_USER='repl',      
    MASTER_PASSWORD='123456',   
    MASTER_LOG_FILE='mysql-bin.000002',
    MASTER_LOG_POS=747;
# 开启主从
mysql> start slave;

myql> show slave status\G;
Slave_IO_State                | Waiting for master to send event
Master_Host                   | localhost
Master_User                   | repl
Master_Port                   | 3316
Connect_Retry                 | 60
Master_Log_File               | mysql-bin.000002
Read_Master_Log_Pos           | 1768
Relay_Log_File                | localhost-relay-bin.000002
Relay_Log_Pos                 | 1341
Relay_Master_Log_File         | mysql-bin.000002
Slave_IO_Running              | Yes
Slave_SQL_Running             | Yes
```



#### 4. 在主库创建数据库和表

```bash
mysql> create database db;
mysql> use db;
mysql> create table t1 (id int primary key);
mysql> insert into t1 values(1),(2),(3),(4);
mysql> select * from t1;
```



#### 5. 去从库验证

```mysql
mysql> show databases;
mysql> use db;
mysql> select * from t1;
+------+
| id   |
|------|
| 1    |
| 2    |
| 3    |
| 4    |
+------+
4 rows in set
Time: 0.008s
```



## 总结

至此MySQL的主从同步搭建完成 (异步方式)