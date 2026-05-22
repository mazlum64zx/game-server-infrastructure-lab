# Phase 10 — Hardcoded IPs & Isolated VM Networking

## Overview

Phase 10 focused on solving one of the first real infrastructure-level problems inside the vSRO188 server architecture.

Unlike previous phases which mainly involved:
- installation
- deployment
- configuration
- VM setup

this phase entered much deeper territory involving:
- runtime service identity
- service certification
- TCP verification
- hardcoded executable IPs
- binary behavior
- virtualization networking
- isolated multi-NIC VM architecture
- infrastructure debugging

The main objective was to understand why several vSRO services failed certification even though:
- ports were open
- config files were correct
- services appeared online

This phase became an important transition from:

```text
simple setup work
```

into:

```text
real infrastructure debugging and runtime analysis
```

---

# 1. Initial Problem

After successfully starting:
- CustomCertificationServer
- GlobalManager

the next services:
- MachineManager
- GatewayServer
- FarmManager
- AgentServer

failed certification.

The GlobalManager displayed errors such as:

```text
cannot certify server body : [MachineManager][25.73.87.49]
```
<img width="827" height="582" alt="Screenshot 2026-05-21 211201" src="https://github.com/user-attachments/assets/cf17d945-1936-477a-adb6-f1203361f21c" />

or later:

```text
cannot certify server body : [MachineManager][192.168.122.198]
```

This proved:

- the service could reach GlobalManager
- TCP communication existed
- the service was not crashing immediately

However:

```text
the runtime identity was rejected during certification.
```

This became the first major realization that:
- network connectivity
- and service certification

are two different layers.

---

# 2. Understanding Runtime Identity

One of the most important discoveries in this phase was:

```text
A successful TCP connection does NOT automatically mean successful certification.
```

A service can:
- connect to GlobalManager
- open sockets correctly
- appear in topology

and still fail certification because:
- the runtime identity is invalid
- the reported IP does not match expectations
- the service body is rejected

This became one of the first major lessons about distributed service architectures inside vSRO.

---

# 3. Config Layer vs Executable Layer

Another major discovery was understanding that:

```text
Config layer and executable layer are separate things.
```

Possible runtime identity sources inside vSRO included:

- server.cfg
- packt.dat
- srNodeType.ini
- hardcoded EXE strings
- network adapter detection

This meant:

```text
Editing server.cfg alone is NOT always enough.
```

Even after rebuilding:

```cmd
Convert.exe ini ini dat packt.dat
```

MachineManager still attempted certification using:

```text
25.73.87.49
```

This proved:

```text
The executable itself contained hardcoded runtime IP data.
```

---

# 4. Using tasklist to Verify Running Services

One important debugging technique involved verifying whether services were actually running.

Command:

```cmd
tasklist | findstr Manager
```
<img width="715" height="638" alt="Screenshot 2026-05-21 204324" src="https://github.com/user-attachments/assets/db8c0171-6ce6-4570-9867-da6ece2c9edf" />

Purpose:
- lists all running Windows processes
- filters output to processes containing `Manager`

This helped verify:
- MachineManager.exe
- GlobalManager.exe
- FarmManager.exe

were actually alive.

---

Another useful variation:

```cmd
tasklist | findstr Server
```

Purpose:
- verify server processes
- verify GatewayServer.exe
- verify AgentServer.exe
- verify CertificationServer.exe

Important realization:

```text
GlobalManager topology alone is NOT proof that a service is healthy.
```

The topology may still display nodes even when:
- certification failed
- sockets closed
- processes crashed

---

# 5. Using netstat for Real TCP Verification

Another critical debugging tool was:

```cmd
netstat -ano
```
<img width="749" height="671" alt="Screenshot 2026-05-21 204022" src="https://github.com/user-attachments/assets/624a5996-ba62-4ddf-af8c-8285a321b85d" />

Purpose:
- verify listening ports
- verify active sockets
- verify established TCP sessions
- display process IDs

This allowed verification of the:
- real network state
- real TCP state

instead of relying on:
- assumptions
- startup windows
- GUI topology

---

Useful examples:

```cmd
netstat -ano | findstr 32000
```

Purpose:
- verify CertificationServer socket state

---

```cmd
netstat -ano | findstr 24523
```
<img width="715" height="638" alt="Screenshot 2026-05-21 204324" src="https://github.com/user-attachments/assets/14be5540-a0a7-483c-8f81-2b7c2ed63dbc" />

