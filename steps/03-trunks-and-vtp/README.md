# Step 03 — Trunks & VTP

## Goal
Building A gets its two distribution switches and a second access switch. Every access
switch has two **trunk** uplinks. VLANs are created **once** on DIST1 and spread
automatically by **VTP**.

Ministry requirements: **22.6, 22.7, 22.8, 6 (L2 redundancy)**

## Concepts
- **Trunk (802.1Q)** — one cable carries many VLANs; each frame gets a 4-byte tag with its
  VLAN ID. Frames of the **native VLAN** travel *untagged*. We make the native VLAN 998
  (unused) so an attacker can't use "double tagging" to hop into VLAN 1.
- **Allowed VLAN list** — only carry the VLANs you need. Less broadcast, less attack surface.
- **`switchport nonegotiate`** — turns off DTP; the port will never negotiate on its own.
- **VTP** — VLAN database replication.
  - *Server*: you create/delete VLANs here.
  - *Client*: learns VLANs, can't edit them.
  - *Transparent*: ignores VTP, keeps its own VLANs (forwards VTP ads).
  - ⚠️ **Revision number**: the switch with the highest revision wins and overwrites
    everyone. A lab switch with revision 50 plugged into production can delete all VLANs
    in the company. Safe rule: before joining a switch to a domain, set it to
    `transparent` (resets revision to 0), then `client`.
- Each Haifa building is its own VTP domain (`HFA-A`, `HFA-B`, `HFA-C`) because the
  buildings are connected by **routed** links — VLANs don't cross between them.

## Build

| Device | Name | Note |
|---|---|---|
| 3650-24PS | `SWL3-HFA-A-DIST1` | ⚠️ add **AC-POWER-SUPPLY** in the Physical tab |
| 3650-24PS | `SWL3-HFA-A-DIST2` | ⚠️ power supply |
| 2960-24TT | `SW-HFA-A-ACC2` | |
| 2 PCs | `PC-HFA-ENG-03` (ACC2 Fa0/1), `PC-HFA-FIN-03` (ACC2 Fa0/11) | |

| From | To |
|---|---|
| ACC1 Gi0/1 | DIST1 Gi1/0/1 |
| ACC1 Gi0/2 | DIST2 Gi1/0/1 |
| ACC2 Gi0/1 | DIST1 Gi1/0/2 |
| ACC2 Gi0/2 | DIST2 Gi1/0/2 |

(DIST1 ↔ DIST2 comes in step 04 as an EtherChannel.)

## Configure

### 1. Basic hardening on DIST1, DIST2, ACC2
Repeat the "identity & basic hardening" block from step 02 (hostname, `no ip domain-lookup`,
`enable secret`, `service password-encryption`, banner, console). **Type it again** — by
the third device you will know it by heart.

### 2. SWL3-HFA-A-DIST1 — VTP server, owns the VLANs
```
configure terminal
vtp domain HFA-A
vtp version 2
vtp password Intel#Vtp
vtp mode server

vlan 10
 name ENGINEERING
vlan 20
 name FINANCE
vlan 30
 name PRINTERS-A
vlan 99
 name MGMT-A
vlan 110
 name WIFI-EMP
vlan 120
 name WIFI-GUEST
vlan 998
 name NATIVE
vlan 999
 name PARKING
 exit

! downlinks to the access switches
interface range gi1/0/1 - 2
 description TRUNK to access switch
 ! 3650 is dot1q-only: no "encapsulation" command needed (see pt-gotchas)
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,99,110,120,998
 switchport nonegotiate
 no shutdown

! unused ports -> parking
interface range gi1/0/3 - 22
 switchport mode access
 switchport access vlan 999
 shutdown
end
wr
```

### 3. SWL3-HFA-A-DIST2 — VTP client
```
configure terminal
vtp mode transparent
vtp domain HFA-A
vtp version 2
vtp password Intel#Vtp
vtp mode client

interface range gi1/0/1 - 2
 description TRUNK to access switch
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,99,110,120,998
 switchport nonegotiate
 no shutdown
interface range gi1/0/3 - 22
 switchport mode access
 switchport access vlan 999
 shutdown
end
wr
```

### 4. SW-HFA-A-ACC1 and SW-HFA-A-ACC2 — VTP clients, trunk uplinks
ACC1 already has VLANs from step 02 (its revision number is > 0!). The `transparent`
line resets it to 0 so it can't overwrite DIST1.

```
configure terminal
vtp mode transparent
vtp domain HFA-A
vtp version 2
vtp password Intel#Vtp
vtp mode client

interface range gi0/1 - 2
 description TRUNK uplink to DIST1 / DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,99,110,120,998
 switchport nonegotiate
 no shutdown
end
wr
```

On **ACC2** also configure the access ports (port plan from step 02):
```
configure terminal
interface range fa0/1 - 10
 description ENGINEERING users
 switchport mode access
 switchport access vlan 10
interface range fa0/11 - 20
 description FINANCE users
 switchport mode access
 switchport access vlan 20
interface fa0/21
 description printer
 switchport mode access
 switchport access vlan 30
interface fa0/22
 description AP-HFA-A-EMP (step 10)
 switchport mode access
 switchport access vlan 110
interface fa0/23
 description AP-HFA-A-GUEST (step 10)
 switchport mode access
 switchport access vlan 120
interface fa0/24
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
end
wr
```
PCs: `PC-HFA-ENG-03` = 10.10.16.103/24, `PC-HFA-FIN-03` = 10.10.17.103/24.

## Verify

```
SW-HFA-A-ACC2# show vtp status
```
Expected: `VTP Operating Mode : Client`, `VTP Domain Name : HFA-A`, and the **same
configuration revision** as DIST1.

```
SW-HFA-A-ACC2# show vlan brief           <- all 8 VLANs, learned from DIST1
SWL3-HFA-A-DIST1# show interfaces trunk  <- Gi1/0/1-2 trunking, native 998
```

| Test | Expected |
|---|---|
| ENG-01 (ACC1) → `ping 10.10.16.103` (ENG-03 on ACC2) | ✅ — VLAN 10 crosses the trunk |
| ENG-01 → `ping 10.10.17.103` | ❌ — still no router |

**VTP experiment**: on DIST1 create `vlan 50` / `name TEST`. Within seconds
`show vlan brief` on ACC2 shows it. Now delete it on DIST1 (`no vlan 50`). Gone everywhere.

## Checkpoint
`my-work/step03.pkt` + commit.

## Common mistakes
- **Native VLAN mismatch** (998 on one side, 1 on the other) → CDP prints
  `%CDP-4-NATIVE_VLAN_MISMATCH` and traffic leaks between VLANs. Both ends must match.
- VTP password or domain typo → client never learns VLANs. `show vtp password` on both.
- `vtp version 2` on the server but not the client — keep them equal.

## Challenge
Run `show spanning-tree vlan 10` on ACC1. One of its two uplinks is **BLK** (blocking). Why
does the switch block a perfectly good cable? Which switch is the root bridge right now,
and why that one?

<details><summary>Answer</summary>

ACC1 → DIST1 → ACC2 → DIST2 → ACC1 is a **loop**. Without STP, a broadcast would circle
forever (broadcast storm) and kill the network in seconds. STP blocks one port to break
the loop; if the active path fails, the blocked port takes over (redundancy).

The root is the switch with the **lowest bridge ID** = priority (default 32768 + VLAN ID)
and then **lowest MAC address**. Since all priorities are equal, the oldest/lowest MAC
wins — probably an access switch, which is a terrible root. We fix this in step 04.
</details>
