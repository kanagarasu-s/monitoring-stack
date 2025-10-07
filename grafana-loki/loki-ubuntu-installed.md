
##  Download Loki Latest Version
```
https://github.com/grafana/loki/releases/download latest version for loki.
```

## Install unzip (if not installed)
```
apt install unzip

```
## Unzip the Loki binary
```
unzip loki-linux-amd64.zip
```
##   Set permissions and move binary
```
chmod +x loki-linux-amd64
sudo mv loki-linux-amd64 /usr/local/bin/loki
sudo chmod +x /usr/local/bin/loki

```
## Create Loki configuration file
```
vi loki-local-config.yaml
```
## Sample loki-local-config.yaml
```
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096
  log_level: debug
  grpc_server_max_concurrent_streams: 1000

common:
  instance_addr: 127.0.0.1
  path_prefix: /tmp/loki
  storage:
    filesystem:
      chunks_directory: /tmp/loki/chunks
      rules_directory: /tmp/loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: memberlist

memberlist:
  join_members:
    - 127.0.0.1

query_range:
  results_cache:
    cache:
      embedded_cache:
        enabled: true
        max_size_mb: 100

limits_config:
  metric_aggregation_enabled: true
  enable_multi_variant_queries: true
  allow_structured_metadata: false
  retention_period: 0s    # 🔑 Keep logs forever (no automatic deletion)

schema_config:
  configs:
    - from: 2020-10-24
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

ingester:
  chunk_idle_period: 5m
  chunk_retain_period: 5m
  max_chunk_age: 2h      # Aligns with TSDB block size (avoid 1h-only issue)

compactor:
  working_directory: /tmp/loki/compactor
  retention_enabled: false   # 🔑 Disable automatic block deletion

pattern_ingester:
  enabled: true
  metric_aggregation:
    loki_address: localhost:3100

ruler:
  alertmanager_url: http://localhost:9093

frontend:
  encoding: protobuf

#analytics:
#reporting_enabled: false
```


## Create Loki systemd service
```
sudo vi /etc/systemd/system/loki.service
```
## Loki service file
```
[Unit]
Description=Loki Log Aggregation System
After=network.target
[Service]
User=root
ExecStart=/usr/local/bin/loki --config.file=/opt/loki-local-config.yaml
Restart=always
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
```

## Start and enable Loki service
```

systemctl start loki

systemctl status loki

systemctl enable loki
```

