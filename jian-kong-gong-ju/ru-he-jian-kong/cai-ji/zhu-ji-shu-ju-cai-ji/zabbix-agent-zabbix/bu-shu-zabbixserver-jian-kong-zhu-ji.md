# 部署Zabbix-server监控主机

## 版本4.0
## 设置yum源
```
1
rpm -ivh  http://mirrors.aliyun.com/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm 
2.
-rw-r--r-- 1 root root  410 Oct  2  2018 zabbix.repo
[root@zabbix-server yum.repos.d]# vim zabbix.repo 
[zabbix]
name=Zabbix Official Repository - $basearch
baseurl=http://mirrors.aliyun.com/zabbix/zabbix/4.0/rhel/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-non-supported]
name=Zabbix Official Repository non-supported - $basearch
baseurl=http://mirrors.aliyun.com/zabbix/non-supported/rhel/7/$basearch/
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
gpgcheck=1
```
# MySQL部署
数据库版本5.7

```
mkdir -p /data/app
tar xf /root/mysql-5.7.20-linux-glibc2.12-x86_64.tar.gz -C /data/app
mv mysql-5.7.20-linux-glibc2.12-x86_64 mysql

#mysql
vim ~/.bash_profile  
MYSQL_HOME=/data/app/mysql
export PATH=$MYSQL_HOME/bin:$PATH
source ~/.bash_profile 
[root@dev01 app]# mysql --version
mysql  Ver 14.14 Distrib 5.7.20, for linux-glibc2.12 (x86_64) using  EditLine wrapper
```
## 建立Mysql用户组和用户
useradd mysql
## 创建相关的目录权限
 chown -R mysql.mysql /data/app/*

## 初始化数据库
```
[root@dev01 app]# mysqld --initialize-insecure  --user=mysql --basedir=/data/app/mysql --datadir=/data/app/data/mysql
2020-05-27T05:17:05.863952Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2020-05-27T05:17:06.064502Z 0 [Warning] InnoDB: New log files created, LSN=45790
2020-05-27T05:17:06.103728Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2020-05-27T05:17:06.166793Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 4e7ba80f-9fd9-11ea-a8f7-fa163edd63e4.
2020-05-27T05:17:06.171653Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2020-05-27T05:17:06.172246Z 1 [Warning] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
```
检查是否有数据
```
[root@dev01 app]# ll /data/app/data/mysql/
total 110628
-rw-r----- 1 mysql mysql       56 May 27 13:17 auto.cnf
-rw-r----- 1 mysql mysql      419 May 27 13:17 ib_buffer_pool
-rw-r----- 1 mysql mysql 12582912 May 27 13:17 ibdata1
-rw-r----- 1 mysql mysql 50331648 May 27 13:17 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 May 27 13:17 ib_logfile1
drwxr-x--- 2 mysql mysql     4096 May 27 13:17 mysql
drwxr-x--- 2 mysql mysql     8192 May 27 13:17 performance_schema
drwxr-x--- 2 mysql mysql     8192 May 27 13:17 sys
```
## 修改默认配置文件

 vim /etc/my.cnf
```
[mysqld]
basedir=/data/app/mysql
datadir=/data/app/data/mysql
socket=/data/app/data/mysql/mysql.sock
port=3306
log-error=/data/app/data/mysql/mysqlerror.log
log_bin=/data/app/data/mysql/mysql-bin
binlog_format=row
lower_case_table_names=1
log-bin-trust-function-creators=1
skip-name-resolve
server-id=7
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
sql_mode=NO_ENGINE_SUBSTITUTION
[mysql]
prompt=3306 [\\d]>
socket=/data/app/data/mysql/mysql.sock
[client]
socket=/data/app/data/mysql/mysql.sock
```
## 使用systemd管理mysql
vim /usr/lib/systemd/system/mysqld.service
```
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/data/app/mysql/bin/mysqld --defaults-file=/etc/my.cnf
LimitNOFILE = 5000
```
## 启动数据库
    第一次启动
    systemctl daemon reload
    systemctl start mysqld
    （其他）直接启动
    systemctl start mysqld
## 检查状态
ps -ef|grep mysqld


ss -tnl|grep 3306
## 初始化密码
```
[root@dev01 app]# mysqladmin -uroot -p password xxxxx (直接回车)
Enter password: （回车，不要输入）
mysqladmin: [Warning] Using a password on the command line interface can be insecure.
Warning: Since password will be sent to server in plain text, use ssl connection to ensure password safety.
```
## 测试登录
```
[root@dev01 app]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.20-log MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

3306 [(none)]>
```
## 导入数据
```
创建数据库
sh mysql_login.sh
创建数据库
create database zabbix  character set utf8 collate utf8_bin;
授权
grant all privileges on zabbix.* to 'zabbix'@'%' identified by 'zabbix';
flush privileges;

导入
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -pzabbix zabbix


```
- 打开前端网址访问


http://192.168.7.70/zabbix/


按照提示步骤进行初始化


初始账号密码
Admin/zabbix

# 总结
zabbix 比较适用于主机监控


Prometheus 适用于应用监控
他们都可以发送数据到grafana中
通过不同的dashboard切换就可以查看不同的监控页面
# Zabbixagent部署
## 安装
```
yum install pcre* -y（可选）
tar xf  zabbix-3.4.11.tar.gz
 ./configure --enable-agent --prefix=/usr/local/zabbix
  make&&make install
配置文件zabbix_agent.conf：server：白名单ip
hostname：填写本机IP地址
agent工作模式
监听10050 端口启动后

zabbixserver 主动去连接xxxx:10050服务
 获取信息
```
## zabbix_get

命令是在server端用来检查agent端的一个命令，在添加完主机或者触发器后，不能正常获得数据，可以用zabbix_get来检查能否采集到数据，以便判断问题症结所在。
```
zabbix_get 参数说明：
-s --host： 指定客户端主机名或者IP
-p --port：客户端端口，默认10050
-I --source-address：指定源IP，写上zabbix server的ip地址即可，一般留空，服务器如果有多ip的时候，你指定一个。
-k --key：你想获取的key
```
如果不知道key参数可以使用 zabbix_agentd -p 寻找自己想要找的参数
```
[root@host~]# zabbix_agentd -p | grep system.cpu.load
system.cpu.load[all,avg1]                     [d|0.040000]
```
如果不知道zabbix_get在什么路径，可以使用find / -name zabbix_get查找
```
[root@host ~]# find / -name zabbix_get
/usr/local/zabbix/bin/zabbix_get
/data/tools/zabbix-4.0.3/src/zabbix_get
/data/tools/zabbix-4.0.3/src/zabbix_get/zabbix_get
```
```
案例：
[root@zabbix ~]# zabbix_get -s 192.168.1.7 -p 10050 -k system.cpu.load[all,avg1]   
0.000000
```
>>zabbix_get  判断客户端是否联通


>>agent.conf的server 类似防火墙的白名单的意思
服务的最后一跳地址 