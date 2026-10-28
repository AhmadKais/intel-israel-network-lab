# Step 09 — Buildings B & C + the OSPF virtual link

> ⏱ 4 lessons · 🎯 ministry requirements 8, 22.16, 22.15, 20.1, 20.2 · 📁 start from: `my-work/step08.pkt` · 💾 save as: `my-work/step09.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 08 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## 🗺 Your missions in this step
**Lesson 1 — build and cable**
- [ ] **Mission 1** — Open step 08 and save as step 09 (2 min)
- [ ] **Mission 2** — Place building B and building C (with 4 power supplies!) (20 min)
- [ ] **Mission 3** — Cable building B (20 min)
- [ ] **Mission 4** — Cable building C (20 min)

**Lesson 2 — building B**
- [ ] **Mission 5** — B-DIST1, part 1: hardening, VTP, VLANs, STP, trunks, EtherChannel (15 min)
- [ ] **Mission 6** — B-DIST1, part 2: gateways, DHCP, uplinks, OSPF, management (20 min)
- [ ] **Mission 7** — B-DIST2: the mirror of B-DIST1 (25 min)
- [ ] **Mission 8** — B-ACC1 and B-ACC2 (25 min)
- [ ] **Mission 9** — Open the core ports toward building B (10 min)
- [ ] **Mission 10** — Building B end devices + check building B (10 min)

**Lesson 3 — building C**
- [ ] **Mission 11** — C-DIST1 (30 min)
- [ ] **Mission 12** — C-DIST2 (25 min)
- [ ] **Mission 13** — C-ACC1 and C-ACC2 (25 min)
- [ ] **Mission 14** — Building C end devices + **see the problem** 🔍 (10 min)

**Lesson 4 — the fix and the finish**
- [ ] **Mission 15** — The virtual links (10 min)
- [ ] **Mission 16** — Update the 7 older devices' guest list (15 min)
- [ ] **Mission 17** — Register the 8 new devices on the AAA server (10 min)
- [ ] **Mission 18** — Final checks (15 min)
- [ ] **Exercises** A and B
- [ ] **🛠 Your own project**

## What you will have at the end
Haifa HQ's LAN is **complete**: 3 buildings, 6 departments, 3-tier design, OSPF areas
0/1/2/3. Building C is connected only to building B, so you'll meet a real OSPF problem and
solve it with a **virtual link**.

```
                       CORE1 ═══════ CORE2                        AREA 0
                    ╱    │    ╲    ╱   │    ╲
       A-DIST1/2 (area 1)      B-DIST1 ═ B-DIST2 (ABRs 0/2)       AREA 2 links
                                 │   ╲  ╱   │
                                 │    ╳     │         10.10.36.x/30
                                 │   ╱  ╲   │
                              C-DIST1 ═ C-DIST2 (ABRs 2/3)        AREA 3
                    ······ virtual links B-DIST ⇄ C-DIST through area 2 ······
```

> **This step is the big revision of steps 02–08.** Every block you type here, you have typed
> before. Each mission says which step it comes from. **🧩 Want a challenge?** Before you open a
> block, try to write it yourself from the tables in Part 1, then compare.

---

## Part 1 — Understand it first (simple English)

### 1.1 Same recipe, new buildings
Building B and C follow exactly the recipe of building A:

| Layer | Building A | Building B | Building C |
|---|---|---|---|
| VTP domain | HFA-A | **HFA-B** | **HFA-C** |
| Departments | Engineering (10), Finance (20) | **HR (40), Marketing (50)** | **R&D (70), IT (80)** |
| Printers VLAN | 30 | **60** | **90** |
| Management VLAN 99 | 10.10.19.0/24 | **10.10.35.0/24** | **10.10.51.0/24** |
| OSPF area | 1 (10.10.16.0/20) | **2 (10.10.32.0/20)** | **3 (10.10.48.0/20)** |
| Uplinks go to | the core | the core | **building B** ⚠️ |

VLAN 99 has the same **number** in every building but a **different subnet** — that's fine,
because buildings are joined by routed links: VLANs don't cross between buildings.

### 1.2 The problem: building C doesn't touch area 0
OSPF has a strict rule: **every area must connect to area 0** (the backbone). Routes between
areas are only passed *through* area 0 — like cities that may only trade through the national
highway.

Building C (area 3) is connected only to building B's area 2. So:
- C-DIST1/2 have no foot in area 0 → they can't pass area 3's routes to the backbone;
- building C becomes an **island**: nobody outside area 2 can reach it, and it can't reach the servers.

You'll **see** this happen first (Mission 14), then fix it (Mission 15).

### 1.3 The fix: a virtual link
A **virtual link** is a logical piece of area 0, "tunnelled" through another area (the
**transit** area — here area 2), between two ABRs:
- one end on a router that sits in **area 0 + area 2** → B-DIST1/2 (router-IDs 10.10.5.21 / .22)
- the other end on a router that sits in **area 2 + area 3** → C-DIST1/2 (router-IDs 10.10.63.1 / .2)

Comparison: building C's town has no highway exit, so we sign an agreement with the
neighbouring town to treat one of its roads **as if** it were highway.

The command uses the **router-ID** of the other end — not an interface address:
`area 2 virtual-link 10.10.63.1`. We build all 4 combinations (B1↔C1, B1↔C2, B2↔C1, B2↔C2),
so losing any one switch keeps area 3 connected.

> Requirement 22.16 asks for exactly this: "multi-area OSPF in one branch including a structure
> that requires a virtual link".

### 1.4 Why the B↔C links are numbered 10.10.36.x
They belong to area 2, so they get addresses **inside area 2's block** (10.10.32.0/20). Then
B's summary `10.10.32.0/20` covers them too and they don't leak into the core as extra routes.
Building C's loopbacks (10.10.63.x) are inside **area 3's** block for the same reason.

### 1.5 The address plan of this step
Keep this open — every mission takes its numbers from here.

| Link | Subnet | Side 1 | Side 2 | Area |
|---|---|---|---|---|
| B-DIST1 Gi1/1/1 ↔ CORE1 Gi1/0/3 | 10.10.4.16/30 | B-DIST1 .17 | CORE1 .18 | 0 |
| B-DIST1 Gi1/1/2 ↔ CORE2 Gi1/0/3 | 10.10.4.20/30 | B-DIST1 .21 | CORE2 .22 | 0 |
| B-DIST2 Gi1/1/1 ↔ CORE1 Gi1/0/4 | 10.10.4.24/30 | B-DIST2 .25 | CORE1 .26 | 0 |
| B-DIST2 Gi1/1/2 ↔ CORE2 Gi1/0/4 | 10.10.4.28/30 | B-DIST2 .29 | CORE2 .30 | 0 |
| B-DIST1 Gi1/1/3 ↔ C-DIST1 Gi1/1/1 | 10.10.36.0/30 | B-DIST1 .1 | C-DIST1 .2 | 2 |
| B-DIST2 Gi1/1/3 ↔ C-DIST1 Gi1/1/2 | 10.10.36.4/30 | B-DIST2 .5 | C-DIST1 .6 | 2 |
| B-DIST1 Gi1/1/4 ↔ C-DIST2 Gi1/1/1 | 10.10.36.8/30 | B-DIST1 .9 | C-DIST2 .10 | 2 |
| B-DIST2 Gi1/1/4 ↔ C-DIST2 Gi1/1/2 | 10.10.36.12/30 | B-DIST2 .13 | C-DIST2 .14 | 2 |

| Device | Loopback0 / router-ID | Mgmt (VLAN 99) |
|---|---|---|
| SWL3-HFA-B-DIST1 / DIST2 | 10.10.5.21 / 10.10.5.22 | 10.10.35.2 / .3 (VIP .1) |
| SW-HFA-B-ACC1 / ACC2 | — | 10.10.35.11 / .12 |
| SWL3-HFA-C-DIST1 / DIST2 | 10.10.63.1 / 10.10.63.2 | 10.10.51.2 / .3 (VIP .1) |
| SW-HFA-C-ACC1 / ACC2 | — | 10.10.51.11 / .12 |

| VLAN | Name | Subnet | Primary (STP root + HSRP active) |
|---|---|---|---|
| 40 | HR | 10.10.32.0/24 | B-DIST1 |
| 50 | MARKETING | 10.10.33.0/24 | B-DIST2 |
| 60 | PRINTERS-B | 10.10.34.0/24 | B-DIST1 |
| 99 | MGMT-B | 10.10.35.0/24 | B-DIST1 |
| 70 | RND | 10.10.48.0/24 | C-DIST1 |
| 80 | IT | 10.10.49.0/24 | C-DIST2 |
| 90 | PRINTERS-C | 10.10.50.0/24 | C-DIST1 |
| 99 | MGMT-C | 10.10.51.0/24 | C-DIST1 |

### 1.6 Two blocks you will type many times
The **management block** is the same on every new switch — SSH (step 07) + the final guest
list + Syslog + AAA (step 08). From now on, network devices may be managed from **all three
MGMT VLANs** and the **IT department**.

**🅼 Management block — for the 4 Layer-3 switches** (B-DIST1/2, C-DIST1/2):
```
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
→ answer **`1024`** → then:
```
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
login block-for 120 attempts 3 within 60
ip access-list standard MGMT-ONLY
 permit 10.10.19.0 0.0.0.255
 permit 10.10.35.0 0.0.0.255
 permit 10.10.51.0 0.0.0.255
 permit 10.10.49.0 0.0.0.255
 deny any log
 exit
service timestamps log datetime msec
logging trap informational
logging host 10.10.1.10
aaa new-model
tacacs-server host 10.10.1.11 key Intel#Tac
aaa authentication login default group tacacs+ local
ip tacacs source-interface loopback 0
line vty 0 15
 transport input ssh
 login authentication default
 access-class MGMT-ONLY in
 exec-timeout 5 0
line console 0
 login authentication default
 logging synchronous
 end
copy running-config startup-config
```

