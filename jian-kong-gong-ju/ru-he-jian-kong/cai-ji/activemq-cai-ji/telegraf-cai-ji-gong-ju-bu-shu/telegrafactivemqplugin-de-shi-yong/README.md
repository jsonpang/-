# Telegraf-activemq-plugin的使用


# ActiveMQ 监控

- 详细的mq队列信息需要通过telegraf activemq插件监控
- 常规的activemq 等信息通过激活mq 的自带jolokia服务监控

## 1.创建一个配置文件
telegraf -sample-config -input-filter activemq -output-filter prometheus_client >activemq-telegraf.conf
## 2.编辑参数
## 3.启动telegraf
参考jokolia相关的描述
## 4.通过Prometheus获取到数据
## 5.grafana展示
## 问题
- 5.14.5采集xml/queue.jsp报错不能解析，是个bug
  - 5.14.6或者5.15.0之后的版本修复此bug