# Prometheus的内置函数

增加一个标签-替代的方式
label_replace(up, "host", "$1", "instance",  "(.*):.*")
新的结果
```
up{host="192.168.7.70",instance="192.168.7.70:9090",job="prometheus"}	
https://prometheus.fuckcloudnative.io/di-san-zhang-prometheus/di-4-jie-cha-xun/functions#label_replace

对于v中的每个timeseries，label_replace（v instant vector、dst_label string、replacement string、src_label string、regex string）将正则表达式regex与label src_label匹配。如果匹配，则返回timeseries，标签dst_label替换为replacement的扩展。$1替换为第一个匹配的子组，$2替换为第二个等。如果正则表达式不匹配，则返回的timeseries将保持不变。
url中提取域名和端口
label_replace(activemq_broker_MemoryLimit,"mq","$2:$3","jolokia_agent_url","(.*)://(.*):(.*)/(.*)/(.*)")
```

```
完全匹配也成为锚点
(25[0-5]|2[0-4]\d|[0-1]\d{2}|[1-9]?\d)\.(25[0-5]|2[0-4]\d|[0-1]\d{2}|[1-9]?\d)\.(25[0-5]|2[0-4]\d|[0-1]\d{2}|[1-9]?\d)\.(25[0-5]|2[0-4]\d|[0-1]\d{2}|[1-9]?\d)\:(\d+)
```
[go 在线工具](https://c.runoob.com/compile/21)