**🅰 Management block — for the 4 access switches** (B-ACC1/2, C-ACC1/2): exactly the same,
**without** the line `ip tacacs source-interface loopback 0` (access switches have no loopback).

> As always: if `login block-for ...` or `ip tacacs source-interface` is rejected, skip that one
> line (see the notes in steps 07–08).

---

## Mission 1 — Open step 08 and save as step 09 (how? → M1)
**File → Open** → `my-work/step08.pkt` → **File → Save As** → `my-work/step09.pkt`.

---

## Mission 2 — Place building B and building C

### 2a — Building B (inside the "BUILDING B" rectangle — how? → M2, M3, M4)
| Model | Display Name |
|---|---|
| 3650-24PS | `SWL3-HFA-B-DIST1`, `SWL3-HFA-B-DIST2` |
| 2960 | `SW-HFA-B-ACC1`, `SW-HFA-B-ACC2` |
| PC ×4 | `PC-HFA-HR-01`, `PC-HFA-HR-02`, `PC-HFA-MKT-01`, `PC-HFA-MKT-02` |
| Printer | `PRN-HFA-B-01` |

### 2b — Building C (inside the "BUILDING C" rectangle)
| Model | Display Name |
|---|---|
| 3650-24PS | `SWL3-HFA-C-DIST1`, `SWL3-HFA-C-DIST2` |
| 2960 | `SW-HFA-C-ACC1`, `SW-HFA-C-ACC2` |
| PC ×4 | `PC-HFA-RND-01`, `PC-HFA-RND-02`, `PC-HFA-IT-01`, `PC-HFA-IT-02` |
| Printer | `PRN-HFA-C-01` |

### 2c — Power supplies on all **4** new 3650s (⚠️ how? → M8)
Each 3650 → **Physical** → drag **AC-POWER-SUPPLY** into the empty slot → close. Tick:
- [ ] SWL3-HFA-B-DIST1   - [ ] SWL3-HFA-B-DIST2   - [ ] SWL3-HFA-C-DIST1   - [ ] SWL3-HFA-C-DIST2

✅ 18 new devices, all named, 4 green-lit 3650s.

---

## Mission 3 — Cable building B (how? → M5)
Switch ↔ switch = **Cross-Over**; end device ↔ switch = **Straight-Through**.

| # | Cable | Click first | Port | Click second | Port |
|---|---|---|---|---|---|
| 1 | Cross-Over | SW-HFA-B-ACC1 | Gi0/1 | SWL3-HFA-B-DIST1 | Gi1/0/1 |
| 2 | Cross-Over | SW-HFA-B-ACC1 | Gi0/2 | SWL3-HFA-B-DIST2 | Gi1/0/1 |
| 3 | Cross-Over | SW-HFA-B-ACC2 | Gi0/1 | SWL3-HFA-B-DIST1 | Gi1/0/2 |
| 4 | Cross-Over | SW-HFA-B-ACC2 | Gi0/2 | SWL3-HFA-B-DIST2 | Gi1/0/2 |
| 5 | Cross-Over | SWL3-HFA-B-DIST1 | Gi1/0/23 | SWL3-HFA-B-DIST2 | Gi1/0/23 |
| 6 | Cross-Over | SWL3-HFA-B-DIST1 | Gi1/0/24 | SWL3-HFA-B-DIST2 | Gi1/0/24 |
| 7 | Cross-Over | SWL3-HFA-B-DIST1 | **Gi1/1/1** | SWL3-HFA-CORE1 | **Gi1/0/3** |
| 8 | Cross-Over | SWL3-HFA-B-DIST1 | **Gi1/1/2** | SWL3-HFA-CORE2 | **Gi1/0/3** |
| 9 | Cross-Over | SWL3-HFA-B-DIST2 | **Gi1/1/1** | SWL3-HFA-CORE1 | **Gi1/0/4** |
| 10 | Cross-Over | SWL3-HFA-B-DIST2 | **Gi1/1/2** | SWL3-HFA-CORE2 | **Gi1/0/4** |
| 11 | Straight | PC-HFA-HR-01 | Fa0 | SW-HFA-B-ACC1 | Fa0/1 |
| 12 | Straight | PC-HFA-HR-02 | Fa0 | SW-HFA-B-ACC2 | Fa0/1 |
| 13 | Straight | PC-HFA-MKT-01 | Fa0 | SW-HFA-B-ACC1 | Fa0/11 |
| 14 | Straight | PC-HFA-MKT-02 | Fa0 | SW-HFA-B-ACC2 | Fa0/11 |
| 15 | Straight | PRN-HFA-B-01 | Fa0 | SW-HFA-B-ACC1 | Fa0/21 |

