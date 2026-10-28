# Step 08 — Core layer, OSPF, server farm, AAA & Syslog

> ⏱ 3 lessons · 🎯 ministry requirements 8 (start), 20.1, 20.9, 20.10, 22.5, 22.15, 22.18, 6 · 📁 start from: `my-work/step07.pkt` · 💾 save as: `my-work/step08.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 07 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## 🗺 Your missions in this step
**Lesson 1 — the core**
- [ ] **Mission 1** — Open step 07 and save as step 08 (2 min)
- [ ] **Mission 2** — Place the 2 cores (with power), the server switch, 2 servers (10 min)
- [ ] **Mission 3** — Cable everything to the exact ports (15 min)
- [ ] **Mission 4** — CORE1: hardening, routing, VLANs, Loopback0 (10 min)
- [ ] **Mission 5** — CORE1: routed links, server port, server gateway (15 min)
- [ ] **Mission 6** — CORE1: OSPF + SSH (10 min)
- [ ] **Mission 7** — CORE2: the same, with its own values (25 min)

**Lesson 2 — OSPF areas**
- [ ] **Mission 8** — DIST1: uplinks + OSPF areas 0 and 1 (15 min)
- [ ] **Mission 9** — DIST2: the same, with its own values (10 min)
- [ ] **Mission 10** — Check OSPF: neighbours, summary route, pings (15 min)

**Lesson 3 — servers, AAA, Syslog**
- [ ] **Mission 11** — SW-HFA-SRV: the server switch (15 min)
- [ ] **Mission 12** — The two servers: addresses and services (10 min)
- [ ] **Mission 13** — Fill in the AAA server (15 min)
- [ ] **Mission 14** — AAA + Syslog on all 7 switches (25 min)
- [ ] **Mission 15** — Test AAA, its backup, and Syslog (10 min)
- [ ] **Exercises** A, B and C
- [ ] **🛠 Your own project**

## What you will have at the end
- Two **core** switches — the backbone of Haifa HQ.
- Building A connected to the core with **routed** links (not trunks).
- **OSPF** routing: core = **area 0**, building A = **area 1**, building A summarized to **one** route.
- A small **server farm** with a **Syslog** server (collects logs) and an **AAA** server
  (checks every admin login centrally).

```
              SRV-HFA-SYSLOG  SRV-HFA-AAA
                     └──── SW-HFA-SRV ────┘
                       Gi0/1 │      │ Gi0/2        VLAN 100 = 10.10.1.0/24
                             │      │
   SWL3-HFA-CORE1 ═══ Gi1/0/23 (10.10.4.48/30) ═══ SWL3-HFA-CORE2          OSPF AREA 0
     Gi1/0/1  Gi1/0/2                            Gi1/0/1  Gi1/0/2
        │        ╲──────────────╲    ╱─────────────╱        │
        │                         ╳                          │
        │        ╱──────────────╱    ╲─────────────╲        │
     Gi1/1/1  Gi1/1/2                            Gi1/1/1  Gi1/1/2
   SWL3-HFA-A-DIST1 ══════════ Po1 ══════════ SWL3-HFA-A-DIST2            ABRs 0/1
            (VLANs 10,20,30,99,110,120 below)                            OSPF AREA 1
