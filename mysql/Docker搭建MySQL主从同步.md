# Docker 搭建MySQL主从同步

## 手动改容器内部的方式

### 环境准备

- 安装docker并配置docker
- 下载 mysql 5.7 镜像



### 准备容器

#### 1、启动 master 和 slave

```shell
# master 使用3316
docker run -d -p 3316:3306 --name mysql-master -e MYSQL_ROOT_PASSWORD=123456  mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

# slave 使用 3326

docker run -d -p 3326:3306 --name mysql-slave -e MYSQL_ROOT_PASSWORD=123456  mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

```

#### 2、进入容器修改配置

##### maser配置

```bash
docker exec -it mysql-master /bin/bash
# 进入新的命令行窗口
cd /etc/mysql/myql.conf.d/
# 写入配置，由于没有 vim ，安装vim还需要更新 apt-get比较麻烦 ，所以使用 echo
echo 'server_id = 1' >> mysqld.cnf
echo 'sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES' >> mysqld.cnf
echo 'log_bin=mysql-bin' >> mysqld.cnf
echo 'binlog-format=Row' >> mysqld.cnf
```

##### slave配置

```bash
docker exec -it mysql-slave /bin/bash
# 进入新的命令行窗口
cd /etc/mysql/myql.conf.d/
# 写入配置，由于没有 vim ，安装vim还需要更新 apt-get比较麻烦 ，所以使用 echo
echo 'server_id = 2' >> mysqld.cnf
echo 'sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES' >> mysqld.cnf
echo 'log_bin=mysql-bin' >> mysqld.cnf
echo 'binlog-format=Row' >> mysqld.cnf
```



#### 3、重启容器

```bash
docker restart mysql-master
docker restart mysql-slave
```



### 开始配置主从

https://xiaozefeng.github.io/#/mysql/MySQL%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6



## Dockerfile的方式

### 准备主从配置文件

##### 主配置

`vim ./primary/mysqld.cnf`

```properties
# Copyright (c) 2014, 2016, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
datadir		= /var/lib/mysql
#log-error	= /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address	= 127.0.0.1

server_id = 1
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
log_bin=mysql-bin
binlog-format=Row

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
```



##### 从配置 

`vim ./second/mysqld.cnf`

```properties

# Copyright (c) 2014, 2016, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
datadir		= /var/lib/mysql
#log-error	= /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address	= 127.0.0.1

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

server_id = 2
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
log_bin=mysql-bin
binlog-format=Row

```



### 准备Dockerfile

主和从都是一样的，只是放在不同的目录

```dockerfile
FROM mysql:5.7

COPY mysqld.cnf /etc/mysql/mysql.conf.d/mysqld.cnf
```



### 创建镜像

##### 创建primary镜像

```bash
cd ./primary
ls
# Dockerfile mysqld.cnf
docker build -t mysqlprimary:v1
```

##### 创建second镜像

```bash
cd ./second
ls
# Dockerfile mysqld.cnf
docker build -t mysqlsecond:v1
```



### 启动容器

```bash
docker run -d -p 3316:3306 --name mysql-primary -e MYSQL_ROOT_PASSWORD=123456  mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

docker run -d -p 3326:3306 --name mysql-second -e MYSQL_ROOT_PASSWORD=123456  mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```



!> 需要特别注意的是在开启主从同步时 , `ip` 和 `端口`一定要注意, 容器内部的端口是 3306

```mysql
CHANGE MASTER TO
    MASTER_HOST='localhost',  
    MASTER_PORT = 3306,
    MASTER_USER='repl',      
    MASTER_PASSWORD='123456',   
    MASTER_LOG_FILE='mysql-bin.000002',
    MASTER_LOG_POS=747;
```

!> ip的获取可以通过 `docker inspect --format '{{ .NetworkSettings.IPAddress }}'  容器id`