> If PT refuses copper on the 3650's `Gi1/1/x` ports, use the orange **Fiber** cable (as in step 08).

✅ 15 cables. Check every label (M6). Tick off each row as you go — it's easy to skip one.

---

## Mission 4 — Cable building C (its uplinks go to **building B**)
| # | Cable | Click first | Port | Click second | Port |
|---|---|---|---|---|---|
| 1 | Cross-Over | SW-HFA-C-ACC1 | Gi0/1 | SWL3-HFA-C-DIST1 | Gi1/0/1 |
| 2 | Cross-Over | SW-HFA-C-ACC1 | Gi0/2 | SWL3-HFA-C-DIST2 | Gi1/0/1 |
| 3 | Cross-Over | SW-HFA-C-ACC2 | Gi0/1 | SWL3-HFA-C-DIST1 | Gi1/0/2 |
| 4 | Cross-Over | SW-HFA-C-ACC2 | Gi0/2 | SWL3-HFA-C-DIST2 | Gi1/0/2 |
| 5 | Cross-Over | SWL3-HFA-C-DIST1 | Gi1/0/23 | SWL3-HFA-C-DIST2 | Gi1/0/23 |
| 6 | Cross-Over | SWL3-HFA-C-DIST1 | Gi1/0/24 | SWL3-HFA-C-DIST2 | Gi1/0/24 |
| 7 | Cross-Over | SWL3-HFA-C-DIST1 | **Gi1/1/1** | SWL3-HFA-B-DIST1 | **Gi1/1/3** |
| 8 | Cross-Over | SWL3-HFA-C-DIST1 | **Gi1/1/2** | SWL3-HFA-B-DIST2 | **Gi1/1/3** |
| 9 | Cross-Over | SWL3-HFA-C-DIST2 | **Gi1/1/1** | SWL3-HFA-B-DIST1 | **Gi1/1/4** |
| 10 | Cross-Over | SWL3-HFA-C-DIST2 | **Gi1/1/2** | SWL3-HFA-B-DIST2 | **Gi1/1/4** |
| 11 | Straight | PC-HFA-RND-01 | Fa0 | SW-HFA-C-ACC1 | Fa0/1 |
| 12 | Straight | PC-HFA-RND-02 | Fa0 | SW-HFA-C-ACC2 | Fa0/1 |
| 13 | Straight | PC-HFA-IT-01 | Fa0 | SW-HFA-C-ACC1 | Fa0/11 |
| 14 | Straight | PC-HFA-IT-02 | Fa0 | SW-HFA-C-ACC2 | Fa0/11 |
| 15 | Straight | PRN-HFA-C-01 | Fa0 | SW-HFA-C-ACC1 | Fa0/21 |

✅ 15 cables, labels checked. **File → Save**.

---

## Mission 5 — B-DIST1, part 1: hardening, VTP, VLANs, STP, trunks, EtherChannel
👉 **Where:** **SWL3-HFA-B-DIST1** → CLI → **Enter** → `Switch>` (new switch, no passwords yet).

### 5a — Hardening (step 02)
⌨️ Type:
```
enable
configure terminal
hostname SWL3-HFA-B-DIST1
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
```

### 5b — VTP server of building B + VLANs (step 03)
⌨️ Type:
```
vtp domain HFA-B
vtp version 2
vtp password Intel#Vtp
vtp mode server
vlan 40
 name HR
vlan 50
 name MARKETING
vlan 60
 name PRINTERS-B
vlan 99
 name MGMT-B
vlan 998
 name NATIVE
vlan 999
 name PARKING
 exit
```

### 5c — Spanning Tree, trunks, EtherChannel, unused ports (steps 03–04)
B-DIST1 is boss of 40, 60, 99; deputy of 50.

⌨️ Type:
```
spanning-tree mode rapid-pvst
spanning-tree vlan 40,60,99 root primary
spanning-tree vlan 50 root secondary
interface range gi1/0/1 - 2
 description TRUNK to access switch
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 40,50,60,99,998
 switchport nonegotiate
 spanning-tree guard root
 no shutdown
interface range gi1/0/23 - 24
 description Po1 member to SWL3-HFA-B-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 40,50,60,99,998
 switchport nonegotiate
 channel-group 1 mode active
 no shutdown
interface port-channel 1
 description EtherChannel to SWL3-HFA-B-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 40,50,60,99,998
 switchport nonegotiate
interface range gi1/0/3 - 22
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
```
🔍 **Check now:** `do show vlan brief` → VLANs 40, 50, 60, 99, 998, 999 with their names.

---

## Mission 6 — B-DIST1, part 2: gateways, DHCP, uplinks, OSPF, management
👉 **Where:** B-DIST1, prompt `SWL3-HFA-B-DIST1(config)#`.

### 6a — Gateways: SVIs + HSRP (step 05)
Priority **110** on 40, 60, 99 (B-DIST1 is Active); **90** on 50.

⌨️ Type:
```
ip routing
interface vlan 40
 description GW HR
 ip address 10.10.32.2 255.255.255.0
 standby version 2
 standby 40 ip 10.10.32.1
 standby 40 priority 110
 standby 40 preempt
 no shutdown
interface vlan 50
 description GW MARKETING
 ip address 10.10.33.2 255.255.255.0
 standby version 2
 standby 50 ip 10.10.33.1
 standby 50 priority 90
 standby 50 preempt
 no shutdown
interface vlan 60
 description GW PRINTERS-B
 ip address 10.10.34.2 255.255.255.0
 standby version 2
 standby 60 ip 10.10.34.1
 standby 60 priority 110
 standby 60 preempt
 no shutdown
interface vlan 99
 description GW MGMT-B
 ip address 10.10.35.2 255.255.255.0
 standby version 2
 standby 99 ip 10.10.35.1
 standby 99 priority 110
 standby 99 preempt
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
 exit
```

### 6b — DHCP split scope: this switch gives .11–.130 (step 05)
⌨️ Type:
```
ip dhcp excluded-address 10.10.32.1 10.10.32.10
ip dhcp excluded-address 10.10.32.131 10.10.32.254
ip dhcp excluded-address 10.10.33.1 10.10.33.10
ip dhcp excluded-address 10.10.33.131 10.10.33.254
ip dhcp pool HFA-HR
 network 10.10.32.0 255.255.255.0
 default-router 10.10.32.1
 dns-server 10.10.200.11
 domain-name intel.lab
ip dhcp pool HFA-MARKETING
 network 10.10.33.0 255.255.255.0
 default-router 10.10.33.1
 dns-server 10.10.200.11
 domain-name intel.lab
 exit
```

### 6c — Routed uplinks: to the core (area 0) and to building C (area 2) (step 08)
⌨️ Type:
```
interface loopback 0
 ip address 10.10.5.21 255.255.255.255
interface gi1/1/1
 description to SWL3-HFA-CORE1 Gi1/0/3
 no switchport
 ip address 10.10.4.17 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/1/2
 description to SWL3-HFA-CORE2 Gi1/0/3
 no switchport
 ip address 10.10.4.21 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/1/3
 description to SWL3-HFA-C-DIST1 Gi1/1/1
 no switchport
 ip address 10.10.36.1 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/1/4
 description to SWL3-HFA-C-DIST2 Gi1/1/1
 no switchport
 ip address 10.10.36.9 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
 exit
```

