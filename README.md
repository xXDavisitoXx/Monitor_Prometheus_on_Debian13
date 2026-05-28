# Monitor Prometheus on Debian13

### Install basic software

Install the prerequisite packages:
```bash
sudo apt-get install -y apt-transport-https wget gnupg
```

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
