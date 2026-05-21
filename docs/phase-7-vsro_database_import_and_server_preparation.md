# Phase 7 — vSRO Database Import & Server Preparation

## Goal

After successfully installing Microsoft SQL Server 2014 Express, the next step was preparing the actual Silkroad Online backend environment.

This phase focused on:
- Installing SQL Server Management Studio (SSMS)
- Connecting to the SQL Server instance
- Understanding SQL architecture basics
- Importing the vSRO databases
- Understanding the internal Silkroad database structure
- Preparing SQL authentication for the server modules
- Configuring localhost networking for the vSRO environment

---

# Installing SQL Server Management Studio (SSMS)

To manage the SQL Server instance, SQL Server Management Studio 22 was installed.

Although the backend itself uses SQL Server 2014 Express, SSMS 22 remains fully compatible.

### Why SSMS?

SSMS acts as:
- a database management interface
- a SQL query editor
- a database restore utility
- a server administration console

Important understanding:

SSMS is NOT the database server itself.

The actual database engine is:

```text
SQL Server (SQLEXPRESS)
```

SSMS only communicates with the SQL Server service.

---

# Connecting to SQL Server

After installation, SSMS was connected to:

```text
localhost\SQLEXPRESS
```

using:

```text
Windows Authentication
```

---

# Understanding the SQL Instance

The SQL Server service was installed as:

```text
SQL Server (SQLEXPRESS)
```

This means:

| Part | Meaning |
|---|---|
| localhost | current machine |
| SQLEXPRESS | SQL Server instance |

The instance name matters because many vSRO files explicitly reference:

```text
.\SQLEXPRESS
```

inside their configuration files.

---

# First SQL Query

A temporary test database was created using:

```sql
CREATE DATABASE TESTDB;
```

This confirmed:
- SQL queries execute correctly
- database creation works
- SSMS communicates properly with SQL Server

The test database was later deleted again.

---

# Understanding Basic SQL Structure

During testing, several important SQL concepts were learned.

| SQL Component | Meaning |
|---|---|
| Database | container for data |
| Table | structured data storage |
| Column | individual data field |
| Row | single data entry |

---

# Importing vSRO Databases

The following Silkroad database backups were imported:

```text
SRO_VT_ACCOUNT
SRO_VT_SHARD
SRO_VT_SHARDLOG
```

The databases were restored from:

```text
.bak
```

backup files.

---

# Understanding SQL Restore

The `.bak` files are not directly opened like normal files.

Instead, SQL Server reconstructs:
- tables
- indexes
- relationships
- stored data

from the backup archive.

This process is called:

```text
Database Restore
```

---

# Understanding the vSRO Database Architecture

## SRO_VT_ACCOUNT

Stores:
- user accounts
- login information
- account privileges

---

## SRO_VT_SHARD

The most important database.

Contains:
- characters
- inventories
- skills
- guilds
- quests
- world data
- NPC data
- item data

Example tables discovered:

| Table | Purpose |
|---|---|
| dbo._Char | character data |
| dbo._Inventory | inventory data |
| dbo._Items | item data |
| dbo._Guild | guild system |
| dbo._CharSkill | skill data |
| dbo._CharQuest | quest data |

Important understanding:

The entire Silkroad world is heavily database-driven.

---

## SRO_VT_SHARDLOG

Stores:
- logs
- events
- historical server data
- transactions

---

# Understanding `dbo`

Many tables were prefixed with:

```text
dbo.
```

Meaning:

```text
database owner
```

This is the default owner schema for SQL Server tables.

Example:

```text
dbo._Char
```

means:

```text
The _Char table belongs to the database owner schema.
```

---

# Preparing the Server Files

After the databases were imported successfully, the vSRO 1.188 server files were extracted.

The server package contained modules such as:

| Module | Purpose |
|---|---|
| GlobalManager | central service management |
| MachineManager | module coordination |
| GatewayServer | login gateway |
| AgentServer | player authentication |
| SR_GameServer | actual game world |
| SR_ShardManager | shard management |

Important realization:

vSRO is NOT a single executable.

It is a collection of independent services communicating over:
- SQL
- IP networking
- internal ports

---

# Understanding SQL Connection Strings

Inside the configuration files, the following SQL connection structure was discovered:

```ini
query=DRIVER={SQL Server};SERVER=.\SQLEXPRESS;DSN=SRO_VT_ACCOUNT;UID=vsro;PWD=vsro;DATABASE=SRO_VT_ACCOUNT
```

This revealed:
- SQL authentication is required
- the files expect a SQL user named `vsro`
- the SQL instance must be `SQLEXPRESS`

---

# Enabling SQL Authentication

SQL Server was already configured with:

```text
Mixed Mode Authentication
```

Meaning both:
- Windows Authentication
- SQL Authentication

were enabled.

This is required for older vSRO environments.

---

# Creating the vSRO SQL Login

A dedicated SQL login was created:

```text
User: vsro
Password: vsro
```

The user received:

```text
db_owner
```

permissions for:
- SRO_VT_ACCOUNT
- SRO_VT_SHARD
- SRO_VT_SHARDLOG

---

# Why This Matters

The vSRO services themselves authenticate directly against SQL Server.

They do NOT use the Windows user account.

Without the dedicated SQL login:
- AgentServer cannot connect
- GatewayServer fails
- GameServer startup breaks

---

# Localhost Network Configuration

The original server files contained external IP addresses such as:

```text
25.73.87.49
```

Examples found:

```ini
DivisionManager "25.73.87.49",16274
```

and:

```ini
wip=25.73.87.49
nip=25.73.87.49
```

For the local homelab environment, all external IPs were replaced with:

```text
127.0.0.1
```

---

# Understanding localhost

| IP | Meaning |
|---|---|
| 127.0.0.1 | current machine |
| 192.168.x.x | local network |
| public IP | internet |

All vSRO services now communicate internally inside the VM.

---

# Security Lessons Learned

During testing, several server file releases triggered Windows Defender detections.

Important lessons:

- old Silkroad files frequently trigger antivirus heuristics
- loaders and launchers are especially risky
- isolated VMs are essential for safe testing
- suspicious files should never be executed blindly

The KVM virtualized environment provided strong isolation between:
- the Ubuntu host
- the Windows vSRO environment

---

# Current Infrastructure Status

## Virtualization
- Ubuntu Server
- KVM/QEMU
- libvirt
- VirtIO integration
- QEMU Guest Agent

## Remote Access
- Cloudflare SSH
- VNC access

## SQL Environment
- SQL Server 2014 Express
- SSMS 22
- Mixed Mode Authentication
- vSRO SQL user configured

## Silkroad Environment
- vSRO 1.188 databases restored
- server files prepared
- localhost network conversion completed

---

# Lessons Learned

This phase provided practical experience with:
- SQL Server management
- SQL authentication
- SQL restore operations
- MMO database architecture
- legacy Windows game server infrastructure
- connection strings
- localhost networking
- service-based MMO systems
- safe malware-aware virtualization workflows

The environment is now prepared for:
- GlobalManager startup
- runtime debugging
- server module communication
- first local server startup
- client connection testing