### 6d — OSPF: ABR between area 0 and area 2 (step 08)
⌨️ Type:
```
router ospf 1
 router-id 10.10.5.21
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface gi1/1/2
 no passive-interface gi1/1/3
 no passive-interface gi1/1/4
 no passive-interface vlan 99
 network 10.10.0.0 0.0.15.255 area 0
 network 10.10.32.0 0.0.15.255 area 2
 area 2 range 10.10.32.0 255.255.240.0
 exit
```

### 6e — Management
⌨️ Type the **🅼 management block for L3 switches** from section 1.6 (answer `1024` to the key
question).

✅ `[OK]` at the end. 🔍 `show ip interface brief` → Loopback0 `10.10.5.21`, Gi1/1/1–4 and
Vlan40/50/60/99 with the addresses from 1.5.

---

## Mission 7 — B-DIST2: the mirror of B-DIST1
👉 **Where:** **SWL3-HFA-B-DIST2** → CLI → **Enter** → `Switch>`.

What changes compared with B-DIST1:

| | B-DIST1 | **B-DIST2** |
|---|---|---|
| hostname | SWL3-HFA-B-DIST1 | **SWL3-HFA-B-DIST2** |
| VTP | server + creates the VLANs | **transparent → client, no `vlan` commands** |
| STP root primary / secondary | 40,60,99 / 50 | **50 / 40,60,99** |
| SVI addresses | .2 | **.3** |
| HSRP priority 110 on | 40, 60, 99 | **50** |
| DHCP gives | .11–.130 | **.131–.254** |
| Loopback0 / router-id | 10.10.5.21 | **10.10.5.22** |
| Gi1/1/1, Gi1/1/2 | 10.10.4.17, .21 | **10.10.4.25, .29** |
| Gi1/1/3, Gi1/1/4 | 10.10.36.1, .9 | **10.10.36.5, .13** |

### 7a — Hardening, VTP client, STP, trunks, EtherChannel
⌨️ Type:
```
enable
configure terminal
hostname SWL3-HFA-B-DIST2
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
vtp domain HFA-B
vtp version 2
vtp password Intel#Vtp
vtp mode client
spanning-tree mode rapid-pvst
spanning-tree vlan 50 root primary
spanning-tree vlan 40,60,99 root secondary
interface range gi1/0/1 - 2
 description TRUNK to access switch
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 40,50,60,99,998
 switchport nonegotiate
 spanning-tree guard root
 no shutdown
interface range gi1/0/23 - 24
 description Po1 member to SWL3-HFA-B-DIST1
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 40,50,60,99,998
 switchport nonegotiate
 channel-group 1 mode active
 no shutdown
interface port-channel 1
 description EtherChannel to SWL3-HFA-B-DIST1
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 40,50,60,99,998
 switchport nonegotiate
interface range gi1/0/3 - 22
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
```
Press **Fast Forward Time**. 🔍 **Check now:** `do show etherchannel summary` → `Po1(SU)`, both
members `(P)`; `do show vlan brief` → the building-B VLANs arrived by VTP.

### 7b — Gateways and DHCP
⌨️ Type:
```
ip routing
interface vlan 40
 description GW HR
 ip address 10.10.32.3 255.255.255.0
 standby version 2
 standby 40 ip 10.10.32.1
 standby 40 priority 90
 standby 40 preempt
 no shutdown
interface vlan 50
 description GW MARKETING
 ip address 10.10.33.3 255.255.255.0
 standby version 2
 standby 50 ip 10.10.33.1
 standby 50 priority 110
 standby 50 preempt
 no shutdown
interface vlan 60
 description GW PRINTERS-B
 ip address 10.10.34.3 255.255.255.0
 standby version 2
 standby 60 ip 10.10.34.1
 standby 60 priority 90
 standby 60 preempt
 no shutdown
interface vlan 99
 description GW MGMT-B
 ip address 10.10.35.3 255.255.255.0
 standby version 2
 standby 99 ip 10.10.35.1
 standby 99 priority 90
 standby 99 preempt
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
 exit
! DHCP: this switch gives .131-.254
ip dhcp excluded-address 10.10.32.1 10.10.32.130
ip dhcp excluded-address 10.10.33.1 10.10.33.130
ip dhcp pool HFA-HR
 network 10.10.32.0 255.255.255.0
 default-router 10.10.32.1
 dns-server 10.10.200.11
 domain-name intel.lab
ip dhcp pool HFA-MARKETING
 network 10.10.33.0 255.255.255.0
 default-router 10.10.33.1
 dns-server 10.10.200.11
 domain-name intel.lab
 exit
```

### 7c — Uplinks and OSPF
⌨️ Type:
```
interface loopback 0
 ip address 10.10.5.22 255.255.255.255
interface gi1/1/1
 description to SWL3-HFA-CORE1 Gi1/0/4
 no switchport
 ip address 10.10.4.25 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/1/2
 description to SWL3-HFA-CORE2 Gi1/0/4
 no switchport
 ip address 10.10.4.29 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/1/3
 description to SWL3-HFA-C-DIST1 Gi1/1/2
 no switchport
 ip address 10.10.36.5 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/1/4
 description to SWL3-HFA-C-DIST2 Gi1/1/2
 no switchport
 ip address 10.10.36.13 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
 exit
router ospf 1
 router-id 10.10.5.22
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface gi1/1/2
 no passive-interface gi1/1/3
 no passive-interface gi1/1/4
 no passive-interface vlan 99
 network 10.10.0.0 0.0.15.255 area 0
 network 10.10.32.0 0.0.15.255 area 2
 area 2 range 10.10.32.0 255.255.240.0
 exit
```

### 7d — Management
⌨️ Type the **🅼 management block for L3 switches** (section 1.6).

✅ `[OK]`. Press Fast Forward Time. 🔍 `show standby brief` → Active for 50, Standby for 40/60/99.

---

## Mission 8 — B-ACC1 and B-ACC2

### 8a — SW-HFA-B-ACC1
👉 **Where:** **SW-HFA-B-ACC1** → CLI → **Enter** → `Switch>`.

⌨️ **Hardening + VTP client** (steps 02–03):
```
enable
configure terminal
hostname SW-HFA-B-ACC1
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
vtp domain HFA-B
vtp version 2
vtp password Intel#Vtp
vtp mode client
spanning-tree mode rapid-pvst
```
⌨️ **Uplinks, user ports, printer, unused** (steps 02–04, 06):
```
interface range gi0/1 - 2
 description TRUNK uplink to B-DIST1 / B-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 40,50,60,99,998
 switchport nonegotiate
 ip dhcp snooping trust
 no shutdown
interface range fa0/1 - 10
 description HR users
 switchport mode access
 switchport access vlan 40
interface range fa0/11 - 20
 description MARKETING users
 switchport mode access
 switchport access vlan 50
interface range fa0/1 - 20
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 ip dhcp snooping limit rate 10
 no cdp enable
interface fa0/21
 description PRN-HFA-B-01
 switchport mode access
 switchport access vlan 60
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 no cdp enable
interface range fa0/22 - 24
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
```
⌨️ **DHCP snooping + management address** (steps 05–06):
```
ip dhcp snooping
ip dhcp snooping vlan 40,50,60,99
no ip dhcp snooping information option
interface vlan 99
 description MANAGEMENT
 ip address 10.10.35.11 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.10.35.1
```
⌨️ Then the **🅰 management block for access switches** (section 1.6).

