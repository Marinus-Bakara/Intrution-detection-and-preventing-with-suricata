# Suricata IDS/IPS Setup on pfSense

A step-by-step guide to installing and configuring **Suricata** as an Intrusion Detection System (IDS) and Intrusion Prevention System (IPS) on **pfSense**. This lab demonstrates how to detect and block network threats — including Nmap scans — from an external Kali Linux machine targeting an internal Ubuntu machine.

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Testing the IDS/IPS](#testing-the-idsips)
- [Results](#results)

---

## Overview

This project covers:

- Installing Suricata via the pfSense Package Manager
- Configuring WAN and LAN interfaces for monitoring
- Enabling IPS blocking mode to automatically block offending hosts
- Setting up threat intelligence rule sets (ETOpen, Snort GPLv2)
- Verifying detection by running an Nmap scan from a Kali Linux machine on an external network

---

## Prerequisites

- pfSense firewall (running as a VM or physical device)
- Kali Linux machine on an **external** network segment
- Ubuntu machine on an **internal/LAN** network segment
- Internet access from pfSense to download rule sets

---

## Installation

### Step 1 — Install Suricata via Package Manager

Navigate to **System → Package Manager** in the pfSense web UI.

![Installation](screenshoots/1_installation.png)

Search for **Suricata** under Available Packages and install it.

### Step 2 — Access the Suricata Service

After installation, go to **Services → Suricata**.

![Access Suricata](screenshoots/2.png)

---

## Configuration

### Step 3 — Add the WAN Interface

On the **Interfaces** tab, click **+ Add** to create a new interface instance.

![Interfaces Tab](screenshoots/3_.png)

Enable Suricata inspection and select **WAN (em1)** as the interface.

![WAN Interface Selection](screenshoots/4_after_selecting_the_interface_you_can_choose_the_settings_thatg_applies_to_your_orgaqnisation.png)

### Step 4 — Configure Alert and Block Settings (WAN)

Scroll down to the **Alert and Block Settings** section:

1. Check **Block Offenders** to automatically block hosts that trigger a Suricata alert
2. Set **IPS Mode** to `Legacy Mode`
3. Set **Which IP to Block** to `SRC`

![WAN Block Settings](screenshoots/5_.png)

Save the configuration.

![Save Settings](screenshoots/6_.png)

### Step 5 — Add the LAN Interface

Back on the **Interfaces** tab, click **+ Add** again to add the LAN interface.

![Add LAN Interface](screenshoots/7_To_add_your_LAN_interface_too.png)

Enable Suricata inspection and select **LAN (em0)** as the interface.

![LAN Interface Selection](screenshoots/8_selecting_the_LAN_interface_.png)

### Step 6 — Configure Alert and Block Settings (LAN)

Apply the same blocking configuration for the LAN interface:

1. **Block Offenders** — checked
2. **IPS Mode** — `Legacy Mode`
3. **Which IP to Block** — `SRC`

![LAN Block Settings](screenshoots/9_enable_blocking_and_block_the_source_from_gettiong_to_your_machine_and_click_on_save.png)

### Step 7 — Configure Global Settings (Rule Sets)

Go to **Global Settings** and select the rule sets to download:

- ✅ **ETOpen Emerging Threats rules** — free open-source Suricata rules
- ✅ **Snort GPLv2 Community rules** — Talos-certified, distributed free of charge

![Global Settings Rule Sets](screenshoots/10_clcik_on_the_global_settings_an_select_the_option_that_apply_fro_you.png)

### Step 8 — Set Remove Blocked Hosts Interval

Still under **Global Settings**, configure how long blocked hosts remain blocked. **15 minutes** is set here (1 hour is recommended for production environments).

> ⚠️ This setting only applies in **Legacy Mode**. It is ignored in Inline IPS Mode.

![Blocked Hosts Interval](screenshoots/11_after_selecting_that_you_clcik_on_save.png)

Save the settings.

### Step 9 — Update Rule Sets

Go to the **Updates** tab and click **Update** to download the latest rule signatures.

![Update Rules](screenshoots/12_click_on_updates.png)

The installed rule sets with their MD5 hashes will be listed once the update is complete.

---

## Testing the IDS/IPS

### Step 10 — Check Alerts Baseline

Navigate to **Alerts** to confirm no alerts exist before running the test.

![Alerts Baseline](screenshoots/13_checking_whether_we_have_any_alerts_hiting_IDS_and_currently_on_alert_so_we_would_run_nmap_scan_and_see_whether_the_settings_we_did_are_working.png)

### Step 11 — Run Nmap Scan from Kali Linux

From the Kali machine (external network), run an aggressive Nmap scan targeting the Ubuntu machine (internal network):

```bash
nmap -A -T4 -Pn 192.168.47.128
```

![Nmap Scan from Kali](screenshoots/14_am_running_a_network_scan_from_my_kali_to_ubuntu_to_see_whether_the_IDS_will_detect_in_kali_is_on_an_external_network_while_ubuntu_is_on_an_internal_network.png)

---

## Results

### IDS Detection — Alerts Generated

After the Nmap scan, Suricata immediately generated multiple alerts on the **WAN** interface. The alerts show:

- **Source IP:** `192.168.127.130` (Kali — external)
- **Destination IP:** `192.168.47.128` (Ubuntu — internal)
- **Protocol:** ICMP
- **Rule triggered:** `1:2200025` — *SURICATA ICMPv4 unknown code*

![IDS Alerts](screenshoots/15_after_running_the_nmap_command_the_IDS_detects_it_these_are_the_alerts_we_are_getting.png)

### IPS Blocking — Host Blocked in Logs

The **IPS** also blocked the source IP, preventing further access. The `block.log` in **Logs View** confirms the block action:

```
05/18/2026-10:29:52.499506  [Block Src] [**] [1:2200025:2] SURICATA ICMPv4 unknown code [**] [Classification: Generic Protocol Command Decode] [Priority: 3]
```

The blocked host is prevented from reaching the system again until the block interval expires or an administrator manually allows the IP.

![IPS Block Log](screenshoots/16_the_IPS_also_prevent_this_ip_from_reaching_this_system_againuntil_you_allow_or_when_you_give_it_a_specific_tikme.png)

---

## Summary

| Component | Role |
|-----------|------|
| pfSense + Suricata | Firewall with integrated IDS/IPS |
| WAN Interface (em1) | Monitors inbound traffic from external network |
| LAN Interface (em0) | Monitors internal network traffic |
| ETOpen + Snort GPLv2 | Threat detection rule sets |
| Legacy Mode (IPS) | Blocks offending source IPs automatically |
| Kali Linux | Attacker machine (external network) |
| Ubuntu | Target machine (internal network) |

Suricata on pfSense provides a powerful, free solution for network intrusion detection and prevention in both home lab and enterprise environments.
