# Monitor Prometheus on Debian13

### Install basic software

Install the prerequisite packages:
```bash
sudo apt-get install -y apt-transport-https wget gnupg, curl
```
## Prometheus 
Create own user:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin prometheus
```
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus

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
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
sudo cp prometheus.yml /etc/prometheus/
```

Add permissions:
```bash
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

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


