# Step 03 — Trunks & VTP

> ⏱ 2 lessons · 🎯 ministry requirements 22.6, 22.7, 22.8, 6 · 📁 start from: `my-work/step02.pkt` · 💾 save as: `my-work/step03.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 02 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## What you will have at the end
Building A gets its two **distribution** switches and a second access switch. Every access
switch has **two trunk uplinks** (one to each distribution switch). VLANs are created
**once** on DIST1 and copied automatically to the others by **VTP**.

```
        SWL3-HFA-A-DIST1 (3650)            SWL3-HFA-A-DIST2 (3650)
          Gi1/0/1   Gi1/0/2                  Gi1/0/1   Gi1/0/2
             │         ╲                      ╱         │
             │           ╲──────────╲  ╱────╱           │
             │                       ╳                   │
             │           ╱──────────╱  ╲────╲           │
          Gi0/1     Gi0/2                      Gi0/1     Gi0/2
        SW-HFA-A-ACC1 (2960)                SW-HFA-A-ACC2 (2960)
     ACC1 Gi0/1→DIST1 Gi1/0/1   ACC1 Gi0/2→DIST2 Gi1/0/1
     ACC2 Gi0/1→DIST1 Gi1/0/2   ACC2 Gi0/2→DIST2 Gi1/0/2