```

---

## Part 1 — Understand it first (simple English)

### 1.1 Why a core layer?
Building A works on its own. Soon there will be **three** buildings, a firewall and the
internet. Connecting every building to every other building would be a mess. Instead, every
building connects to a fast central **core** — like all city roads leading to one ring road.
This is the **3-tier model**: **access** (users plug in) → **distribution** (each building's
routers) → **core** (the backbone). Requirement 20.1 asks for exactly this.

### 1.2 Routed links instead of trunks
Between the distribution and the core we use **routed links**: the port stops being a switch
port (`no switchport`) and gets its own IP address, like a router port. Each link is a tiny
/30 network with 2 addresses — one for each end.

Why not trunks? With routed links:
- there's **no Spanning Tree** between layers — so **no blocked links**: both uplinks of every
  distribution switch carry traffic at the same time;
- if a link dies, routing moves traffic in under a second;
- VLANs (and their broadcasts) stay inside their building.

### 1.3 Routing — how a router knows where to send packets
A router only knows the networks it is **directly connected** to (`C` routes). To reach
networks further away it needs **routes**. Two ways:
- **Static routes** — you type every route by hand. Fine for 1–2 routes (we use them toward
  the internet in step 12).
- **Dynamic routing protocol** — routers **tell each other** what networks they know, and
  update automatically when something breaks. We use **OSPF** in Haifa.

### 1.4 OSPF in simple words
**OSPF** (Open Shortest Path First):
1. Routers find **neighbours** by sending small "Hello" messages on their links.
2. Neighbours exchange information about their links → each router builds a **map** of the
   network.
3. Each router calculates the **shortest path** to every network on that map.
4. If a link fails, the news spreads, everyone recalculates — in about a second.

**Router-ID** — every OSPF router needs a unique name, written like an IP address. We use the
address of **Loopback0**: a virtual interface that never goes down, so the ID never changes.

### 1.5 OSPF areas — why split the map?
In a big network, one huge map is slow to calculate and every small change disturbs everyone.
OSPF splits the network into **areas**:
- Inside an area, routers know the **detailed** map.
- Between areas, they only exchange **summaries**.
- **Area 0** is the **backbone**: every other area must connect to it.
- A router with feet in two areas is an **ABR** (Area Border Router). DIST1 and DIST2 are
  ABRs: their uplinks are in area 0, their VLANs in area 1.

Comparison: a country (area 0 = national highways) with cities (areas 1, 2, 3). Inside a city
you know every street; between cities you only need "Haifa is that way".

### 1.6 Route summarization
In step 01 we gave building A the block `10.10.16.0/20`. The ABR can now announce **one**
route — `10.10.16.0/20` — instead of six /24s: `area 1 range 10.10.16.0 255.255.240.0`.
The core's table stays small, and a VLAN going up and down in building A doesn't disturb the
rest of the company. (Requirement 22.15.)

### 1.7 Making OSPF safe
- **Passive interface** — OSPF still announces the network, but sends **no Hellos** there.
  On user VLANs this stops an attacker from running OSPF on a laptop and injecting fake routes.
  We use `passive-interface default` (everything passive) and then open only the links toward
  other routers.
- **MD5 authentication** — neighbours must share a secret key, or no neighbourship.
  (Requirement 22.18.)

### 1.8 Syslog — the network's diary
Every device writes messages (`interface down`, `login failed`, …). By default they stay on the
device and are lost when it reboots. A **Syslog server** collects all messages from all devices
in one place. (Requirement 20.9.)

### 1.9 AAA — one central guard for all logins
Until now each switch had its own local user `admin`. With 50 devices, changing a password
means 50 changes. **AAA** = **A**uthentication (who are you?), **A**uthorization (what may you
do?), **A**ccounting (what did you do?). With `aaa new-model` + a **TACACS+** server, every
switch asks the AAA server "is this username/password OK?". The local user stays as a
**backup** if the server is unreachable. (Requirement 20.10.)

### 1.10 The address plan for this step
Keep this table open — every mission takes its numbers from here.

| Link | Subnet | First device | Second device |
|---|---|---|---|
| A-DIST1 Gi1/1/1 ↔ CORE1 Gi1/0/1 | 10.10.4.0/30 | DIST1 **.1** | CORE1 **.2** |
| A-DIST1 Gi1/1/2 ↔ CORE2 Gi1/0/1 | 10.10.4.4/30 | DIST1 **.5** | CORE2 **.6** |
| A-DIST2 Gi1/1/1 ↔ CORE1 Gi1/0/2 | 10.10.4.8/30 | DIST2 **.9** | CORE1 **.10** |
| A-DIST2 Gi1/1/2 ↔ CORE2 Gi1/0/2 | 10.10.4.12/30 | DIST2 **.13** | CORE2 **.14** |
| CORE1 Gi1/0/23 ↔ CORE2 Gi1/0/23 | 10.10.4.48/30 | CORE1 **.49** | CORE2 **.50** |

| Device | Loopback0 (= OSPF router-ID) |
|---|---|
| SWL3-HFA-CORE1 / CORE2 | 10.10.5.1 / 10.10.5.2 |
| SWL3-HFA-A-DIST1 / DIST2 | 10.10.5.11 / 10.10.5.12 |

| VLAN 100 SERVERS — 10.10.1.0/24 | Address |
|---|---|
| HSRP gateway (virtual) | 10.10.1.1 |
| CORE1 / CORE2 | 10.10.1.2 / 10.10.1.3 |
| SW-HFA-SRV (management) | 10.10.1.5 |
| SRV-HFA-SYSLOG | 10.10.1.10 |
| SRV-HFA-AAA | 10.10.1.11 |
| (ASA firewall — step 11) | 10.10.1.254 |

---

## Mission 1 — Open step 07 and save as step 08 (how? → M1)
**File → Open** → `my-work/step07.pkt` → **File → Save As** → `my-work/step08.pkt`.

---

## Mission 2 — Place the 2 cores (with power), the server switch, 2 servers

### 2a — Place and name (how? → M2, M4)
| Where to find it | Model | Display Name | Put it… |
|---|---|---|---|
| Network Devices → Switches | **3650-24PS** | `SWL3-HFA-CORE1` | in the **"HAIFA HQ - CORE"** area at the top, above DIST1 |
| Network Devices → Switches | **3650-24PS** | `SWL3-HFA-CORE2` | next to CORE1, above DIST2 |
| Network Devices → Switches | **2960** | `SW-HFA-SRV` | between and above the two cores |
| End Devices → End Devices | **Server** | `SRV-HFA-SYSLOG` | next to SW-HFA-SRV |
| End Devices → End Devices | **Server** | `SRV-HFA-AAA` | next to SW-HFA-SRV |

### 2b — Power supplies (⚠️ don't skip — how? → M8)
CORE1 and CORE2 → **Physical** tab → drag **AC-POWER-SUPPLY** into the empty slot → close.
(Exactly as in step 03, Mission 2b.)

✅ Both cores show green lights in the Physical tab.

---

## Mission 3 — Cable everything to the exact ports (how? → M5)
Switch ↔ switch = **Copper Cross-Over**. Server ↔ switch = **Copper Straight-Through**.

| # | Cable | Click first | Port | Click second | Port |
|---|---|---|---|---|---|
| 1 | Cross-Over | SWL3-HFA-A-DIST1 | **GigabitEthernet1/1/1** | SWL3-HFA-CORE1 | **GigabitEthernet1/0/1** |
| 2 | Cross-Over | SWL3-HFA-A-DIST1 | **GigabitEthernet1/1/2** | SWL3-HFA-CORE2 | **GigabitEthernet1/0/1** |
| 3 | Cross-Over | SWL3-HFA-A-DIST2 | **GigabitEthernet1/1/1** | SWL3-HFA-CORE1 | **GigabitEthernet1/0/2** |
| 4 | Cross-Over | SWL3-HFA-A-DIST2 | **GigabitEthernet1/1/2** | SWL3-HFA-CORE2 | **GigabitEthernet1/0/2** |
| 5 | Cross-Over | SWL3-HFA-CORE1 | **GigabitEthernet1/0/23** | SWL3-HFA-CORE2 | **GigabitEthernet1/0/23** |
| 6 | Cross-Over | SWL3-HFA-CORE1 | **GigabitEthernet1/0/24** | SW-HFA-SRV | **GigabitEthernet0/1** |
| 7 | Cross-Over | SWL3-HFA-CORE2 | **GigabitEthernet1/0/24** | SW-HFA-SRV | **GigabitEthernet0/2** |
| 8 | Straight-Through | SRV-HFA-SYSLOG | FastEthernet0 | SW-HFA-SRV | **FastEthernet0/1** |
| 9 | Straight-Through | SRV-HFA-AAA | FastEthernet0 | SW-HFA-SRV | **FastEthernet0/2** |

> The 3650's `Gi1/1/x` ports are the **uplink** ports (near the **end** of the port list).
> If PT refuses a copper cable on them, they are fiber ports in your version — use the
> orange **Fiber** cable instead. Everything else stays the same.

✅ Nine new cables. Check each label against the table (M6). Some dots stay orange/red until
the ports are configured — that's normal now.

---

## Mission 4 — CORE1: hardening, routing, VLANs, Loopback0
👉 **Where:** **SWL3-HFA-CORE1** → CLI → **Enter** → `Switch>` (M9).

### 4a — Hardening
⌨️ Type:
```
enable
configure terminal
hostname SWL3-HFA-CORE1
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
```

### 4b — Routing on, own VLAN list, Loopback0
The core doesn't join a building's VTP domain — it keeps its own VLANs (`transparent`).

⌨️ Type:
```
ip routing
vtp mode transparent
spanning-tree mode rapid-pvst
vlan 100
 name SERVERS
