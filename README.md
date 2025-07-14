# task20
я создал новый инстанс и изменил файл prometheus.yml: <br>
```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets:
          - 'localhost-ne:9100'
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        replacement: 'Local machine'

  - job_name: 'aws_node_exporter'
    static_configs:
      - targets:
          - '3.83.83.118:9100'
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        replacement: 'AWS instance'

alerting:
  alertmanagers:
    - scheme: http
      static_configs:
        - targets:
          - 'alertmanager:9093'

rule_files:
  - '/etc/prometheus/alerts.yml'
```

<img width="1912" height="386" alt="image" src="https://github.com/user-attachments/assets/3ed81442-3c79-4780-8c67-96810d10f61c" />


Дальше я добавил новое правило по RAM: <br>

```
groups:
  - name: CPU-RAM-alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 * (1 - avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[1m])) > 0.1)
        for: 1m
        labels:
          severity: critical
          instance: 'Local machine'
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}% (over 10% for 1 minute)"

      - alert: HighRamUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes > 0.50
        for: 1m
        labels:
          severity: critical
          instance: 'AWS instance'
        annotations:
          summary: "High RAM usage on {{ $labels.instance }}"
          description: "RAM usage is {{ $value }}% (over 10% for 1 minute)"

```
<img width="1908" height="370" alt="image" src="https://github.com/user-attachments/assets/c3ab62c6-b74e-4a0b-a492-1be35f184892" />

Итоговое уведомление в SLACK:<br>
<img width="796" height="273" alt="image" src="https://github.com/user-attachments/assets/9ae267a5-f39f-4698-b5a7-7d3788ce21df" />


