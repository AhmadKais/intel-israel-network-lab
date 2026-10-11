# Step 03 — Trunks & VTP

## Goal
Building A gets its two distribution switches and a second access switch. Every access
switch has two **trunk** uplinks. VLANs are created **once** on DIST1 and spread
automatically by **VTP**.

Ministry requirements: **22.6, 22.7, 22.8, 6 (L2 redundancy)**

## Understand it first (simple English)

### 1. The problem: VLANs on more than one switch
In step 02 every VLAN lived on one switch. But Engineering PCs sit on **ACC1 and ACC2**, and
both switches connect to the distribution switches. How do we carry 6 VLANs between
switches? One cable per VLAN would need 6 cables per uplink — impossible at scale.

### 2. The solution: a trunk
A **trunk** is one cable that carries **many VLANs** at once.

Think of a highway between two office buildings. Every car (frame) gets a **coloured
sticker** with its room number (VLAN) when it enters the highway. At the other end, the
switch reads the sticker and sends the car into the right room, then removes the sticker.

- The sticker is called a **tag** (4 bytes, standard **802.1Q**).
- **Access port** = a door into one room → no sticker (the PC never sees tags).
- **Trunk port** = the highway → stickers on.

| | Access port | Trunk port |
|---|---|---|
| Carries | 1 VLAN | many VLANs |
| Connects to | PC, printer, AP | another switch / router |
| Tags | no | yes (except the native VLAN) |

### 3. Native VLAN — the cars without a sticker
On a trunk, one VLAN travels **without** a tag: the **native VLAN** (VLAN 1 by default).
Attackers can abuse this ("double tagging" — hiding a second sticker under the first to
jump into another VLAN). Our fix: make the native VLAN **998**, a VLAN nobody uses.
Both ends of the trunk **must** agree on the native VLAN.

### 4. Allowed VLAN list
By default a trunk carries **all** VLANs. We list only the ones that are needed
(`10,20,30,99,110,120,998`). Less useless broadcast traffic, and fewer ways in for an
attacker.

### 5. DTP — and why we turn it off
**DTP** lets two switch ports *negotiate* to become a trunk on their own. Convenient — but a
laptop can pretend to be a switch and negotiate a trunk, then see every VLAN.
`switchport nonegotiate` turns DTP off: trunks are only trunks because **we** said so.

### 6. VTP — one VLAN list for the whole building
Creating the same 8 VLANs by hand on every switch is slow and error-prone. **VTP** (VLAN
Trunking Protocol) is like a head office that publishes the official list of rooms, and
every branch office copies it automatically.

| VTP mode | Role | Comparison |
|---|---|---|
| **Server** | Create / delete / rename VLANs here; sends the list out | Head office |
| **Client** | Receives the list, can't change it | Branch office |
| **Transparent** | Ignores the list, keeps its own VLANs (just passes the messages on) | An independent shop |

Switches only share a list if they have the same **domain name** and **password**.

⚠️ **The revision number danger.** Every change to the list adds 1 to its **revision
number**. A switch always accepts the list with the **highest** revision — even if that list
is empty or wrong. Real story from many companies: someone takes a lab switch (revision 50)
and plugs it into the production network → its list overwrites everyone's → all VLANs
disappear → the whole company goes offline.
**Safe rule:** before joining any switch to a domain, set it to `vtp mode transparent`
(this resets the revision to 0), then to `client`.

### 7. Why every access switch has two uplinks
If an access switch had one cable to one distribution switch, losing that cable or that
switch would cut off everyone on it. With **two uplinks to two different distribution
switches**, either can fail and users keep working (requirement 6).

Two uplinks create a **loop** (ACC1 → DIST1 → ACC2 → DIST2 → ACC1). Loops are deadly in
switching — a broadcast would go round forever. A protocol called **Spanning Tree** blocks
one link automatically. You'll see it in this step's challenge and control it in step 04.

---

### Short technical summary
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

### What the new commands mean
| Command | In simple English |
|---|---|
| `vtp domain HFA-A` | "I belong to the building-A VLAN club" |
| `vtp password Intel#Vtp` | The club password — only members with it share lists |
| `vtp mode server` / `client` / `transparent` | Head office / branch office / independent (table above) |
| `switchport mode trunk` | "This port is a highway — carry tagged VLANs" |
| `switchport trunk native vlan 998` | "Cars without a sticker belong to the unused VLAN 998" |
| `switchport trunk allowed vlan ...` | "Only these VLANs may use this highway" |
| `switchport nonegotiate` | "Don't negotiate with whoever is at the other end (DTP off)" |

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

---

## 👩‍🏫 Teacher notes
- **Lesson plan (~2 × 90 min):** lesson 1 = trunks (draw the highway with coloured stickers),
  native VLAN, allowed list, DTP; students cable and configure trunks. Lesson 2 = VTP with
  the head-office comparison, the revision-number story, the VTP experiment
  (create/delete VLAN 50 on DIST1 and watch it appear/disappear on ACC2).
- **Show tagging live:** Simulation mode → ping ENG-01 → ENG-03 → click the frame on the
  trunk cable → the 802.1Q header shows `VLAN: 10`. On the PC's cable there is no tag.
- **Common student questions**
  - *"Why don't PCs need to understand tags?"* — The access port adds/removes the tag; the
    PC only ever sees normal frames.
  - *"If VTP is dangerous, why use it?"* — It saves a lot of work in a big network. The danger
    is managed with passwords, the transparent→client rule, and in Kiryat Gat we
    deliberately use transparent mode (a design choice to discuss in the book).
  - *"Why does `show interfaces trunk` not show my trunk?"* — The other end isn't a trunk
    yet, or the port is down. Both ends must be configured.
- **Typical errors:** native VLAN mismatch (CDP warning in the console), forgetting the
  power supply on the 3650s, VTP domain typo (case-sensitive: `HFA-A` ≠ `hfa-a`).
