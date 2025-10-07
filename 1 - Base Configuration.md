# Part 1 – Base Configuration  

## 📘 Overview  
This section covers the **initial setup and configuration** of all Cisco routers and switches in the physical server rack homelab. The goal is to establish console access, verify hardware functionality, and prepare each device for future VLAN, routing, and monitoring configurations.  

This phase focused on restoring, standardizing, and documenting baseline configurations across the entire rack to ensure consistency before implementing advanced features in later stages.  

---

## ⚙️ Objectives  
- Establish console connectivity for all routers and switches  
- Perform password recovery and restore default configurations  
- Configure hostnames, enable secret passwords, and secure privileged modes  
- Verify hardware, interfaces, and flash storage  
- Set up basic management and SSH access  
- Save and back up initial configurations  

---

## 🧰 Equipment Used  
- Cisco 2951 Router (Edge)  
- Cisco 2911 Router (WAN Sim A)  
- Cisco 2901 Router (WAN Sim B)  
- Cisco 3560 L3 Switch (Core)  
- 3 × Cisco 2960 L2 Switches (Access Layer)  
- USB-to-RJ45 Console Cables  
- PC with Cisco Console Access (Tera Term / PuTTY)  

---

## 🔌 Setup Process  

### 1. Console Access  
- Connected each device using USB-to-RJ45 console cables.  
- Verified access using PuTTY and Tera Term.  
- Recorded COM-port mappings and confirmed terminal output.  

### 2. Password Recovery & Reset  
- Entered ROMmon mode on each device using break sequence.  
- Adjusted `config-register 0x2142` to bypass startup configs.  
- Recovered enable secrets and reset privileged passwords.  

### 3. Baseline Configuration  
- Set unique hostnames (e.g., `R2951-Core`, `SW-3560-L3`).  
- Configured domain names and SSH keys.  
- Enabled logging synchronous and exec-timeout controls on consoles.  
- Assigned IP addresses to management interfaces for later connectivity.  

### 4. Verification & Backups  
- Verified interface status with `show ip int brief`.  
- Saved running configs to startup configs (`copy run start`).  
- Exported all base configs via TFTP for GitHub archiving.  

---

## 📋 Sample Outputs  

**Show Version:**  
