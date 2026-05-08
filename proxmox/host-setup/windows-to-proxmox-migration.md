# Windows to Proxmox Migration Guide

## Purpose

This guide documents the process of converting a Windows PC into the official MutaSpace SOC Lab Proxmox host.

The goal is to make the process clear enough that another beginner can follow it and understand what is happening.

## Warning

Installing Proxmox VE directly onto the PC will remove Windows and erase the selected installation disk.

Do not continue until important files have been backed up and the target disk has been confirmed.

## Learning Goals

By completing this stage, the learner should understand:

1. Why a dedicated virtualization host is useful for a SOC lab.
2. Why installing Proxmox replaces the existing operating system.
3. What hardware information should be recorded before installation.
4. What BIOS/UEFI settings matter for virtualization.
5. Why network planning matters before the first Proxmox login.

## Why This PC Is Being Converted

The original MutaSpace SOC Lab was built on a mini PC. That system was valuable for learning and experimentation, but the official lab needs stronger hardware with more memory, storage, and expansion capacity.

The new PC will become the official Proxmox host for:

1. Wazuh SIEM
2. Firewall/router VM
3. Windows endpoint VMs
4. Linux endpoint VMs
5. Suricata network sensor
6. Red-team simulation VM
7. Trust-boundary research VM
8. Phishing/NLP research VM

## Pre-Install Checklist

Before installing Proxmox:

- [ ] Back up personal files.
- [ ] Save or verify BitLocker recovery key if BitLocker is enabled.
- [ ] Confirm Windows license/recovery option if Windows may be reinstalled later.
- [ ] Record hardware specifications.
- [ ] Record disk layout.
- [ ] Record network adapter information.
- [ ] Download the latest Proxmox VE ISO.
- [ ] Create Proxmox USB installer.
- [ ] Confirm BIOS/UEFI boot order.
- [ ] Confirm virtualization support is enabled.
- [ ] Confirm target disk for Proxmox installation.

## Hardware Information to Record

| Item | Value |
|---|---|
| Manufacturer | |
| Model | |
| CPU | |
| CPU cores/threads | |
| RAM installed | |
| Primary disk | |
| Additional disks | |
| Ethernet adapter | |
| Wi-Fi adapter | |
| GPU | |
| BIOS/UEFI version | |

## Disk Information

| Disk | Size | Current Use | Planned Use |
|---|---:|---|---|
| Disk 0 | | Windows / current OS | Proxmox OS |
| Disk 1 | | | VM storage |
| Disk 2 | | | Backups or ISO storage |

## Network Plan

| Item | Planned Value |
|---|---|
| Proxmox hostname | pve-mutaspace-01 |
| Management bridge | vmbr2 or vmbr0, depending final design |
| Management IP | |
| Gateway | |
| DNS server | |
| Domain/search suffix | |

## Installation Summary

The Proxmox installer will be booted from USB. During installation, the target disk will be selected, and Proxmox will install its operating system and management interface.

After installation, the host will be managed from a web browser using:

```text
https://<proxmox-ip-address>:8006