```
(The cable between DIST1 and DIST2 comes in step 04.)

---

## Part 1 — Understand it first (simple English)

### 1.1 The problem: VLANs on more than one switch
Engineering PCs sit on **ACC1 and ACC2**, and both switches connect to the distribution
switches. How do we carry 8 VLANs between switches? One cable per VLAN would need 8 cables
per uplink — impossible.

### 1.2 The solution: a trunk
A **trunk** is one cable that carries **many VLANs** at once.

Think of a highway between two office buildings. Every car (frame) gets a **coloured
sticker** with its room number (VLAN) when it enters the highway. At the other end, the
switch reads the sticker, sends the car into the right room, and removes the sticker.

- The sticker is called a **tag** (4 bytes, standard **802.1Q**).
- **Access port** = a door into one room → no sticker (the PC never sees tags).
- **Trunk port** = the highway → stickers on.

| | Access port | Trunk port |
|---|---|---|
| Carries | 1 VLAN | many VLANs |
| Connects to | PC, printer, AP | another switch / router |
| Tags | no | yes (except the native VLAN) |

### 1.3 Native VLAN — the cars without a sticker
On a trunk, one VLAN travels **without** a tag: the **native VLAN** (VLAN 1 by default).
Attackers can abuse this ("double tagging" — hiding a second sticker under the first to jump
into another VLAN). Our fix: native VLAN **998**, a VLAN nobody uses. Both ends of a trunk
**must** use the same native VLAN.

### 1.4 Allowed VLAN list
By default a trunk carries **all** VLANs. We list only those that are needed
(`10,20,30,99,110,120,998`): less useless traffic and fewer ways in for an attacker.

### 1.5 DTP — and why we turn it off
**DTP** lets two ports *negotiate* to become a trunk on their own. Convenient — but a laptop
can pretend to be a switch, negotiate a trunk and see every VLAN.
`switchport nonegotiate` turns DTP off: a trunk is a trunk only because **we** said so.

### 1.6 VTP — one VLAN list for the whole building
Creating the same 8 VLANs by hand on every switch is slow and error-prone. **VTP** (VLAN
Trunking Protocol) is like a head office that publishes the official list of rooms; every
branch office copies it automatically (the messages travel over the trunks).

| VTP mode | Role | Comparison |
|---|---|---|
| **Server** | Create / delete / rename VLANs here; sends the list | Head office |
| **Client** | Receives the list, can't change it | Branch office |
| **Transparent** | Ignores the list, keeps its own VLANs (passes messages on) | Independent shop |

Switches share a list only with the same **domain name** and **password**. Each Haifa
building is its own domain (`HFA-A`, `HFA-B`, `HFA-C`) because buildings will be joined by
**routed** links, which don't carry VLANs.

⚠️ **The revision-number danger.** Every change to the list adds 1 to its **revision
number**. A switch always accepts the list with the **highest** revision — even if that
list is empty or wrong. A classic disaster: someone plugs a lab switch (revision 50) into the
production network → its list overwrites everyone's → VLANs disappear → the company goes
offline. **Safe rule:** before joining a switch to a domain, set it to `vtp mode transparent`
(resets the revision to 0), then to `client`.

ACC1 already has a revision above 0 (you created 8 VLANs on it in step 02) — so we use the
safe rule on it.

### 1.7 Why every access switch has two uplinks
If an access switch had only one cable to one distribution switch, losing it would cut off
everyone on that switch. With **two uplinks to two different distribution switches**, either
can fail and users keep working (requirement 6).

Two uplinks create a **loop** (ACC1 → DIST1 → ACC2 → DIST2 → ACC1). A broadcast would circle
forever and kill the network. The **Spanning Tree Protocol** blocks one link automatically —
that's why one uplink light will stay **orange**. You'll control it in step 04.

### 1.8 Layer-3 switch (the 3650)
The distribution switches are **Catalyst 3650** — Layer-3 switches. Today we use them only as
switches; in step 05 they also become routers. In Packet Tracer the 3650 comes **without a
power supply** — you must add one or it stays dead.

---

## Part 2 — Prepare
1. **File → Open** → `my-work/step02.pkt` → **File → Save As** → `my-work/step03.pkt`.

---

## Part 3 — Build (click by click)

### 3.1 Add the two 3650 switches
1. **Network Devices → Switches** → drag a **3650-24PS** into Building A, above ACC1.
2. Drag a second **3650-24PS** next to it.
3. Rename them (**Config → Display Name**): `SWL3-HFA-A-DIST1` and `SWL3-HFA-A-DIST2`.

### 3.2 Give each 3650 a power supply (⚠️ don't skip)
For **each** 3650:
1. Click the switch → **Physical** tab.
2. Zoom in on the picture of the device (use the **Zoom In** button in that window).
   Look at the back: there is an **empty power-supply slot**.
3. In the **MODULES** list on the left, find **AC-POWER-SUPPLY**.
4. **Drag** it onto the empty slot. The switch picture changes and its lights come on.
5. Close the window.
(If the module doesn't drop in, make sure you dragged it exactly onto the empty slot.)

### 3.3 Add the second access switch and 2 PCs
1. Drag a **2960** next to ACC1 → rename `SW-HFA-A-ACC2`.
2. Drag 2 PCs below it → rename `PC-HFA-ENG-03` and `PC-HFA-FIN-03`.

### 3.4 Cable — uplinks (switch ↔ switch = **Copper Cross-Over**)
| Click first | Port | Click second | Port |
|---|---|---|---|
| SW-HFA-A-ACC1 | **GigabitEthernet0/1** | SWL3-HFA-A-DIST1 | **GigabitEthernet1/0/1** |
| SW-HFA-A-ACC1 | **GigabitEthernet0/2** | SWL3-HFA-A-DIST2 | **GigabitEthernet1/0/1** |
| SW-HFA-A-ACC2 | **GigabitEthernet0/1** | SWL3-HFA-A-DIST1 | **GigabitEthernet1/0/2** |
| SW-HFA-A-ACC2 | **GigabitEthernet0/2** | SWL3-HFA-A-DIST2 | **GigabitEthernet1/0/2** |

### 3.5 Cable — PCs (Copper **Straight-Through**)
| Device | Port | Switch port |
|---|---|---|
| PC-HFA-ENG-03 | FastEthernet0 | SW-HFA-A-ACC2 **Fa0/1** |
| PC-HFA-FIN-03 | FastEthernet0 | SW-HFA-A-ACC2 **Fa0/11** |

Press **Fast Forward Time** a few times. Most lights turn green; **one or more uplinks stay
orange** — that's Spanning Tree blocking the loop (section 1.7). Correct!

### 3.6 PC addresses (Desktop → IP Configuration → Static)
| PC | IP | Mask |
|---|---|---|
| PC-HFA-ENG-03 | 10.10.16.103 | 255.255.255.0 |
| PC-HFA-FIN-03 | 10.10.17.103 | 255.255.255.0 |

---

## Part 4 — Configure (type every line; `!` lines are comments)

### 4.1 SWL3-HFA-A-DIST1 — basics + VTP **server** + trunks
Click it → **CLI** → press Enter.
```
enable
configure terminal
! ---------- hardening (same as step 02) ----------
hostname SWL3-HFA-A-DIST1
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
line console 0
 password Intel#Con
 login
 logging synchronous
 exec-timeout 10 0
 exit

