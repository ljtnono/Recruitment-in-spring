[TOC]

## MySQL最小安装

1、在官网下载最小安装包，并解压

```shell
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.25-linux-glibc2.17-x86_64-minimal.tar.xz

tar xvf mysql-8.0.25-linux-glibc2.17-x86_64-minimal.tar.xz
mv mysql-8.0.25-linux-glibc2.17-x86_64-minimal.tar.xz mysql8
```

2、初始化mysql

```bash
groupadd mysql
useradd -r -g mysql -s /bin/false mysql
# 进入mysql安装路径,将常用的my.cnf配置文件拷贝到mysql安装目录中
mkdir logs
mkdir data
touch logs/slow_sql.log
touch logs/mysql.log
chown -R mysql:mysql .
# --datadir为mysql安装目录下的data文件夹
bin/mysqld --initialize --user=mysql --datadir=<basedir>/data
bin/mysql_ssl_rsa_setup
# 修改启动脚本中的basedir为mysql安装目录，datadir为mysql安装目录下的data目录
# vim support-files/mysql.server
# basedir=<basedir>
# datadir=<datadir>
# 启动mysql
support-files/mysql.server start --socket=<basedir>/mysql.sock
```

3、修改root用户密码，设置远程访问

```bash
# 其中<password>是初始化时的root密码，可以在logs/mysql.log文件中查看
bin/mysql -uroot -p'<password>' --socket=<basedir>/mysql.sock
```

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY '<你的密码>'
USE mysql;
UPDATE user SET Host = '%' WHERE User = 'root';
FLUSH PRIVILEGES;
```



## 常用my.cnf配置文件

```mysql
# 将以下<baseDir>改为自己的mysql安装路径
[mysqld]
socket = <baseDir>/mysql.sock
basedir = <baseDir>
datadir = <baseDir>/data
# 去除groupBy 需要选择字段
sql_mode = STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
# 慢sql日志
slow_query_log_file = <baseDir>/logs/slow_sql.log
slow_query_log = on
long_query_time = 3
secure_file_priv = <baseDir>
# pid 设置
pid-file = <baseDir>/mysql.pid
[mysqld_safe]
log-error=<baseDir>/logs/mysql.log
```