✅ `[OK]`.

### 8b — SW-HFA-B-ACC2
👉 **Where:** **SW-HFA-B-ACC2** → CLI → **Enter** → `Switch>`.

Exactly the same as 8a, with **two changes**: `hostname SW-HFA-B-ACC2` and management IP
`10.10.35.12`. Here is the full block so you don't have to scroll:

⌨️ Type:
```
enable
configure terminal
hostname SW-HFA-B-ACC2
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
vtp domain HFA-B
vtp version 2
vtp password Intel#Vtp
vtp mode client
spanning-tree mode rapid-pvst
interface range gi0/1 - 2
 description TRUNK uplink to B-DIST1 / B-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 40,50,60,99,998
 switchport nonegotiate
 ip dhcp snooping trust
 no shutdown
interface range fa0/1 - 10
 description HR users
 switchport mode access
 switchport access vlan 40
interface range fa0/11 - 20
 description MARKETING users
 switchport mode access
 switchport access vlan 50
interface range fa0/1 - 20
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 ip dhcp snooping limit rate 10
 no cdp enable
interface fa0/21
 description printer
 switchport mode access
 switchport access vlan 60
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 no cdp enable
interface range fa0/22 - 24
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
ip dhcp snooping
ip dhcp snooping vlan 40,50,60,99
no ip dhcp snooping information option
interface vlan 99
 description MANAGEMENT
 ip address 10.10.35.12 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.10.35.1
```
⌨️ Then the **🅰 management block for access switches**.

✅ `[OK]`. 🔍 On B-ACC2: `show vtp status` → domain `HFA-B`, mode Client;
`show interfaces trunk` → Gi0/1, Gi0/2 trunking, native 998.

---

## Mission 9 — Open the core ports toward building B
Those ports have been parked and shut since step 08. Now they become routed links.

### 9a — SWL3-HFA-CORE1
👉 CORE1 → CLI → log in (`admin` / `Intel#Adm1n`) → `SWL3-HFA-CORE1#`.

⌨️ Type:
```
configure terminal
interface gi1/0/3
 description to SWL3-HFA-B-DIST1 Gi1/1/1
 no switchport
 ip address 10.10.4.18 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/0/4
 description to SWL3-HFA-B-DIST2 Gi1/1/1
 no switchport
 ip address 10.10.4.26 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
 exit
router ospf 1
 no passive-interface gi1/0/3
 no passive-interface gi1/0/4
 end
copy running-config startup-config
```

### 9b — SWL3-HFA-CORE2
👉 CORE2 → CLI → log in → `SWL3-HFA-CORE2#`. Addresses `.22` and `.30`:

⌨️ Type:
```
configure terminal
interface gi1/0/3
 description to SWL3-HFA-B-DIST1 Gi1/1/2
 no switchport
 ip address 10.10.4.22 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/0/4
 description to SWL3-HFA-B-DIST2 Gi1/1/2
 no switchport
 ip address 10.10.4.30 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
 exit
router ospf 1
 no passive-interface gi1/0/3
 no passive-interface gi1/0/4
 end
copy running-config startup-config
```
Press **Fast Forward Time** a few times.

✅ `%OSPF-5-ADJCHG ... Nbr 10.10.5.21 ... to FULL` and `... 10.10.5.22 ... to FULL` on both cores.

---

## Mission 10 — Building B end devices + check building B

### 10a — End devices
- **PCs** HR-01, HR-02, MKT-01, MKT-02: Desktop → IP Configuration → **DHCP** (M16).
- **PRN-HFA-B-01** (M15): Config → FastEthernet0 → **Static**, IP `10.10.34.10`, mask
  `255.255.255.0`; Config → Settings → Gateway `10.10.34.1`.

### 10b — Check building B before you continue
| # | Where | Type | ✅ Expected |
|---|---|---|---|
| 1 | CORE1 | `show ip route ospf` | `O IA 10.10.32.0/20` via B-DIST1 **and** B-DIST2 |
| 2 | B-DIST1 | `show ip ospf neighbor` | CORE1, CORE2 and B-DIST2 — all FULL |
| 3 | B-DIST1 | `show standby brief` | Active 40/60/99, Standby 50 |
| 4 | B-DIST1 | `show etherchannel summary` | `Po1(SU)`, both `(P)` |
| 5 | PC-HFA-HR-01 | `ipconfig` | an address `10.10.32.x`, gateway `10.10.32.1` |
| 6 | PC-HFA-HR-01 | `ping` an Engineering PC in building A | replies |

❌ Don't start building C until all six pass. **File → Save**.

---

## Mission 11 — C-DIST1
👉 **Where:** **SWL3-HFA-C-DIST1** → CLI → **Enter** → `Switch>`.

Building C is like building B with its own numbers: VTP `HFA-C`, VLANs 70/80/90/99, C-DIST1 is
boss of 70, 90, 99. **The big difference is OSPF (11d): no area 0 here.**

### 11a — Hardening, VTP server, VLANs
⌨️ Type:
```
enable
configure terminal
hostname SWL3-HFA-C-DIST1
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp domain HFA-C
vtp version 2
vtp password Intel#Vtp
vtp mode server
vlan 70
 name RND
vlan 80
 name IT
vlan 90
 name PRINTERS-C
vlan 99
 name MGMT-C
vlan 998
 name NATIVE
vlan 999
 name PARKING
 exit
```

### 11b — STP, trunks, EtherChannel, unused
⌨️ Type:
```
spanning-tree mode rapid-pvst
spanning-tree vlan 70,90,99 root primary
spanning-tree vlan 80 root secondary
interface range gi1/0/1 - 2
 description TRUNK to access switch
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 70,80,90,99,998
 switchport nonegotiate
 spanning-tree guard root
 no shutdown
interface range gi1/0/23 - 24
 description Po1 member to SWL3-HFA-C-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 70,80,90,99,998
 switchport nonegotiate
 channel-group 1 mode active
 no shutdown
interface port-channel 1
 description EtherChannel to SWL3-HFA-C-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 70,80,90,99,998
 switchport nonegotiate
interface range gi1/0/3 - 22
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
```

### 11c — Gateways and DHCP (.11–.130)
⌨️ Type:
```
ip routing
interface vlan 70
 description GW RND
 ip address 10.10.48.2 255.255.255.0
 standby version 2
 standby 70 ip 10.10.48.1
 standby 70 priority 110
 standby 70 preempt
 no shutdown
interface vlan 80
 description GW IT
 ip address 10.10.49.2 255.255.255.0
 standby version 2
 standby 80 ip 10.10.49.1
 standby 80 priority 90
 standby 80 preempt
 no shutdown
interface vlan 90
 description GW PRINTERS-C
 ip address 10.10.50.2 255.255.255.0
 standby version 2
 standby 90 ip 10.10.50.1
 standby 90 priority 110
 standby 90 preempt
 no shutdown
interface vlan 99
 description GW MGMT-C
 ip address 10.10.51.2 255.255.255.0
 standby version 2
 standby 99 ip 10.10.51.1
 standby 99 priority 110
 standby 99 preempt
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
 exit
ip dhcp excluded-address 10.10.48.1 10.10.48.10
ip dhcp excluded-address 10.10.48.131 10.10.48.254
ip dhcp excluded-address 10.10.49.1 10.10.49.10
ip dhcp excluded-address 10.10.49.131 10.10.49.254
ip dhcp pool HFA-RND
 network 10.10.48.0 255.255.255.0
 default-router 10.10.48.1
 dns-server 10.10.200.11
 domain-name intel.lab
ip dhcp pool HFA-IT
 network 10.10.49.0 255.255.255.0
 default-router 10.10.49.1
 dns-server 10.10.200.11
 domain-name intel.lab
 exit
```