! ---------- VTP: this switch is the head office of building A ----------
vtp domain HFA-A
vtp version 2
vtp password Intel#Vtp
vtp mode server

! ---------- the VLAN list (created only here) ----------
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

! ---------- downlinks to ACC1 and ACC2 = trunks ----------
interface range gi1/0/1 - 2
 description TRUNK to access switch
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,99,110,120,998
 switchport nonegotiate
 no shutdown

! ---------- unused ports -> parking + off ----------
interface range gi1/0/3 - 22
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 end
copy running-config startup-config
```
(Ports Gi1/0/23-24 are left alone — they're used in step 04.)

> **3650 note:** the 3650 only speaks 802.1Q, so it doesn't need (or accept) the command
> `switchport trunk encapsulation dot1q`. If your PT asks for it, type it before `switchport mode trunk`.

### 4.2 SWL3-HFA-A-DIST2 — basics + VTP **client** + trunks
```
enable
configure terminal
hostname SWL3-HFA-A-DIST2
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
line console 0
 password Intel#Con
 login
 logging synchronous
 exec-timeout 10 0
 exit

! ---------- safe join: transparent first (revision -> 0), then client ----------
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
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 end
copy running-config startup-config
```
No `vlan` commands here — DIST2 receives the list from DIST1.

> You may see a warning like `% Access VLAN does not exist` on `switchport access vlan 999`.
> That's normal: DIST2 hasn't received the VLAN list yet (it arrives once the access switches'
> trunks are up, in 4.3–4.4). The setting is kept and starts working when VLAN 999 arrives.

### 4.3 SW-HFA-A-ACC1 — join VTP + trunk uplinks
ACC1 was configured in step 02. Log in (console password `Intel#Con`, enable `Intel#En4ble`):
```
enable
configure terminal
! ACC1 has revision > 0 from step 02 -> safe join
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
copy running-config startup-config
```

### 4.4 SW-HFA-A-ACC2 — everything (it's new)
```
enable
configure terminal
hostname SW-HFA-A-ACC2
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
line console 0
 password Intel#Con
 login
 logging synchronous
 exec-timeout 10 0
 exit

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
copy running-config startup-config
```
ACC2's port plan: same as ACC1, except Fa0/22-23 are reserved for the two WiFi access points.

### 4.5 What the new commands mean
| Command | In simple English |
|---|---|
| `vtp domain HFA-A` | "I belong to the building-A VLAN club" |
| `vtp password Intel#Vtp` | The club password — only members with it share lists |
| `vtp mode server` / `client` / `transparent` | Head office / branch office / independent |
| `vtp version 2` | Use VTP version 2 (all members must match) |
| `switchport mode trunk` | "This port is a highway — carry tagged VLANs" |
| `switchport trunk native vlan 998` | "Cars without a sticker belong to the unused VLAN 998" |
| `switchport trunk allowed vlan ...` | "Only these VLANs may use this highway" |
| `switchport nonegotiate` | "Don't negotiate with whoever is at the other end (DTP off)" |

---

## Part 5 — Check your work

### 5.1 VTP
On **SW-HFA-A-ACC2**:
```
show vtp status
```
Look for:
```
VTP Version capable             : 1 to 2
VTP version running             : 2
VTP Domain Name                 : HFA-A
VTP Operating Mode              : Client
Configuration Revision          : 16         <- your number may differ - it must be the SAME as on DIST1
```
Run `show vtp status` on DIST1 too: the **Configuration Revision** must be the same.

