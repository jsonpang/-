# 部署Zabbix-server监控主机

## 部署安装
下载一个zabbix-3.4.11.tar.gz


解压


tar xf  zabbix-3.4.11.tar.gz

安装

```
./configure --enable-agent --prefix=/usr/local/zabbix
  make&&make install
  useradd zabbix
```
启动

```
[root@jz-demo01 sbin]# ./zabbix_agentd -c ../etc/zabbix_agentd.conf
[root@jz-demo01 sbin]# ps -ef|grep zabbix
zabbix   27656     1  0 14:11 ?        00:00:00 ./zabbix_agentd -c ../etc/zabbix_agentd.conf
zabbix   27657 27656  0 14:11 ?        00:00:00 ./zabbix_agentd: collector [idle 1 sec]
zabbix   27658 27656  0 14:11 ?        00:00:00 ./zabbix_agentd: listener #1 [waiting for connection]
zabbix   27659 27656  0 14:11 ?        00:00:00 ./zabbix_agentd: listener #2 [waiting for connection]
zabbix   27660 27656  0 14:11 ?        00:00:00 ./zabbix_agentd: listener #3 [waiting for connection]
zabbix   27661 27656  0 14:11 ?        00:00:00 ./zabbix_agentd: active checks #1 [idle 1 sec]
root     28150  4490  0 14:11 pts/14   00:00:00 grep --color=auto zabbix
[root@jz-demo01 sbin]# ss -tnl|grep 10050
LISTEN     0      128          *:10050                    *:*  
```
工作模式
监听10050 端口启动后

zabbixserver 主动去连接xxxx:10050服务
 获取信息

### zabbix_get命令

       server端用来检查agent端的一个命令，在添加完主机或者触发器后，不能正常获得数据，可以用zabbix_get来检查能否采集到数据，以便判断问题症结所在。
```
zabbix_get 参数说明：
-s --host： 指定客户端主机名或者IP
-p --port：客户端端口，默认10050
-I --source-address：指定源IP，写上zabbix server的ip地址即可，一般留空，服务器如果有多ip的时候，你指定一个。
-k --key：你想获取的key
至于使用长参数还是短的，自己选，我经常使用-s而不是-host，
如果不知道key参数可以使用 zabbix_agentd -p 寻找自己想要找的参数
```
```
[root@host~]# zabbix_agentd -p | grep system.cpu.load
system.cpu.load[all,avg1]                     [d|0.040000]
如果不知道zabbix_get在什么路径，可以使用find / -name zabbix_get查找
[root@host ~]# find / -name zabbix_get
/usr/local/zabbix/bin/zabbix_get
/data/tools/zabbix-4.0.3/src/zabbix_get
/data/tools/zabbix-4.0.3/src/zabbix_get/zabbix_get
案例：
[root@zabbix ~]# zabbix_get -s 192.168.1.7 -p 10050 -k system.cpu.load[all,avg1]   
0.000000
```
zabbix_get  判断客户端是否联通
**zabbix_agentd.conf的server 类似防火墙的白名单的意思表示服务的最后一跳地址**