Purpose:
- verify SR_GameServer or AgentServer ports

---

```cmd
netstat -ano | findstr 16274
```

Purpose:
- verify MachineManager communication

---

Important TCP states learned:

```text
LISTENING
```

Meaning:
- service is waiting for incoming connections

---

```text
ESTABLISHED
```

Meaning:
- active TCP connection currently exists

This became one of the most important debugging techniques during this phase.

---

# 6. Discovering Hardcoded IPs

A major breakthrough came from using:

```cmd
findstr /S /M "25.73.87.49" *.*
```
<img width="842" height="653" alt="Screenshot 2026-05-21 214754" src="https://github.com/user-attachments/assets/d50bbfd2-0d64-48aa-ae8c-a36886c317e2" />

Explanation:

```cmd
/S
```

search recursively through subfolders.

```cmd
/M
```

display only filenames containing the string.

```cmd
*.*
```

search all file types.

---

Result:

```text
MachineManager.exe
AgentServer.exe
SR_GameServer.exe
```

This proved:

```text
The old runtime IP was stored directly inside executable binaries.
```

This completely changed the debugging direction.

The issue was no longer:
- firewall
- port forwarding
- wrong config

The issue became:

```text
hardcoded runtime identity inside the EXE.
```

---

# 7. Why Rebuilding packt.dat Was Still Important

Even though the main issue involved hardcoded executable IPs,
rebuilding:

```text
packt.dat
```

remained important.

Command:

```cmd
Convert.exe ini ini dat packt.dat
```
<img width="550" height="155" alt="Screenshot 2026-05-21 212840" src="https://github.com/user-attachments/assets/6ccf1963-66f5-4a52-b1c3-f67424b5f158" />

Purpose:
- rebuild runtime certification data
- compile updated INI configuration into DAT format

Explanation:

```cmd
Convert.exe
```

runs the vSRO converter utility.

```cmd
ini
```

input format.

```cmd
ini
```

input folder.

```cmd
dat
```

output format.

```cmd
packt.dat
```

generated runtime certification database.

This step ensured:
- node configuration changes
- certification updates
- INI modifications

were compiled into the runtime data used by:

```text
CustomCertificationServer.exe
```

---

# 8. Starting CertificationServer

Command:

```cmd
CustomCertificationServer.exe packt.dat
```

Purpose:
- start CertificationServer
- load packt.dat as certification database
<img width="662" height="113" alt="image" src="https://github.com/user-attachments/assets/73b16782-57e5-4302-a040-19056a66921c" />

Expected output:

```text
Certification Server started on 127.0.0.1:32000
```

Meaning:
- CertificationServer is listening successfully
- TCP port 32000 is active

---

# 9. Discovering Runtime Hardcoded IPs

Initially MachineManager sent:

```text
MachineManager[25.73.87.49]
```

Later, after replacing the executable with another version:

```text
MachineManager[192.168.1.121]
```
<img width="1531" height="1289" alt="Screenshot 2026-05-21 223518" src="https://github.com/user-attachments/assets/1e11fc55-1533-435f-9973-6cb4aa7298fb" />

This proved:

```text
The runtime IP was embedded directly inside the executable.
```

Even supposedly:
- clean files
- replacement executables

still contained:
- predefined runtime identities
- embedded IP assumptions

---

# 10. Why Simple Hex Editing Was Dangerous

At first, HxD hex editing was considered.
<img width="1475" height="904" alt="Screenshot 2026-05-21 221600" src="https://github.com/user-attachments/assets/de18e2d5-365f-4caa-82d5-04a1cacf5bdc" />

However:

```text
192.168.1.121
```

contains:

```text
13 characters
```

while:

```text
192.168.122.198
```

contains:

```text
15 characters
```

This created a serious binary safety issue.

Replacing a string with a longer one can:
- shift offsets
- corrupt instructions
- break executable structures
- crash the process

Important lesson:

```text
Inline hex replacement is only safe when the replacement string is equal length or shorter.
```

This explained why:
- direct HxD editing
- ASCII replacement

became unsafe for larger IPs.

---

# 11. Avoiding Dangerous Binary Repointing

At this point, another major realization occurred:

```text
Binary patching should be avoided unless absolutely necessary.
```

A cleaner infrastructure design was often:
- safer
- easier
- more maintainable

than:
- ASM patching
- executable repointing
- code caves
- PE modifications

This shifted the strategy away from:

```text
modifying the executable
```

toward:

