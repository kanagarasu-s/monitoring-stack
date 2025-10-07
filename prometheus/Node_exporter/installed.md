Install Node Exporter (for Server Metrics)
## Download Node Exporter
```
cd /opt
sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
```
## Extract & Move Binaries
```
sudo tar -xvzf node_exporter-1.8.2.linux-amd64.tar.gz
sudo mv node_exporter-1.8.2.linux-amd64 node_exporter
sudo cp node_exporter/node_exporter /usr/local/bin/
```
## Create Node Exporter User
```
sudo useradd --no-create-home --shell /bin/false nodeusr
sudo chown nodeusr:nodeusr /usr/local/bin/node_exporter
```
## Create Systemd Service for Node Exporter
```
sudo vi /etc/systemd/system/node_exporter.service
```
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=nodeusr
Group=nodeusr
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
## Start Node Exporter
```
sudo systemctl daemon-reexec
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
systemctl status node_exporter
```
## Open Firewall (if enabled)
```
sudo firewall-cmd --add-port=9100/tcp --permanent
sudo firewall-cmd --reload
```
## Verify Node Exporter
```
http://<your-server-ip>:9100/metrics
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
