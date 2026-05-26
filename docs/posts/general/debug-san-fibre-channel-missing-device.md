---
date: 2026-05-10
authors: [mohammadhosein]
description: >
  Troubleshooting Fibre Channel SAN Multipath: Missing Devices & "LUNZ" Issue
categories:
  - General
tags:
  - Devops
  - Linux
  - SAN
  - Multipath
---

# Troubleshooting Fibre Channel SAN Multipath: Missing Devices & "LUNZ" Issue

## 📝 Problem Summary
A bare-metal Ubuntu server was physically connected to a Fibre Channel (FC) SAN. The multipath service was installed and running, but no device-mapper multipath devices (e.g., `/dev/mapper/mpathX`) were being created. 

## 🔍 Root Cause
The issue was **not** with the Ubuntu OS or the multipath configuration. The root cause was an incorrect/missing **WWPN (World Wide Port Name) mapping** on the SAN Storage Array. 

Because the server's WWPNs were not properly mapped to a real Storage Volume (LUN) on the array, the storage array presented fake "dummy" LUNs to the server. The OS saw these as `0-Byte` disks. The Linux device mapper (multipath) automatically ignores `0-Byte` disks, which is why no multipath devices were created.

---

## 🛠️ Step-by-Step Troubleshooting Guide

Below are the commands used to diagnose the issue, what each command does, and the actual outputs that led to the solution.

### Step 1: Verify Physical Fibre Channel Link
**Command:**
```bash
cat /sys/class/fc_host/host*/port_state
```
**What it does:** Checks the physical link status of the Host Bus Adapters (HBAs) to see if they are successfully talking to the SAN switch/fabric.
**Result:**
```text
Online
Online
```
*Conclusion:* The physical fiber cables, SFPs, and SAN switch zoning were working perfectly.

### Step 2: Rescan the SCSI Bus
**Command:**
```bash
for host in /sys/class/scsi_host/host*/scan; do echo "- - -" | sudo tee $host; done
```
**What it does:** Forces the Linux kernel to rescan the SCSI host adapters for new storage targets and LUNs.
**Result:** 
```text
- - -
- - -
- - -
```
*Conclusion:* The rescan signals were successfully sent to the HBAs.

### Step 3: Verify Multipath Service Status
**Command:**
```bash
sudo systemctl status multipathd
```
**What it does:** Checks if the Multipath Daemon is running and actively monitoring block devices.
**Result:**
```text
● multipathd.service - Device-Mapper Multipath Device Controller
     Loaded: loaded (/usr/lib/systemd/system/multipathd.service; enabled; preset: enabled)
     Active: active (running)
```
*Conclusion:* The multipath service was healthy and running.

### Step 4: Check OS Block Devices (The First Clue)
**Command:**
```bash
lsblk
```
**What it does:** Lists all block storage devices attached to the system and their respective sizes.
**Result:**
```text
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0  4.4T  0 disk 
├─sda1   8:1    0    1G  0 part /boot/efi
└─sda2   8:2    0  4.4T  0 part /
sdb      8:16   0    0B  0 disk 
sdc      8:32   0    0B  0 disk 
sdd      8:48   0    0B  0 disk 
sde      8:64   0    0B  0 disk 
```
*Conclusion:* The OS saw 4 paths from the SAN (`sdb`, `sdc`, `sdd`, `sde`), but they all had a size of **0B (0 Bytes)**. Device Mapper cannot build multipath devices out of 0-byte disks.

### Step 5: Deep Dive with SCSI Rescan Script (The Smoking Gun)
**Command:**
```bash
sudo rescan-scsi-bus.sh -r
```

[Here](https://bash.cyberciti.biz/diskadmin/rescan-linux-scsi-bus/) is the documentation for the rescan-scsi-bus.sh script.

**What it does:** A robust script that syncs filesystems, scans for new/changed/removed SCSI devices, and queries the hardware for Vendor and Model information.
**Result:** *(Note: This command took a very long time to run due to I/O timeouts on the 0-byte disks).*
```text
 Scanning for device 1 0 0 0 ... 
OLD: Host: scsi1 Channel: 00 Id: 00 Lun: 00
      Vendor: DGC      Model: LUNZ             Rev: 5201
      Type:   Direct-Access                    ANSI SCSI revision: 06
```
*Conclusion:* The hardware reported the Vendor as **DGC** (Dell/EMC) and the Model as **LUNZ**. 
* **LUNZ** is a pseudo-LUN. Dell EMC storage arrays present a "LUNZ" to a server when the server is physically connected, but no actual data LUNs have been mapped to that server's WWPNs.

---

## ✅ Resolution

To fix the issue, the exact WWPNs of the server needed to be retrieved and configured on the SAN Storage Array.

### 1. Retrieve the Server WWPNs
**Command:**
```bash
cat /sys/class/fc_host/host*/port_name
```
**What it does:** Outputs the World Wide Port Names (WWPNs) of the server's Fibre Channel HBAs.
**Result:** Returns two 16-character hexadecimal strings (e.g., `0x100000109bXXXXXX`).

### 2. Fix Mapping on the SAN Storage
The WWPNs retrieved in the previous step were provided to the Storage Administrator (or entered into the Storage Array UI). 
* The WWPNs were properly registered in the Storage Host Group.
* A real Storage LUN (e.g., a 2TB volume) was mapped to the host, overriding the fake `LUNZ`.

### 3. Final Verification
After the SAN team corrected the WWPN mapping:
1. Ran `sudo rescan-scsi-bus.sh -r` again (the script ran instantly this time).
2. Ran `lsblk` — the disks `sdb`, `sdc`, `sdd`, `sde` now showed their real size instead of `0B`.
3. Ran `sudo multipath -ll` — the device mapper successfully aggregated the disks into a single multipath volume (e.g., `/dev/mapper/mpatha`), ready to be formatted and mounted.