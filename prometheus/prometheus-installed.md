Install and Configure Prometheus 3.6.0 with Node Exporter on CentOS 7.9
## Update System & Install Dependencies
```
sudo yum clean all
sudo yum -y install wget tar
```
## Download Prometheus 3.6.0
```
cd /opt
sudo wget https://github.com/prometheus/prometheus/releases/download/v3.6.0/prometheus-3.6.0.linux-amd64.tar.gz
```
## Extract & Move Files
```
sudo tar -xvzf prometheus-3.6.0.linux-amd64.tar.gz
sudo mv prometheus-3.6.0.linux-amd64 prometheus
```
## Create Prometheus User
```
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus /var/lib/prometheus
```
## Move Binaries
```
cd /opt/prometheus
sudo cp prometheus promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
```
## Move Config Files
```
sudo cp prometheus.yml /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus
```
## Create Prometheus Systemd Service
```
sudo vi /etc/systemd/system/prometheus.service
```
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ \
    --web.listen-address=0.0.0.0:9090 \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
## Start Prometheus Service
```
sudo systemctl daemon-reexec
sudo systemctl enable prometheus
sudo systemctl start prometheus
systemctl status prometheus
```
## Open Firewall (if enabled)
```
sudo firewall-cmd --add-port=9090/tcp --permanent
sudo firewall-cmd --reload
```
## Access Prometheus
```
http://<your-server-ip>:9090
```
## Configure Prometheus to Scrape Node Exporter
```
sudo vi /etc/prometheus/prometheus.yml
```
```
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```
## Restart Prometheus:
```
sudo systemctl restart prometheus
```
