# Monitor Prometheus on Debian13

### Install basic software

Install the prerequisite packages:
```bash
sudo apt install apt-transport-https wget gnupg curl
```
## Prometheus 
Create own user:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin prometheus
```

Download packet:

```bash
curl -LO https://github.com/prometheus/prometheus/releases/download/v3.11.3/prometheus-3.11.3.linux-amd64.tar.gz
```
Descompress tar:

```bash
tar -xvf prometheus-3.11.3.linux-amd64.tar.gz
cd prometheus-3.11.3.linux-amd64
```

Copy binaries:
```bash
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
```

Create directories:
```bash
sudo mkdir /etc/prometheus
sudo mkdir -p /mnt/prometheus-db/prometheus
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
sudo cp prometheus.yml /etc/prometheus/
```

Add permissions:
```bash
sudo chown -R prometheus:prometheus /etc/prometheus /mnt/prometheus-data/prometheus
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
  --storage.tsdb.path=/mnt/prometheus-db/prometheus \
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
ReadWritePaths=/mnt/prometheus-db/prometheus
ReadOnlyPaths=/etc/prometheus

# Límites
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
Enable and Start service:

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable --now prometheus
sudo systemctl status prometheus
```
Try web acces:
http://YOUR-IP:9090

---

## Node-Exporter
Create user:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin node_exporter
```

Download and install:
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
tar xvfz node_exporter-1.10.2.linux-amd64.tar.gz
cd node_exporter-1.10.2.linux-amd64
./node_exporter
```

Copy binaries:
```bash
sudo cp node_exporter-1.10.2.linux-amd64/node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
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
Hardenizado:
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
  --web.listen-address=:9100

Restart=on-failure
RestartSec=5s

# ===== Seguridad =====

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

# ===== CONTROL DE RED =====
IPAddressDeny=any
IPAddressAllow=127.0.0.1
IPAddressAllow=::1

# Solo permitir familias de sockets necesarias
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX

# Node Exporter necesita leer /proc y /sys
ProcSubset=pid

# Permitir acceso de solo lectura al sistema
ReadOnlyPaths=/

# Permitir acceso explícito a métricas kernel
ReadWritePaths=/run

# Capabilities
CapabilityBoundingSet=
AmbientCapabilities=

# Limitar recursos
LimitNOFILE=65536
TasksMax=1024

[Install]
WantedBy=multi-user.target
```

Start and enable service:
```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable --now node_exporter
sudo systemctl status node_exporter
```

Test metrics:
```bash
curl http://localhost:9100/metrics
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


