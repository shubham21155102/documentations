# 📊 Monitoring

> Frequently used commands and configurations for Prometheus, Grafana, and alerting.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [Prometheus](#prometheus)
- [Node Exporter](#node-exporter)
- [Alertmanager](#alertmanager)
- [Grafana](#grafana)
- [Loki & Promtail](#loki--promtail)
- [PromQL Queries](#promql-queries)
- [Kubernetes Monitoring (kube-prometheus-stack)](#kubernetes-monitoring-kube-prometheus-stack)

---

## Prometheus

### Installation

```bash
# Docker
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v ./prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus

# Docker Compose
```

```yaml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=15d'
      - '--web.enable-lifecycle'

volumes:
  prometheus_data:
```

### `prometheus.yml`

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    monitor: 'production'

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - "alerts/*.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'myapp'
    metrics_path: /metrics
    static_configs:
      - targets: ['myapp:8080']
    labels:
      env: production

  # Kubernetes service discovery
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

### Prometheus API

```bash
# Query via HTTP API
curl 'http://localhost:9090/api/v1/query?query=up'
curl 'http://localhost:9090/api/v1/query_range?query=up&start=2024-01-01T00:00:00Z&end=2024-01-02T00:00:00Z&step=15s'

# Reload config
curl -X POST http://localhost:9090/-/reload

# Check targets
curl http://localhost:9090/api/v1/targets | jq

# Check rules
curl http://localhost:9090/api/v1/rules | jq

# Check alerts
curl http://localhost:9090/api/v1/alerts | jq
```

---

## Node Exporter

```bash
# Docker
docker run -d \
  --name node-exporter \
  --net=host \
  --pid=host \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /:/rootfs:ro \
  prom/node-exporter \
  --path.procfs=/host/proc \
  --path.sysfs=/host/sys \
  --collector.filesystem.ignored-mount-points='^/(sys|proc|dev|host|etc)($$|/)'

# Verify metrics
curl http://localhost:9100/metrics
```

**Useful Node Exporter metrics:**

| Metric | Description |
|--------|-------------|
| `node_cpu_seconds_total` | CPU time by mode |
| `node_memory_MemAvailable_bytes` | Available memory |
| `node_disk_io_time_seconds_total` | Disk I/O time |
| `node_network_receive_bytes_total` | Network receive bytes |
| `node_filesystem_avail_bytes` | Available disk space |
| `node_load1` | 1-minute load average |

---

## Alertmanager

### `alertmanager.yml`

```yaml
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'alerts@example.com'
  smtp_auth_username: 'alerts@example.com'
  smtp_auth_password: 'app-password'

route:
  receiver: 'default'
  group_by: ['alertname', 'job']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h

  routes:
    - match:
        severity: critical
      receiver: pagerduty
    - match:
        severity: warning
      receiver: slack

receivers:
  - name: 'default'
    email_configs:
      - to: 'team@example.com'
        require_tls: true

  - name: 'slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'
        channel: '#alerts'
        send_resolved: true
        title: '{{ .Status | toUpper }} {{ .CommonLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

  - name: 'pagerduty'
    pagerduty_configs:
      - routing_key: 'your-pagerduty-integration-key'

inhibit_rules:
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal: ['alertname', 'instance']
```

### Alert Rules

```yaml
# alerts/node.yml
groups:
  - name: node-alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value | printf \"%.1f\" }}% on {{ $labels.instance }}"

      - alert: LowDiskSpace
        expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 < 15
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Only {{ $value | printf \"%.1f\" }}% disk space remaining on {{ $labels.mountpoint }}"

      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"

      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
```

---

## Grafana

### Installation

```bash
# Docker
docker run -d \
  --name grafana \
  -p 3000:3000 \
  -e GF_SECURITY_ADMIN_PASSWORD=admin \
  -v grafana_data:/var/lib/grafana \
  grafana/grafana

# Default: http://localhost:3000  admin/admin
```

### Grafana CLI

```bash
# Inside Grafana container
grafana-cli plugins list-remote
grafana-cli plugins install grafana-piechart-panel
grafana-cli plugins install grafana-clock-panel

# Reset admin password
grafana-cli admin reset-admin-password newpassword
```

### Grafana API

```bash
# Create datasource
curl -X POST \
  -H "Content-Type: application/json" \
  -u admin:admin \
  http://localhost:3000/api/datasources \
  -d '{
    "name": "Prometheus",
    "type": "prometheus",
    "url": "http://prometheus:9090",
    "access": "proxy",
    "isDefault": true
  }'

# Import dashboard
curl -X POST \
  -H "Content-Type: application/json" \
  -u admin:admin \
  http://localhost:3000/api/dashboards/import \
  -d @dashboard.json

# List dashboards
curl -u admin:admin http://localhost:3000/api/search | jq

# Export dashboard
curl -u admin:admin http://localhost:3000/api/dashboards/uid/DASHBOARD_UID | jq
```

---

## Loki & Promtail

### Docker Compose

```yaml
version: '3.8'
services:
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    volumes:
      - ./loki-config.yml:/etc/loki/config.yml
      - loki_data:/loki
    command: -config.file=/etc/loki/config.yml

  promtail:
    image: grafana/promtail:latest
    volumes:
      - /var/log:/var/log:ro
      - ./promtail-config.yml:/etc/promtail/config.yml
    command: -config.file=/etc/promtail/config.yml

volumes:
  loki_data:
```

### `promtail-config.yml`

```yaml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log

  - job_name: app
    static_configs:
      - targets:
          - localhost
        labels:
          job: myapp
          env: production
          __path__: /var/log/myapp/*.log
    pipeline_stages:
      - json:
          expressions:
            level: level
            msg: message
      - labels:
          level:
```

---

## PromQL Queries

```promql
# CPU Usage
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory Usage %
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk Usage %
(1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100

# Network Receive Rate (MB/s)
rate(node_network_receive_bytes_total[5m]) / 1024 / 1024

# HTTP Request Rate
rate(http_requests_total[5m])

# HTTP Error Rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) * 100

# 95th Percentile Latency
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# Service Up
up{job="myapp"}

# Container CPU Usage
rate(container_cpu_usage_seconds_total{container!=""}[5m]) * 100

# Container Memory Usage
container_memory_working_set_bytes{container!=""}

# Pod Restart Count
increase(kube_pod_container_status_restarts_total[1h])
```

---

## Kubernetes Monitoring (kube-prometheus-stack)

```bash
# Install via Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm upgrade --install kube-prometheus-stack \
  prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  -f values.yaml

# Access Grafana
kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
# Default credentials: admin/prom-operator

# Access Prometheus
kubectl port-forward svc/kube-prometheus-stack-prometheus 9090:9090 -n monitoring

# Access Alertmanager
kubectl port-forward svc/kube-prometheus-stack-alertmanager 9093:9093 -n monitoring
```

**`values.yaml` snippet:**

```yaml
grafana:
  adminPassword: "admin"
  persistence:
    enabled: true
    size: 5Gi

prometheus:
  prometheusSpec:
    retention: 15d
    storageSpec:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 50Gi

alertmanager:
  alertmanagerSpec:
    storage:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 5Gi
```

---

[← Back to Home](../README.md)
