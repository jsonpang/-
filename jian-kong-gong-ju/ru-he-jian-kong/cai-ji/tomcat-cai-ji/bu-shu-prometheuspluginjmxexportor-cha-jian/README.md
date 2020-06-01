# 部署Prometheus-plugin-jmx\_exportor插件
# 1.下载
项目地址：https://github.com/prometheus/jmx_exporter

# 2.收集tomcat数据
## Jar包应用
安装github上的方式启动就好了
```
java -javaagent:./jmx_prometheus_javaagent-0.9.jar=9151:config.yaml -jar yourJar.jar
```
## Tomcat war包应用
进入bin目录$TOMCAT_HOME/bin，将jmx_exporter.jar包文件和下载得到的tomcat.yml 改名为config.yaml文件复制到这里。然后修改里面的一个catalina.sh的脚本，找到JAVA_OPTS,加上以下配置(代理)：
标准config文件--这个指标文件没有配套的grafana的dashboard模板使用
wget https://github.com/prometheus/jmx_exporter/blob/master/example_configs/tomcat.yml
config文件
```
[root@bsd-dev bin]# cat simple-config.yml 
---
lowercaseOutputLabelNames: true
lowercaseOutputName: true
whitelistObjectNames: ["java.lang:type=OperatingSystem"]
rules:
 - pattern: 'java.lang<type=OperatingSystem><>((?!process_cpu_time)\w+):'
   name: os_$1
   type: GAUGE
   attrNameSnakeCase: true
```
启动参数
```
JAVA_OPTS="-javaagent:./jmx_prometheus_javaagent-0.9.jar=39081:./simple-config.yml"
```
如果有多tomcat,建议将jmx_prometheus_javaagent和config.yaml文件放到固定的目录,$TOMCAT_HOME/bin/catalina.sh文件中写绝对路径.
#修改bin/catalina.sh 文件
添加：
```
JAVA_OPTS="-javaagent:bin/jmx_prometheus_javaagent-0.9.jar=39081:bin/config.yaml"
```
绝对路径
```
JAVA_OPTS="-javaagent:／root/jmx_prometheus_javaagent-0.9.jar=39081:／root/config.yaml"
```


访问服务器是否采集到数据
部署后
检查是否成功
Metrics will now be accessible at http://localhost:port/metrics
 curl -s http://192.168.8.15:39081/metrics|less
 ![图 2](https://i.loli.net/2020/05/24/ArCTfwhj15FyNmn.png)  



3.配置prometheus版本	2.18.1
注意 ：采集的数据相差8个小时这个问题，如果最后展示使用grafana的话不需要处理，在grafana展示是正常的

修改prometheus配置文件增加job 设置自动发现
```
  - job_name: 'tomcat'
    file_sd_configs:
    - files: ['/app/prometheus/sd_config/tomcat.yml']
      refresh_interval: 180s
```
配置文件
```
- targets:
  - 192.168.8.15:39081
  labels:
    idc: bj_company
    service: tomcat
```

重载配置文件
```
kill -hup `ps -ef |grep prometheus|grep -v grep|awk '{print $2}'`
```

到Prometheus控制台查询

[![tS4zzn.png](https://s1.ax1x.com/2020/05/24/tS4zzn.png)](https://imgchr.com/i/tS4zzn)
grafana-6.7
1.先添加数据源
[![tS50w8.png](https://s1.ax1x.com/2020/05/24/tS50w8.png)](https://imgchr.com/i/tS50w8)
![tS52yq.png](https://s1.ax1x.com/2020/05/24/tS52yq.png)
![tS5gln.png](https://s1.ax1x.com/2020/05/24/tS5gln.png)



2.import dashboard

  使用这个dashboard
https://grafana.com/grafana/dashboards/8563
其他的根据版本原因不能用
启用Prometheus自带的dashboard



 
问题排查
prometheus 没有数据 

1.首先判断Prometheus 有没有数据

查看能不能正常获取指标
1.获取指标名称

任意获取一个指标名称

比如
os_process_cpu_load
点击

回到首页
查询一下指标情况

可见是能获取到数据，如果没有数据的话，就可以判断是获取数据的时候出问题了，这个就分两种情况排查，如果在本机执行
curl http://192.168.8.15:39081 能正常获取数据
那就是Prometheus获取数据出了问题，往这个方向排查
02 Grafana配置datasource dashborad 没有数据
分几个情况讨论
1.先判断单个指标能不能收到数据





输入os_process_cpu_load，可以得到下图结果

如果没有数据的话

返回正确结果的过程

返回错误结果的话可以吧grafana拼接的url放到浏览器看看
http://192.168.7.70:9090/api/v1/query_range?query=os_process_cpu_load&start=1590030720&end=1590031620&step=15
我遇到一个坑就是 添加Prometheus数据源的时候http后面的冒号用了中文字符，一直没有拿到Prometheus的数据，导致浪费了很多时间，不过结果是摸索到这个方式可以完成简单的调试
使用datasource 注意事项
注意版本问题


测试1 把prometheus的config 换成指向文件的方式启动
注意jobname 要和grafana中导入模板设置的一样

现在要尝试把jmx-exporter tomcat-config。yml文件换成其他的，看看结果的变化情况

通过设置label

区分不同的应用