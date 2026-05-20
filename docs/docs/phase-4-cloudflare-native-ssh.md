# Phase 4 - Cloudflare Native SSH Access

## Objective

Enable native SSH access over Cloudflare Tunnel while keeping browser-based SSH access available.

This allows:
- native SSH
- SCP
- SSH tunneling
- VNC forwarding
- remote infrastructure management

from outside the home network.

---

# Existing Infrastructure

Current setup already included:

```text
ssh.mazlum.uk → ssh://localhost:22
```

inside the Cloudflare Tunnel routes.

This means:
Cloudflare already forwarded traffic to the Ubuntu SSH service.

---

# Important Architecture Difference

Two different Cloudflare systems were involved:

| Component | Purpose |
|---|---|
| Tunnel Routes | network routing |
| Applications | access control / browser SSH |

---

# Tunnel Route

Inside:
```text
Cloudflare Tunnel
→ Public Hostnames
```

the SSH route already existed:

```text
ssh.mazlum.uk
→ ssh://localhost:22
```

Purpose:
Forward SSH traffic to the Ubuntu server SSH daemon.

---

# Browser SSH Application

Inside:
```text
Zero Trust
→ Access
→ Applications
```

a Browser SSH application existed.

Purpose:
Provide SSH access directly inside the web browser.

---

# Problem

Browser SSH worked successfully:

```text
https://ssh.mazlum.uk
```

However native SSH commands failed:

```bash
ssh mazlum@ssh.mazlum.uk
```

Result:

```text
Unknown error
```

or:

```text
Connection timed out
```

---

# Root Cause

The client PC did not have:
- Cloudflare Access integration
- cloudflared SSH proxy support

As a result:
OpenSSH attempted to connect directly to port 22.

Cloudflare Tunnel requires:
```text
cloudflared access ssh
```

for native SSH connections.

---

# Install cloudflared

## Install Using winget

```powershell
winget install --id Cloudflare.cloudflared
```

Purpose:
Install the Cloudflare Access client on Windows.

---

# Verify Installation

```powershell
cloudflared --version
```

Purpose:
Verify that cloudflared was successfully installed.

---

# Cloudflare Access Login

```powershell
cloudflared access login https://ssh.mazlum.uk
```

Purpose:
Authenticate the client device with Cloudflare Access.

---

## Login Request

![Cloudflare Login](../screenshots/6.png)

---

## Successful Authentication

![Cloudflare Success](../screenshots/7.png)

---

# SSH Configuration

## Create SSH Config Directory

```powershell
mkdir $env:USERPROFILE\.ssh
```

Purpose:
Create the OpenSSH configuration directory.

---

# Create SSH Config File

Path:

```text
C:\Users\UserS2025\.ssh\config
```

Content:

```text
Host ssh.mazlum.uk
  ProxyCommand cloudflared access ssh --hostname %h
```

Purpose:
Force OpenSSH to use Cloudflare Access proxying for this domain.

---

## SSH Config Setup

![SSH Config](../screenshots/8.png)

---

# Common Windows Issue

The config file was initially saved as:

```text
config.txt
```

instead of:

```text
config
```

This prevented OpenSSH from loading the configuration.

Fix:
Rename the file to:

```text
config
```

without `.txt`.

---

# Verify SSH Config

```powershell
type $env:USERPROFILE\.ssh\config
```

Purpose:
Display SSH configuration contents.

---

# Native SSH Test

```powershell
ssh mazlum@ssh.mazlum.uk
```

Result:
Successful native SSH connection through Cloudflare Tunnel.

---

## Successful Native SSH Connection

![Native SSH](../screenshots/9.png)

---

# SSH Debugging

```powershell
ssh -v mazlum@ssh.mazlum.uk
```

Purpose:
Verbose SSH debugging.

Used to verify:
- SSH config loading
- ProxyCommand usage
- Cloudflare integration

---

# Cloudflare Native SSH Architecture

```text
Laptop
↓
OpenSSH
↓
cloudflared access ssh
↓
Cloudflare Tunnel
↓
Ubuntu SSH Service
↓
Homeserver
```

---

# Advantages

- No open router ports required
- Secure remote access
- Cloudflare authentication
- SCP support
- SSH tunneling support
- VNC forwarding support
- Works outside home network

---

# Future Use Cases

This setup now supports:

```bash
ssh mazlum@ssh.mazlum.uk
```

```bash
scp file.txt mazlum@ssh.mazlum.uk:/home/mazlum
```

```bash
ssh -L 5900:127.0.0.1:5900 mazlum@ssh.mazlum.uk
```

---

# Learnings

- Difference between Cloudflare Tunnel and Access
- Browser SSH vs native SSH
- SSH ProxyCommand
- cloudflared Access integration
- OpenSSH configuration
- Windows SSH config paths
- Remote infrastructure management
- Secure SSH over Cloudflare

---

# Current Infrastructure Status

```text
Ubuntu Homelab Server
├── KVM/QEMU
├── libvirt
├── Windows VM
├── Docker/CasaOS
├── Cloudflare Tunnel
├── Browser SSH
└── Native SSH Access
```

---

# Next Phase

Phase 5:
- Remote VNC over Cloudflare SSH
- Continue Windows VM setup
- Install VM tools/drivers
- Prepare Silkroad server environment