### 11d — Loopback, uplinks to building B, OSPF areas 2 and 3 (no area 0 here!)
⌨️ Type:
```
! loopback inside AREA 3's block
interface loopback 0
 ip address 10.10.63.1 255.255.255.255
! uplinks to building B (area 2)
interface gi1/1/1
 description to SWL3-HFA-B-DIST1 Gi1/1/3
 no switchport
 ip address 10.10.36.2 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/1/2
 description to SWL3-HFA-B-DIST2 Gi1/1/3
 no switchport
 ip address 10.10.36.6 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
 exit
router ospf 1
 router-id 10.10.63.1
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface gi1/1/2
 no passive-interface vlan 99
 network 10.10.36.0 0.0.0.15 area 2
 network 10.10.48.0 0.0.15.255 area 3
 area 3 range 10.10.48.0 255.255.240.0
 exit
```
> `network 10.10.36.0 0.0.0.15` = the 16 addresses 10.10.36.0–15: exactly the four B↔C links.

### 11e — Management
⌨️ Type the **🅼 management block for L3 switches** (section 1.6).

✅ `[OK]`.

---

## Mission 12 — C-DIST2
👉 **Where:** **SWL3-HFA-C-DIST2** → CLI → **Enter** → `Switch>`.

| | C-DIST1 | **C-DIST2** |
|---|---|---|
| VTP | server + VLANs | **transparent → client** |
| STP root primary / secondary | 70,90,99 / 80 | **80 / 70,90,99** |
| SVIs / HSRP 110 on | .2 / 70, 90, 99 | **.3 / 80** |
| DHCP gives | .11–.130 | **.131–.254** |
| Loopback0 / router-id | 10.10.63.1 | **10.10.63.2** |
| Gi1/1/1, Gi1/1/2 | 10.10.36.2, .6 | **10.10.36.10, .14** |

### 12a — Hardening, VTP, STP, trunks, EtherChannel
⌨️ Type:
```
enable
configure terminal
hostname SWL3-HFA-C-DIST2
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
vtp domain HFA-C
vtp version 2
vtp password Intel#Vtp
vtp mode client
spanning-tree mode rapid-pvst
spanning-tree vlan 80 root primary
spanning-tree vlan 70,90,99 root secondary
interface range gi1/0/1 - 2
 description TRUNK to access switch
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 70,80,90,99,998
 switchport nonegotiate
 spanning-tree guard root
 no shutdown
interface range gi1/0/23 - 24
 description Po1 member to SWL3-HFA-C-DIST1
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 70,80,90,99,998
 switchport nonegotiate
 channel-group 1 mode active
 no shutdown
interface port-channel 1
 description EtherChannel to SWL3-HFA-C-DIST1
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 70,80,90,99,998
 switchport nonegotiate
interface range gi1/0/3 - 22
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
```

### 12b — Gateways and DHCP (.131–.254)
⌨️ Type:
```
ip routing
interface vlan 70
 description GW RND
 ip address 10.10.48.3 255.255.255.0
 standby version 2
 standby 70 ip 10.10.48.1
 standby 70 priority 90
 standby 70 preempt
 no shutdown
interface vlan 80
 description GW IT
 ip address 10.10.49.3 255.255.255.0
 standby version 2
 standby 80 ip 10.10.49.1
 standby 80 priority 110
 standby 80 preempt
 no shutdown
interface vlan 90
 description GW PRINTERS-C
 ip address 10.10.50.3 255.255.255.0
 standby version 2
 standby 90 ip 10.10.50.1
 standby 90 priority 90
 standby 90 preempt
 no shutdown
interface vlan 99
 description GW MGMT-C
 ip address 10.10.51.3 255.255.255.0
 standby version 2
 standby 99 ip 10.10.51.1
 standby 99 priority 90
 standby 99 preempt
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
 exit
ip dhcp excluded-address 10.10.48.1 10.10.48.130
ip dhcp excluded-address 10.10.49.1 10.10.49.130
ip dhcp pool HFA-RND
 network 10.10.48.0 255.255.255.0
 default-router 10.10.48.1
 dns-server 10.10.200.11
 domain-name intel.lab
ip dhcp pool HFA-IT
 network 10.10.49.0 255.255.255.0
 default-router 10.10.49.1
 dns-server 10.10.200.11
 domain-name intel.lab
 exit
```

### 12c — Loopback, uplinks, OSPF
⌨️ Type:
```
interface loopback 0
 ip address 10.10.63.2 255.255.255.255
interface gi1/1/1
 description to SWL3-HFA-B-DIST1 Gi1/1/4
 no switchport
 ip address 10.10.36.10 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/1/2
 description to SWL3-HFA-B-DIST2 Gi1/1/4
 no switchport
 ip address 10.10.36.14 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
 exit
router ospf 1
 router-id 10.10.63.2
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface gi1/1/2
 no passive-interface vlan 99
 network 10.10.36.0 0.0.0.15 area 2
 network 10.10.48.0 0.0.15.255 area 3
 area 3 range 10.10.48.0 255.255.240.0
 exit
```

### 12d — Management
⌨️ Type the **🅼 management block for L3 switches** (section 1.6).

✅ `[OK]`.

---

## Mission 13 — C-ACC1 and C-ACC2
The same as B-ACC1/2 (Mission 8) with building C's values: VTP `HFA-C`, VLANs **70** (R&D,
Fa0/1–10), **80** (IT, Fa0/11–20), **90** (printer, Fa0/21), management `10.10.51.11` / `.12`,
gateway `10.10.51.1`.

### 13a — SW-HFA-C-ACC1
👉 **Where:** **SW-HFA-C-ACC1** → CLI → **Enter** → `Switch>`.

⌨️ Type:
```
enable
configure terminal
hostname SW-HFA-C-ACC1
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
vtp domain HFA-C
vtp version 2
vtp password Intel#Vtp
vtp mode client
spanning-tree mode rapid-pvst
interface range gi0/1 - 2
 description TRUNK uplink to C-DIST1 / C-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 70,80,90,99,998
 switchport nonegotiate
 ip dhcp snooping trust
 no shutdown
interface range fa0/1 - 10
 description RND users
 switchport mode access
 switchport access vlan 70
interface range fa0/11 - 20
 description IT users
 switchport mode access
 switchport access vlan 80
interface range fa0/1 - 20
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 ip dhcp snooping limit rate 10
 no cdp enable
interface fa0/21
 description PRN-HFA-C-01
 switchport mode access
 switchport access vlan 90
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 no cdp enable
interface range fa0/22 - 24
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
ip dhcp snooping
ip dhcp snooping vlan 70,80,90,99
no ip dhcp snooping information option
interface vlan 99
 description MANAGEMENT
 ip address 10.10.51.11 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.10.51.1
```
⌨️ Then the **🅰 management block for access switches**. ✅ `[OK]`.

