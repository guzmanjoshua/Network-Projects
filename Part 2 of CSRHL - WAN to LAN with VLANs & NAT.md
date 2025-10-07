# Part 2 of CSRHL - WAN to LAN with VLANs & NAT

## 📌 Overview

This project documents how I connected my **Cisco WAN edge router (2951)** to the **LAN/core switch (3560)** and access switches (2960s). The goal was to:

- Provide **NAT** for Internet access.
- Create **VLAN segmentation** (Mgmt, Servers, Users).
- Enable **inter-VLAN routing** via SVIs.
- Demonstrate DHCP services and PC connectivity.

---

## 🩠 Problem

After initial baseline config (Part 1), the rack was functional but had no:

- VLAN segmentation.
- NAT translation for Internet access.
- DHCP for end devices.

---

## 🔧 Step 1: Configure WAN Edge Router (R2951)

```bash
# WAN side (DHCP from home router / ISP sim)
interface g0/0
ip address dhcp
ip nat outside
no shutdown

# LAN side (link to 3560 VLAN 10)
interface g0/1
description Link to 3560 VLAN 10
ip address 10.0.11.1 255.255.255.0
ip nat inside
no shutdown

# Optional: link for VLAN 20 traffic
interface g0/2
ip address 10.0.20.1 255.255.255.0
ip nat inside
no shutdown
```

### NAT Overload Config

```bash
access-list 1 permit 10.0.11.0 0.0.0.255
access-list 1 permit 10.0.20.0 0.0.0.255

ip nat inside source list 1 interface g0/0 overload
```

---

## 🔧 Step 2: Configure Core Switch (3560)

Enable Layer 3 routing:

```bash
ip routing
hostname S3560
```

### VLAN Interfaces (SVIs)

```bash
# VLAN 10 - Management
interface vlan 10
ip address 10.0.11.2 255.255.255.0
no shutdown

# VLAN 20 - Servers
interface vlan 20
ip address 10.0.20.2 255.255.255.0
no shutdown

# VLAN 30 - Users
interface vlan 30
ip address 10.0.30.2 255.255.255.0
no shutdown
```

---

## 🔧 Step 3: Configure Access Switch (2960 Example)

```bash
# Trunk to 3560
interface g0/1
switchport mode trunk
switchport trunk allowed vlan 10,20,30

# Access ports for VLAN 10 PCs
interface range fastethernet0/2-12
switchport mode access
switchport access vlan 10
spanning-tree portfast
no shutdown
```

---

## 🔧 Step 4: Static PC Configuration Example

```
IP: 10.0.11.50
Subnet mask: 255.255.255.0
Default gateway: 10.0.11.2  (3560 SVI)
DNS: 8.8.8.8 / 1.1.1.1
```

---

## 🔧 Step 5: Verify Connectivity

Key verification commands:

```bash
# On router
show ip interface brief
show ip nat translations

# On switch
show vlan brief
show ip route
```

On PC:

- Ping VLAN gateway (10.0.11.2).
- Ping external IP (after NAT).

---

## ✅ Results

- VLAN segmentation established (Mgmt, Servers, Users).
- NAT overload functional → LAN devices can reach Internet via R2951.
- Inter-VLAN routing works through 3560 SVIs.
- PCs successfully tested with both **static IPs** and DHCP (later moved to Pi in Part 3).

---

## 🚀 Skills Demonstrated

- VLAN and trunk configuration.
- Layer 3 switching with SVIs.
- Router NAT overload setup.
- DHCP relay with `ip helper-address` (if Pi DHCP is used).
- Basic end-device integration with Cisco infrastructure.
