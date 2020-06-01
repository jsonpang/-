```
# my global config
global:
  scrape_interval:     15s # 抓取的时间间隔Set，用来指定Prometheus评估规则的频率 the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # 用来指定Prometheus评估规则的频率Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration 
# 设置告警
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['192.168.7.70:9090','192.168.8.15:9274','192.168.8.15:9275','192.168.8.15:9276','47.93.6.224:29273','47.93.6.224:29274','47.93.6.224:29275']
    metric_relabel_configs:
    - source_labels: [jolokia_agent_url]
      regex: (.*)://(.*):(.*)/(.*)/(.*) 
      target_label: mq_console
      replacement: $2:$3
      action: replace
    - source_labels: [instance]
      regex: (.*):.*
      target_label: host
      replacement: $1
      action: replace
  - job_name: 'java'
    file_sd_configs:
    - files: ['/app/prometheus/sd_config/tomcat.yml']
      refresh_interval: 30s
    metric_relabel_configs:
    - source_labels: [instance]
      regex: 192.168.8.15:39081
      target_label: servername
      replacement: bps-9081
      action: replace
    - source_labels: [instance]
      regex: 192.168.8.15:39082
      target_label: servername
      replacement: tomcat-bsd
      action: replace
    - source_labels: [instance]
      regex: 192.168.8.15:39083
      target_label: servername
      replacement: jncb-yewu
      action: replace
    - source_labels: [instance]
      regex: 192.168.8.15:39084
      target_label: servername
      replacement: bkrcb-yewu
      action: replace
    - source_labels: [instance]
      regex: (.*):.*
      target_label: host
      replacement: $1
      action: replace
```
参考链接地址
https://prometheus.io/docs/prometheus/latest/configuration/configuration/

## 支持服务发现target地址