### 13b — SW-HFA-C-ACC2
👉 **Where:** **SW-HFA-C-ACC2** → CLI → **Enter** → `Switch>`.

Type the **whole 13a block again** with these **two** lines changed:
- `hostname SW-HFA-C-ACC2`
- under `interface vlan 99`: `ip address 10.10.51.12 255.255.255.0`

(and `description printer` on Fa0/21 if you like — it's only a label).
Then the **🅰 management block**. ✅ `[OK]`.

🔍 On C-ACC2: `show vlan brief` → VLANs 70/80/90/99 arrived; Fa0/1 in 70, Fa0/11 in 80.

---

## Mission 14 — Building C end devices + see the problem 🔍

### 14a — End devices
- PCs RND-01, RND-02, IT-01, IT-02 → **DHCP**.
- PRN-HFA-C-01 → IP `10.10.50.10` / `255.255.255.0`, gateway `10.10.50.1`.

✅ The PCs get `10.10.48.x` (R&D) and `10.10.49.x` (IT) — DHCP works **inside** the building.

### 14b — See the problem
Press **Fast Forward Time** a few times, then:

| # | Where | Type | What you see | Means |
|---|---|---|---|---|
| 1 | B-DIST1 | `show ip ospf neighbor` | C-DIST1 and C-DIST2 are FULL | area 2 works |
| 2 | CORE1 | `show ip route ospf` | `10.10.16.0/20`, `10.10.32.0/20` — but **no** `10.10.48.0/20` | area 3 is invisible to the core |
| 3 | PC-HFA-RND-01 | `ping 10.10.1.10` (Syslog server) | ❌ Request timed out | building C can't leave |
| 4 | C-DIST1 | `show ip route` | no `O IA` routes from areas 0 and 1 | the island is complete |

✅ All four as described. **This is the expected result** — the rule from 1.2 in action.
Write in your notebook, in one sentence, **why** it happens. Then fix it in Mission 15.

---

## Mission 15 — The virtual links
🧠 **Why:** give building C a "piece of area 0" through area 2.

### 15a — On B-DIST1 **and** B-DIST2 (ABRs 0/2): point to C's router-IDs
⌨️ Type on **each** of the two:
```
configure terminal
router ospf 1
 area 2 virtual-link 10.10.63.1
 area 2 virtual-link 10.10.63.2
 end
copy running-config startup-config
```

### 15b — On C-DIST1 **and** C-DIST2 (ABRs 2/3): point to B's router-IDs
⌨️ Type on **each** of the two:
```
configure terminal
router ospf 1
 area 2 virtual-link 10.10.5.21
 area 2 virtual-link 10.10.5.22
 end
copy running-config startup-config
```
Press **Fast Forward Time**.

✅ You see `%OSPF-5-ADJCHG: Process 1, Nbr 10.10.63.1 on OSPF_VL0 from LOADING to FULL`.

| Command | In simple English |
|---|---|
| `area 2 virtual-link 10.10.63.1` | "Build a piece of area 0 **through area 2**, to the router whose router-ID is 10.10.63.1" |

🔍 **Check now:** on B-DIST1 `show ip ospf virtual-links`:
```
Virtual Link OSPF_VL0 to router 10.10.63.1 is up
  Transit area 2, via interface GigabitEthernet1/1/3, Cost of using 1
  ...
Virtual Link OSPF_VL1 to router 10.10.63.2 is up
```
❌ `is down` → you typed an **interface IP** instead of the **router-ID**, or the transit area
isn't `2`.

---

## Mission 16 — Update the 7 older devices' guest list
🧠 **Why:** IT and the new MGMT VLANs must be allowed to manage building A, the cores and the
server switch too.

On **SW-HFA-A-ACC1, SW-HFA-A-ACC2, SWL3-HFA-A-DIST1, SWL3-HFA-A-DIST2, SWL3-HFA-CORE1,
SWL3-HFA-CORE2, SW-HFA-SRV** (log in with `admin` / `Intel#Adm1n`):

⌨️ Type:
```
configure terminal
ip access-list standard MGMT-ONLY
 no deny any log
 permit 10.10.35.0 0.0.0.255
 permit 10.10.51.0 0.0.0.255
 permit 10.10.49.0 0.0.0.255
 deny any log
 end
copy running-config startup-config
```
> Why remove the `deny` first? New lines are added at the **bottom** of a named ACL. If the
> `deny any` stayed above them, the new permits would never be reached.

🔍 On each: `show access-lists MGMT-ONLY` → 4 permits, then the deny. Tick:
- [ ] A-ACC1  - [ ] A-ACC2  - [ ] A-DIST1  - [ ] A-DIST2  - [ ] CORE1  - [ ] CORE2  - [ ] SW-HFA-SRV

---

## Mission 17 — Register the 8 new devices on the AAA server
👉 **Where:** SRV-HFA-AAA → **Services → AAA → Network Configuration** (as in step 08, Mission 13).
For each row: Client Name, Client IP, Secret `Intel#Tac`, ServerType **Tacacs** → **Add**.

| Client Name | Client IP |
|---|---|
| SWL3-HFA-B-DIST1 | 10.10.5.21 |
| SWL3-HFA-B-DIST2 | 10.10.5.22 |
| SW-HFA-B-ACC1 | 10.10.35.11 |
| SW-HFA-B-ACC2 | 10.10.35.12 |
| SWL3-HFA-C-DIST1 | 10.10.63.1 |
| SWL3-HFA-C-DIST2 | 10.10.63.2 |
| SW-HFA-C-ACC1 | 10.10.51.11 |
| SW-HFA-C-ACC2 | 10.10.51.12 |

✅ The client list now has 15 lines.

---

## Mission 18 — Final checks

### 18a — The core now sees every building as one route
👉 On **CORE1**: `show ip route ospf`

✅ You should see:
```
O IA  10.10.16.0/20 ...   <- building A (area 1)
O IA  10.10.32.0/20 ...   <- building B (area 2)
O IA  10.10.48.0/20 ...   <- building C (area 3) — arrived through the virtual link!
```

### 18b — Pings and SSH
| # | From | Type | Expected |
|---|---|---|---|
| 1 | PC-HFA-RND-01 | `ping 10.10.1.10` | ✅ (failed before the virtual link) |
| 2 | PC-HFA-IT-01 | `ping` an Engineering PC in building A | ✅ |
| 3 | PC-HFA-HR-01 | `ping` an R&D PC | ✅ |
| 4 | PC-HFA-IT-01 | `ssh -l admin 10.10.19.11` (ACC1 in building A) | ✅ IT may manage devices |
| 5 | PC-HFA-RND-01 | `ssh -l admin 10.10.51.11` | ❌ refused (not on the guest list) |

### 18c — Save
`copy running-config startup-config` on every changed switch → **File → Save**.

---

## Checkpoint
- [ ] Building B and C PCs get DHCP addresses from their own DIST switches
- [ ] All 4 virtual links up
- [ ] CORE1 has exactly three `O IA` /20 routes
- [ ] Every PC in every building reaches the servers
- [ ] IT can SSH to every switch; R&D can't

## If something doesn't work
| Problem | Fix |
|---|---|
| Virtual link stays down | You used an **interface IP** instead of the **router-ID**; or the transit area isn't `2`; or C-DIST has no FULL neighbour in area 2 yet |
| C-DIST has no area-2 neighbours | MD5 key typo on a 10.10.36.x link, or `network 10.10.36.0 0.0.0.15 area 2` missing |
| Core sees many small routes from area 2 | Missing `area 2 range 10.10.32.0 255.255.240.0` on B-DIST1/2 |
| Building C PCs get no address | VTP domain `HFA-C` typo on an access switch, or snooping without `trust` on uplinks |
| `% Access VLAN does not exist` warnings | Normal during VTP learning — see step 03 |
| A new 3650 has every link red | Its power supply is missing (Mission 2c) |

---

## 🏋 Exercises

### Exercise A — Remove the summary ⭐⭐
- 🎯 **Task:** see what the area-2 summary hides from the core.
- 📁 **Start from:** your `step09.pkt` (Save As `step09-exA.pkt`).
- 👣 **Steps:**
  1. On B-DIST1 **and** B-DIST2: `configure terminal` → `router ospf 1` →
     `no area 2 range 10.10.32.0 255.255.240.0` → `end`.
  2. Fast Forward Time. On CORE1: `show ip route ospf`. Count the lines that start with `10.10.3`.
  3. Put the summary back on both.
- ✅ **Done when:** you can say how many extra lines appeared, and why the `10.10.36.x` links are
  among them.

<details><summary>💡 Hint</summary>

Which networks are in area 2? The VLANs of building B… and what else?
</details>
<details><summary>✔ Answer</summary>

Without the summary, every area-2 network is announced separately as `O IA`: the 4 VLANs
(40, 50, 60, 99) **and** the four /30 links to building C — about 8 lines instead of 1. The /30
links are area-2 networks too. With the summary they're hidden inside 10.10.32.0/20 — that's
why we numbered them 10.10.36.x and not 10.10.4.x.
</details>

### Exercise B — Break one virtual link ⭐⭐
- 🎯 **Task:** prove that building C survives losing one of the two B-DIST switches.
- 📁 **Start from:** your `step09.pkt` — **File → Save** first.
- 👣 **Steps:**
  1. From PC-HFA-RND-01 start `ping -t 10.10.1.10`.
  2. Delete **SWL3-HFA-B-DIST1** (Delete tool).
  3. Watch the ping. On C-DIST1: `show ip ospf virtual-links`.
  4. **File → Open** your saved `step09.pkt` (answer **No** to saving).
- ✅ **Done when:** you can explain why the ping came back, and which virtual links are still up.

<details><summary>✔ Answer</summary>

A few pings are lost, then they continue: the virtual links to `10.10.5.22` (B-DIST2) are still
up, so area 3 still has a path to area 0. That's why we built all 4 combinations (B1↔C1, B1↔C2,
B2↔C1, B2↔C2) and not just one.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

