# Part 4 of CSRHL - Centralized Logging with Splunk

## 📌 Overview

This project documents deploying **Splunk Enterprise** and **Splunk Universal Forwarders** in my home lab to create a **centralized log collection and monitoring platform**.

Logs were collected from:

- Cisco routers (2951, 2901, 2911)
- Cisco switches (3560, 2960s)
- Raspberry Pi servers (Pi4, Pi5 running Linux services)

Splunk allowed me to **search, enrich, and analyze network and system logs**, similar to how SIEM platforms are used in enterprise environments.

---

## 🛠️ Problem

Initially, logs from Cisco devices and Pis were only visible locally. This made it hard to:

- Track issues across multiple devices.
- Correlate events between routers, switches, and servers.
- Build dashboards for troubleshooting or security monitoring.

I solved this by introducing **Splunk as a centralized SIEM-like tool**.

---

## 🔧 Step 1: Install Splunk Enterprise on PC

On my Windows PC (indexer + search head):

1. Download Splunk Enterprise (Windows 64-bit).
2. Install → accept defaults.
3. Access GUI at:

    ```
    http://localhost:8000
    ```

4. Create admin account.

✅ PC now acts as **Splunk Enterprise** → the central GUI, indexer, and search head.

---

## 🔧 Step 2: Install Splunk Universal Forwarder on Pi5

On Raspberry Pi 5:

```bash
mkdir ~/splunk
cd ~/splunk
wget -O splunkforwarder.tgz "https://download.splunk.com/products/universalforwarder/releases/10.0.0/linux/splunkforwarder-10.0.0-e8eb0c4654f8-linux-arm64.tgz"
sudo tar -xvzf splunkforwarder.tgz -C /opt
cd /opt/splunkforwarder/bin

# Start Splunk
sudo ./splunk start --accept-license
```

Point the forwarder to the Splunk Enterprise PC:

```bash
sudo ./splunk add forward-server 10.0.11.100:9997 -auth admin:YourPassword
```

---

## 🔧 Step 3: Configure Cisco Devices for Syslog

On router/switch:

```bash
conf t
logging host 10.0.20.11       # Pi5 IP
logging trap informational
logging on
end
write memory
```

---

## 🔧 Step 4: Configure Syslog on Pi5

On Pi5, configure rsyslog to receive logs:

```bash
sudo nano /etc/rsyslog.d/cisco.conf

module(load="imudp")
input(type="imudp" port="514")

if ($fromhost-ip == '10.0.10.3') then /var/log/cisco_2960.log
& stop

if ($fromhost-ip == '10.0.10.2') then /var/log/cisco_3560.log
& stop
```

Restart rsyslog:

```bash
sudo systemctl restart rsyslog
```

---

## 🔧 Step 5: Monitor Logs with Splunk UF

Tell Splunk UF to monitor Cisco logs:

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/cisco_2960.log -index main -sourcetype cisco:ios
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/cisco_3560.log -index main -sourcetype cisco:ios
sudo systemctl restart SplunkForwarder
```

---

## 🔧 Step 6: Enable Receiving on Splunk Enterprise

On PC GUI (`http://localhost:8000`):

1. Go to **Settings → Forwarding and Receiving**.
2. Enable port **9997** for data input.

---

## 🔎 Step 7: Verify in Splunk

Search example:

```
index=* sourcetype=cisco:ios
```

Expected: log entries from Cisco switches/routers appear in Splunk search results.

---

## ✅ Results

- Cisco routers/switches now forward logs → Pi5 → Splunk.
- Syslogs are centralized and searchable.
- Able to build alerts and dashboards in Splunk GUI.
- Demonstrates **SIEM-style monitoring in a home lab**.

---

## 🚀 Skills Demonstrated

- Cisco syslog configuration.
- Linux rsyslog + Splunk UF integration.
- Splunk Enterprise installation & administration.
- Indexing, source typing, and searching in Splunk.
- Hands-on SIEM/log management experience.
