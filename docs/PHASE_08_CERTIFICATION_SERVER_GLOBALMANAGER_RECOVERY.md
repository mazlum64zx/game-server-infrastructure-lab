# Phase 8 — Certification Server & GlobalManager Recovery

## Overview

This phase focused entirely on debugging the vSRO188 certification system and recovering the `GlobalManager` startup process.

The server stack initially failed during certification, causing:
- `GlobalManager.exe` crashes
- automatic `.dmp` dump creation
- `GatewayServer.exe` immediate shutdown
- endless:

```text
failed to request certification
```

errors.

This phase documents the complete recovery process.

---

# Initial Symptoms

## GlobalManager Crash

`GlobalManager.exe` continuously created dump files:

```text
GlobalManager_[timestamp].dmp
```

while showing:

```text
failed to request certification
```

---

## MachineManager Status

`MachineManager.exe` displayed:

```text
request server certification
```

without completing registration.

---

## GatewayServer

`GatewayServer.exe` closed instantly without any visible error.

---

# Root Cause Analysis

The issue was NOT related to:

- SQL Server
- ODBC
- database credentials
- firewall
- missing DLLs
- corrupted media

The real problem was:

```text
broken certification configuration
```

caused by:
- invalid certification ports
- inconsistent `srNodeData`
- incorrect `server.cfg` values
- WAN IP usage instead of localhost

---

# Step 1 — Locate Certification Configuration

## Important Files

### srNodeData

Path:

```text
1X Local Files\bin\ini\srNodeData
```

---

### server.cfg

Path:

```text
1X Local Files\Server.cfg
```

---

# Step 2 — Detect Wrong Certification Port

Initially the Certification Server was configured around:

```text
24173
```

This appeared in:
- `srNodeData`
- runtime logs
- old leaked configurations

However vSRO188 internally expects:

```text
32000
```

as the stable certification endpoint.

---

# Step 3 — Fix srNodeData

Inside:

```text
1X Local Files\bin\ini\srNodeData
```

multiple references to:

```ini
port=24173
```

were found.

---

## Required Change

Example correction:

```ini
[entry1]
node_id=1
port=32000
```

All active certification-related entries were aligned to `32000`.

---

# Step 4 — Rebuild packt.dat

After ANY `.ini` modification:

open CMD inside:

```text
1X Local Files\bin
```

and execute:

```cmd
Convert.exe ini ini dat packt.dat
```

Successful output:

```text
The process has completed successfully
```

This step is mandatory because the server reads from `packt.dat`, not directly from `.ini`.

---

# Step 5 — Start Certification Server

Still inside:

```text
1X Local Files\bin
```

execute:

```cmd
CustomCertificationServer.exe packt.dat
```

Correct result:

```text
Certification Server started on 127.0.0.1:32000
```

---

# Step 6 — Verify Active Port

Open any CMD window and run:

```cmd
netstat -ano | findstr 32000
```

Expected:

```text
TCP    127.0.0.1:32000    0.0.0.0:0    LISTENING
```

This confirmed the Certification Server was active.

---

# Step 7 — Fix server.cfg

Inside:

```text
1X Local Files\Server.cfg
```

the original configuration used:

```cfg
GlobalManager {
    Certification "25.73.87.49", 24173
}
```

Problems:
- wrong IP
- wrong port
- external WAN address usage

---

## Correct Configuration

Changed to:

```cfg
GlobalManager {
    Certification "127.0.0.1", 32000
}
```

Only the `GlobalManager` block was modified initially.

Other services remained untouched during early recovery.

---

# Step 8 — Restart Entire Certification Chain

Required order:

## 1. Rebuild

```cmd
Convert.exe ini ini dat packt.dat
```

---

## 2. Start Cert Server

```cmd
CustomCertificationServer.exe packt.dat
```

---

## 3. Start GlobalManager

```text
GlobalManager.exe
```

---

# Successful Recovery

After the fixes:

`GlobalManager.exe` finally initialized correctly.

---

## Successful Logs

```text
successfully server certificated
```

and:

```text
GlobalManager is initialized successfully
```

appeared.

---

# MachineManager Recovery

`MachineManager.exe` connected successfully afterward and certification requests completed properly.

---

# Important Findings

## DMP Files

The `.dmp` files were generated directly by:
- failed certification initialization
- invalid certification socket configuration

NOT by database failures.

---

## Empty ReportLog

`ReportLog` remained empty because the crash occurred BEFORE normal logging initialization.

---

# Key Lessons Learned

## vSRO Leak Configurations Are Often Broken

Many leaked vSRO setups contain:
- invalid WAN IPs
- random certification ports
- mismatched node configurations
- incomplete localhost rewrites

Never trust default leaked configs blindly.

---

# Current Stable State

## Working

```text
✓ SQL Server
✓ ODBC
✓ Certification Server
✓ packt.dat compilation
✓ GlobalManager
✓ MachineManager
```

---

## Remaining Work

Still pending:
- GatewayServer
- DownloadServer
- FarmManager
- AgentServer
- full gameserver chain

---

# Most Important Fixes

## server.cfg

```cfg
GlobalManager {
    Certification "127.0.0.1", 32000
}
```

---

## srNodeData

```ini
port=32000
```

---

## Mandatory Recompile

```cmd
Convert.exe ini ini dat packt.dat
```

without this step, configuration changes do NOT apply.

---

# Final Result

Phase 8 successfully recovered:
- Certification communication
- GlobalManager startup
- MachineManager registration

and established the first stable internal server communication state for the vSRO188 environment.