- [ ] Open `my-project/step08.pkt` → **Save As** `my-project/step09.pkt`
- [ ] Write your own tables first, like 1.5: links, loopbacks, VLANs, primaries
- [ ] Write your own two management blocks, like 1.6, with your guest list
- [ ] Build, power and cable your other HQ buildings
- [ ] Configure the building that touches the core; check it (like Mission 10b)
- [ ] Configure the building that touches only the other building; **see the problem**
- [ ] Add the virtual links; update the old devices' ACL; register the new devices on AAA
- [ ] Mission 18 checks pass in your project

| In the example | In your project — take it from |
|---|---|
| Buildings B and C, their VLANs 40–90 | plan part D (your other HQ buildings / departments) |
| VTP domains `HFA-B`, `HFA-C` | one domain per building, your names |
| Area 3 not touching area 0 → virtual link through area 2 | the ministry requires an OSPF virtual link (req. 8): your design must have one area that reaches area 0 only through another area |
| Summaries per area | your /20 per area |

**Check:** `show ip ospf virtual-links` shows the link **up**; every department at HQ can ping
every other.

💾 Save as `my-project/step09.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★3 Where did you configure a virtual link? Why?**

<details><summary>Answer</summary>

Every OSPF area must touch area 0. The building whose area has no direct link to area 0 is joined to the backbone **through** a transit area: `area <transit> virtual-link <router-ID of the other ABR>` on both ABRs. The ministry requires this structure (req. 8, 22.16).

**Show it:** `show ip ospf virtual-links` → "is up"
</details>

**2. ◆ Why can't the virtual link's transit area be a stub?**

<details><summary>Answer</summary>

A virtual link needs full routing information in the transit area; stub areas block it, so the virtual link can only cross a normal area.

**Show it:** —
</details>

**3. ★3 What is a STUB area? Which LSAs does it block and receive?**

<details><summary>Answer</summary>

An area that doesn't receive external routes: type 4 and 5 LSAs are blocked; it receives types 1, 2, 3 and a default route from the ABR. Configured with `area X stub` on **all** routers of the area.

**Show it:** —
</details>

**4. ★3 What is a TOTALLY STUBBY area, and how do you make a stub area totally stubby?**

<details><summary>Answer</summary>

It also blocks type 3 (inter-area) LSAs; the area receives only types 1, 2 and a default route. On the ABR only: `area X stub no-summary`.

**Show it:** —
</details>

**5. ★3 What is an NSSA, and how do you make it totally NSSA?**

<details><summary>Answer</summary>

Not-So-Stubby Area: like a stub, but a router inside it may import external routes, which travel as type 7 LSAs and are converted to type 5 by the ABR. Totally NSSA: `area X nssa no-summary` on the ABR.

**Show it:** —
</details>

**6. ◆ How do you summarize an area in OSPF?**

<details><summary>Answer</summary>

On the ABR: `area <N> range <network> <mask>` — one route for the whole /20 of the building is sent to the backbone.

**Show it:** `show ip route` on a core switch → one `O IA` per building
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

In a copy of your step-09 file, delete the virtual link on both ABRs. Show that the far area
is no longer reachable (`show ip route`), then add the virtual link again without this page.
Target: **5 minutes**. Explain to the examiner why the area needs it.

---

## 👩‍🏫 Teacher notes
- **Lesson 1:** revision game — students say which step each block of B-DIST1's config comes
  from (hardening = 02, VTP = 03, STP/EtherChannel = 04, SVIs/HSRP/DHCP = 05, …). Then
  Missions 1–4 (build and cable both buildings). Check the 4 power supplies before they leave.
- **Lesson 2:** Missions 5–10 (building B). Don't let anyone start building C until Mission 10b
  passes.
- **Lesson 3:** Missions 11–14 (building C); **stop at Mission 14b** and let the class discover
  the problem. Ask: "OSPF neighbours are FULL — so why doesn't the core see area 3?" Lead them
  to the "every area must touch area 0" rule.
- **Lesson 4:** Missions 15–18 → exercises.
- **Differentiation:** strong students try each device from the tables in 1.5 before opening
  the block (🧩); weaker students type the blocks. Both reach the same checks.
- **Common student questions**
  - *"Why not just connect building C to the core?"* — In a real campus that's the better design!
    We build it this way because the ministry requires a virtual link (22.16). Say so in the book.
  - *"Why use the router-ID and not an IP address?"* — The virtual link is between two *routers*,
    not two interfaces. OSPF identifies routers by router-ID.
- **Typical errors:** missing power supplies (4 new 3650s!); wrong /30 side on the B↔C links;
  forgetting to update the ACL on the old devices.
