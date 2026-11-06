# Home_lab_attak_similation
This lab present a simulation of exploiting a vernurability on ftp protocole (vsftp 2.3.4)
#  Isolated Virtual Network and Attack Simulation

![Status](https://img.shields.io/badge/status-complete-brightgreen)
![Platform](https://img.shields.io/badge/platform-VirtualBox%20%7C%20GNS3-blue)
![Tools](https://img.shields.io/badge/tools-Nmap%20%7C%20Metasploit%20%7C%20Netcat-orange)
![Focus](https://img.shields.io/badge/focus-offensive%20%2B%20defensive-red)

A self-contained, segmented home lab built to practice reconnaissance, service enumeration, exploitation, and remediation — documented end to end.

##  Disclaimer
All activities conducted as part of this project were carried out within an isolated virtual network ("host-only" mode), targeting an intentionally vulnerable machine (Metasploitable 2) that I own and control. No testing was performed on systems for which I do not have explicit authorization. The techniques presented here are for educational purposes and are intended to strengthen defensive security.

---

##  Contents
1. [Objective](#1-objective)
2. [Lab Architecture](#2-lab-architecture)
3. [Environment Setup](#3-environment-setup)
4. [Phase 1 — Reconnaissance & Enumeration](#4-phase-1--reconnaissance--enumeration)
5. [Phase 2 — Exploitation](#5-phase-2--exploitation)
6. [Findings & Impact](#6-findings--impact)
7. [Mitigation & Remediation](#7-mitigation--remediation)
8. [Lessons Learned](#8-lessons-learned)
9. [Repository Structure](#9-repository-structure)
10. [Appendix — Command Cheat Sheet](#10-appendix--command-cheat-sheet)

---

## 1. Objective
Build an isolated virtual environment to:
- Practice **network segmentation** by placing an attacker and a vulnerable target on a non-routable virtual LAN.
- Perform **port scanning and service enumeration** with Nmap to map the target's attack surface.
- Exploit a confirmed vulnerability to demonstrate real-world impact.
- Document **attack vectors and mitigation steps** as a repeatable, defensive-minded report.

---

## 2. Lab Architecture

**Design principle:** 
- The vulnerable target must never touch the internet or my physical LAN.
- The attacker machine gets a second adapter for updates only.
- This is the segmentation that makes the lab safe to run.

### Network Plan

| VM | Role | Adapter 1 | Adapter 2 | IP Address |
| :--- | :--- | :--- | :--- | :--- |
| **Kali Linux** | Attacker | NAT (Internet for tooling) | Host-Only (`vboxnet0`) | `192.168.56.103` / `10.0.2.15` |
| **Metasploitable 2** | Target | Host-Only (`vboxnet0`) | — | `192.168.56.4` |

### Topology
![Topology](/evidence/01-architecture-virtualbox-hostnetworkmanager.png)

---

## 3. Environment Setup

### 3.1 Host-Only Network (VirtualBox)
- **Path:** File → Host Network Manager → Create
- **Adapter:** `vboxnet0`
- **IPv4:** `192.168.56.1/24` (VirtualBox default)
- **DHCP:** Enabled (or assign static IPs manually)

### 3.2 Metasploitable 2
| Setting | Value |
| :--- | :--- |
| Type / Version | Linux / Other (64-bit) |
| RAM | 512 MB |
| Disk | Existing `.vmdk` (Metasploitable.vmdk) |
| Network | Adapter 1 → Host-Only → `vboxnet0` |
| Default creds | `msfadmin` / `msfadmin` |

### 3.3 Connectivity Verification

**Kali Linux (`ifconfig`):**
```text
(kali@kali)-[~] $ ifconfig eth1
eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.103  netmask 255.255.255.0  broadcast 192.168.56.255
        inet6 fe80::db05:1d1d:45b3:6ef9  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:bb:7c:d4  txqueuelen 1000  (Ethernet)
