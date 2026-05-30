# Monitor Prometheus on Debian13

### Install basic software

Install the prerequisite packages:
```bash
sudo apt install curl net-tools
```
## Prometheus 
Create own user:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin prometheus
```

Download packet:

Last version auto command:
```bash
curl -LO $( \
  curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest \
  | grep "browser_download_url.*linux-amd64.tar.gz" \
  | cut -d '"' -f 4 \
)
```

static version guide:
```bash
curl -LO https://github.com/prometheus/prometheus/releases/download/v3.12.0/prometheus-3.12.0.linux-amd64.tar.gz
```
Descompress tar:

```bash
tar -xvf prometheus-3.11.3.linux-amd64.tar.gz
cd prometheus-3.12.0.linux-amd64
```

Copy binaries:
```bash
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
```

Create directories:
```bash
sudo mkdir /etc/prometheus
sudo mkdir -p /mnt/prometheus-data/prometheus
```

Copy config file:
```bash
sudo cp prometheus.yml /etc/prometheus/
```
Restrict permissions to prometheus user:
```bash
sudo chown -R prometheus:prometheus /mnt/prometheus-data/prometheus
sudo chmod -R 750 /mnt/prometheus-data/prometheus
```

Create service:

```bash
sudo nano /etc/systemd/system/prometheus.service
```
```bash
[Unit]
Description=Prometheus Monitor Server
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus

ExecStartPre=/usr/local/bin/promtool check config /etc/prometheus/prometheus.yml

ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/mnt/prometheus-data/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

Restart=on-failure
RestartSec=5s

# Seguridad
NoNewPrivileges=true
PrivateTmp=true

ProtectSystem=strict
ProtectHome=true

ProtectControlGroups=true
ProtectKernelModules=true
ProtectKernelTunables=true

LockPersonality=true
MemoryDenyWriteExecute=true
RestrictRealtime=true
RestrictNamespaces=true

# Permisos necesarios
ReadWritePaths=/mnt/prometheus-data/prometheus
ReadOnlyPaths=/etc/prometheus

# Límites
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
Enable and Start service:
Reaload daemon and trey start service:
```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
```

⚠️ If dont run use this command and read errors:
```bash
journalctl -xeu prometheus
sudo journalctl -u prometheus --no-pager -n 50
```

✔️ If this script run use:
```bash
sudo systemctl status prometheus
sudo systemctl enable --now prometheus
```

Try web acces:
http://YOUR-IP:9090

## Node-Exporter
Create user:
```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin node_exporter
```

Download automatic latest command:
```bash
curl -LO $( \
  curl -s https://api.github.com/repos/prometheus/node_exporter/releases/latest \
  | grep "browser_download_url.*linux-amd64.tar.gz" \
  | cut -d '"' -f 4 \
)
```

Manual version download:
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
```
Descomppress:
```bash
tar xvfz node_exporter-1.10.2.linux-amd64.tar.gz
cd node_exporter-1.10.2.linux-amd64
```

Copy binaries:
```bash
sudo cp node_exporter /usr/local/bin/
```

Create service: 
```bash
sudo nano /etc/systemd/system/node_exporter.service
```

```bash
[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
Hardened:
```bash
[Unit]
Description=Prometheus Node Exporter
Documentation=https://github.com/prometheus/node_exporter
Wants=network-online.target
After=network-online.target

[Service]
Type=simple

User=node_exporter
Group=node_exporter

ExecStart=/usr/local/bin/node_exporter \
  --web.listen-address=127.0.0.1:9100

Restart=on-failure
RestartSec=5s

# ===== Seguridad Fuerte =====
NoNewPrivileges=true
PrivateTmp=true
PrivateDevices=true

ProtectSystem=strict
ProtectHome=true

ProtectControlGroups=true
ProtectKernelModules=true
ProtectKernelTunables=true
ProtectClock=true
ProtectHostname=true

LockPersonality=true
MemoryDenyWriteExecute=true

RestrictRealtime=true
RestrictNamespaces=true
RestrictSUIDSGID=true

RemoveIPC=true

SystemCallArchitectures=native

# ===== Control de Red (Localhost Seguro) =====
IPAddressDeny=any
IPAddressAllow=127.0.0.1
IPAddressAllow=::1

# AF_NETLINK es vital para las métricas de red (dev, tcp, etc.)
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX AF_NETLINK

# Permitir acceso a recursos de ejecución necesarios
ReadWritePaths=/run

# Capabilities vacías (No necesita privilegios de root para leer /proc)
CapabilityBoundingSet=
AmbientCapabilities=

# Limitar recursos
LimitNOFILE=65536
TasksMax=1024

[Install]
WantedBy=multi-user.target

```

Reload service:
```bash
sudo systemctl daemon-reload
```

start service:
```bash
sudo systemctl start node_exporter
```

❗If the service fail check nad resolv errors:
```bash
journalctl -xeu prometheus
sudo journalctl -u node_exporter --no-pager -n 50
```

✔️ If the service start good enable service:
```bash
sudo systemctl enable --now node_exporter
sudo systemctl status node_exporter
```

Test services listen:
```bash
netstat -tnlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 10.255.255.254:53       0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:9100          0.0.0.0:*               LISTEN      1134/node_exporter
tcp6       0      0 :::9090                 :::*                    LISTEN      1253/prometheus
```

Test metrics in to file:
```bash
curl http://localhost:9100/metrics > metrics.txt
```
Add node_exporter server metrics to Prometheus server:
```bash
nano /etc/prometheus/prometheus.yml
```

```bash
# [=== Node Exporter Prometheus Server===]
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
        labels:
          app: "node_exporter"
```
Add automatic targets folder:
```bash
# [=== Node Exporter Target Hosts===]
  - job_name: 'dynamic_hosts'
    file_sd_configs:
      - files:
          - '/etc/prometheus/targets/*.yml'
          - '/etc/prometheus/targets/*.json'
        refresh_interval: 5m
```

```bash

```
---
## Grafana 
Import the GPG key:

```bash
sudo mkdir -p /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/grafana.asc https://apt.grafana.com/gpg-full.key
sudo chmod 644 /etc/apt/keyrings/grafana.asc
```

Add stable repository:

```bash
echo "deb [signed-by=/etc/apt/keyrings/grafana.asc] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
Refresh APT: 

```bash
apt update
```

Install Grafana:

```bash
apt install grafana
```

Start and enable service:

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

Check status service:

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
```

Try web acces:

http://YOUR-IP:3000/


