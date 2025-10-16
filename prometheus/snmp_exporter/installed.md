## SNMP Exporter for pfSense

This guide focuses on customizing the snmp_exporter configuration for pfSense.
Prerequisites
Before starting, ensure the following:
Enable SNMP service with modules on your pfSense host.
```
Enabled modules:
MibII
PF
Host Resources
UCD
```
## Install snmpwalk package:

snmpwalk comes from the package net-snmp-utils, and it’s asking if you want to install it. You should go ahead and install it so you can run SNMP queries.
```
sudo dnf install net-snmp-utils
```
## Test PFSense SNMP access to snmp_exporter:

```
snmpwalk -v2c -c public 192.168.130.254 1.3.6.1.2.1.1.1.0
```
## Download pfSense MIBs (SSH login must be enabled temporarily):
```
mkdir -p pfmibs
scp admin@192.168.130.254:/usr/share/snmp/mibs/* /usr/share/snmp/pfmibs
```
To build snmp_exporter and its generator. You have to install git and go on your local system.

Running Prometheus and Grafana server to collect and display metrics as dashboard.
## Install prerequisites on your local system:

1.git
2.go
```
sudo yum install git -y
```
```
sudo yum install golang -y
```
## Check the installed version:
```
go version
```
## Install net-snmp development package
Since you’re on CentOS/RHEL, run:
```
sudo yum install net-snmp net-snmp-devel -y
```
## Verify
Check the header file location:
```
find /usr/include -name net-snmp-config.h
```

## Build and Run snmp_exporter
Clone snmp_exporter:
```
git clone https://github.com/prometheus/snmp_exporter.git
```
## Build snmp_exporter and test on your local environment.
Go back to your snmp_exporter folder:
```
cd /opt/snmp_exporter
make clean
make
```
## Copy pfSense MIBs:
```
/usr/share/snmp/mibs
```
above location in pfsense to snmp_exporter server
```
scp file name root@192.168.55.160:/opt/snmp_exporter/generator/mibs
```
## Build generator and download default MIBs:
```
cd generate
make generator mibs
```
## Backup and edit generator.yml:
```
cp generator.yml generator.yml.bak
```
## new file create generator.yml
```
vim generator.yml
```
```
auths:
  public_v1:
    version: 1
  public_v2:
    version: 2

modules:
  shelbyville:
    walk:
      - 1.3.6.1.2.1.2                 # MIB2::interfaces
      - 1.3.6.1.2.1.31.1              # MIB2::ifMIB::ifXTable
      - 1.3.6.1.4.1.2021.4            # UCD(ucdavis)::memory
      - 1.3.6.1.4.1.2021.9            # UCD(ucdavis)::disk
      - 1.3.6.1.4.1.2021.10           # UCD(ucdavis)::loadAverage
      - 1.3.6.1.4.1.2021.11           # UCD(ucdavis)::cpu
      - 1.3.6.1.4.1.12325.1.200.1.1   # BEGEMOT-PF-MIB::pfStatus
      - 1.3.6.1.4.1.12325.1.200.1.8   # BEGEMOT-PF-MIB::pfInterface
      - 1.3.6.1.4.1.12325.1.200.1.11  # BEGEMOT-PF-MIB::pfLabels

    lookups:
      - source_indexes: [ifIndex]
        lookup: ifAlias
      - source_indexes: [ifIndex]
        lookup: 1.3.6.1.2.1.31.1.1.1.1 # ifName
      - source_indexes: [pfInterfacesIfIndex]
        lookup: pfInterfacesIfDescr
      - source_indexes: [pfLabelsLblIndex]
        lookup: pfLabelsLblName

    overrides:
      ifAlias:
        ignore: true
      ifDescr:
        ignore: true
      ifName:
        ignore: true
      ifType:
        type: EnumAsInfo
      pfInterfacesIfDescr:
        ignore: true
        type: DisplayString
      pfLabelsLblName:
        ignore: true
        type: DisplayString
      pfLogInterfaceName:
        ignore: true
        type: DisplayString
```
## Generate new snmp.yml:
```
MIBDIRS=./mibs ./generator generate
```
## in case any error
this command use
```
MIBDIRS=./mibs ./generator generate --no-fail-on-parse-errors
```
## generator location snmp.yml copy
```
 cp snmp.yml snmp-Shelbyville.yml
```
## that file move snmp_exporter execute file location
```
[root@itsec snmp_exporter]# ls
auth-split-migration.md  collector       CONTRIBUTING.md  generator  LICENSE         Makefile         README.md    snmp_exporter         snmp.yml
CHANGELOG.md             config          Dockerfile       go.mod     main.go         Makefile.common  scraper      snmp-mixin            testdata
CODE_OF_CONDUCT.md       config_test.go  examples         go.sum     MAINTAINERS.md  NOTICE           SECURITY.md  snmp-Shelbyville.yml  VERSION
```
## Create the service file
```
sudo nano /etc/systemd/system/snmp_exporter.service
```
```
[Unit]
Description=SNMP Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/opt/snmp_exporter/snmp_exporter --config.file=/opt/snmp_exporter/snmp-Shelbyville.yml
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```
## Reload systemd
```
sudo systemctl daemon-reload
```
## Enable and start the service
```
sudo systemctl enable snmp_exporter
sudo systemctl start snmp_exporter
```
## Check service status
```
sudo systemctl status snmp_exporter
```
## Test snmp_exporter
```
curl http://localhost:9116/metrics
```


