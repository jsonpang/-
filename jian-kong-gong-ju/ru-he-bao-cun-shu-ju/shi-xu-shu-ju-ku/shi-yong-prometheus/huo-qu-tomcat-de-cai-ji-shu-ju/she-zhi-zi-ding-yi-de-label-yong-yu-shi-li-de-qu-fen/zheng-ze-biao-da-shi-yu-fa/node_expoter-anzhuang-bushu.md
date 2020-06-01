# 下载
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.0.0/node_exporter-1.0.0.linux-amd64.tar.gz
tar xf node_exporter-*linux-amd64.tar.gz
cp node_exporter-*/node_exporter /usr/local/bin/ 
```

## 测试
```
[root@bsd-dev ~]# node_exporter --version
node_exporter, version 1.0.0 (branch: HEAD, revision: b9c96706a7425383902b6143d097cf6d7cfd1960)
  build user:       root@3e55cc20ccc0
  build date:       20200526-06:01:48
  go version:       go1.14.3
```
## 启动

```
前端启动
node_exporter --web.listen-address=":9100" --web.telemetry-path="/metrics"
daemon启动
node_exporter --web.listen-address=":9100" --web.telemetry-path="/metrics" &
```
### 查看采集结果
```
curl -s http://192.168.8.15:9100/metrics|less
# TYPE node_netstat_Tcp_OutSegs untyped
node_netstat_Tcp_OutSegs 1.66600738e+08
# HELP node_netstat_Tcp_PassiveOpens Statistic TcpPassiveOpens.
# TYPE node_netstat_Tcp_PassiveOpens untyped
node_netstat_Tcp_PassiveOpens 1.208879e+06
# HELP node_netstat_Tcp_RetransSegs Statistic TcpRetransSegs.
# TYPE node_netstat_Tcp_RetransSegs untyped
node_netstat_Tcp_RetransSegs 64070
# HELP node_netstat_Udp6_InDatagrams Statistic Udp6InDatagrams.
# TYPE node_netstat_Udp6_InDatagrams untyped
node_netstat_Udp6_InDatagrams 0
```
## 自定义指标
textfile收集器采集逻辑

收集器通过扫描指定目录中的文件，提取所有格式为Prometheus指标的字符串，然后暴露它们以
便抓取

**提示**
在真实环境中，建议使用配置管理工具来编辑该文件。例如，在配置新主机时，可以从模板
创建元数据指标，这可以让你自动对主机和服务进行分类

#### 启用textfile
默认会被Node-exporter加载，需要指定textfile_exporter目录，以便Node Exporter知道在哪里可以找到自定义指标。为此，我们需要指定--
collector.textfile.directory参数。