vlan 999
 name PARKING
 exit

! Loopback0 = stable router-ID
interface loopback 0
 ip address 10.10.5.1 255.255.255.255
 exit
```
🔍 **Check now:** `do show ip interface brief` → the last line is `Loopback0  10.10.5.1  YES manual up up`.

---

## Mission 5 — CORE1: routed links, server port, server gateway
👉 **Where:** CORE1, prompt `SWL3-HFA-CORE1(config)#`.

### 5a — Routed links to building A and to CORE2 (area 0, MD5)
Every routed link has the same 5 lines; only the description and the address change.
`no switchport` **must** come before `ip address`.

⌨️ Type:
```
interface gi1/0/1
 description to SWL3-HFA-A-DIST1 Gi1/1/1
 no switchport
 ip address 10.10.4.2 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/0/2
 description to SWL3-HFA-A-DIST2 Gi1/1/1
 no switchport
 ip address 10.10.4.10 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/0/23
 description to SWL3-HFA-CORE2 Gi1/0/23
 no switchport
 ip address 10.10.4.49 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
```
❌ `% Invalid input` on `ip address` → you skipped `no switchport` on that interface.

### 5b — The server switch port, unused ports, the server gateway
⌨️ Type:
```
interface gi1/0/24
 description to SW-HFA-SRV Gi0/1
 switchport mode access
 switchport access vlan 100
 no shutdown

! unused ports (Gi1/0/3-4 are used in step 09)
interface range gi1/0/3 - 22
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown

! gateway for the servers (HSRP, CORE1 = active)
interface vlan 100
 description GW SERVERS
 ip address 10.10.1.2 255.255.255.0
 standby version 2
 standby 100 ip 10.10.1.1
 standby 100 priority 110
 standby 100 preempt
 no shutdown
 exit
```
🔍 **Check now:** `do show ip interface brief` → `GigabitEthernet1/0/1` `10.10.4.2`,
`Gi1/0/2` `10.10.4.10`, `Gi1/0/23` `10.10.4.49`, `Vlan100` `10.10.1.2`, `Loopback0` `10.10.5.1`.
(Status may be `down` on links whose other end isn't configured yet — fine for now.)

---

## Mission 6 — CORE1: OSPF + SSH
👉 **Where:** CORE1, prompt `SWL3-HFA-CORE1(config)#`.

### 6a — OSPF
⌨️ Type:
```
router ospf 1
 router-id 10.10.5.1
 passive-interface default
 no passive-interface gi1/0/1
 no passive-interface gi1/0/2
 no passive-interface gi1/0/23
 network 10.10.0.0 0.0.15.255 area 0
 exit
```
✅ The prompt was `(config-router)#` inside `router ospf 1`.

> **What does `network 10.10.0.0 0.0.15.255 area 0` mean?** "Every interface whose address is
> inside 10.10.0.0/20 joins OSPF area 0." It catches the /30 links (10.10.4.x), Loopback0
> (10.10.5.x) and VLAN 100 (10.10.1.x) in **one line** — the payoff of planning one block per
> area in step 01. The `0.0.15.255` is a wildcard (like in ACLs, step 07) for a /20.

### 6b — SSH management (the same as step 07)
⌨️ Type:
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
 deny any log
 exit
line vty 0 15
 transport input ssh
 login local
 access-class MGMT-ONLY in
 exec-timeout 5 0
line console 0
 login local
 end
copy running-config startup-config
```
✅ `[OK]`.

> As in step 07: if a switch rejects `login block-for ...`, skip that one line on that switch.

---

## Mission 7 — CORE2: the same, with its own values
👉 **Where:** **SWL3-HFA-CORE2** → CLI → **Enter** → `Switch>`.

The **only** differences from CORE1:

| | CORE1 | **CORE2** |
|---|---|---|
| hostname | SWL3-HFA-CORE1 | **SWL3-HFA-CORE2** |
| Loopback0 / router-id | 10.10.5.1 | **10.10.5.2** |
| Gi1/0/1 | 10.10.4.2 (to DIST1 Gi1/1/1) | **10.10.4.6** (to DIST1 Gi1/1/2) |
| Gi1/0/2 | 10.10.4.10 (to DIST2 Gi1/1/1) | **10.10.4.14** (to DIST2 Gi1/1/2) |
| Gi1/0/23 | 10.10.4.49 | **10.10.4.50** |
| Gi1/0/24 | to SW-HFA-SRV Gi0/1 | to SW-HFA-SRV **Gi0/2** |
| Vlan100 | 10.10.1.2, priority 110 | **10.10.1.3, priority 90** |

### 7a — Hardening, routing, VLANs, Loopback0
⌨️ Type:
```
enable
configure terminal
hostname SWL3-HFA-CORE2
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
ip routing
vtp mode transparent
spanning-tree mode rapid-pvst
vlan 100
 name SERVERS
vlan 999
 name PARKING
 exit
interface loopback 0
 ip address 10.10.5.2 255.255.255.255
 exit
```

### 7b — Links, server port, server gateway
⌨️ Type:
```
interface gi1/0/1
 description to SWL3-HFA-A-DIST1 Gi1/1/2
 no switchport
 ip address 10.10.4.6 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/0/2
 description to SWL3-HFA-A-DIST2 Gi1/1/2
 no switchport
 ip address 10.10.4.14 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/0/23
 description to SWL3-HFA-CORE1 Gi1/0/23
 no switchport
 ip address 10.10.4.50 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/0/24
 description to SW-HFA-SRV Gi0/2
 switchport mode access
 switchport access vlan 100
 no shutdown
interface range gi1/0/3 - 22
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
interface vlan 100
 description GW SERVERS
 ip address 10.10.1.3 255.255.255.0
 standby version 2
 standby 100 ip 10.10.1.1
 standby 100 priority 90
 standby 100 preempt
 no shutdown
 exit
```

### 7c — OSPF and SSH
⌨️ Type:
```
router ospf 1
 router-id 10.10.5.2
 passive-interface default
 no passive-interface gi1/0/1
 no passive-interface gi1/0/2
 no passive-interface gi1/0/23
 network 10.10.0.0 0.0.15.255 area 0
 exit
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
→ **`1024`** → then:
```
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
login block-for 120 attempts 3 within 60
ip access-list standard MGMT-ONLY
 permit 10.10.19.0 0.0.0.255
 deny any log
 exit
line vty 0 15
 transport input ssh
 login local
 access-class MGMT-ONLY in
 exec-timeout 5 0
line console 0
 login local
 end
copy running-config startup-config
```
Press **Fast Forward Time** a few times (M18).

✅ On CORE2 you see `%OSPF-5-ADJCHG: Process 1, Nbr 10.10.5.1 on GigabitEthernet1/0/23 from
LOADING to FULL` — **the two cores are OSPF neighbours**.
🔍 **Check now:** `show ip ospf neighbor` → one line, `10.10.5.1`, state `FULL`.
❌ No neighbour → compare the MD5 key on Gi1/0/23 of both cores (`show running-config interface gi1/0/23`).

---

## Mission 8 — DIST1: uplinks + OSPF areas 0 and 1
👉 **Where:** **SWL3-HFA-A-DIST1** → CLI → log in (`admin` / `Intel#Adm1n` since step 07) →
`SWL3-HFA-A-DIST1#`.

DIST1 already has everything from steps 03–07. We **add** to it.

### 8a — Loopback0 and the two uplinks
⌨️ Type:
```
configure terminal
interface loopback 0
 ip address 10.10.5.11 255.255.255.255
interface gi1/1/1
 description to SWL3-HFA-CORE1 Gi1/0/1
 no switchport
 ip address 10.10.4.1 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/1/2
 description to SWL3-HFA-CORE2 Gi1/0/1
 no switchport
 ip address 10.10.4.5 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
```

### 8b — DIST1 and DIST2 are also neighbours inside area 1, over VLAN 99
⌨️ Type:
```
interface vlan 99
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 exit
```

### 8c — OSPF with two areas and the summary
⌨️ Type:
```
router ospf 1
 router-id 10.10.5.11
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface gi1/1/2
 no passive-interface vlan 99
 ! uplinks + loopback -> area 0
 network 10.10.0.0 0.0.15.255 area 0
 ! building A VLANs -> area 1
 network 10.10.16.0 0.0.15.255 area 1
 ! announce building A as ONE route
 area 1 range 10.10.16.0 255.255.240.0
 end
copy running-config startup-config
```
Press **Fast Forward Time**.

✅ Messages `%OSPF-5-ADJCHG: ... Nbr 10.10.5.1 ... to FULL` and `... Nbr 10.10.5.2 ... to FULL`.
🔍 **Check now:** `show ip ospf neighbor` → CORE1 (`10.10.5.1`) and CORE2 (`10.10.5.2`), both `FULL`.

---

## Mission 9 — DIST2: the same, with its own values
👉 **Where:** **SWL3-HFA-A-DIST2** → CLI → log in → `SWL3-HFA-A-DIST2#`.

Differences from DIST1: Loopback0 / router-id **10.10.5.12**; Gi1/1/1 **10.10.4.9** (to CORE1
Gi1/0/2); Gi1/1/2 **10.10.4.13** (to CORE2 Gi1/0/2).

⌨️ Type:
```
configure terminal
interface loopback 0
 ip address 10.10.5.12 255.255.255.255
interface gi1/1/1
 description to SWL3-HFA-CORE1 Gi1/0/2
 no switchport
 ip address 10.10.4.9 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface gi1/1/2
 description to SWL3-HFA-CORE2 Gi1/0/2
 no switchport
 ip address 10.10.4.13 255.255.255.252
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 no shutdown
interface vlan 99
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
 exit
router ospf 1
 router-id 10.10.5.12
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface gi1/1/2
 no passive-interface vlan 99
 network 10.10.0.0 0.0.15.255 area 0
 network 10.10.16.0 0.0.15.255 area 1
 area 1 range 10.10.16.0 255.255.240.0
 end
copy running-config startup-config
```
Press **Fast Forward Time** a few times.

✅ `%OSPF-5-ADJCHG` messages: neighbours CORE1, CORE2 and DIST1 reach `FULL`.

### What the new commands mean
| Command | In simple English |
|---|---|
| `no switchport` | "Stop being a switch port, become a router port" |
| `interface loopback 0` | A virtual interface that is always up — the device's ID card |
| `router ospf 1` | Start OSPF (process number 1 — only matters locally) |
| `router-id 10.10.5.1` | My OSPF name |
| `passive-interface default` | "Don't send Hellos anywhere…" |
| `no passive-interface gi1/0/1` | "…except on this link toward another router" |
| `network 10.10.0.0 0.0.15.255 area 0` | "Interfaces in this range join area 0" |
| `area 1 range 10.10.16.0 255.255.240.0` | "Announce area 1 as one summary route" |
| `ip ospf message-digest-key 1 md5 KEY` + `ip ospf authentication message-digest` | "Only talk OSPF with neighbours that know this key" |

---

## Mission 10 — Check OSPF: neighbours, summary route, pings

### 10a — Neighbours
👉 On **CORE1**: `show ip ospf neighbor`

✅ You should see:
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.10.5.11        1   FULL/DR         00:00:35    10.10.4.1       GigabitEthernet1/0/1
10.10.5.12        1   FULL/DR         00:00:36    10.10.4.9       GigabitEthernet1/0/2
10.10.5.2         1   FULL/BDR        00:00:33    10.10.4.50      GigabitEthernet1/0/23
```
Three neighbours, all **FULL**. (DR/BDR are roles on shared links — ignore them for now. Which
switch is DR depends on which one came up first, so your DR/BDR labels may differ.)

👉 On **DIST1**: `show ip ospf neighbor` → ✅ CORE1, CORE2 **and** DIST2 (over Vlan99).

### 10b — The summary route
👉 On **CORE1**: `show ip route ospf`

✅ You should see:
```
     10.0.0.0/8 is variably subnetted, ...
O IA    10.10.16.0/20 [110/2] via 10.10.4.1, GigabitEthernet1/0/1
                      [110/2] via 10.10.4.9, GigabitEthernet1/0/2
O       10.10.5.11/32 [110/2] via 10.10.4.1, ...
...
```
**O IA** = OSPF route from **another area** (Inter-Area). **One** summary for all of building
A, reachable through **two** paths (DIST1 and DIST2) — load sharing.

👉 On **DIST1**: `show ip ospf interface` → for each interface look at the `Area` line.
✅ Gi1/1/1, Gi1/1/2 and Loopback0 are in **area 0**; the VLANs are in **area 1**.

### 10c — Pings (how? → M17)
| # | From | Type | Expected |
|---|---|---|---|
| 1 | an ENG PC | `ping 10.10.1.1` | ✅ the servers' gateway, through the core |
| 2 | an ENG PC | `tracert 10.10.1.1` | `10.10.16.2` → `10.10.4.2` (or `.6`) → `10.10.1.x` |
| 3 | PC-HFA-ADMIN-01 | `ssh -l admin 10.10.5.1` | ✅ login to CORE1 (local user for now) |

### 10d — See the passive interfaces (Simulation mode — how? → M21)
**Simulation** → **Show All/None** → **Edit Filters** → tick only **OSPF** → close.
Press **Capture / Forward** a few times.

✅ Hello envelopes travel only on the routed links and VLAN 99 — **never** toward the PCs.

Back to **Realtime**. **File → Save**.

---

## Mission 11 — SW-HFA-SRV: the server switch
👉 **Where:** **SW-HFA-SRV** → CLI → **Enter** → `Switch>`.

### 11a — Hardening and VLAN 100
⌨️ Type:
```
enable
configure terminal
hostname SW-HFA-SRV
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
spanning-tree mode rapid-pvst
vlan 100
 name SERVERS
 exit
```

### 11b — Ports and management address
⌨️ Type:
```
interface range gi0/1 - 2
 description uplinks to CORE1 / CORE2
 switchport mode access
 switchport access vlan 100
interface range fa0/1 - 23
 description servers
 switchport mode access
 switchport access vlan 100
 spanning-tree portfast
 spanning-tree bpduguard enable
interface fa0/24
 description to ASA-HFA-FW1 inside (step 11)
 switchport mode access
 switchport access vlan 100
interface vlan 100
 description MANAGEMENT
 ip address 10.10.1.5 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.10.1.1
```

### 11c — SSH
⌨️ Type:
```
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
→ **`1024`** → then:
```
ip ssh version 2
login block-for 120 attempts 3 within 60
ip access-list standard MGMT-ONLY
 permit 10.10.19.0 0.0.0.255
 deny any log
 exit
line vty 0 15
 transport input ssh
 login local
 access-class MGMT-ONLY in
 exec-timeout 5 0
line console 0
 login local
 end
copy running-config startup-config
```
🔍 **Check now:** `ping 10.10.1.1` → ✅ replies (the HSRP gateway on the cores).

---

## Mission 12 — The two servers: addresses and services

### 12a — Addresses (how? → M14)
For each server: click → **Desktop** → **IP Configuration** → **Static**:

| Server | IPv4 Address | Subnet Mask | Default Gateway | DNS Server |
|---|---|---|---|---|
| SRV-HFA-SYSLOG | 10.10.1.10 | 255.255.255.0 | 10.10.1.1 | 10.10.200.11 |
| SRV-HFA-AAA | 10.10.1.11 | 255.255.255.0 | 10.10.1.1 | 10.10.200.11 |

### 12b — One job per server (how? → M20)
On **each** server → **Services** tab → for **HTTP**, **DHCP**, **DNS**, **EMAIL**, **FTP**, **NTP**
→ set each to **Off** (HTTP: switch both HTTP and HTTPS off).

Then:
- **SRV-HFA-SYSLOG:** **Services** → **SYSLOG** → **On**.

✅ From an ENG PC: `ping 10.10.1.10` and `ping 10.10.1.11` → both reply.

---

## Mission 13 — Fill in the AAA server
👉 **Where:** click **SRV-HFA-AAA** → **Services** tab → **AAA** (in the list on the left).

1. **Service:** click **On**.
2. **Network Configuration** (the upper part) — add one line per network device. For each row
   of the table below:
   1. Click in **Client Name**, type the name.
   2. Click in **Client IP**, type the IP.
   3. **Secret**: `Intel#Tac`
   4. **ServerType**: choose **Tacacs**.
   5. Click **Add**. The line appears in the list underneath.

   | Client Name | Client IP |
   |---|---|
   | SW-HFA-A-ACC1 | 10.10.19.11 |
   | SW-HFA-A-ACC2 | 10.10.19.12 |
   | SWL3-HFA-A-DIST1 | 10.10.5.11 |
   | SWL3-HFA-A-DIST2 | 10.10.5.12 |
   | SWL3-HFA-CORE1 | 10.10.5.1 |
   | SWL3-HFA-CORE2 | 10.10.5.2 |
   | SW-HFA-SRV | 10.10.1.5 |

3. **User Setup** (the lower part) — **Username** `admin`, **Password** `Intel#Adm1n` → **Add**.

✅ The client list has 7 lines; the user list has `admin`.
❌ A line has a typo → click it in the list → **Remove** → add it again.

> Why these client IPs? The AAA server only answers devices it knows **by the source IP of
> their request**. Access switches send from their VLAN-99 address. L3 switches would send
> from whichever uplink they use — so we tell them to always use Loopback0 (Mission 14b).

---

## Mission 14 — AAA + Syslog on all 7 switches

### 14a — The block for every switch
Type this block on **all 7**. Log in to each with `admin` / `Intel#Adm1n`.

⌨️ Type:
```
configure terminal
! ---------- Syslog ----------
service timestamps log datetime msec
logging trap informational
logging host 10.10.1.10

! ---------- AAA: ask the TACACS+ server, local user as backup ----------
aaa new-model
tacacs-server host 10.10.1.11 key Intel#Tac
aaa authentication login default group tacacs+ local
line vty 0 15
 login authentication default
line console 0
 login authentication default
 end
copy running-config startup-config
```
> PT older syntax: if `logging host` is refused, type `logging 10.10.1.10`.

Tick each switch when it shows `[OK]`:
- [ ] SW-HFA-A-ACC1
- [ ] SW-HFA-A-ACC2
- [ ] SWL3-HFA-A-DIST1
- [ ] SWL3-HFA-A-DIST2
- [ ] SWL3-HFA-CORE1
- [ ] SWL3-HFA-CORE2
- [ ] SW-HFA-SRV

### 14b — The 4 L3 switches only: talk to AAA from Loopback0
On **DIST1, DIST2, CORE1, CORE2**:

⌨️ Type:
```
configure terminal
ip tacacs source-interface loopback 0
end
copy running-config startup-config
```
> If `ip tacacs source-interface` is refused, register the L3 switches in the AAA server by the
> IPs they send from instead. DIST1 and DIST2 have two equal uplinks, so add **both** for each:
> DIST1 10.10.4.1 and 10.10.4.5, DIST2 10.10.4.9 and 10.10.4.13, CORE1 10.10.1.2, CORE2 10.10.1.3.

### What the new commands mean
| Command | In simple English |
|---|---|
| `logging host 10.10.1.10` | "Send a copy of every message to the Syslog server" |
| `service timestamps log datetime msec` | "Put date and time on every message" |
| `aaa new-model` | "Switch on AAA" |
| `tacacs-server host ... key ...` | "This is the AAA server and our shared secret" |
| `aaa authentication login default group tacacs+ local` | "Check logins with TACACS+; if it doesn't answer, use my local users" |
| `ip tacacs source-interface loopback 0` | "When you ask the AAA server, send from Loopback0" |

---

## Mission 15 — Test AAA, its backup, and Syslog

### 15a — Login through AAA
From **PC-HFA-ADMIN-01**: `ssh -l admin 10.10.5.1` → `Intel#Adm1n`.
✅ You land on `SWL3-HFA-CORE1#` — this time the password was checked by the AAA server.

### 15b — The backup works
1. Click **SRV-HFA-AAA** → **Physical** tab → click its **power button** (light goes off).
2. SSH again from the admin PC to `10.10.5.1`.
   ✅ Still works with `admin` / `Intel#Adm1n` — the switch used its local user (backup). (It
   may take a few seconds longer while the switch waits for the dead server.)
3. Power the AAA server back on.

### 15c — Syslog
1. On **ACC1**: `configure terminal` → `interface fa0/24` → `no shutdown` → `shutdown` → `end`.
2. Click **SRV-HFA-SYSLOG** → **Services** → **SYSLOG**.
   ✅ The messages from ACC1 are listed, with time and source IP.

📸 Screenshot the Syslog list for your book. Then **File → Save**.

---

## Checkpoint
- [ ] CORE1 has 3 OSPF neighbours in FULL state
- [ ] CORE1 sees `O IA 10.10.16.0/20` via two paths
- [ ] PCs reach the servers
- [ ] SSH works through AAA, and still works when the AAA server is off
- [ ] Syslog server receives messages

## If something doesn't work
| Problem | Fix |
|---|---|
| `% Invalid input` on `ip address` under Gi1/1/1 | Type `no switchport` first |
| Neighbours never reach FULL (stuck INIT/EXSTART) or none at all | MD5 key typo on one side; or one side still passive; or IP on the wrong side of the /30 (`show ip interface brief`) |
| `show ip ospf neighbor` empty on DIST2 → DIST1 | VLAN 99 not in `no passive-interface`, or no MD5 on Vlan99 |
| Core doesn't see `10.10.16.0/20` | Missing `network 10.10.16.0 0.0.15.255 area 1` on the DIST switches |
| SSH always uses the local user (AAA ignored) | Client IP in the AAA server doesn't match the request's source IP. See the note in Mission 14b |
| Syslog server shows nothing | Server's SYSLOG service off, or wrong `logging host` IP, or the device can't ping 10.10.1.10 |

---

## 🏋 Exercises

### Exercise A — Kill a core switch ⭐⭐
- 🎯 **Task:** prove that Haifa HQ survives the complete loss of CORE1.
- 📁 **Start from:** your `step08.pkt` — **File → Save** first (you will need it back).
- 👣 **Steps:**
  1. From an ENG PC start `ping -t 10.10.1.10` (the Syslog server).
  2. Delete CORE1 (Delete tool → click CORE1).
  3. Watch the ping. Then on DIST1 run `show ip route ospf` and on CORE2 `show standby brief`.
  4. **File → Open** your saved `step08.pkt` to get CORE1 back (answer **No** to "save changes").
- ✅ **Done when:** you can explain what happened to (a) the ping, (b) DIST1's routes,
  (c) the servers' gateway 10.10.1.1.

<details><summary>💡 Hint</summary>

Before: how many paths did DIST1 have to `10.10.1.0/24`? Which core was HSRP Active for VLAN 100?
</details>
<details><summary>✔ Answer</summary>

DIST1's two equal routes become one (via CORE2); a few pings are lost while OSPF notices the
dead neighbour. CORE2 becomes HSRP **Active** for VLAN 100, so the servers' gateway 10.10.1.1
keeps working. (Had CORE1 only rebooted, `preempt` would give VLAN 100 back to it when it returns.)
</details>

### Exercise B — The wrong key ⭐⭐
- 🎯 **Task:** see what a wrong MD5 key does to an OSPF neighbour.
- 📁 **Start from:** your `step08.pkt` (Save As `step08-exB.pkt`).
- 👣 **Steps:**
  1. On CORE2: `interface gi1/0/23` → `no ip ospf message-digest-key 1 md5 Intel#Ospf` →
     `ip ospf message-digest-key 1 md5 WRONG` → `end`.
  2. Press Fast Forward Time for a while (the dead timer is 40 s).
  3. On CORE1: `show ip ospf neighbor`. Is CORE2 still there?
  4. Put the right key back.
- ✅ **Done when:** you saw the neighbour disappear and come back, and can say why this protects
  the network.

<details><summary>✔ Answer</summary>

After the dead time CORE2 (`10.10.5.2`) disappears from CORE1's neighbour list: the keys don't
match, so the Hellos are rejected. The same thing happens to an attacker's laptop that tries to
inject fake routes without the key (req. 22.18). With the right key back, the neighbour returns
to FULL.
</details>

### Exercise C — Read the routing table ⭐
- 🎯 **Task:** on DIST1, find the route to the server VLAN and explain every part of the line.
- 👣 **Steps:** on DIST1 run `show ip route 10.10.1.0`, then `show ip route ospf`.
- ✅ **Done when:** you can explain `O`, `[110/2]`, `via 10.10.4.2`, and why there are two `via`
  lines.

<details><summary>✔ Answer</summary>

`O` = learned by OSPF inside the same area (area 0 — DIST1 is in area 0 on its uplinks).
`110` = OSPF's administrative distance (how much the router trusts OSPF), `2` = the cost (sum of
link costs). `via 10.10.4.2` = the next hop (CORE1). Two `via` lines = two equal-cost paths
(CORE1 and CORE2) — the traffic is shared between them.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

