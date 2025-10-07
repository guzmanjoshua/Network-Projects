# Part 3 of CSRHL - DHCP Migration to Raspberry Pi

## 📌 Overview

This project documents migrating **DHCP services** from the Cisco 2951 router to a **Raspberry Pi 4** running Linux. The goal was to centralize address assignment on a lightweight server, while keeping the Cisco 3560 core switch responsible for DHCP relays.

This demonstrates how enterprise environments often move DHCP/DNS to dedicated servers (Windows/Linux/Infoblox), while routers act only as forwarders.

---

## 🛠️ Problem

Initially, the **2951 router handled DHCP** for VLANs. While functional, this limited flexibility and didn’t reflect real-world environments.

By moving DHCP to a Raspberry Pi, I gained:

- Easier lease monitoring and control.
- More realistic enterprise-style design.
- Experience configuring Linux network services.

---

## 🔧 Step 1: Configure DHCP Server on Pi4

Install and configure ISC DHCP server:

```bash
sudo apt update
sudo apt install isc-dhcp-server -y
```

### `dhcpd.conf` Example

```
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 10.0.10.0 netmask 255.255.255.0 {
  range 10.0.10.20 10.0.10.200;
  option routers 10.0.10.1;
  option domain-name-servers 8.8.8.8;
}

subnet 10.0.20.0 netmask 255.255.255.0 {
  range 10.0.20.20 10.0.20.200;
  option routers 10.0.20.1;
  option domain-name-servers 8.8.8.8;
}

subnet 10.0.30.0 netmask 255.255.255.0 {
  range 10.0.30.20 10.0.30.200;
  option routers 10.0.30.1;
  option domain-name-servers 8.8.8.8;
}
```

Enable DHCP service:

```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl enable isc-dhcp-server
```

---

## 🔧 Step 2: Set Static IP for Pi4

`/etc/dhcpcd.conf`:

```
interface eth0
static ip_address=10.0.20.2/24
static routers=10.0.20.1
static domain_name_servers=8.8.8.8
```

---

## 🔧 Step 3: Configure Cisco Core Switch (DHCP Relay)

On the 3560, point SVIs to the Pi’s IP:

```bash
interface vlan 10
ip helper-address 10.0.20.2

interface vlan 20
ip helper-address 10.0.20.2

interface vlan 30
ip helper-address 10.0.20.2
```

---

## 🔧 Step 4: Remove Old DHCP Config from 2951 Router

```bash
conf t
no ip dhcp pool VLAN10-POOL
no ip dhcp pool VLAN20-POOL
no ip dhcp pool VLAN30-POOL
no ip dhcp excluded-address 10.0.10.1 10.0.10.10
no ip dhcp excluded-address 10.0.20.1 10.0.20.10
no ip dhcp excluded-address 10.0.30.1 10.0.30.10
end
write memory
```

---

## 🔧 Step 5: Verification

On Pi4:

```bash
cat /var/lib/dhcp/dhcpd.leases
```

On Cisco:

```bash
show ip dhcp binding
```

On PC:

- Confirm IP is received in the correct VLAN range.
- Gateway is correct (.1).
- Internet works via NAT on R2951.

---

## ✅ Results

- Pi4 now acts as the **central DHCP server** for VLAN10, VLAN20, VLAN30.
- Cisco 3560 correctly relays DHCP requests to the Pi.
- NAT on the 2951 still provides Internet access.
- Design now mirrors **enterprise separation of services**.

---

## 🚀 Skills Demonstrated

- Linux system administration (Pi4 DHCP server).
- Cisco DHCP relay configuration (`ip helper-address`).
- Router DHCP decommissioning.
- Verifying leases and bindings across devices.
- Hybrid Cisco + Linux integration.