```
show vlan brief
```
on ACC2 → all 8 VLANs are there, although you never created them on ACC2. ✅ VTP works.

### 5.2 Trunks
On **SWL3-HFA-A-DIST1**:
```
show interfaces trunk
```
```
Port        Mode         Encapsulation  Status        Native vlan
Gig1/0/1    on           802.1q         trunking      998
Gig1/0/2    on           802.1q         trunking      998

Port        Vlans allowed on trunk
Gig1/0/1    10,20,30,99,110,120,998
...
```
✅ Both ports `trunking`, native 998, our allowed list.

### 5.3 Ping across the trunk
| From | Type | Expected | Why |
|---|---|---|---|
| PC-HFA-ENG-01 (on ACC1) | `ping 10.10.16.103` (ENG-03 on ACC2) | ✅ | VLAN 10 crosses the trunks via a distribution switch |
| PC-HFA-FIN-01 (ACC1) | `ping 10.10.17.103` (FIN-03 on ACC2) | ✅ | VLAN 20 too |
| PC-HFA-ENG-01 | `ping 10.10.17.103` | ❌ | still no router |

### 5.4 Watch VTP live
1. On **DIST1**: `configure terminal` → `vlan 50` → `name TEST` → `end`.
2. On **ACC2**: `show vlan brief` → VLAN 50 TEST is there within seconds.
3. On DIST1: `configure terminal` → `no vlan 50` → `end`. Check ACC2 again → gone.
4. Try on ACC2: `configure terminal` → `vlan 60` → you get
   `VTP VLAN configuration not allowed when device is in CLIENT mode.` ✅

### 5.5 See the tag (Simulation mode)
1. Click **Simulation** (bottom right). In the panel: **Edit Filters** → only **ICMP**.
2. **Add Simple PDU** (envelope) → click PC-HFA-ENG-01 → click PC-HFA-ENG-03.
3. Press **Capture / Forward** until the envelope is on an uplink cable. Click the envelope.
4. In **Outbound PDU Details** you see an **802.1Q** header with `VLAN: 10`. On the PC's own
   cable there is no such header. That's the sticker!
5. Back to **Realtime**.

---

## Part 6 — Save
`copy running-config startup-config` on all 4 switches (done above), then **File → Save**.

## Checkpoint
- [ ] DIST1 and DIST2 have power supplies and are on
- [ ] `show vtp status` — same domain and revision on all 4 switches; DIST1 server, others client
- [ ] `show interfaces trunk` — all uplinks trunking, native 998
- [ ] VLAN 10 and VLAN 20 pings work between ACC1 and ACC2

## If something doesn't work
| Problem | Fix |
|---|---|
| 3650 is dark, no ports | Power supply missing (3.2) |
| Uplink light red | Switch ↔ switch needs **Cross-Over**. Delete the cable and redo it |
| ACC2 doesn't learn VLANs | `show vtp status` on both: domain, password (`show vtp password`), version must match **exactly** (case-sensitive) |
| `%CDP-4-NATIVE_VLAN_MISMATCH` messages | One end of a trunk has native 998, the other doesn't. Fix the missing side |
| `show interfaces trunk` shows nothing | The port isn't a trunk on one side, or it's shut down |
| Ping ENG-01 → ENG-03 fails | Is VLAN 10 in the allowed list on **both** ends? Are the PCs in VLAN 10 (`show vlan brief`)? |

## Challenge
Run `show spanning-tree vlan 10` on ACC1. One of its two uplinks is **BLK** (blocking). Why
does the switch block a perfectly good cable? Which switch is the **root bridge** right now,
and why that one?

<details><summary>Answer</summary>

ACC1 → DIST1 → ACC2 → DIST2 → ACC1 is a **loop**. Without Spanning Tree, a broadcast would
circle forever (a "broadcast storm") and kill the network in seconds. STP blocks one port to
break the loop; if the active path fails, the blocked port takes over (redundancy).

