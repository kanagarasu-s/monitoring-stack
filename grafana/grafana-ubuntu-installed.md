## Grafana Installation on Ubuntu 24.04
## update your system
```
sudo apt update
```
## Install required dependencies
```
sudo apt install -y software-properties-common apt-transport-https wget
```
## Add Grafana GPG key 
```
wget -q -O - https://packages.grafana.com/gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/grafana.gpg
```
# Add Grafana APT repository
```
echo "deb [signed-by=/usr/share/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
```
# install Grafana
```
sudo apt update
sudo apt install grafana -y
```

# Enable and start the Grafana service
```
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```
# Check the status of the Grafana service
```
systemctl status grafana-server
```
# Allow Grafana through the firewall
```
sudo ufw allow 3000/tcp
sudo ufw reload
```
# Access Grafana
```
Open your browser and go to: http://<your-server-ip>:3000
```
## Default credentials:
```
Username: admin
Password: admin
```
You will be prompted to change the default password upon first login.

