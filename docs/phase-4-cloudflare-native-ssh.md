# Phase 4 - Cloudflare Native SSH & Remote VNC Access

## Objective

Enable secure remote administration of the homelab environment from outside the home network.

Goals:
- native SSH access
- browser SSH access
- encrypted remote access
- VNC forwarding over SSH
- remote Windows VM management

without exposing public router ports.

---

# Existing Infrastructure

The homelab already used:
- Ubuntu Server
- KVM/QEMU virtualization
- libvirt
- Windows 10 VM
- Cloudflare Tunnel
- Cloudflare Zero Trust

The existing Cloudflare Tunnel route already forwarded SSH traffic:

```text
ssh.mazlum.uk → ssh://localhost:22
```

This enabled:
- browser-based SSH access
- Cloudflare-protected SSH routing

---

# Browser SSH vs Native SSH

Two separate Cloudflare systems were involved.

| Component | Purpose |
|---|---|
| Tunnel Routes | network forwarding |
| Access Applications | authentication and browser SSH |

---

# Browser SSH

Browser SSH already worked successfully using:

```text
https://ssh.mazlum.uk
```

This opened a terminal directly inside the browser.

However:
native OpenSSH connections still failed.

---

# Problem

Running:

```powershell
ssh mazlum@ssh.mazlum.uk
```

resulted in:

```text
Unknown error
```

or:

```text
Connection timed out
```

Reason:
OpenSSH attempted to connect directly to port 22 instead of using Cloudflare Access authentication.

---

# Install cloudflared

The Cloudflare Access client was installed on the remote Windows laptop.

## Installation

```powershell
winget install --id Cloudflare.cloudflared
```

Purpose:
Install native Cloudflare Access support for SSH proxying.

---

# Verify Installation

```powershell
cloudflared --version
```

Purpose:
Verify successful installation.

---

# Authenticate with Cloudflare Access

```powershell
cloudflared access login https://ssh.mazlum.uk
```

Purpose:
Authenticate the client device with Cloudflare Zero Trust.

---

## Cloudflare Access Login

![Cloudflare Login](screenshots/cloudflare-login.png)

---

## Successful Authentication

![Cloudflare Success](screenshots/cloudflare-success.png)

---

# Configure OpenSSH

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
Force OpenSSH to use Cloudflare Access proxying for this hostname.

---

## SSH Config Setup

![SSH Config](screenshots/ssh-config.png)

---

# Common Windows Issue

Initially the configuration file was saved as:

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

without a file extension.

---

# Successful Native SSH Connection

After configuring the SSH proxy,
native SSH connections worked successfully.

Command:

```powershell
ssh mazlum@ssh.mazlum.uk
```

---

## Successful Native SSH Session

![Native SSH](screenshots/native-ssh-success.png)

---

# Remote VNC Access over Cloudflare

After native SSH was working,
an SSH tunnel was used to forward the VM VNC port securely through Cloudflare.

---

# Start the Windows VM

On the Ubuntu server:

```bash
virsh start windows10
```

Purpose:
Start the Windows virtual machine.

---

# Verify VM Status

```bash
virsh list --all
```

Result:

```text
windows10 running
```

---

# Verify VNC Display

```bash
virsh vncdisplay windows10
```

Result:

```text
127.0.0.1:0
```

This confirmed:
- VNC was enabled
- VNC listened locally on port 5900

---

# Verify Local VNC Listener

```bash
ss -tulpn | grep 5900
```

Result:

```text
127.0.0.1:5900 LISTEN
```

Purpose:
Verify the VNC server was actively listening.

---

# Create SSH VNC Tunnel

On the remote Windows laptop:

```powershell
ssh -L 5900:127.0.0.1:5900 mazlum@ssh.mazlum.uk
```

Purpose:
Forward the remote VM VNC session securely through Cloudflare Tunnel and SSH.

---

# Verify Tunnel Locally

```powershell
netstat -ano | findstr :5900
```

Result:

```text
127.0.0.1:5900 LISTEN
```

This confirmed:
the local laptop now forwarded traffic directly into the remote VM VNC service.

---

# Connect with TigerVNC

TigerVNC target:

```text
127.0.0.1:5900
```

Result:
successful remote graphical access to the Windows VM.

---

## Successful Remote VNC Session

![Remote VNC](screenshots/remote-vnc-over-cloudflare.png)

---

# Final Remote Workflow

## 1. Start VM

```bash
virsh start windows10
```

---

## 2. Open SSH Tunnel

```powershell
ssh -L 5900:127.0.0.1:5900 mazlum@ssh.mazlum.uk
```

---

## 3. Open TigerVNC

Target:

```text
127.0.0.1:5900
```

---

# Security Advantages

- No public router ports required
- SSH protected through Cloudflare Zero Trust
- End-to-end encrypted remote access
- VNC traffic tunneled securely
- Remote infrastructure management from anywhere
- Reduced attack surface

---

# Current Infrastructure Status

```text
Ubuntu Homelab Server
├── KVM/QEMU
├── libvirt
├── Windows 10 VM
├── Docker/CasaOS
├── Cloudflare Tunnel
├── Browser SSH
├── Native SSH Access
└── Remote VNC Access
```

---

# Learnings

- Cloudflare Tunnel architecture
- Cloudflare Access authentication
- Browser SSH vs native SSH
- SSH ProxyCommand configuration
- OpenSSH Windows configuration
- SSH tunneling
- Secure VNC forwarding
- Remote virtualization management
- Cloud-native homelab access

---

# Next Phase

Phase 5:
- VM optimization
- VirtIO drivers
- Windows tooling
- Silkroad server preparation
- Database deployment
- Game server environment setup
