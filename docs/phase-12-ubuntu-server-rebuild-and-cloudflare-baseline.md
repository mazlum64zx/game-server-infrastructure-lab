# Phase 12 — Ubuntu Server Rebuild & Infrastructure Baseline

## Objective

Complete infrastructure rebuild after instability caused by:
- mixed legacy binaries
- broken remote access
- inconsistent networking
- failed Cloudflare setup
- unreproducible server state

Goal:
- clean reproducible Linux baseline
- stable remote management
- rollback capability
- documented infrastructure phases

---

# Operating System Installation

## OS
- Ubuntu Server 22.04 LTS
- UEFI installation
- GPT partitioning

## Host Configuration
| Setting | Value |
|---|---|
| Hostname | `mazoshome` |
| Username | `mazlum` |

## OpenSSH
Enabled during installation:
```text
Install OpenSSH server
```

---

# Initial System Update

```bash
sudo apt update && sudo apt upgrade -y
```

---

# LVM Storage Expansion

## Problem

Ubuntu allocated only ~100 GB to the root logical volume while ~135 GB remained unused inside the volume group.

---

## Verify Volume Group

```bash
sudo vgdisplay
```

Result:
- VG Size: ~235 GB
- Free Space: ~135 GB

---

## Extend Logical Volume

```bash
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```

---

## Resize Filesystem Online

```bash
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

---

## Verify

```bash
df -h
```

Final result:
- `/` = ~232 GB
- ~214 GB free

---

# Timeshift Snapshot System

## Installation

```bash
sudo apt update && sudo apt install timeshift -y
```

---

## List Available Devices

```bash
sudo timeshift --list-devices
```

Correct snapshot target:
```text
/dev/dm-0
```

---

# Baseline Snapshot

## Create Snapshot

```bash
sudo timeshift --create --comments "clean-server-updated" --tags D
```

---

## Verify Snapshots

```bash
sudo timeshift --list
```

---

# Headless Server Configuration

## Problem

SSH became unreachable after reboot.

Root cause:
- Laptop entered suspend/sleep mode when lid was closed.

---

# Solution

## Configure logind

File:
```bash
/etc/systemd/logind.conf
```

Required values:

```ini
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

---

## Apply Automatically via CLI

```bash
sudo sed -i 's/#HandleLidSwitch=suspend/HandleLidSwitch=ignore/' /etc/systemd/logind.conf && sudo sed -i 's/#HandleLidSwitchExternalPower=suspend/HandleLidSwitchExternalPower=ignore/' /etc/systemd/logind.conf && sudo sed -i 's/#HandleLidSwitchDocked=ignore/HandleLidSwitchDocked=ignore/' /etc/systemd/logind.conf
```

---

## Verify Configuration

```bash
grep HandleLid /etc/systemd/logind.conf
```

Expected result:

```ini
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

---

## Reload logind

```bash
sudo systemctl restart systemd-logind
```

---

# SSH Verification

## Verify SSH Listener

```bash
sudo ss -tulpn | grep :22
```

Result:
- sshd listening on:
  - `0.0.0.0:22`
  - `[::]:22`

---

# Cloudflare Tunnel Setup

## Tunnel Type

Selected:
```text
Cloudflared
```

Not used:
```text
Mesh
```

---

# SSH Tunnel Route

## Hostname

```text
ssh.mazlum.uk
```

## Service Type

```text
SSH
```

## Internal Target

```text
localhost:22
```

---

# DNS Conflict Resolution

## Error

```text
A, AAAA, or CNAME record already exists
```

## Cause

Old DNS record from previous infrastructure setup.

## Resolution

- removed old DNS record
- recreated tunnel route

---

# SSH via Cloudflare Tunnel

## Connect

```powershell
ssh mazlum@ssh.mazlum.uk
```

---

# SSH Host Key Reset

After reinstalling Ubuntu:

```powershell
ssh-keygen -R ssh.mazlum.uk
```

Then reconnect normally.

---

# Snapshot Verification

## Check Snapshot Storage Usage

```bash
sudo du -sh /timeshift
```

Current usage:
```text
5.2 GB
```

---

# Current Infrastructure Status

## Operational Components

- Ubuntu Server
- OpenSSH
- LVM expanded
- Headless server mode
- Timeshift snapshot system
- Cloudflare Tunnel
- Remote SSH access
- Domain-based infrastructure access

---

# Snapshot Strategy

## Current Snapshots

| Snapshot | Purpose |
|---|---|
| `clean-server-updated` | Fresh updated Linux baseline |
| `cloudflare-ssh-working` | Stable remote-access baseline |

---

# Infrastructure Philosophy

New workflow:

```text
Small change
→ verification
→ snapshot
→ next infrastructure layer
```

---

# Next Planned Phase

## Phase 02

Planned components:
- Docker
- Docker Compose
- Reverse Proxy
- Monitoring Stack
- Infrastructure Services

Next planned snapshot:
```text
docker-working
```