- [ ] Open `my-project/step07.pkt` → **Save As** `my-project/step08.pkt`
- [ ] Write your own link + loopback table first (like 1.10) from your area-0 block
- [ ] Two cores with power, server switch, Syslog + AAA servers, cabled
- [ ] Cores: hardening, routing, routed links with MD5, server VLAN gateway, OSPF area 0, SSH
- [ ] Distribution: uplinks, OSPF areas 0 + your building area, summary
- [ ] Servers + AAA server filled in; AAA + Syslog on every switch
- [ ] Mission 10 and Mission 15 checks pass

| In the example | In your project — take it from |
|---|---|
| Area 0 block `10.10.0.0/20`, area 1 `10.10.16.0/20` | plan part C (your HQ areas) |
| /30 links `10.10.4.x`, loopbacks `10.10.5.x` | plan part C — make your own link and loopback table from your area-0 block |
| Server VLAN 100 and server IPs (Syslog, AAA) | plan part D (your server VLAN and fixed addresses) |
| OSPF MD5 key `Intel#Ospf`, TACACS+ key `Intel#Tac` | plan part F |
| `area 1 range 10.10.16.0 255.255.240.0` | your building's /20 (summarization — req. 22.15) |

**Check:** `show ip ospf neighbor` shows FULL on every link; `show ip route` shows **one**
summary route per building; login through your AAA server works; your Syslog server receives logs.

