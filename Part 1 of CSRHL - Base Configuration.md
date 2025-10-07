# Part 1 of CSRHL – Base Configuration

## 📌 Overview

This project documents the initial **rack bring-up and baseline configuration** of Cisco routers and switches in my home lab. The focus was on **console access, clearing old configs, setting hostnames, configuring interfaces, and saving a clean starting point** for later projects.

Hardware used:

- Cisco **Router 2951** (R2951)
- Cisco **Router 2901** (R2901)
- Cisco **Router 2911** (R2911)
- Cisco **Catalyst 3560** (Layer 3 switch, core)
- Cisco **Catalyst 2960** (Layer 2 switches, access)

---

## 🩠 Problem

During the initial setup, the **router interfaces were not coming online**:

- Ethernet LEDs were dark on the routers (even though they glowed on the switch side).
- Old configurations (with passwords) were still present from previous environments.

This required:

1. **Console access** to each device via PuTTY + USB-to-RJ45 console cable.
2. **Password recovery and config wipe**.
3. **Assigning fresh IP addresses** and verifying connectivity.

---

## 🔧 Step 1: Console Access

- Identify the COM port under **Device Manager → Ports (COM & LPT)**.
- Open PuTTY → **Connection type: Serial** → Serial line `COM3` → press *Open*.
- When prompted with the initial setup dialog: choose **No**.

---

## 🔧 Step 2: Basic Router Setup

```bash
enable
configure terminal
hostname R2951

# Configure G0/0
interface g0/0
ip address 10.0.0.1 255.255.255.252
no shutdown
exit

# Save config
write memory
```

Repeat for other interfaces:

- G0/0 → 10.0.0.1/30
- G0/1 → 10.0.10.1/30
- G0/2 → 10.0.20.1/30

---

## 🔧 Step 3: Fixing the Config Register

Sometimes devices boot with the wrong config register (0x2142).

Check with:

```bash
show version
```

If set to `0x2142`, reset it:

```bash
configure terminal
config-register 0x2102
end
write memory
reload
```

---

## 🔧 Step 4: Password Recovery (if old configs present)

1. **Reboot the router** and send a *break* signal during startup.
2. At the ROMMON prompt:
    
    ```bash
    confreg 0x2142
    reset
    ```
    
3. After boot, erase old configs:
    
    ```bash
    erase startup-config
    delete vlan.dat
    reload
    ```

This ensures a clean baseline.

---

## ✅ Results

- All router interfaces successfully configured and brought **up** (`no shutdown`).
- Old configs removed → devices now boot with a fresh startup configuration.
- Ready for Part 2: **WAN edge → LAN with VLANs & NAT**.

---

## 🚀 Skills Demonstrated

- Serial console access & troubleshooting
- Cisco IOS privileged/exec/config modes
- Password recovery via config register
- Basic interface configuration (IP addressing + `no shutdown`)
- Persisting configs with `write memory
