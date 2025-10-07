# Part 5 of CSRHL - Metrics Monitoring with Prometheus & Grafana

## 📌 Overview

This project adds **metrics-based monitoring** to my Cisco home lab using **Prometheus** (data collection & time-series database) and **Grafana** (visual dashboards).

Whereas Splunk (Part 4) centralizes logs, Prometheus + Grafana provides **real-time performance insights** — CPU, memory, disk usage, and Cisco interface stats (errors, drops, bandwidth).

---

## 🛠️ Problem

Logs alone (Splunk) did not provide a **metrics view** of my devices. I needed a way to:

- See **live CPU/memory/disk usage** on Pis.
- Track **interface errors and bandwidth** on Cisco routers/switches.
- Build **visual dashboards** for faster troubleshooting.

---

## 🔧 Step 1: Install Grafana on Pi4

```bash
sudo apt update
sudo apt install -y wget gnupg
wget -q -O - https://apt.grafana.com/gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/grafana.gpg
echo "deb [signed-by=/usr/share/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install -y grafana
sudo systemctl enable --now grafana-server
```

Access Grafana GUI on your PC:

```
http://10.0.20.12:3000
```

---

## 🔧 Step 2: Install Prometheus on Pi4

```bash
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.55.1/prometheus-2.55.1.linux-armv7.tar.gz
tar xvf prometheus-2.55.1.linux-armv7.tar.gz
cd prometheus-2.55.1.linux-armv7

sudo cp prometheus promtool /usr/local/bin/
sudo mkdir /etc/prometheus /var/lib/prometheus
sudo cp -r consoles/ console_libraries/ /etc/prometheus/
```

### Config: `/etc/prometheus/prometheus.yml`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
```

Enable service:

```bash
sudo systemctl daemon-reexec
sudo systemctl enable --now prometheus
```

Access Prometheus GUI:

```
http://10.0.20.12:9090
```

---

## 🔧 Step 3: Add Node Exporter on Pi4 + Pi5

```bash
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-armv7.tar.gz
tar xvf node_exporter-1.8.2.linux-armv7.tar.gz
cd node_exporter-1.8.2.linux-armv7
sudo cp node_exporter /usr/local/bin/
```

Create systemd service:

```
[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Enable + start:

```bash
sudo systemctl daemon-reexec
sudo systemctl enable --now node_exporter
```

✅ Test from browser:

```
http://<pi-ip>:9100/metrics
```

---

## 🔧 Step 4: Add Cisco Devices via SNMP Exporter

Install SNMP Exporter on Pi4:

```bash
cd /tmp
wget https://github.com/prometheus/snmp_exporter/releases/download/v0.26.0/snmp_exporter-0.26.0.linux-armv7.tar.gz
tar xvf snmp_exporter-0.26.0.linux-armv7.tar.gz
sudo cp snmp_exporter-*/snmp_exporter /usr/local/bin/
```

Update Prometheus config:

```yaml
scrape_configs:
  - job_name: 'cisco-switches'
    static_configs:
      - targets: ['10.0.10.3','10.0.10.4','10.0.10.5']
    metrics_path: /snmp
    params:
      module: [if_mib]

  - job_name: 'cisco-routers'
    static_configs:
      - targets: ['10.0.10.2','10.0.12.1','10.0.11.1']
    metrics_path: /snmp
    params:
      module: [if_mib]
```

Restart Prometheus:

```bash
sudo systemctl restart prometheus
```

---

## 🔧 Step 5: Configure Cisco Devices for SNMP

```bash
conf t
snmp-server community public RO
snmp-server contact Joshua
snmp-server location Homelab-Rack
end
```

Test from Pi:

```bash
snmpwalk -v2c -c public 10.0.10.3
```

---

## 🔧 Step 6: Add Prometheus to Grafana

1. In Grafana: **Connections → Data Sources → Add Prometheus**.
2. URL = `http://localhost:9090`.
3. Save & Test.
4. Import dashboards (example: SNMP Exporter ID 21962).

---

## ✅ Results

- Grafana dashboards now display **live metrics**:
    - Pi CPU, memory, disk usage.
    - Cisco switch/router interface stats (traffic, errors).
- Prometheus scrapes both Linux Node Exporter and Cisco SNMP data.
- Combined with Splunk (Part 4), lab now has **logs + metrics monitoring**.

---

## 🚀 Skills Demonstrated

- Prometheus + Grafana deployment.
- Node Exporter setup on Linux systems.
- SNMP-based monitoring for Cisco devices.
- Grafana dashboard creation & import.
- Full-stack observability (logs + metrics).