💾 Save as `my-project/step08.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★1 What is the advantage of dividing the network into OSPF areas? What is the recommended maximum of routers in an area?**

<details><summary>Answer</summary>

Each router keeps a detailed map (LSDB) only of its own area; between areas only summaries are exchanged. Smaller databases, less CPU, and a change inside one area doesn't make every router recalculate. A common guideline is about **50 routers** per area.

**Show it:** `show ip ospf database`
</details>

**2. ★1 Show me area 0. What is its role? What is it called?**

<details><summary>Answer</summary>

Area 0 is the **backbone**: every other area must connect to it, and traffic between areas passes through it. In this project it holds the core switches, the server farm and the core links.

**Show it:** `show ip ospf interface` on a core switch → Area 0
</details>

**3. ★1 What are the Hello and Dead timers in OSPF?**

<details><summary>Answer</summary>

On Ethernet (broadcast and point-to-point): Hello 10 s, Dead 40 s. Neighbors must have the same timers to become adjacent.

**Show it:** `show ip ospf interface <int>`
</details>

**4. ★1 How do you see OSPF's topology table? And the neighbor table?**

<details><summary>Answer</summary>

Topology (link-state database): `show ip ospf database`. Neighbors: `show ip ospf neighbor`. Routing table: `show ip route ospf`.

**Show it:** those three commands
</details>

**5. ★2 How is the OSPF Router-ID chosen?**

<details><summary>Answer</summary>

1) the `router-id` command; 2) otherwise the highest IP on a loopback; 3) otherwise the highest IP on an active interface. This project sets it with the `router-id` command (the same value as Loopback0).

**Show it:** `show ip protocols`, `show ip ospf`
</details>

**6. ★2 Which router is the DR and which the BDR? How are they chosen? What is the difference?**

<details><summary>Answer</summary>

On broadcast segments (Ethernet) one DR and one BDR are elected: highest OSPF priority, then highest Router-ID. Everyone exchanges updates only with the DR (and BDR), which saves traffic. The BDR is the backup and becomes DR if the DR fails.

**Show it:** `show ip ospf neighbor` → State column (FULL/DR, FULL/BDR, …)
</details>

**7. ★2 If the DR fails, who takes over? What happens when the old DR comes back?**

<details><summary>Answer</summary>

The BDR becomes DR and a new BDR is elected. The election is **not pre-emptive**: when the old DR returns it does not take the role back — it waits until the current DR/BDR fail.

**Show it:** Shut down the DR's interface, then `show ip ospf neighbor`.
</details>

**8. ★2 What is a point-to-point network and what is a broadcast network?**

<details><summary>Answer</summary>

Point-to-point: exactly two routers on the link — no DR/BDR election. Broadcast (Ethernet): many routers can share the segment — a DR/BDR is elected. An Ethernet /30 is type broadcast by default unless you set `ip ospf network point-to-point`.

**Show it:** `show ip ospf interface <int>` → Network Type
</details>

**9. ★2 What is an LSA type 1, and how is it marked in the routing table? And type 3?**

<details><summary>Answer</summary>

Type 1 (Router LSA): every router describes its own links inside its area → routes marked `O`. Type 3 (Summary LSA): sent by an ABR to describe networks of another area → marked `O IA` (inter-area).

**Show it:** `show ip route` on a distribution switch
</details>

**10. ★2 Show me which router is the ABR (area border router).**

<details><summary>Answer</summary>

A router with interfaces in area 0 and in another area. In this project the distribution switches of each building are ABRs (area 0 toward the core, area N toward the building).

**Show it:** `show ip ospf` ("It is an area border router")
</details>

**11. ◆ How is OSPF secured in your project?**

<details><summary>Answer</summary>

MD5 authentication on the routed links (`ip ospf authentication message-digest` + `ip ospf message-digest-key 1 md5 <key>`) — a router without the key can't become a neighbor and inject routes (req. 22.18).

**Show it:** `show ip ospf interface <int>` → "Message digest authentication enabled"
</details>

**12. ★1 What is the role of the NTP server? Show the configuration on the server and on a switch.**

<details><summary>Answer</summary>

It keeps the clocks of all devices the same (UDP 123), so log times match between devices and certificates/keys work. Server: Services → NTP → On. Device: `ntp server <IP>`.

**Show it:** `show ntp status`, `show clock`
</details>

**13. ★1 What is the role of the Syslog server? Which device does it work with?**

<details><summary>Answer</summary>

It collects log messages from all routers and switches in one place (UDP 514), so the admin can see events of the whole network. The devices send to it with `logging host <IP>`.

**Show it:** Server → Services → SYSLOG (the list of messages); `show logging`
</details>

**14. ★1 What is an AAA server? What are its advantages and disadvantages?**

<details><summary>Answer</summary>

Authentication (who are you), Authorization (what may you do), Accounting (what did you do). Advantage: one central user database for all devices; changes in one place; logging. Disadvantage: if it is unreachable nobody can log in — so a local user is kept as a fallback (`… group tacacs+ local`).

**Show it:** `show running-config | include aaa`
</details>

**15. ★1/★2 Which AAA servers exist? What is the difference between RADIUS and TACACS+?**

<details><summary>Answer</summary>

RADIUS (open standard, UDP 1812/1813, encrypts only the password, combines authentication and authorization — common for network access/WiFi). TACACS+ (Cisco, TCP 49, encrypts the whole packet, separates the three A's — common for device administration). This project uses TACACS+ for device logins.

**Show it:** Server → Services → AAA
</details>

**16. ★2 What are the main AAA settings? How do you secure the AAA server?**

<details><summary>Answer</summary>

On the device: `aaa new-model`, the server (`tacacs-server host <IP> key <key>`), and the method list (`aaa authentication login default group tacacs+ local`). On the server: the client (device name + IP + key) and the users. Secure it with a strong shared key, by placing it in the server VLAN behind an ACL, and by keeping a local fallback user.

**Show it:** `show running-config` (aaa / tacacs lines) and the AAA service page
</details>

**17. ★3 Explain the AAA lines on the router and the AAA configuration on the server.**

<details><summary>Answer</summary>

Walk through every `aaa` and `tacacs-server` line on the device, then the AAA page on the server: client name, client IP, key (must match), server type TACACS, then the user list.

**Show it:** both screens side by side
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

In a copy of your finished step-08 file, remove OSPF from one distribution switch and the AAA lines from
one device. Without this page, configure them again from your plan (router-ID, networks/areas,
MD5, summary, AAA with local fallback). Target: **20 minutes**. Then show
`show ip ospf neighbor` (FULL), `show ip route` (one summary per building), an SSH login through
the AAA server, and a log message arriving at the Syslog server.

---

## 👩‍🏫 Teacher notes
- **Lesson 1 (90 min):** 3-tier model and routed links → static vs dynamic routing → OSPF in
  simple words (neighbours, map, shortest path) → Missions 1–7 (build, cable, CORE1, CORE2).
  End of lesson: the two cores are FULL neighbours.
- **Lesson 2 (90 min):** areas & ABRs (country/city comparison) → summarization → Missions 8–10
  → passive interfaces in Simulation mode.
- **Lesson 3 (90 min):** Syslog and AAA (one guard for all doors) → Missions 11–15 → exercises.
- **Use the 🔍 checks.** Each core and distribution mission ends with a neighbour check; a
  student who reaches Mission 10 with no FULL neighbours has skipped them.
- **Common student questions**
  - *"Why do PCs not run OSPF?"* — Only routers need the map. PCs just use their gateway.
  - *"What is `O IA`?"* — A route learned from another area. `O` alone = same area.
  - *"Why is the wildcard 0.0.15.255?"* — A /20 contains 16 /24s → the third number may vary
    by 15.
- **Typical errors:** forgetting `no switchport`; wrong side of a /30 (both ends `.1`);
  copying the MD5 key with a typo; forgetting the 3650 power supplies.
