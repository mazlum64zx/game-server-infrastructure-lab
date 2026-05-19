# game-server-infrastructure-lab
Self-hosted infrastructure project for running and managing a private Silkroad Online server environment using Linux, virtualization, Docker and networking.


# Phase 1 - Virtualization Environment Setup

## Objective

Prepare the Ubuntu server for virtualization and future Windows-based game server testing.

---

# System Environment

| Component | Value |
|---|---|
| OS | Ubuntu 26.04 LTS |
| CPU | Intel i5-10310U |
| RAM | 16 GB |
| Storage | 256 GB NVMe |
| Existing Services | CasaOS, Docker |

---

# Virtualization Validation

## Check CPU Virtualization Support

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

Purpose:
Checks whether hardware virtualization is supported.

Result:
- Intel VT-x detected
- Virtualization supported successfully

---

## Verify KVM Availability

```bash
sudo kvm-ok
```

Purpose:
Checks if KVM acceleration can be used by the system.

Result:
- `/dev/kvm` detected
- KVM acceleration available

---

# Installed Virtualization Components

```bash
sudo apt install qemu-system-x86 libvirt-daemon-system bridge-utils virtinst -y
```

Installed:
- QEMU
- libvirt
- bridge-utils
- virtinst

Purpose:
Prepare the server for virtual machine management.

---

# User Permissions

## Add User To libvirt Group

```bash
sudo usermod -aG libvirt $USER
```

Purpose:
Allows VM management without always using sudo.

---

## Reload Group Permissions

```bash
newgrp libvirt
```

Purpose:
Reloads group permissions immediately.

---

# Virtualization Management

## Check Available Virtual Machines

```bash
virsh list --all
```

Purpose:
Displays all virtual machines managed by libvirt.

Result:
- No VMs currently configured
- libvirt working successfully

---

# Storage Preparation

## Create VM Storage Structure

```bash
mkdir -p ~/vm-storage
cd ~/vm-storage
mkdir iso
mkdir disks
```

Resulting Structure:

```text
/home/mazlum/vm-storage
├── iso
└── disks
```

Purpose:
- `iso` stores operating system installation files
- `disks` stores virtual machine disk files

---

# Resource Validation

## Disk Usage

```bash
df -h
```

Result:
- ~198 GB available storage

---

## Memory Usage

```bash
free -h
```

Result:
- ~12 GB available RAM

---

## CPU Information

```bash
lscpu
```

Result:
- 4 Cores
- 8 Threads
- Intel VT-x enabled

---

# Learnings

- Basic Linux filesystem navigation
- Linux user/group permissions
- KVM virtualization basics
- libvirt VM management
- Server resource validation
- VM storage organization
