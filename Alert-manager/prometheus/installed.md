## Setup of Prometheus Alertmanager in CentOS 

## Update the server

```
dnf update -y
yum update
```

## Download Alertmanager

```
cd /opt
wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz
```

## install and extract Alertmanager

```
tar -xvf alertmanager-0.25.0.linux-amd64.tar.gz
mv alertmanager-0.25.0.linux-amd64 /usr/local/bin/alertmanager
```

## Create the Alertmanager service file

```
vi /etc/systemd/system/alertmanager.service
```

```
[Unit]
Description=Prometheus Alertmanager Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/alertmanager/alertmanager \
    --config.file=/usr/local/bin/alertmanager/alertmanager.yml \
    --cluster.advertise-address="xx.xx.xx.xx:9093"
Restart=on-failure

[Install]
WantedBy=multi-user.target

```

## Verify Alertmanager configuration

```
/usr/local/bin/alertmanager/amtool check-config /usr/local/bin/alertmanager/alertmanager.yml
```

## Reload daemon, enable, start, and check status

```
systemctl daemon-reload
systemctl enable --now alertmanager.service
systemctl status alertmanager.service
```

## Allow port 9093 in the firewall

```
firewall-cmd --permanent --add-port=9093/tcp
firewall-cmd --reload
```

## Access in browser

```
http://<server_ip>:9093
```

## Create Alert Rules for Prometheus

```
mkdir -p /etc/prometheus/rules
```

```
vi /etc/prometheus/rules/alert_rules.yml
```

```
groups:
- name: alert-rules
  rules:


  - alert: Linux Server Host Out Of DiskSpace
    expr: 100 - ((node_filesystem_avail_bytes{mountpoint="/",fstype!="tmpfs"} * 100) / node_filesystem_size_bytes{mountpoint="/",fstype!="tmpfs"}) > 85
    for: 2m
    labels:
      severity: warning
    annotations:
      description: "Disk filled\n  VALUE = {{ $value }}"


  - alert: Linux Server Host Out Of Memory
    expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 < 15
    for: 5m
    labels:
      severity: warning
    annotations:
      description: "Memory remaining\n  VALUE = {{ $value }}"

  - alert: Linux Server HostHighCpuLoad
    expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 85
    for: 5m
    labels:
      severity: warning
    annotations:
      description: "CPU load\n  VALUE = {{ $value }}"

  - alert: Windows Server LowDiskSpace
    expr: (1 - (windows_logical_disk_free_bytes / windows_logical_disk_size_bytes)) * 100 > 85
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Low disk space on {{ $labels.instance }} ({{ $labels.volume }})"
      description: "Disk usage is above 90% on drive {{ $labels.volume }}."

  - alert: Windows Server HighMemoryUsage
    expr: (1 - (windows_memory_available_bytes / windows_memory_physical_total_bytes)) * 100 > 85
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High Memory usage on {{ $labels.instance }}"
      description: "Memory usage is above 85% for more than 2 minutes."

  - alert: Windows Server HighCPUUsage
    expr: (100 - (avg by (instance) (irate(windows_cpu_time_total{mode="idle"}[5m])) * 100)) > 85
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "High CPU usage on {{ $labels.instance }}"
      description: "CPU usage is above 85% for more than 2 minutes."

  - alert: pfSenseHighCPU
    expr: avg by (instance) (hrProcessorLoad) > 85
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "High CPU usage on pfSense {{ $labels.instance }}"
      description: "CPU usage above 85% for more than 2 minutes."

  - alert: pfSenseHighMemory
    expr: ((memTotalReal - memAvailReal) / memTotalReal) * 100 > 85
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High Memory usage on pfSense {{ $labels.instance }}"
      description: "Memory usage above 85% for more than 2 minutes."

  - alert: pfSenseLowDisk
    expr: (hrStorageUsed{hrStorageDescr="/"} / hrStorageSize{hrStorageDescr="/"} * 100) > 85
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Low Disk space on pfSense {{ $labels.instance }}"
      description: "Disk usage above 90% on {{ $labels.hrStorageDescr }}."

  - alert: LinuxServerDown
    expr: up{job=~"prometheus-server|uat-server|dev-server|prodsecondary-server|sasqlbackup-server"} == 0
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "Linux Server Down: {{ $labels.instance }}"
      description: "The Linux server {{ $labels.job }} ({{ $labels.instance }}) has been unreachable for more than 2 minutes."


  - alert: pfSenseServerDown
    expr: up{job=~"Bangalore|Indiana|Shelbyville"} == 0
    for: 2m
    labels:
      severity: critical
      type: pfsense
    annotations:
      summary: "pfSense Down: {{ $labels.instance }}"
      description: |
        pfSense device {{ $labels.instance }} is unreachable or SNMP Exporter stopped responding
        for more than 2 minutes.


  - alert: WindowsServerDown
    expr: up{job=~"adbdc2019|bucolfs"} == 0
    for: 2m
    labels:
      severity: critical
      type: windows
    annotations:
      summary: "Windows Server Down: {{ $labels.instance }}"
      description: |
        Windows server {{ $labels.instance }} is unreachable or exporter stopped responding
        for more than 2 minutes.
```

## Update Prometheus Configuration

```
vi /etc/prometheus/prometheus.yml
```

```
global:
  scrape_interval: 15s

rule_files:
  - /etc/prometheus/rules/*.yml

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - "localhost:9093"
```

## Reload Prometheus

```
systemctl daemon-reload
systemctl restart prometheus
systemctl status prometheus
```

## Configure Alertmanager for Notification

```
vi /usr/local/bin/alertmanager/alertmanager.yml
```

```
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'yourmail@gmail.com'
  smtp_auth_username: 'yourmail@gmail.com'
  smtp_auth_password: 'your_app_password'
  smtp_require_tls: true

route:
  receiver: 'email-alert'

receivers:
  - name: 'email-alert'
    email_configs:
      - to: 'admin@example.com'
```

## Microsoft Teams Notification

```
route:
  receiver: 'teams-notify'

receivers:
  - name: 'teams-notify'
    webhook_configs:
      - url: 'https://outlook.office.com/webhook/XXXXXXXXXXXXXX'
```

## Reload Alertmanager

```
systemctl restart alertmanager
systemctl status alertmanager
```


