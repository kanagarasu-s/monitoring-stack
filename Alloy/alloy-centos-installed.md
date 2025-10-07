## install and run Grafana Alloy on CentOS 7.9

## Preparation (packages & facts)
```
sudo yum update -y
sudo yum install -y wget tar gzip policycoreutils-python   # policycoreutils-python needed if you want semanage for SELinux ports
```
## Why: wget/tar unpack the release; semanage (in policycoreutils) is useful only if you need to adjust SELinux ports later.
## (General install docs: Alloy supports Linux AMD64/ARM64 and is distributed as a binary or package).
## Download the correct Alloy binary
```
cd /tmp
wget https://github.com/grafana/alloy/releases/download/vX.Y.Z/alloy-linux-amd64.tar.gz
tar -xzvf alloy-linux-amd64.tar.gz
# the archive should contain the 'alloy' binary
sudo mv alloy /usr/local/bin/alloy
sudo chmod 0755 /usr/local/bin/alloy
```
## Check the binary:
```
/usr/local/bin/alloy -v
```
## Create a dedicated system user and directories
```
sudo useradd --system --no-create-home --shell /sbin/nologin alloy
sudo mkdir -p /etc/alloy /var/lib/alloy
sudo chown -R alloy:alloy /etc/alloy /var/lib/alloy
sudo chmod 750 /etc/alloy /var/lib/alloy
```
## Create your config.alloy (example: Prometheus scrape → remote_write)
```
prometheus.remote_write "default" {
  endpoint {
    url = "https://prometheus-us-central1.grafana.net/api/prom/push"
    basic_auth {
      username = sys.env("GRAFANA_CLOUD_USERNAME")
      password = sys.env("GRAFANA_CLOUD_API_KEY")
    }
  }
}

prometheus.scrape "node" {
  targets = [{"__address__" = "127.0.0.1:9100"}]  # example: node_exporter
  forward_to = [prometheus.remote_write.default.receiver]
}
```
## Set environment file for the service (CentOS / RHEL style)
```
sudo vi /etc/sysconfig/alloy
```
``` 
# Configuration file used by the systemd unit
CONFIG_FILE="/etc/alloy/config.alloy"
# Example: only listen on localhost by default. To expose UI externally change 127.0.0.1 to 0.0.0.0
CUSTOM_ARGS="--server.http.listen-addr=127.0.0.1:12345 --storage.path=/var/lib/alloy"
# Put any secrets for sys.env() here:
GRAFANA_CLOUD_USERNAME="your-org-or-username"
GRAFANA_CLOUD_API_KEY="REPLACE_WITH_SECRET"
```
## Set Permissions
```
sudo chown root:root /etc/sysconfig/alloy
sudo chmod 640 /etc/sysconfig/alloy
```
## Create a systemd service (so Alloy runs on boot)
```
sudo vi /etc/systemd/system/alloy.service
```
```
[Unit]
Description=Grafana Alloy
After=network.target

[Service]
Type=simple
User=alloy
Group=alloy
EnvironmentFile=/etc/sysconfig/alloy
ExecStart=/usr/local/bin/alloy run --config.file=${CONFIG_FILE} ${CUSTOM_ARGS}
Restart=on-failure
RestartSec=5s
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
## Enable and Start the Service
```
sudo systemctl daemon-reload
sudo systemctl enable --now alloy
```
## Validate and Check the Configuration
```
sudo /usr/local/bin/alloy validate /etc/alloy/config.alloy
```
## Check Service Status and Logs
```
sudo systemctl status alloy --no-pager
sudo journalctl -u alloy -f
```
## Alloy UI (Debugging UI) & Firewall
```
sudo firewall-cmd --permanent --add-port=12345/tcp
sudo firewall-cmd --reload
```
## SELinux (CentOS 7) — quick troubleshooting steps
```
sudo setenforce 0    # debug only
# restart alloy, check logs; then re-enable:
sudo setenforce 1
```
## Production Solution
Production solution: create a small SELinux policy module (use audit2allow) or use semanage to add port labeling if needed. (Installing policycoreutils-python earlier gives you semanage.)