```text
designing a better virtual network architecture.
```

---

# 12. Designing an Isolated Dual-NIC VM Architecture

The final solution became:

```text
Adapter 1:
192.168.122.x
Purpose:
Internet / NAT / VM management

Adapter 2:
192.168.1.121
Purpose:
Internal vSRO certification identity
```

This design solved:
- hardcoded runtime IP problems
- certification mismatch
- VM isolation concerns

while preserving:
- internet access
- NAT networking
- host security

---

# 13. Creating an Isolated libvirt Network

A new isolated libvirt network was created.
<img width="613" height="23" alt="Screenshot 2026-05-22 003850" src="https://github.com/user-attachments/assets/1ff3cdaf-a773-4561-9270-0c6fba7d18e1" />

Command:

```bash
nano isolated-vsro.xml
```

Purpose:
- create a new libvirt network definition file

---

Network XML:
<img width="803" height="199" alt="Screenshot 2026-05-22 003906" src="https://github.com/user-attachments/assets/393f8f9a-db4f-4c43-8b4f-01b13dabf951" />

```xml
<network>
  <name>vsroisolated</name>
  <bridge name='virbr1'/>
  <ip address='192.168.1.1' netmask='255.255.255.0'>
  </ip>
</network>
```

Explanation:

```xml
<name>
```

defines the libvirt network name.

---

```xml
<bridge name='virbr1'/>
```

creates a new isolated virtual bridge interface.

---

```xml
192.168.1.1
```

becomes the host-side gateway/interface of the isolated network.

Important:

```text
This network exists only between:
- host
- VM
```

It does NOT expose the VM to the real home LAN.

---

Saving in nano:

```text
CTRL + O
```

save file.

```text
ENTER
```

confirm filename.

```text
CTRL + X
```

exit nano.

---

# 14. Registering the New libvirt Network
<img width="833" height="392" alt="Screenshot 2026-05-22 003926" src="https://github.com/user-attachments/assets/64fe4774-cf8f-47bf-81cb-09220c69e5a5" />

Command:

```bash
virsh net-define isolated-vsro.xml
```

Purpose:
- register the network definition inside libvirt

---

Command:

```bash
virsh net-start vsroisolated
```

Purpose:
- start the isolated virtual network

---

Command:

```bash
virsh net-autostart vsroisolated
```

Purpose:
- automatically start the network after host reboot

---

Command:

```bash
virsh net-list --all
```
<img width="803" height="177" alt="Screenshot 2026-05-22 003945" src="https://github.com/user-attachments/assets/b36abb0e-5972-4588-ba82-0d60a62e1c39" />

Purpose:
- display all libvirt networks and their status

Expected result:

```text
vsroisolated active
```

---

# 15. Attaching a Second Virtual NIC to the VM

Command:
<img width="1662" height="111" alt="Screenshot 2026-05-22 004020" src="https://github.com/user-attachments/assets/12cd6642-5559-4c4e-8d52-0f4f93fc531c" />

```bash
virsh attach-interface \
  --domain windows10 \
  --type network \
  --source vsroisolated \
  --model e1000e \
  --config \
  --live
```

Purpose:
- add a second virtual network interface to the Windows VM

---

Parameter explanation:

```bash
--domain windows10
```

select VM.

---

```bash
--type network
```

use a libvirt virtual network.

---

```bash
--source vsroisolated
```

connect adapter to isolated vSRO network.

---

```bash
--model e1000e
```

use Intel virtual NIC model.

---

```bash
--config
```

make adapter persistent after reboot.

---

```bash
--live
```

attach immediately while VM is running.

---

# 16. Verifying VM Interfaces

Command:

```bash
virsh domiflist windows10
```

Purpose:
- display all virtual network interfaces attached to the VM

Expected final structure:

```text
1x default NAT adapter
1x vsroisolated adapter
```

---

# 17. Configuring Windows VM Adapters

After attaching the second NIC, Windows displayed:
- Ethernet
- Ethernet 2
- Ethernet 3
<img width="1426" height="1124" alt="Screenshot 2026-05-22 004135" src="https://github.com/user-attachments/assets/7a4920de-908c-41f6-a06c-4718170390ad" />

The accidental extra adapter was disabled.

---

## Ethernet


Purpose:

```text
Internet / NAT
```

Configuration:

```text
DHCP automatic
```

Expected IP:

```text
192.168.122.198
```

---

## Ethernet 2

Purpose:

```text
Internal isolated vSRO identity network
```

Manual IPv4 configuration:

```text
IP:
192.168.1.121

Mask:
255.255.255.0

Gateway:
empty

DNS:
empty
```

Important:

```text
No gateway
No DNS
```

because this NIC should NOT route internet traffic.

It only exists for:

```text
MachineManager certification identity.
```

---

# 18. Verifying Final Network State

Command:

```cmd
ipconfig
```
<img width="1235" height="741" alt="Screenshot 2026-05-22 004426" src="https://github.com/user-attachments/assets/84be0d2d-e27a-443b-8c24-49b77996f9e0" />

Purpose:
- display all active Windows network adapters and IP addresses

Expected result:

```text
Ethernet:
192.168.122.198

Ethernet 2:
192.168.1.121
```

Meaning:

```text
The VM keeps internet access through NAT,
while simultaneously satisfying the hardcoded runtime IP.
```

---

# 19. Final Successful Certification

Final startup order:

```text
1. CustomCertificationServer.exe packt.dat
2. GlobalManager.exe
3. MachineManager.exe
```

Expected MachineManager output:

```text
request server certification
successfully server certificated
server cord established
```

Expected GlobalManager output:

```text
Certification request from:
MachineManager[192.168.1.121]
```
<img width="1758" height="732" alt="Screenshot 2026-05-22 004614" src="https://github.com/user-attachments/assets/c98a15e9-7ebf-443d-be06-1b91bc6210fd" />

This proved:

```text
The hardcoded runtime IP problem was finally solved successfully.
```

---

# 20. Final Architecture

Final network design:

```text
Linux Host
│
├── NAT Network (virbr0)
│      └── Windows VM Ethernet
│              └── 192.168.122.198
│              └── Internet / NAT
│
└── Isolated vSRO Network (virbr1)
       └── Windows VM Ethernet 2
               └── 192.168.1.121
               └── internal certification identity
```

This architecture achieved:
- internet access
- VM isolation
- hardcoded IP compatibility
- stable certification
- safe infrastructure separation

without:
- dangerous binary repointing
- full bridge networking
- exposing the VM to the entire home LAN

---

# 21. Lessons Learned

## Understanding Runtime Identity vs Configuration

One of the most important discoveries in this phase was:

```text
A successful TCP connection does NOT automatically mean successful certification.
```

A service can:
- connect to GlobalManager
- open sockets correctly
- appear in topology

and still fail certification because:
- the runtime identity is invalid
- the reported IP does not match expectations
- the service body is rejected

---

## GlobalManager Topology Alone Is Not Enough

The topology GUI may still display services even when:
- certification failed
- sockets closed
- processes crashed

Real verification requires:
- tasklist
- netstat
- certification logs

---

## tasklist Verifies Processes

```cmd
tasklist | findstr Manager
```

proved whether services were actually running.

---

## netstat Verifies Real TCP State

```cmd
netstat -ano
```

became one of the most important infrastructure debugging tools.

It verified:
- ports
- sockets
- active TCP state
- established sessions

instead of relying on assumptions.

---

## findstr Can Reveal Hardcoded Runtime Data

```cmd
findstr /S /M "25.73.87.49" *.*
```

revealed hardcoded runtime IPs directly inside executables.

This became a major reverse engineering discovery.

---

## Inline Hex Editing Is Dangerous for Longer Strings

Replacing:

```text
192.168.1.121
```

with:

```text
192.168.122.198
```

was unsafe because:
- the replacement string was longer
- binary structures could shift
- offsets could break

---

## Infrastructure Design Is Often Better Than Binary Patching

Instead of:
- ASM patching
- executable repointing
- code cave injection

the cleaner solution became:

```text
better network architecture
```

using:
- virtualization
- isolated NICs
- separated responsibilities

---

## Multi-NIC VM Design Is Extremely Powerful

Using:
- one NIC for internet
- one NIC for isolated internal services

allowed:
- safe VM isolation
- hardcoded identity compatibility
- stable networking
- clean architecture

without exposing the VM directly to the home LAN.

---

## Systematic Debugging Always Wins

The final solution came from:
1. observing logs
2. validating processes
3. validating TCP state
4. validating runtime IPs
5. isolating variables
6. testing assumptions
7. designing proper infrastructure

instead of:
- random tutorials
- blind config edits
- unsafe binary modifications

This phase became one of the first true:

```text
infrastructure engineering and runtime debugging phases
```

inside the vSRO project.
