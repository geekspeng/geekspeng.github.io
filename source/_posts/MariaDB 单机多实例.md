---
title: MariaDB 单机多实例
date: 2019-10-21
updated: 2019-10-23
tags: [MariaDB,数据库]
categories: [MariaDB]

---

# 安装mariadb

```bash
# yum install mariadb-server -y
```
# 创建对应的目录文件

```bash
# mkdir -p /mariadb/data{3306,3307,3308}
# chown -R mysql:mysql /mariadb
```
<!-- more -->

# 初始化数据库文件

```bash
# mysql_install_db  --datadir=/mariadb/data3306 --user=mysql
# mysql_install_db  --datadir=/mariadb/data3307 --user=mysql
# mysql_install_db  --datadir=/mariadb/data3308 --user=mysql
```
可能会报如下的错误

```bash
Neither host 'galera-57561c9a' nor 'localhost' could be looked up with
'/usr/libexec/resolveip'
Please configure the 'hostname' command to return a correct
hostname.
If you want to solve this at a later stage, restart this script
with the --force option
```
如果出现如上的错误，就按提示上加上 --force 选项
```bash
# mysql_install_db  --datadir=/mariadb/data3306 --user=mysql  --force
# mysql_install_db  --datadir=/mariadb/data3307 --user=mysql  --force
# mysql_install_db  --datadir=/mariadb/data3308 --user=mysql  --force
```
# 手动启动流程

## 创建对应配置文件

```ini
[mysqld]
port=3306
socket=/tmp/mysql3306.sock
pid-file=/tmp/mysql3306.pid
datadir=/mariadb/data3306[mysqld_safe]
log-error=/mysql/3306/log/mariadb.log
pid-file=/mysql/3306/pid/mariadb.pid
```
## mysqld_safe 方式启动

配置文件

```ini
# vi /etc/my.cnf.d/3306.cnf
[mysqld]
port=3306
socket=/tmp/mysql3306.sock
pid-file=/tmp/mysql3306.pid
datadir=/mariadb/data3306
log-error=/var/log/mariadb/3306.log
```
```ini
# vi /etc/my.cnf.d/3307.cnf
[mysqld]
port=3307
socket=/tmp/mysql3307.sock
pid-file=/tmp/mysql3307.pid
datadir=/mariadb/data3307
log-error=/var/log/mariadb/3307.log
```
```ini
# vi /etc/my.cnf.d/3308.cnf
[mysqld]
port=3308
socket=/tmp/mysql3308.sock
pid-file=/tmp/mysql3308.pid
datadir=/mariadb/data3308
log-error=/var/log/mariadb/3308.log
```
- mysqld_safe 启动

```bash
# mysqld_safe --defaults-file=/etc/my.cnf.d/3306.cnf
# mysqld_safe --defaults-file=/etc/my.cnf.d/3307.cnf
# mysqld_safe --defaults-file=/etc/my.cnf.d/3308.cnf
```
- 通过脚本启动


```shell
#!/bin/bash
#chkconfig: 345 80 2
port=3306
mysql_user="root"
mysql_pwd=""
cmd_path="/usr/bin"
defaults-file="/etc/my.cnf.d/3306.cnf"
mysql_sock="/tmp/mysql3306.sock"function_start_mysql()
{
    if [ ! -e "$mysql_sock" ];then
      printf "Starting MySQL...\n"
      ${cmd_path}/mysqld_safe --defaults-file=${defaults-file} &> /dev/null  &
    else
      printf "MySQL is running...\n"
      exit
    fi
}function_stop_mysql()
{
    if [ ! -e "$mysql_sock" ];then
       printf "MySQL is stopped...\n"
       exit
    else
       printf "Stoping MySQL...\n"
       ${cmd_path}/mysqladmin -u ${mysql_user} -p${mysql_pwd} -S ${mysql_sock} shutdown
   fi
}function_restart_mysql()
{
    printf "Restarting MySQL...\n"
    function_stop_mysql
    sleep 2
    function_start_mysql
}case $1 in
start)
    function_start_mysql
;;
stop)
    function_stop_mysql
;;
restart)
    function_restart_mysql
;;
*)
    printf "Usage: ./mysqld3306 {start|stop|restart}\n"
```
## mysqld_multi

### 配置文件

```ini
# cp -a /etc/my.cnf /etc/my.cnf.bak
# vi /etc/my.cnf # 添加如下代码,里面没有列出来的值都是保持默认的值
[mysqld_multi]
mysqld     = /usr/bin/mysqld_safe
user       = mysql

[mysqld3306]
port=3306
socket=/tmp/mysql3306.sock
pid-file=/tmp/mysql3306.pid
datadir=/mariadb/data3306
log-error=/var/log/mariadb/3306.log

[mysqld3307]
port=3307
socket=/tmp/mysql3307.sock
pid-file=/tmp/mysql3307.pid
datadir=/mariadb/data3307
log-error=/var/log/mariadb/3307.log

[mysqld3308]
port=3308
socket=/tmp/mysql3308.sock
pid-file=/tmp/mysql3308.pid
datadir=/mariadb/data3308
log-error=/var/log/mariadb/3308.log
```
### 启动实例

注：[mysqld3306]，[mysqld3307]，[mysqld3308] 分别对应3306，3307,3308

```bash
# mysqld_multi --defaults-extra-file=/etc/my.cnf start 3306
# mysqld_multi --defaults-extra-file=/etc/my.cnf start 3307
# mysqld_multi --defaults-extra-file=/etc/my.cnf start 3308
```
### 查看启动的实例

```bash
# mysqld_multi --defaults-extra-file=/etc/my.cnf report 
```
# 客户端登录

## 通过TCP/IP连接

```bash
# mysql -P3306 -hlocalhost --protocol=tcp
```
## 通过连接实例的方式（只能本地连接，不能用于远程连接）

```bash
# mysql -S /tmp/mysql3307.sock
```
# 停止实例

```bash
# mysqladmin -u root -p -S /tmp/mysql3306.sock shutdown
# mysqladmin -u root -p -S /tmp/mysql3307.sock shutdown
# mysqladmin -u root -p -S /tmp/mysql3308.sock shutdown
```
# 修改密码

```bash
# mysqladmin --no-defaults --port=3306 --user=root --protocol=tcp password '123456'
# mysqladmin --no-defaults --port=3307--user=root --protocol=tcp password '123456'
# mysqladmin --no-defaults --port=3308 --user=root --protocol=tcp password '123456'
```
另一种方式

```bash
# systemctl restart mariadb --skip-grant-tables --skip-networking
# mysql -e"UPDATE mysql.user SET password=password('somenewpassword') WHERE user='root'"
# systemctl restart mariadb
```