The root is the switch with the **lowest bridge ID** = priority (default 32768 + VLAN number)
and then the **lowest MAC address**. All priorities are equal, so the lowest MAC wins —
maybe an access switch, which is a bad root (traffic would take strange paths). We fix it
in step 04 by choosing the root ourselves.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

| In the example | In your project — take it from |
|---|---|
| Native VLAN `998` | plan part D (your NATIVE VLAN) |
| Allowed VLAN list on the trunks | plan part D (only **your** VLANs of that building) |
| VTP domain `HFA-A` | your own domain name per building, e.g. `<SITE>-A` |
| VTP password `Intel#Vtp` | plan part F |
| Device names and uplink ports | plan parts B and D |

**Check:** `show interfaces trunk` lists your native VLAN and only your VLANs;
`show vtp status` shows your domain on every switch of the building.

💾 Save as `my-project/step03.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★2 What is 802.1Q and why is it used?**

<details><summary>Answer</summary>

The trunking standard: it inserts a 4-byte tag with the 12-bit VLAN ID into each Ethernet frame, so one link can carry many VLANs and the other side knows which VLAN each frame belongs to. It is used between switches and between a switch and a router (router-on-a-stick).

**Show it:** `show interfaces trunk`
</details>

**2. ◆ What is the native VLAN and why is yours an unused VLAN?**

<details><summary>Answer</summary>

Frames of the native VLAN cross the trunk **untagged**. If the native VLAN is a user VLAN, an attacker can double-tag frames and hop into another VLAN. An unused native VLAN with no ports and no IP blocks that.

**Show it:** `show interfaces trunk` → Native vlan column
</details>

**3. ◆ Why limit the allowed VLANs on a trunk?**

<details><summary>Answer</summary>

Only the VLANs that are needed cross the link: less unnecessary broadcast traffic, and a VLAN that shouldn't be in a building can't leak into it.

**Show it:** `show interfaces trunk` → "Vlans allowed on trunk"
</details>

**4. ◆ What is VTP? What are server, client and transparent?**

<details><summary>Answer</summary>

VTP shares the VLAN list between switches in one domain. Server: creates/deletes VLANs and advertises them. Client: learns VLANs from the server, cannot change them. Transparent: keeps its own VLANs and only passes advertisements on.

**Show it:** `show vtp status`
</details>

**5. ◆ What is the danger of VTP and how do you protect against it?**

<details><summary>Answer</summary>

A switch with a higher configuration revision number, added to the domain, can overwrite (even delete) the VLANs of every switch. Protection: a VTP password, the same domain name only where intended, and resetting the revision of a new switch before connecting it.

**Show it:** `show vtp status` → Configuration Revision
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

In a copy of your step-02 file: add the second access switch and the two distribution
switches, cable the uplinks, and type the trunk and VTP configuration from your plan, without
this page. Target: **20 minutes**. Then `show interfaces trunk` and `show vtp status` on every
switch, and explain the native VLAN and allowed list you chose.

---

## 👩‍🏫 Teacher notes
- **Lesson 1 (90 min):** trunks (draw the highway with coloured stickers), native VLAN,
  allowed list, DTP → build: 3650s + power supplies, cabling with the right cable types →
  configure DIST1.
- **Lesson 2 (90 min):** VTP with the head-office comparison + the revision-number story →
  configure DIST2, ACC1, ACC2 → checks 5.1–5.3 → live VTP demo (5.4) → tagging in Simulation
  mode (5.5) on the projector.
- **Common student questions**
  - *"Why don't PCs need to understand tags?"* — The access port adds/removes the tag; the PC
    only ever sees normal frames.
  - *"If VTP is dangerous, why use it?"* — It saves a lot of work. The danger is managed with
    passwords and the transparent→client rule. In Kiryat Gat (step 14) we deliberately use
    transparent mode — a design choice to discuss in the project book.
  - *"Why is one light orange?"* — Spanning Tree blocks the loop. It's protection, not a fault.
- **Typical errors:** forgetting the 3650 power supply; straight-through instead of
  cross-over between switches; VTP domain typo (`HFA-A` ≠ `hfa-a`); native VLAN set on only
  one side.
