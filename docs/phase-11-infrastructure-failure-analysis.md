# Phase 11 — Infrastructure Failure Analysis, ODBC/DB Architecture Learning & Full Reset Decision

## Goal

Stabilize the vSRO188 service stack after successful network/IP migration and understand the full database/service architecture deeply enough to rebuild and debug independently.

---

# Initial Situation

After successfully replacing all previously hardcoded IP addresses with my clean internal network IP:

192.168.1.121

all core services were able to establish network-level connectivity.

This was an important milestone because it confirmed:

- hardcoded binary IP replacement worked
- service discovery worked
- certification handshake worked
- internal socket routing worked
- machine/global/farm communication worked

At this point I believed the infrastructure layer was solved.

However:

`SR_ShardManager` consistently failed with:

DB Connection Failed

followed by:

Schedule Manager : g_pServerBodyOfMyself is NULL !!

and later:

Can't find associated locale setting from config!!! [-1]

---

# Investigation Process

## 1. Database Layer Analysis

We investigated:

- SQL Server connectivity
- login credentials (`vsro`)
- database existence
- DB ownership / permissions
- stored procedures
- table accessibility
- shard initialization

Verified working:

- `SRO_VT_ACCOUNT`
- `SRO_VT_SHARD`
- `SRO_VT_SHARDLOG`

User:

vsro

Permissions:

db_owner on all databases

Stored procedures executed successfully.

This proved:

The SQL layer itself was healthy.

---

## 2. ODBC Architecture Discovery

This was one of the biggest learning moments.

I initially assumed:

SSMS connection = server connection

This was false.

vSRO uses:

**32-bit Windows ODBC System DSN connectors**

This introduced:

- DSN
- ODBC driver abstraction
- System DSN vs User DSN
- runtime SQL connector mapping

Created:

- `SRO_VT_ACCOUNT`
- `SRO_VT_SHARD`
- `SRO_VT_LOG`
- later `SRO_VT_SHARDLOG`

This taught me:

vSRO does not connect "directly" through SSMS.

It connects through:

Windows ODBC abstraction layer

which stores:

- server address
- login
- password
- default database
- protocol behavior

This was my first real enterprise-style database connector architecture lesson.

---

## 3. Config / Binary Packaging Discovery

A critical mistake was discovered:

I assumed editing `.ini` files immediately affected runtime behavior.

False.

For this file set:

configuration changes often require rebuilding:

`packt.dat`

using:

Convert.exe

This explained many false-negative tests.

This was a major infrastructure lesson:

Some vSRO builds embed config into compiled runtime packages.

---

## 4. Config Mismatch Discovery

We eventually discovered my server stack had become mixed from several sources.

Because earlier hardcoded IP replacement looked impossible, I had imported:

- alternate GlobalManager binaries
- alternate service files
- alternate config structures

This created:

version mismatch between:

- binaries
- cfg structure
- expected locale definitions
- runtime service registration schema

This caused:

BSObj / locale crashes
despite SQL and networking being correct.

---

# Critical Discovery

I later found the original clean files included an IP Spoofer utility.

This tool allows replacing longer hardcoded IP strings safely inside binaries.

That means:

I never needed to replace binaries from foreign file sets.

The infrastructure corruption was self-inflicted through mixing incompatible versions while trying to solve hardcoded-IP limitations manually.

This was the major root cause.

---

# Final Decision

Instead of patching a corrupted mixed stack further, I decided to:

## Full Clean Reset

Rebuild from original clean files only.

Use:

IP Spoofer

to patch hardcoded IPs properly.

Avoid:

- importing foreign binaries
- copying alternate managers
- mixing cfg versions
- patching around architectural corruption

This ensures:

single-version consistency across:

- GlobalManager
- MachineManager
- FarmManager
- ShardManager
- GameServer
- GatewayServer
- config expectations
- locale definitions
- binary runtime structures

---

# Strategic Reasoning

This is not "starting over."

This is:

architectural reset using knowledge gained.

The first attempt was exploration.

The second build will be deliberate engineering.

---

# Core Lessons Learned

## Network Layer

Learned:

- service certification flow
- internal socket registration
- hardcoded binary endpoint replacement

---

## Database Layer

Learned:

- SQL auth structure
- database ownership
- DB role validation
- runtime stored procedure expectations

---

## ODBC Layer

Learned:

- 32-bit vs 64-bit ODBC separation
- System DSN architecture
- Windows database connector abstraction
- runtime DSN lookup behavior

---

## Packaging Layer

Learned:

- config compilation into packt.dat
- runtime package rebuild requirements

---

## Infrastructure Debugging

Learned systematic elimination:

1. network
2. certification
3. SQL login
4. DB rights
5. table existence
6. ODBC mapping
7. runtime config
8. binary/config compatibility

This was my first full-stack infrastructure debugging cycle.

---

# Phase 12 Goal

Deeply understand the complete vSRO database/service architecture.

I want to fully master:

- how services authenticate
- where DB credentials are read
- how ODBC resolves runtime access
- exact SQL handshake path
- service registration lifecycle
- cert/global/farm/shard/game interaction
- packt.dat compile pipeline
- binary/runtime config loading

Goal:

Be able to rebuild and debug the entire service architecture independently from scratch without relying on trial-and-error.

This phase transitions from:

"getting it to work"

to:

**true infrastructure engineering understanding**
