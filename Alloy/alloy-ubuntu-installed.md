## alloy install on ubuntu 24.04

## Add Grafana Package Repository and GPG Key.
```
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
```
## Update Package Repositories.
```
sudo apt-get update
```
# install alloy.
```
sudo apt-get install alloy
```
# Start Alloy Service.
```
sudo systemctl start alloy
```
# Enable Alloy to Start on Boot.
```
sudo systemctl enable alloy.service
```
# Check Alloy Service Status.
```
sudo systemctl status alloy
```
# Restart Alloy Service.
```
sudo systemctl restart alloy
```
# Stop Alloy Service.
```
sudo systemctl stop alloy
```
# View Alloy Logs.
```
sudo journalctl -u alloy
```
# Default Configuration File
```
vi /etc/default/alloy
```
Path:
 Description: Grafana Alloy settings
 Type:        string
 Default:     ""
 ServiceRestart: alloy

Command line options for Alloy.

The configuration file holding the Alloy config.
CONFIG_FILE="/etc/alloy/config.alloy"

User-defined arguments to pass to the run command.
CUSTOM_ARGS=" "

Restart on system upgrade. Defaults to true.
RESTART_ON_UPGRADE=true

## Edit /etc/default/alloy to set configuration values
```
CONFIG_FILE="/etc/alloy"
CUSTOM_ARGS="--server.http.listen-addr=0.0.0.0:12345"
```

## Edit the Alloy user name systemd service file.
```
 vi /usr/lib/systemd/system/alloy.service
```

# Systemd Service Configuration.
```
[Unit]
Description= Vendor-agnostic OpenTelemetry Collector distribution with programmable pipelines
Documentation=https://grafana.com/docs/alloy
Wants=network-online.target
After=network-online.target

[Service]
Restart=always
User=root
Environment=HOSTNAME=%H
Environment=ALLOY_DEPLOY_MODE=deb
EnvironmentFile=/etc/default/alloy
WorkingDirectory=/var/lib/alloy
ExecStart=/usr/bin/alloy run $CUSTOM_ARGS --storage.path=/var/lib/alloy/data $CONFIG_FILE
ExecReload=/usr/bin/env kill -HUP $MAINPID
TimeoutStopSec=20s
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
```





