# Phase 2 - ISO Preparation & Remote File Transfer

## Objective

Prepare the virtualization environment for the first Windows virtual machine installation.

---

# VM Storage Structure

## Navigate To ISO Directory

```bash
cd vm-storage
cd iso
```

Purpose:
Move into the directory used for operating system ISO files.

---

# Linux Navigation Basics

## Show Current Directory

```bash
pwd
```

Purpose:
Displays the current working directory.

Example Result:

```text
/home/mazlum/vm-storage/iso
```

---

## List Files & Directories

```bash
ls
```

Purpose:
Displays files and folders inside the current directory.

---

# Windows ISO Download

Downloaded:
- Windows 10 Consumer Editions 22H2 x64 ISO

Selected Edition:
- Windows 10 Pro

Reason:
- Better suited for virtualization and infrastructure labs
- Commonly used in technical environments

---

# Remote File Transfer

## Navigate To Desktop In PowerShell

```powershell
cd ~
cd desktop
```

Purpose:
Move into the Windows desktop directory where the ISO was downloaded.

---

## Verify Desktop Files

```powershell
ls
```

Purpose:
Check if the Windows ISO file exists locally.

---

# Secure Copy (SCP)

## Upload ISO To Ubuntu Server

```powershell
scp de-de_windows_10_consumer_editions_version_22h2_updated_oct_2025_x64_dvd_38efd00d.iso mazlum@192.168.0.216:/home/mazlum/vm-storage/iso
```

Purpose:
Transfer the Windows ISO from the local PC to the Ubuntu virtualization server using SSH.

---

# SCP Basics

| Part | Meaning |
|---|---|
| `scp` | secure copy over SSH |
| `DATEI.iso` | local file |
| `mazlum@192.168.0.216` | target Linux server |
| `:/home/...` | destination path on server |

---

# Learnings

- Linux directory navigation
- Difference between local and remote systems
- SSH-based infrastructure workflows
- SCP remote file transfers
- Linux path structures
- PowerShell terminal navigation
- VM ISO organization

---

# Current Infrastructure Status

```text
Ubuntu Server
├── KVM/QEMU
├── libvirt
├── VM storage structure
└── Windows 10 ISO ready
```

---

# Next Phase

Phase 3:
- Create first Windows virtual machine
- Allocate CPU/RAM resources
- Create virtual disk
- Configure VM networking
- Boot Windows installer
