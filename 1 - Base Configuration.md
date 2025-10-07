# Part 1 of CHL – Base Configuration  

## 📌 Overview  

This project documents the initial **rack bring-up and baseline configuration** of Cisco routers and switches in my home lab.  
The focus was on **console access, clearing old configs, setting hostnames, configuring interfaces, and saving a clean starting point** for later projects.

### Hardware Used
- Cisco **Router 2951 (R2951)**
- Cisco **Router 2901 (R2901)**
- Cisco **Router 2911 (R2911)**
- Cisco **Catalyst 3560** (Layer 3 Core Switch)
- Cisco **Catalyst 2960** (Layer 2 Access Switches)

---

## 🛠️ Problem  

During the initial setup, the **router interfaces were not coming online**:

- Ethernet LEDs were dark on the routers (though active on the switch side).  
- Old configurations with unknown passwords remained from prior environments.

This required:  
1. **Console access** via PuTTY and a USB-to-RJ45 console cable.  
2. **Password recovery and configuration wipe.**  
3. **Fresh IP assignment** and connectivity verification.

---

## 🔧 Step 1 – Console Access  

1. Identify the COM port under **Device Manager → Ports (COM & LPT)**.  
2. Open **PuTTY** → *Connection type:* Serial → *Serial line:* `COM3` → click **Open**.  
3. When prompted with the initial setup dialog, select **No**.  

---

## 🔧 Step 2 – Basic Router Setup  

```bash
enable
configure terminal
hostname R2951

# Configure G0/0
interface g0/0
ip address 10.0.0.1 255.255.255.252
no shutdown
exit

# Save configuration
write memory
