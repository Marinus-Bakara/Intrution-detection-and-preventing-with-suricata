# 🛡️ pfSense Suricata IDS/IPS Setup Guide

A step-by-step guide to installing and configuring **Suricata** as an Intrusion Detection System (IDS) and Intrusion Prevention System (IPS) on **pfSense Community Edition**. This setup monitors both WAN and LAN interfaces, detects network threats, and automatically blocks malicious hosts.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
  - [Step 1 – Navigate to Package Manager](#step-1--navigate-to-package-manager)
  - [Step 2 – Access Suricata via Services](#step-2--access-suricata-via-services)
  - [Step 3 – Add WAN Interface](#step-3--add-wan-interface)
  - [Step 4 – Configure WAN General Settings](#step-4--configure-wan-general-settings)
  - [Step 5 – Configure Alert & Block Settings (WAN)](#step-5--configure-alert--block-settings-wan)
  - [Step 6 – Save the WAN Interface](#step-6--save-the-wan-interface)
  - [Step 7 – Add LAN Interface](#step-7--add-lan-interface)
  - [Step 8 – Configure LAN General Settings](#step-8--configure-lan-general-settings)
  - [Step 9 – Configure Alert & Block Settings (LAN)](#step-9--configure-alert--block-settings-lan)
  - [Step 10 – Configure Global Settings & Rules](#step-10--configure-global-settings--rules)
  - [Step 11 – Set Block Interval & Logging Options](#step-11--set-block-interval--logging-options)
  - [Step 12 – Update Rule Sets](#step-12--update-rule-sets)
- [Testing the IDS/IPS](#testing-the-idsips)
  - [Step 13 – Check Alerts (Before Test)](#step-13--check-alerts-before-test)
  - [Step 14 – Run Nmap Scan from Kali Linux](#step-14--run-nmap-scan-from-kali-linux)
  - [Step 15 – Verify Alerts Are Triggered](#step-15--verify-alerts-are-triggered)
  - [Step 16 – Confirm IPS Block in Logs View](#step-16--confirm-ips-block-in-logs-view)
- [Architecture Overview](#architecture-overview)
- [Notes & Recommendations](#notes--recommendations)

---

## Overview

This guide demonstrates how to deploy **Suricata 7.0** on pfSense to act as both an IDS (alert-only) and IPS (active blocking). The test scenario uses:

- **Kali Linux** on an external/WAN-side network as the attacker machine
- **Ubuntu** on the internal/LAN network as the target
- **pfSense** (192.168.47.10) as the firewall/IDS-IPS gateway

---

## Prerequisites

- pfSense Community Edition installed and running
- Admin access to the pfSense web interface
- Internet connectivity from pfSense (to download rule sets)
- Basic familiarity with networking concepts (WAN/LAN, IDS/IPS)

---

## Installation

### Step 1 – Navigate to Package Manager

Go to **System → Package Manager** in the pfSense top navigation bar.

![Step 1 – Package Manager](screenshots/01_installation.png)

> Navigate to **System** in the top menu, then select **Package Manager** from the dropdown. You should see any already-installed packages listed here (e.g., `suricata`).

---

## Configuration

### Step 2 – Access Suricata via Services

Once Suricata is installed, access it through **Services → Suricata**.

![Step 2 – Services > Suricata](screenshots/02_services_suricata.png)

> Click **Services** in the top navigation bar and select **Suricata** from the dropdown list to open the Suricata management dashboard.

---

### Step 3 – Add WAN Interface

In the Suricata **Interfaces** tab, click the **+ Add** button to add a new interface.

![Step 3 – Add Interface](screenshots/03_interfaces_add.png)

> The Interface Settings Overview table will be empty initially. Click **+ Add** (highlighted in green, bottom right) to begin configuring your first interface.

---

### Step 4 – Configure WAN General Settings

Enable Suricata on the **WAN (em1)** interface.

![Step 4 – WAN General Settings](screenshots/04_wan_general_settings.png)

| Setting | Value |
|---|---|
| **Enable** | ✅ Checked — enables Suricata inspection on this interface |
| **Interface** | WAN (em1) |
| **Description** | WAN |

> Select the interface that faces the internet (WAN). After selecting the interface, you can configure the settings that apply to your organisation's environment.

---

### Step 5 – Configure Alert & Block Settings (WAN)

Scroll down on the WAN interface settings page to configure blocking behaviour.

![Step 5 – Alert and Block Settings](screenshots/05_alert_block_settings.png)

| Setting | Value | Notes |
|---|---|---|
| **Block Offenders** | ✅ Checked | Automatically blocks hosts that generate a Suricata alert |
| **IPS Mode** | Legacy Mode | Uses PCAP engine; suitable for most NIC drivers |
| **Kill States** | ✅ Checked | Kills firewall states for the blocked IP |
| **Which IP to Block** | SRC | Blocks the source IP; BOTH is also recommended |

> **Legacy Mode** inspects copies of packets via PCAP. Some packet leakage may occur before Suricata can determine if traffic should be blocked. **Inline Mode** offers zero leakage but requires compatible NIC drivers.

---

### Step 6 – Save the WAN Interface

Scroll to the bottom of the WAN interface settings page and click **Save**.

![Step 6 – Save](screenshots/06_save.png)

> After configuring all settings for the WAN interface, click the **Save** button to apply the configuration. Any additional Suricata configuration parameters can be added in the Advanced Configuration Pass-Through field.

---

### Step 7 – Add LAN Interface

Back on the **Interfaces** tab, you will now see the WAN interface listed. Click **+ Add** again to add the LAN interface.

![Step 7 – Add LAN Interface](screenshots/07_lan_add.png)

> The WAN interface now appears in the Interface Settings Overview table with **LEGACY MODE** blocking enabled. Click **+ Add** to add the LAN interface as well.

---

### Step 8 – Configure LAN General Settings

Enable Suricata on the **LAN (em0)** interface.

![Step 8 – LAN General Settings](screenshots/08_lan_general_settings.png)

| Setting | Value |
|---|---|
| **Enable** | ✅ Checked |
| **Interface** | LAN (em0) |
| **Description** | LAN |

> Adding the LAN interface allows Suricata to monitor internal traffic and detect threats originating from within the network.

---

### Step 9 – Configure Alert & Block Settings (LAN)

Apply the same blocking configuration to the LAN interface.

![Step 9 – LAN Block Settings](screenshots/09_lan_block_settings.png)

| Setting | Value |
|---|---|
| **Block Offenders** | ✅ Checked |
| **IPS Mode** | Legacy Mode |
| **Kill States** | ✅ Checked |
| **Which IP to Block** | SRC |

> Enable blocking and block the source from getting to your machine, then click **Save**.

---

### Step 10 – Configure Global Settings & Rules

Navigate to **Global Settings** tab and select the rule sets to download.

![Step 10 – Global Settings](screenshots/10_global_settings.png)

| Rule Set | Status | Description |
|---|---|---|
| **ETOpen Emerging Threats** | ✅ Enabled | Free open-source Suricata rules (limited vs ETPro) |
| **ETPro Emerging Threats** | ☐ Disabled | Paid; daily updates with extensive malware coverage |
| **Snort Rules** | ☐ Disabled | Requires free Registered User or paid Subscriber account |
| **Snort GPLv2 Community Rules** | ✅ Enabled | Free GPLv2 Talos-certified ruleset, updated daily |

> Click on **Global Settings** and select the options that apply to your organisation. ETOpen and Snort GPLv2 Community Rules are free and provide solid baseline coverage.

---

### Step 11 – Set Block Interval & Logging Options

Scroll down in Global Settings to configure logging and block duration.

![Step 11 – Block Interval](screenshots/11_block_interval.png)

| Setting | Value | Notes |
|---|---|---|
| **Remove Blocked Hosts Interval** | 15 MINS | Duration hosts remain blocked (Legacy Mode only) |
| **Log to System Log** | ✅ Checked | Copies Suricata messages to the firewall system log |
| **Log Facility** | LOCAL1 | System log facility for reporting |
| **Log Priority** | NOTICE | Log priority level |
| **Keep Suricata Settings After Deinstall** | ✅ Checked | Settings persist through package removal |
| **Clear Blocked Hosts After Deinstall** | ✅ Checked | Clears blocked hosts when package is removed |

> After selecting your options, click **Save**. Note: the block interval setting is only applicable when using Legacy Mode blocking; it is ignored in Inline IPS Mode. In most cases, **1 hour** is a good choice for production.

---

### Step 12 – Update Rule Sets

Navigate to the **Updates** tab and click **Update** to download the latest rule signatures.

![Step 12 – Updates](screenshots/12_updates.png)

After a successful update, you will see MD5 hashes and timestamps for each enabled rule set:

| Rule Set | MD5 Signature Date |
|---|---|
| Emerging Threats Open Rules | Sunday, 17-May-26 17:53:17 UTC |
| Snort GPLv2 Community Rules | Sunday, 17-May-26 17:53:17 UTC |
| Feodo Tracker Botnet C2 IP Rules | Sunday, 17-May-26 17:52:38 UTC |
| ABUSE.ch SSL Blacklist Rules | Sunday, 17-May-26 18:04:35 UTC |

> **Last Update:** May-17-2026 18:04 — **Result: success**. Click **Force** to force a re-download of all rules even if they appear current.

---

## Testing the IDS/IPS

### Step 13 – Check Alerts (Before Test)

Navigate to the **Alerts** tab to view the alert log. Before running any scans, the log should be empty.

![Step 13 – Alerts Empty](screenshots/13_alerts_empty.png)

> The Alerts view shows the last 250 alert entries. The instance is set to **(WAN) WAN**. Currently in alert-only mode — we will run an Nmap scan to verify that the IDS/IPS detects it.

---

### Step 14 – Run Nmap Scan from Kali Linux

From the **Kali Linux** machine (external network), run an aggressive Nmap scan against the Ubuntu target (internal network):

```bash
nmap -A -T4 -Pn 192.168.47.128
```

![Step 14 – Nmap Scan](screenshots/14_nmap_scan.png)

| Flag | Meaning |
|---|---|
| `-A` | Aggressive scan (OS detection, version detection, script scanning, traceroute) |
| `-T4` | Timing template 4 (fast scan) |
| `-Pn` | Skip host discovery, treat all hosts as online |

> Kali Linux is on the **external network** while Ubuntu is on the **internal network**. This simulates an external attacker probing an internal host through the pfSense firewall.

---

### Step 15 – Verify Alerts Are Triggered

After the Nmap scan completes, return to the **Alerts** tab in Suricata. You should now see multiple alerts:

![Step 15 – Alerts Triggered](screenshots/15_alerts_triggered.png)

Example alerts generated:

| Date | Proto | Class | Src IP | Dst IP | GID:SID | Description |
|---|---|---|---|---|---|---|
| 05/18/2026 10:29:54 | ICMP | Generic Protocol Command Decode | 192.168.127.130 | 192.168.47.128 | 1:2200025 | SURICATA ICMPv4 unknown code |
| 05/18/2026 10:29:54 | ICMP | Generic Protocol Command Decode | 192.168.127.130 | 192.168.47.128 | 1:2200025 | SURICATA ICMPv4 unknown code |

> The IDS successfully detects the Nmap scan traffic. Multiple alerts are raised from source IP **192.168.127.130** (Kali) targeting **192.168.47.128** (Ubuntu).

---

### Step 16 – Confirm IPS Block in Logs View

Navigate to **Logs View**, select the **LAN** instance, and choose **block.log** to confirm the IPS is actively blocking the attacker IP.

![Step 16 – Logs View Block](screenshots/16_logs_view_block.png)

**Log entry example:**
```
05/18/2026-10:29:52.499506  [Block Src] [**] [1:2200025:2] SURICATA ICMPv4 unknown code
[**] [Classification: Generic Protocol Command Decode] [Priority: 3]
```

> Navigate to **Logs View → (LAN) LAN → block.log**. The log confirms that the IPS has blocked the source IP from reaching the system again — until you allow it or when the configured block interval expires (15 minutes in this setup).

---

## Architecture Overview

```
┌─────────────────────┐         ┌──────────────────────────────┐         ┌──────────────────┐
│   Kali Linux        │         │        pfSense Firewall        │         │   Ubuntu Target  │
│  (Attacker)         │  WAN    │  ┌──────────────────────────┐ │  LAN    │  (Internal Host) │
│  192.168.127.130    │◄───────►│  │  Suricata IDS/IPS        │ │◄───────►│  192.168.47.128  │
│  External Network   │         │  │  WAN (em1) + LAN (em0)   │ │         │  Internal Network│
└─────────────────────┘         │  │  Legacy Mode Blocking    │ │         └──────────────────┘
                                │  │  ETOpen + Snort Rules    │ │
                                │  └──────────────────────────┘ │
                                │         192.168.47.10          │
                                └──────────────────────────────┘
```

---

## Notes & Recommendations

- **IPS Mode:** Legacy Mode is the default and works with most NICs. Switch to **Inline Mode** for zero packet leakage if your NIC supports it (bnxt, cc, cxgbe, em, ena, ice, igb, ix, ixgbe, lem, re, vmx, vtnet).
- **Which IP to Block:** `SRC` blocks the attacker's source IP. Setting this to **BOTH** (source and destination) is recommended for broader protection.
- **Block Interval:** 15 minutes is suitable for testing. Consider increasing to **1 hour or more** in production environments.
- **Rule Sets:** ETOpen + Snort GPLv2 Community Rules provide free, solid coverage. For enterprise environments, consider **ETPro** for daily threat intelligence updates.
- **Pass Lists:** Configure Pass Lists to whitelist trusted IPs and prevent false positives from blocking legitimate traffic.
- **Suppress Lists:** Use the Suppress tab to silence noisy rules that cause excessive false positives.
- **Regular Updates:** Schedule automatic rule updates (daily recommended) via the Updates tab to stay current with emerging threats.

---

## License

This project is for educational purposes. Suricata is open-source software licensed under the [GPLv2](https://suricata.io/). pfSense Community Edition is licensed under the [Apache 2.0 License](https://www.pfsense.org/).
