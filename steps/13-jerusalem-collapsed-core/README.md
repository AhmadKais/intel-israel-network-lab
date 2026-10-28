# Step 13 — Jerusalem: collapsed core, EIGRP, DHCP server, dynamic NAT, CBAC

> ⏱ 4 lessons · 🎯 ministry requirements 5, 10 (dynamic), 17, 20.x, 22.11, 22.13, 22.14, 22.15, 22.18 · 📁 start from: `my-work/step12.pkt` · 💾 save as: `my-work/step13.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 12 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## 🗺 Your missions in this step
**Lesson 1 — build**
- [ ] **Mission 1** — Open step 12, save as step 13, draw the branch (3 min)
- [ ] **Mission 2** — Place the devices (with 2 power supplies!) (15 min)
- [ ] **Mission 3** — Cable the network devices (15 min)
- [ ] **Mission 4** — Cable the end devices (15 min)

**Lesson 2 — the collapsed core**
- [ ] **Mission 5** — CC1, part 1: hardening, VTP, VLANs, STP, PAgP, trunks (15 min)
- [ ] **Mission 6** — CC1, part 2: gateways + DHCP relay (15 min)
- [ ] **Mission 7** — CC1, part 3: EIGRP and management (15 min)
- [ ] **Mission 8** — CC2: the mirror of CC1 (30 min)

**Lesson 3 — access, servers, DHCP**
- [ ] **Mission 9** — The three floor switches (30 min)
- [ ] **Mission 10** — SW-JLM-SRV (10 min)
- [ ] **Mission 11** — The three servers (DHCP server click by click) (20 min)
- [ ] **Mission 12** — End devices + check DHCP through the relay (10 min)

**Lesson 4 — the edge router**
- [ ] **Mission 13** — R-JLM-EDGE, part 1: interfaces and EIGRP (15 min)
- [ ] **Mission 14** — R-JLM-EDGE, part 2: dynamic NAT (10 min)
- [ ] **Mission 15** — R-JLM-EDGE, part 3: CBAC firewall and management (15 min)
- [ ] **Mission 16** — Final tests + the dynamic-NAT limit (15 min)
- [ ] **Exercises** A and B
- [ ] **🛠 Your own project**

## What you will have at the end
The second branch, built **differently** from Haifa (the ministry forbids copying one branch
to another):

| | Haifa | **Jerusalem** |
|---|---|---|
| Design | 3 tiers (access / distribution / core) | **Collapsed core** — 2 tiers |
| Routing | OSPF | **EIGRP** |
| EtherChannel | LACP | **PAgP** |
| DHCP | on the L3 switches | **dedicated DHCP server** |
| NAT | static + PAT | **dynamic NAT** |
| Firewall | ASA | **CBAC** on the edge router |

```
                        R-ISPB-PE (ISP-B)
                              │ 200.2.1.0/30
                         R-JLM-EDGE   (EIGRP + static default + dynamic NAT + CBAC)
                   Gi0/0 ╱          ╲ Gi0/2          10.20.4.0/30 and 10.20.4.4/30
           SWL3-JLM-CC1 ════ Po1 (PAgP) ════ SWL3-JLM-CC2        collapsed core
          ╱    │     │    ╲               ╱    │     │    ╲
     F1-ACC1  F2-ACC1  F3-ACC1  SW-JLM-SRV  (every access switch has a cable to BOTH CCs)
     Floor 1  Floor 2  Floor 3  DHCP/Syslog/AAA servers
```

---

## Part 1 — Understand it first (simple English)

### 1.1 Collapsed core
In Haifa, distribution and core are separate layers — right for a big campus. Jerusalem is
smaller: a separate core would be expensive and add nothing. So the **core and distribution
are "collapsed" into one pair of L3 switches** (CC1, CC2). They are the VLAN gateways **and**
the backbone. Still fully redundant: every access switch has two uplinks, one to each CC.
The ministry requires a collapsed core on L3 switches in at least one branch (req. 5).

### 1.2 EIGRP — a different routing protocol
**EIGRP** is Cisco's routing protocol. Compared with OSPF:

| | OSPF (Haifa) | EIGRP (Jerusalem) |
|---|---|---|
| How it learns | Everyone builds the full **map** | Neighbours tell each other **distances** ("I can reach X at cost 5") |
| Speed on failure | Fast (recalculates the map) | Very fast — keeps a **ready backup path** (feasible successor) |
| Configuration | areas, router-IDs | simpler: one AS number (`100`) |
| Vendors | standard | Cisco (now open, but mostly Cisco) |

Important EIGRP settings:
- `no auto-summary` — otherwise EIGRP would shorten our routes to `10.0.0.0/8` (old behaviour).
- **Manual summary**: the 6 department subnets 10.20.10.0 – 10.20.15.0 fit inside **10.20.8.0/21**
  (remember step 01 exercise 4). We announce that one route toward the edge router:
  `ip summary-address eigrp 100 10.20.8.0 255.255.248.0` (req. 22.15).
- **Authentication** with a **key chain** (MD5) — like OSPF's key in Haifa (req. 22.18).
- **Passive interfaces** — no EIGRP messages toward users.

### 1.3 PAgP EtherChannel
Same idea as Haifa's LACP bundle, but with Cisco's own protocol **PAgP** — mode `desirable` on both
sides. Using both protocols in the project shows you know both.

### 1.4 DHCP from a dedicated server + DHCP relay
Jerusalem's addresses come from a **server** (`SRV-JLM-DHCP`) in the server VLAN, not from the
switches (req. 22.11: routers in one branch, a server in another).

Problem: a PC's DHCP Discover is a **broadcast** — it never leaves its own VLAN, and the server
sits in VLAN 100. Solution: **DHCP relay**. On each user VLAN's gateway we write
`ip helper-address 10.20.1.10`: the switch catches the broadcast and **forwards it as a normal
packet** to the server. The server sees which gateway sent it and chooses the right pool.

Comparison: a receptionist on each floor who takes your request and phones the central office for you.

### 1.5 Dynamic NAT
Jerusalem gets a **pool** of 6 public addresses: `200.2.10.1 – 200.2.10.6`. Each PC that goes
to the internet borrows one address from the pool, one PC per address. The 7th PC at the same
time gets **nothing** — a real limitation you'll test, and the reason most companies use PAT.

### 1.6 CBAC — a firewall inside the router
Jerusalem has no ASA. Its edge router protects it with **CBAC** (`ip inspect`):
1. An inbound ACL on the internet side **blocks everything**.
2. CBAC watches connections **leaving** the branch and opens a temporary hole **only for their replies**.

So employees can browse, but nobody on the internet can start a connection into Jerusalem.
(The ACL already lets in the VPN from Haifa — used in step 15.)

### 1.7 Addresses of this branch
Keep this open — every mission takes its numbers from here.

| VLAN | Name | Subnet | Gateway VIP | CC1 | CC2 | Primary |
|---|---|---|---|---|---|---|
| 41 | SALES | 10.20.10.0/24 | .1 | .2 | .3 | CC1 |
| 42 | LOGISTICS | 10.20.11.0/24 | .1 | .2 | .3 | CC2 |
| 43 | PRODUCTION | 10.20.12.0/24 | .1 | .2 | .3 | CC1 |
| 44 | QA | 10.20.13.0/24 | .1 | .2 | .3 | CC2 |
| 45 | SECURITY | 10.20.14.0/24 | .1 | .2 | .3 | CC1 |
| 46 | ADMIN | 10.20.15.0/24 | .1 | .2 | .3 | CC2 |
| 99 | MGMT | 10.20.0.0/24 | .1 | .2 | .3 | CC1 |
| 100 | SERVERS | 10.20.1.0/24 | .1 | .2 | .3 | CC1 |
| 130 | PRINTERS | 10.20.2.0/24 | .1 | .2 | .3 | CC1 |

| Link / device | Address |
|---|---|
| R-JLM-EDGE Gi0/0 ↔ CC1 Gi1/1/1 | 10.20.4.1 / 10.20.4.2 (/30) |
| R-JLM-EDGE Gi0/2 ↔ CC2 Gi1/1/1 | 10.20.4.5 / 10.20.4.6 (/30) |
| R-JLM-EDGE Gi0/1 ↔ R-ISPB-PE Gi0/1 | 200.2.1.2 / 200.2.1.1 (/30) |
| Loopbacks | CC1 10.20.5.1 · CC2 10.20.5.2 · EDGE 10.20.5.10 |
| Mgmt (VLAN 99) | F1-ACC1 .11 · F2-ACC1 .12 · F3-ACC1 .13 · SW-JLM-SRV .14 · admin PC .100 |
| Servers | DHCP 10.20.1.10 · SYSLOG .11 · AAA .12 |
| Dynamic NAT pool | 200.2.10.1 – 200.2.10.6 |

> **Reminders:** `crypto key generate rsa` → answer **`1024`**. New routers: answer **`no`** to
> the setup dialog. If `login block-for` or `ip tacacs source-interface` is refused, skip that line.

---

## Mission 1 — Open step 12 and save as step 13 (how? → M1, M19)
**File → Open** → `my-work/step12.pkt` → **File → Save As** → `my-work/step13.pkt`.
Draw a rectangle labelled `JERUSALEM BRANCH` with three smaller ones inside: `FLOOR 1`,
`FLOOR 2`, `FLOOR 3`.

---

## Mission 2 — Place the devices (how? → M2, M3, M4, M8)
| Model | Display Name |
|---|---|
| 2911 | `R-JLM-EDGE` |
| 3650-24PS | `SWL3-JLM-CC1`, `SWL3-JLM-CC2` |
| 2960 | `SW-JLM-F1-ACC1`, `SW-JLM-F2-ACC1`, `SW-JLM-F3-ACC1`, `SW-JLM-SRV` |
| Server ×3 | `SRV-JLM-DHCP`, `SRV-JLM-SYSLOG`, `SRV-JLM-AAA` |
| PC ×13 | `PC-JLM-SAL-01/02`, `PC-JLM-LOG-01/02`, `PC-JLM-PRD-01/02`, `PC-JLM-QA-01/02`, `PC-JLM-SEC-01/02`, `PC-JLM-ADM-01/02`, `PC-JLM-ADMIN-01` |
| Printer ×3 | `PRN-JLM-F1-01`, `PRN-JLM-F2-01`, `PRN-JLM-F3-01` |

⚠️ **Power supplies** on both 3650s (Physical → drag **AC-POWER-SUPPLY** into the slot — M8):
- [ ] CC1  - [ ] CC2

> `PC-JLM-ADM-01/02` = the **Admin department** (VLAN 46). `PC-JLM-ADMIN-01` = the **network
> admin's** PC (VLAN 99). Different jobs — don't mix their names.

---

## Mission 3 — Cable the network devices (how? → M5)
| # | Cable | Click first | Port | Click second | Port |
|---|---|---|---|---|---|
| 1 | Cross-Over | SWL3-JLM-CC1 | Gi1/0/1 | SW-JLM-F1-ACC1 | Gi0/1 |
| 2 | Cross-Over | SWL3-JLM-CC2 | Gi1/0/1 | SW-JLM-F1-ACC1 | Gi0/2 |
| 3 | Cross-Over | SWL3-JLM-CC1 | Gi1/0/2 | SW-JLM-F2-ACC1 | Gi0/1 |
| 4 | Cross-Over | SWL3-JLM-CC2 | Gi1/0/2 | SW-JLM-F2-ACC1 | Gi0/2 |
| 5 | Cross-Over | SWL3-JLM-CC1 | Gi1/0/3 | SW-JLM-F3-ACC1 | Gi0/1 |
| 6 | Cross-Over | SWL3-JLM-CC2 | Gi1/0/3 | SW-JLM-F3-ACC1 | Gi0/2 |
| 7 | Cross-Over | SWL3-JLM-CC1 | Gi1/0/4 | SW-JLM-SRV | Gi0/1 |
| 8 | Cross-Over | SWL3-JLM-CC2 | Gi1/0/4 | SW-JLM-SRV | Gi0/2 |
| 9 | Cross-Over | SWL3-JLM-CC1 | Gi1/0/23 | SWL3-JLM-CC2 | Gi1/0/23 |
| 10 | Cross-Over | SWL3-JLM-CC1 | Gi1/0/24 | SWL3-JLM-CC2 | Gi1/0/24 |
| 11 | Straight | SWL3-JLM-CC1 | **Gi1/1/1** | R-JLM-EDGE | **Gi0/0** |
| 12 | Straight | SWL3-JLM-CC2 | **Gi1/1/1** | R-JLM-EDGE | **Gi0/2** |
| 13 | Cross-Over | R-JLM-EDGE | Gi0/1 | R-ISPB-PE | Gi0/1 |

> Switch ↔ router = Straight (different device types). Router ↔ router = Cross-Over.

✅ 13 cables, labels checked (M6).

---

## Mission 4 — Cable the end devices (all Straight-Through)
| Device | Switch | Port |
|---|---|---|
| PC-JLM-SAL-01 / -02 | SW-JLM-F1-ACC1 | Fa0/1 / Fa0/2 |
| PC-JLM-LOG-01 / -02 | SW-JLM-F1-ACC1 | Fa0/11 / Fa0/12 |
| PRN-JLM-F1-01 | SW-JLM-F1-ACC1 | Fa0/21 |
| PC-JLM-ADMIN-01 | SW-JLM-F1-ACC1 | Fa0/22 |
| PC-JLM-PRD-01 / -02 | SW-JLM-F2-ACC1 | Fa0/1 / Fa0/2 |
| PC-JLM-QA-01 / -02 | SW-JLM-F2-ACC1 | Fa0/11 / Fa0/12 |
| PRN-JLM-F2-01 | SW-JLM-F2-ACC1 | Fa0/21 |
| PC-JLM-SEC-01 / -02 | SW-JLM-F3-ACC1 | Fa0/1 / Fa0/2 |
| PC-JLM-ADM-01 / -02 | SW-JLM-F3-ACC1 | Fa0/11 / Fa0/12 |
| PRN-JLM-F3-01 | SW-JLM-F3-ACC1 | Fa0/21 |
| SRV-JLM-DHCP / SYSLOG / AAA | SW-JLM-SRV | Fa0/1 / Fa0/2 / Fa0/3 |

✅ 19 cables. **File → Save**.

---

## Mission 5 — CC1, part 1: hardening, VTP, VLANs, STP, PAgP, trunks
👉 **Where:** **SWL3-JLM-CC1** → CLI → **Enter** → `Switch>`.

### 5a — Hardening + VTP server + VLANs
⌨️ Type:
```
enable
configure terminal
hostname SWL3-JLM-CC1
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp domain JLM
vtp version 2
vtp password Intel#Vtp
vtp mode server
vlan 41
 name SALES
vlan 42
 name LOGISTICS
vlan 43
 name PRODUCTION
vlan 44
 name QA
vlan 45
 name SECURITY
vlan 46
 name ADMIN
vlan 99
 name MGMT
vlan 100
 name SERVERS
vlan 130
 name PRINTERS
vlan 998
 name NATIVE
vlan 999
 name PARKING
 exit
```
🔍 **Check now:** `do show vlan brief` → 11 VLANs with their names.

### 5b — STP, PAgP EtherChannel, trunks, unused ports
CC1 is boss of 41, 43, 45, 99, 100, 130; deputy of 42, 44, 46.

⌨️ Type:
```
spanning-tree mode rapid-pvst
spanning-tree vlan 41,43,45,99,100,130 root primary
spanning-tree vlan 42,44,46 root secondary
interface range gi1/0/23 - 24
 description Po1 member to SWL3-JLM-CC2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 41-46,99,100,130,998
 switchport nonegotiate
 channel-group 1 mode desirable
 no shutdown
interface port-channel 1
 description EtherChannel (PAgP) to SWL3-JLM-CC2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 41-46,99,100,130,998
 switchport nonegotiate
interface range gi1/0/1 - 4
 description TRUNK to access switch
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 41-46,99,100,130,998
 switchport nonegotiate
 spanning-tree guard root
 no shutdown
interface range gi1/0/5 - 22
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
```
| Command | In simple English |
|---|---|
| `channel-group 1 mode desirable` | Join bundle 1 using **PAgP** (Cisco's protocol) |

---

## Mission 6 — CC1, part 2: gateways + DHCP relay
👉 **Where:** CC1, prompt `SWL3-JLM-CC1(config)#`.

🧠 Each user VLAN gets: an address `.2`, an `ip helper-address` (the relay to the DHCP server)
and HSRP. Priority **110** where CC1 is primary (41, 43, 45, 99, 130), **90** where it's deputy.

### 6a — The six department VLANs
⌨️ Type:
```
ip routing
interface vlan 41
 description GW SALES
 ip address 10.20.10.2 255.255.255.0
 ip helper-address 10.20.1.10
 standby version 2
 standby 41 ip 10.20.10.1
 standby 41 priority 110
 standby 41 preempt
 no shutdown
interface vlan 42
 description GW LOGISTICS
 ip address 10.20.11.2 255.255.255.0
 ip helper-address 10.20.1.10
 standby version 2
 standby 42 ip 10.20.11.1
 standby 42 priority 90
 standby 42 preempt
 no shutdown
interface vlan 43
 description GW PRODUCTION
 ip address 10.20.12.2 255.255.255.0
 ip helper-address 10.20.1.10
 standby version 2
 standby 43 ip 10.20.12.1
 standby 43 priority 110
 standby 43 preempt
 no shutdown
interface vlan 44
 description GW QA
 ip address 10.20.13.2 255.255.255.0
 ip helper-address 10.20.1.10
 standby version 2
 standby 44 ip 10.20.13.1
 standby 44 priority 90
 standby 44 preempt
 no shutdown
interface vlan 45
 description GW SECURITY
 ip address 10.20.14.2 255.255.255.0
 ip helper-address 10.20.1.10
 standby version 2
 standby 45 ip 10.20.14.1
 standby 45 priority 110
 standby 45 preempt
 no shutdown
interface vlan 46
 description GW ADMIN
 ip address 10.20.15.2 255.255.255.0
 ip helper-address 10.20.1.10
 standby version 2
 standby 46 ip 10.20.15.1
 standby 46 priority 90
 standby 46 preempt
 no shutdown
```
| Command | In simple English |
|---|---|
| `ip helper-address 10.20.1.10` | "Forward DHCP broadcasts from this VLAN to the DHCP server" (relay) |

### 6b — MGMT and PRINTERS (no relay: static addresses there)
⌨️ Type:
```
interface vlan 99
 description GW MGMT
 ip address 10.20.0.2 255.255.255.0
 standby version 2
 standby 99 ip 10.20.0.1
 standby 99 priority 110
 standby 99 preempt
 no shutdown
interface vlan 130
 description GW PRINTERS
 ip address 10.20.2.2 255.255.255.0
 standby version 2
 standby 130 ip 10.20.2.1
 standby 130 priority 110
 standby 130 preempt
 no shutdown
 exit
```
🔍 **Check now:** `do show ip interface brief` → Vlan41–46, 99, 130 with `.2` addresses. (They may show `down` until the access switches are configured — that's fine.)

---

## Mission 7 — CC1, part 3: EIGRP and management

### 7a — The EIGRP key, VLAN 100 and the uplink
⌨️ Type:
```
key chain EIGRP-KEYS
 key 1
  key-string Intel#Eigrp
  exit
 exit
interface vlan 100
 description GW SERVERS (+ EIGRP peering CC1-CC2)
 ip address 10.20.1.2 255.255.255.0
 standby version 2
 standby 100 ip 10.20.1.1
 standby 100 priority 110
 standby 100 preempt
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 EIGRP-KEYS
 no shutdown
interface loopback 0
 ip address 10.20.5.1 255.255.255.255
interface gi1/1/1
 description to R-JLM-EDGE Gi0/0
 no switchport
 ip address 10.20.4.2 255.255.255.252
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 EIGRP-KEYS
 ip summary-address eigrp 100 10.20.8.0 255.255.248.0
 no shutdown
 exit
```
| Command | In simple English |
|---|---|
| `key chain EIGRP-KEYS` / `key 1` / `key-string ...` | A named key ring holding the EIGRP password |
| `ip authentication mode eigrp 100 md5` + `key-chain ...` | "Use that key on this link" |
| `ip summary-address eigrp 100 10.20.8.0 255.255.248.0` | "Announce the 6 departments as one /21 on this link" |

> VLAN 100 is where CC1 and CC2 meet as EIGRP neighbours, so it gets the key too.

### 7b — EIGRP
⌨️ Type:
```
router eigrp 100
 eigrp router-id 10.20.5.1
 network 10.20.0.0 0.0.255.255
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface vlan 100
 no auto-summary
 exit
```
| Command | In simple English |
|---|---|
| `router eigrp 100` | Start EIGRP, autonomous system 100 (must match on all routers) |
| `network 10.20.0.0 0.0.255.255` | "Every interface in 10.20.x.x joins EIGRP" |
| `no auto-summary` | "Don't shorten my routes to 10.0.0.0/8" |

### 7c — Management
⌨️ Type:
```
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
 remark --- Jerusalem MGMT + Haifa IT (over the VPN) ---
 permit 10.20.0.0 0.0.0.255
 permit 10.10.49.0 0.0.0.255
 deny any log
 exit
service timestamps log datetime msec
logging trap informational
logging host 10.20.1.11
ntp server 10.10.200.14
aaa new-model
tacacs-server host 10.20.1.12 key Intel#Tac
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
> `ntp server 10.10.200.14` — CC1 is the **branch** NTP server: it syncs to Haifa's main NTP
> (works after the VPN, step 15), and every other Jerusalem device syncs to CC1 (10.20.5.1).

✅ `[OK]`.

---

## Mission 8 — CC2: the mirror of CC1
👉 **Where:** **SWL3-JLM-CC2** → CLI → **Enter** → `Switch>`.

| | CC1 | **CC2** |
|---|---|---|
| hostname | SWL3-JLM-CC1 | **SWL3-JLM-CC2** |
| VTP | server + creates VLANs | **transparent → client, no `vlan` commands** |
| STP root primary / secondary | 41,43,45,99,100,130 / 42,44,46 | **42,44,46 / 41,43,45,99,100,130** |
| SVI addresses | .2 | **.3** |
| HSRP priority 110 on | 41, 43, 45, 99, 100, 130 | **42, 44, 46** |
| Loopback / router-id | 10.20.5.1 | **10.20.5.2** |
| Gi1/1/1 | 10.20.4.2 (to EDGE Gi0/0) | **10.20.4.6 (to EDGE Gi0/2)** |
| NTP server | 10.10.200.14 (Haifa) | **10.20.5.1 (CC1)** |

### 8a — Hardening, VTP client, STP, PAgP, trunks
⌨️ Type:
```
enable
configure terminal
hostname SWL3-JLM-CC2
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
vtp domain JLM
vtp version 2
vtp password Intel#Vtp
vtp mode client
spanning-tree mode rapid-pvst
spanning-tree vlan 42,44,46 root primary
spanning-tree vlan 41,43,45,99,100,130 root secondary
interface range gi1/0/23 - 24
 description Po1 member to SWL3-JLM-CC1
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 41-46,99,100,130,998
 switchport nonegotiate
 channel-group 1 mode desirable
 no shutdown
interface port-channel 1
 description EtherChannel (PAgP) to SWL3-JLM-CC1
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 41-46,99,100,130,998
 switchport nonegotiate
interface range gi1/0/1 - 4
 description TRUNK to access switch
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 41-46,99,100,130,998
 switchport nonegotiate
 spanning-tree guard root
 no shutdown
interface range gi1/0/5 - 22
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
```
Press **Fast Forward Time**. 🔍 **Check now:** `do show etherchannel summary` →
`Po1(SU)  PAgP  Gi1/0/23(P) Gi1/0/24(P)`; `do show vlan brief` → the 11 VLANs arrived by VTP.

### 8b — Gateways
⌨️ Type:
```
ip routing
interface vlan 41
 description GW SALES
 ip address 10.20.10.3 255.255.255.0
 ip helper-address 10.20.1.10
 standby version 2
 standby 41 ip 10.20.10.1
 standby 41 priority 90
 standby 41 preempt
 no shutdown
interface vlan 42
 description GW LOGISTICS
 ip address 10.20.11.3 255.255.255.0
 ip helper-address 10.20.1.10
 standby version 2
 standby 42 ip 10.20.11.1
 standby 42 priority 110
 standby 42 preempt
 no shutdown
interface vlan 43
 description GW PRODUCTION
 ip address 10.20.12.3 255.255.255.0
 ip helper-address 10.20.1.10
 standby version 2
 standby 43 ip 10.20.12.1
 standby 43 priority 90
 standby 43 preempt
 no shutdown
interface vlan 44
 description GW QA
 ip address 10.20.13.3 255.255.255.0
 ip helper-address 10.20.1.10
 standby version 2
 standby 44 ip 10.20.13.1
 standby 44 priority 110
 standby 44 preempt
 no shutdown
interface vlan 45
 description GW SECURITY
 ip address 10.20.14.3 255.255.255.0
 ip helper-address 10.20.1.10
 standby version 2
 standby 45 ip 10.20.14.1
 standby 45 priority 90
 standby 45 preempt
 no shutdown
interface vlan 46
 description GW ADMIN
 ip address 10.20.15.3 255.255.255.0
 ip helper-address 10.20.1.10
 standby version 2
 standby 46 ip 10.20.15.1
 standby 46 priority 110
 standby 46 preempt
 no shutdown
interface vlan 99
 description GW MGMT
 ip address 10.20.0.3 255.255.255.0
 standby version 2
 standby 99 ip 10.20.0.1
 standby 99 priority 90
 standby 99 preempt
 no shutdown
interface vlan 130
 description GW PRINTERS
 ip address 10.20.2.3 255.255.255.0
 standby version 2
 standby 130 ip 10.20.2.1
 standby 130 priority 90
 standby 130 preempt
 no shutdown
 exit
```

### 8c — EIGRP
⌨️ Type:
```
key chain EIGRP-KEYS
 key 1
  key-string Intel#Eigrp
  exit
 exit
interface vlan 100
 description GW SERVERS (+ EIGRP peering CC1-CC2)
 ip address 10.20.1.3 255.255.255.0
 standby version 2
 standby 100 ip 10.20.1.1
 standby 100 priority 90
 standby 100 preempt
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 EIGRP-KEYS
 no shutdown
interface loopback 0
 ip address 10.20.5.2 255.255.255.255
interface gi1/1/1
 description to R-JLM-EDGE Gi0/2
 no switchport
 ip address 10.20.4.6 255.255.255.252
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 EIGRP-KEYS
 ip summary-address eigrp 100 10.20.8.0 255.255.248.0
 no shutdown
 exit
router eigrp 100
 eigrp router-id 10.20.5.2
 network 10.20.0.0 0.0.255.255
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface vlan 100
 no auto-summary
 exit
```
✅ `%DUAL-5-NBRCHANGE: EIGRP-IPv4 100: Neighbor 10.20.1.2 (Vlan100) is up` — CC1 and CC2 are
EIGRP neighbours.

### 8d — Management
⌨️ Type:
```
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
 permit 10.20.0.0 0.0.0.255
 permit 10.10.49.0 0.0.0.255
 deny any log
 exit
service timestamps log datetime msec
logging trap informational
logging host 10.20.1.11
ntp server 10.20.5.1
aaa new-model
tacacs-server host 10.20.1.12 key Intel#Tac
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
🔍 **Check now:** `show standby brief` → Active for 42/44/46, Standby for the rest.

---

## Mission 9 — The three floor switches
All three follow the same recipe (step 02–06): VTP client, trunks with snooping trust, user
ports with port-security, printer, management. Only the VLANs and the address change:

| Switch | Fa0/1–10 | Fa0/11–20 | Fa0/21 | Fa0/22 | Mgmt IP |
|---|---|---|---|---|---|
| SW-JLM-F1-ACC1 | 41 SALES | 42 LOGISTICS | 130 printer | **99 admin PC** | 10.20.0.11 |
| SW-JLM-F2-ACC1 | 43 PRODUCTION | 44 QA | 130 printer | unused | 10.20.0.12 |
| SW-JLM-F3-ACC1 | 45 SECURITY | 46 ADMIN | 130 printer | unused | 10.20.0.13 |

### 9a — SW-JLM-F1-ACC1 (Sales + Logistics + admin PC)
👉 **Where:** **SW-JLM-F1-ACC1** → CLI → **Enter** → `Switch>`.

⌨️ **Hardening, VTP, uplinks, user ports:**
```
enable
configure terminal
hostname SW-JLM-F1-ACC1
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
vtp domain JLM
vtp version 2
vtp password Intel#Vtp
vtp mode client
spanning-tree mode rapid-pvst
interface range gi0/1 - 2
 description TRUNK uplink to CC1 / CC2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 41-46,99,100,130,998
 switchport nonegotiate
 ip dhcp snooping trust
 no shutdown
interface range fa0/1 - 10
 description SALES users
 switchport mode access
 switchport access vlan 41
interface range fa0/11 - 20
 description LOGISTICS users
 switchport mode access
 switchport access vlan 42
interface range fa0/1 - 20
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 ip dhcp snooping limit rate 10
 no cdp enable
```
⌨️ **Printer, admin PC, unused, snooping, management address:**
```
interface fa0/21
 description PRN-JLM-F1-01
 switchport mode access
 switchport access vlan 130
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 no cdp enable
interface fa0/22
 description PC-JLM-ADMIN-01 (management)
 switchport mode access
 switchport access vlan 99
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 no cdp enable
interface range fa0/23 - 24
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
ip dhcp snooping
ip dhcp snooping vlan 41-46,99,130
no ip dhcp snooping information option
interface vlan 99
 description MANAGEMENT
 ip address 10.20.0.11 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.20.0.1
```
> `no ip dhcp snooping information option` — **must** be here, or the DHCP server will ignore
> the requests (the option-82 trap from step 06).

⌨️ **Management:**
```
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
 permit 10.20.0.0 0.0.0.255
 permit 10.10.49.0 0.0.0.255
 deny any log
 exit
service timestamps log datetime msec
logging trap informational
logging host 10.20.1.11
ntp server 10.20.5.1
aaa new-model
tacacs-server host 10.20.1.12 key Intel#Tac
aaa authentication login default group tacacs+ local
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
✅ `[OK]`. 🔍 `show vlan brief` → Fa0/1–10 in 41, Fa0/11–20 in 42, Fa0/21 in 130, Fa0/22 in 99.

### 9b — SW-JLM-F2-ACC1 (Production + QA)
👉 **Where:** **SW-JLM-F2-ACC1** → CLI → **Enter** → `Switch>`.

⌨️ **Hardening, VTP, uplinks, user ports:**
```
enable
configure terminal
hostname SW-JLM-F2-ACC1
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
vtp domain JLM
vtp version 2
vtp password Intel#Vtp
vtp mode client
spanning-tree mode rapid-pvst
interface range gi0/1 - 2
 description TRUNK uplink to CC1 / CC2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 41-46,99,100,130,998
 switchport nonegotiate
 ip dhcp snooping trust
 no shutdown
interface range fa0/1 - 10
 description PRODUCTION users
 switchport mode access
 switchport access vlan 43
interface range fa0/11 - 20
 description QA users
 switchport mode access
 switchport access vlan 44
interface range fa0/1 - 20
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 ip dhcp snooping limit rate 10
 no cdp enable
```
⌨️ **Printer, unused, snooping, management address:**
```
interface fa0/21
 description PRN-JLM-F2-01
 switchport mode access
 switchport access vlan 130
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
ip dhcp snooping vlan 41-46,99,130
no ip dhcp snooping information option
interface vlan 99
 description MANAGEMENT
 ip address 10.20.0.12 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.20.0.1
```
⌨️ **Management:** the same block as 9a (from `ip domain-name intel.lab` to
`copy running-config startup-config`). ✅ `[OK]`.

### 9c — SW-JLM-F3-ACC1 (Security + Admin)
👉 **Where:** **SW-JLM-F3-ACC1** → CLI → **Enter** → `Switch>`.

⌨️ **Hardening, VTP, uplinks, user ports:**
```
enable
configure terminal
hostname SW-JLM-F3-ACC1
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
vtp domain JLM
vtp version 2
vtp password Intel#Vtp
vtp mode client
spanning-tree mode rapid-pvst
interface range gi0/1 - 2
 description TRUNK uplink to CC1 / CC2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 41-46,99,100,130,998
 switchport nonegotiate
 ip dhcp snooping trust
 no shutdown
interface range fa0/1 - 10
 description SECURITY users
 switchport mode access
 switchport access vlan 45
interface range fa0/11 - 20
 description ADMIN users
 switchport mode access
 switchport access vlan 46
interface range fa0/1 - 20
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 ip dhcp snooping limit rate 10
 no cdp enable
```
⌨️ **Printer, unused, snooping, management address:**
```
interface fa0/21
 description PRN-JLM-F3-01
 switchport mode access
 switchport access vlan 130
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
ip dhcp snooping vlan 41-46,99,130
no ip dhcp snooping information option
interface vlan 99
 description MANAGEMENT
 ip address 10.20.0.13 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.20.0.1
```
⌨️ **Management:** the same block as 9a. ✅ `[OK]`.

---

## Mission 10 — SW-JLM-SRV
👉 **Where:** **SW-JLM-SRV** → CLI → **Enter** → `Switch>`.

⌨️ Type:
```
enable
configure terminal
hostname SW-JLM-SRV
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
vtp domain JLM
vtp version 2
vtp password Intel#Vtp
vtp mode client
spanning-tree mode rapid-pvst
interface range gi0/1 - 2
 description TRUNK uplink to CC1 / CC2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 41-46,99,100,130,998
 switchport nonegotiate
 no shutdown
interface range fa0/1 - 3
 description servers
 switchport mode access
 switchport access vlan 100
 spanning-tree portfast
 spanning-tree bpduguard enable
interface range fa0/4 - 24
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
interface vlan 99
 description MANAGEMENT
 ip address 10.20.0.14 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.20.0.1
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
→ **`1024`** → then:
```
ip ssh version 2
login block-for 120 attempts 3 within 60
ip access-list standard MGMT-ONLY
 permit 10.20.0.0 0.0.0.255
 permit 10.10.49.0 0.0.0.255
 deny any log
 exit
service timestamps log datetime msec
logging host 10.20.1.11
ntp server 10.20.5.1
aaa new-model
tacacs-server host 10.20.1.12 key Intel#Tac
aaa authentication login default group tacacs+ local
line vty 0 15
 transport input ssh
 login authentication default
 access-class MGMT-ONLY in
 exec-timeout 5 0
line console 0
 login authentication default
 end
copy running-config startup-config
```
> No DHCP snooping here: the **real** DHCP server is plugged into this switch.

---

## Mission 11 — The three servers

### 11a — IP settings (how? → M14)
Desktop → IP Configuration → **Static**, all with mask `255.255.255.0`, gateway `10.20.1.1`,
DNS `10.10.200.11`:

| Server | IP |
|---|---|
| SRV-JLM-DHCP | 10.20.1.10 |
| SRV-JLM-SYSLOG | 10.20.1.11 |
| SRV-JLM-AAA | 10.20.1.12 |

On each: **Services** tab → switch **Off** every service it doesn't need (M20).
🔍 **Check now:** SRV-JLM-DHCP → Command Prompt → `ping 10.20.1.1` → replies.

### 11b — SRV-JLM-SYSLOG
**Services → SYSLOG → On**.

### 11c — SRV-JLM-AAA
**Services → AAA → On**. **Network Configuration** — for each row: Client Name, Client IP, Secret
`Intel#Tac`, ServerType **Tacacs** → **Add**:

| Client Name | Client IP |
|---|---|
| SWL3-JLM-CC1 | 10.20.5.1 |
| SWL3-JLM-CC2 | 10.20.5.2 |
| R-JLM-EDGE | 10.20.5.10 |
| SW-JLM-F1-ACC1 | 10.20.0.11 |
| SW-JLM-F2-ACC1 | 10.20.0.12 |
| SW-JLM-F3-ACC1 | 10.20.0.13 |
| SW-JLM-SRV | 10.20.0.14 |

**User Setup:** Username `admin`, Password `Intel#Adm1n` → **Add**.

### 11d — SRV-JLM-DHCP (click by click)
👉 SRV-JLM-DHCP → **Services** tab → **DHCP** (left list).
1. Click **On** (top right).
2. The built-in pool **serverPool** is selected. Change its fields to the server's own subnet:
   Default Gateway `10.20.1.1`, DNS Server `10.10.200.11`, Start IP Address `10.20.1.100`,
   Subnet Mask `255.255.255.0`, Maximum Number of Users `50` → click **Save**.
3. Now add one pool per department. For **each** row: type the Pool Name and the other fields →
   click **Add** (not Save!):

| Pool Name | Default Gateway | DNS Server | Start IP Address | Subnet Mask | Max Users |
|---|---|---|---|---|---|
| JLM-SALES | 10.20.10.1 | 10.10.200.11 | 10.20.10.11 | 255.255.255.0 | 240 |
| JLM-LOGISTICS | 10.20.11.1 | 10.10.200.11 | 10.20.11.11 | 255.255.255.0 | 240 |
| JLM-PRODUCTION | 10.20.12.1 | 10.10.200.11 | 10.20.12.11 | 255.255.255.0 | 240 |
| JLM-QA | 10.20.13.1 | 10.10.200.11 | 10.20.13.11 | 255.255.255.0 | 240 |
| JLM-SECURITY | 10.20.14.1 | 10.10.200.11 | 10.20.14.11 | 255.255.255.0 | 240 |
| JLM-ADMIN | 10.20.15.1 | 10.10.200.11 | 10.20.15.11 | 255.255.255.0 | 240 |

✅ The table at the bottom shows **7** pools (serverPool + 6).

❌ **Clicked Save instead of Add?** You overwrote the selected pool. Fix that row and click
**Save**, then add the missing one with **Add**.

> **The Default Gateway is the most important field.** The server picks the pool by the relay
> address (e.g. 10.20.10.2) — it uses the pool whose gateway is in the same subnet.
> The DNS server is Haifa's (10.10.200.11) — reachable after the VPN in step 15. Until then, test
> with IP addresses.

---

## Mission 12 — End devices + check DHCP through the relay

### 12a — End devices
- All 12 department PCs → **DHCP** (M16).
- Printers (M15): F1 `10.20.2.11`, F2 `10.20.2.12`, F3 `10.20.2.13`, mask `255.255.255.0`,
  gateway `10.20.2.1`.
- PC-JLM-ADMIN-01 → **Static** `10.20.0.100` / `255.255.255.0` / gateway `10.20.0.1`.

### 12b — Check
| # | Where | Type | ✅ Expected |
|---|---|---|---|
| 1 | PC-JLM-SAL-01 | `ipconfig /all` | `10.20.10.11` (or higher), gateway `10.20.10.1`, DHCP server `10.20.1.10` |
| 2 | PC-JLM-QA-01 | `ipconfig` | `10.20.13.x` |
| 3 | CC1 | `show etherchannel summary` | `Po1(SU)  PAgP  Gi1/0/23(P) Gi1/0/24(P)` |
| 4 | CC1 | `show standby brief` | Active for 41/43/45/99/100/130 |
| 5 | CC1 | `show spanning-tree vlan 41` | "This bridge is the root" |
| 6 | PC-JLM-SAL-01 | `ping 10.20.13.11` (a QA PC) | replies — inter-VLAN routing works |

❌ **PCs get `169.254.x.x`:** `ip helper-address` missing on the SVI; snooping uplinks not
trusted; the `no ip dhcp snooping information option` line missing; server pool typo.
❌ **Address from the wrong pool:** that pool's Default Gateway doesn't match the VLAN's gateway.

💾 **File → Save**.

---

## Mission 13 — R-JLM-EDGE, part 1: interfaces and EIGRP
👉 **Where:** **R-JLM-EDGE** → CLI → `no` to the dialog → `Router>`.

### 13a — Hardening, key, interfaces
⌨️ Type:
```
enable
configure terminal
hostname R-JLM-EDGE
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
key chain EIGRP-KEYS
 key 1
  key-string Intel#Eigrp
  exit
 exit
interface loopback 0
 ip address 10.20.5.10 255.255.255.255
interface gi0/0
 description to SWL3-JLM-CC1
 ip address 10.20.4.1 255.255.255.252
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 EIGRP-KEYS
 ip nat inside
 no shutdown
interface gi0/2
 description to SWL3-JLM-CC2
 ip address 10.20.4.5 255.255.255.252
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 EIGRP-KEYS
 ip nat inside
 no shutdown
interface gi0/1
 description WAN to R-ISPB-PE
 ip address 200.2.1.2 255.255.255.252
 ip nat outside
 no shutdown
 exit
```

### 13b — Static default to the ISP, shared inside with EIGRP
⌨️ Type:
```
ip route 0.0.0.0 0.0.0.0 200.2.1.1
router eigrp 100
 eigrp router-id 10.20.5.10
 network 10.20.4.0 0.0.0.7
 network 10.20.5.10 0.0.0.0
 redistribute static
 no auto-summary
 exit
```
| Command | In simple English |
|---|---|
| `network 10.20.4.0 0.0.0.7` | Both /30 links to the CCs (10.20.4.0–7) join EIGRP |
| `redistribute static` | "Share my static routes (the default to the ISP) with EIGRP" |

Press **Fast Forward Time**.

🔍 **Check now:**
```
R-JLM-EDGE(config)# do show ip eigrp neighbors
H   Address         Interface   Hold Uptime    SRTT   RTO   Q   Seq
0   10.20.4.2       Gi0/0       12   00:01:10  ...
1   10.20.4.6       Gi0/2       13   00:01:05  ...
R-JLM-EDGE(config)# do show ip route eigrp
D     10.20.8.0/21 [90/...] via 10.20.4.2, Gi0/0
                   [90/...] via 10.20.4.6, Gi0/2     <- the summary, two paths
D     10.20.0.0/24 ..., D 10.20.1.0/24 ..., D 10.20.2.0/24 ...
```
And on **CC1**: `show ip route | include 0.0.0.0` → `D*EX 0.0.0.0/0 [170/...] via 10.20.4.1`.
`D` = EIGRP route, `EX` = external (came from `redistribute`).

❌ **No neighbours:** key-chain name or key-string typo; AS number not `100` on one side.
❌ **Six separate /24 instead of the /21:** `ip summary-address` missing on the CC's Gi1/1/1.

---

## Mission 14 — R-JLM-EDGE, part 2: dynamic NAT
⌨️ Type:
```
ip nat pool JLM-POOL 200.2.10.1 200.2.10.6 netmask 255.255.255.248
access-list 10 permit 10.20.0.0 0.0.255.255
ip nat inside source list 10 pool JLM-POOL
```
| Command | In simple English |
|---|---|
| `ip nat pool JLM-POOL 200.2.10.1 200.2.10.6 ...` | The pool of 6 public addresses |
| `ip nat inside source list 10 pool JLM-POOL` | "Translate Jerusalem's addresses using the pool" = **dynamic NAT** |

> Compare with Haifa: there it ends in `interface gi0/1 overload` (PAT). Here it ends in
> `pool JLM-POOL` with **no** `overload` — one public address per PC.

🔍 **Check now:** PC-JLM-SAL-01 → `ping 200.1.100.11` → replies. On the edge:
`do show ip nat translations` → `--- 200.2.10.1  10.20.10.11 ---` — one pool address per PC.

---

## Mission 15 — R-JLM-EDGE, part 3: CBAC firewall and management

### 15a — CBAC
⌨️ Type:
```
ip inspect name JLM-FW tcp
ip inspect name JLM-FW udp
ip inspect name JLM-FW icmp
ip access-list extended WAN-IN
 remark --- VPN from Haifa (step 15) ---
 permit udp host 200.1.1.2 host 200.2.1.2 eq isakmp
 permit esp host 200.1.1.2 host 200.2.1.2
 permit gre host 200.1.1.2 host 200.2.1.2
 remark --- everything else only as CBAC return traffic ---
 deny ip any any
 exit
interface gi0/1
 ip inspect JLM-FW out
 ip access-group WAN-IN in
 exit
```
| Command | In simple English |
|---|---|
| `ip inspect name JLM-FW tcp` (udp, icmp) | "Watch these kinds of outgoing connections" |
| `ip inspect JLM-FW out` | "…on traffic leaving through this interface" |
| `ip access-group WAN-IN in` | "Block everything arriving from the internet, except the VPN and CBAC replies" |

🔍 **Check now:** PC-JLM-SAL-01 → `ping 200.1.100.11` → **still** replies (CBAC opened the way
back). PC-HOME-01 → `ping 200.2.1.2` → ❌ blocked.

### 15b — Management
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
 permit 10.20.0.0 0.0.0.255
 permit 10.10.49.0 0.0.0.255
 deny any log
 exit
service timestamps log datetime msec
logging host 10.20.1.11
ntp server 10.20.5.1
aaa new-model
tacacs-server host 10.20.1.12 key Intel#Tac
aaa authentication login default group tacacs+ local
ip tacacs source-interface loopback 0
line vty 0 15
 transport input ssh
 login authentication default
 access-class MGMT-ONLY in
 exec-timeout 5 0
line console 0
 login authentication default
 end
copy running-config startup-config
```

---

## Mission 16 — Final tests + the dynamic-NAT limit

### 16a — Tests
| # | From | Do | Expected |
|---|---|---|---|
| 1 | PC-JLM-SAL-01 | `ping 200.1.100.11` | ✅ |
| 2 | PC-JLM-SAL-01 | `tracert 200.1.100.11` | 10.20.10.2 → 10.20.4.1 → 200.2.1.1 → … |
| 3 | R-JLM-EDGE | `show ip nat translations` | one pool address per PC |
| 4 | R-JLM-EDGE | `show ip inspect sessions` (while a PC pings) | sessions listed |
| 5 | PC-HOME-01 | `ping 200.2.1.2` | ❌ CBAC drops unsolicited traffic |
| 6 | PC-JLM-ADMIN-01 | `ssh -l admin 10.20.0.12` | ✅ `SW-JLM-F2-ACC1#` |

### 16b — The dynamic-NAT limit
1. On R-JLM-EDGE: `clear ip nat translation *`.
2. Open **7 different** department PCs. On each, start `ping -t 200.1.100.11` (one after the
   other, leave them running).
3. The **7th** PC: `Request timed out`.
4. On R-JLM-EDGE: `show ip nat statistics` → all 6 pool addresses in use.
5. Stop the pings (Ctrl+C) and `clear ip nat translation *`.

📝 Write this observation in the book — it's why most companies use PAT.

💾 `copy running-config startup-config` on every device → **File → Save**.

---

## Checkpoint
- [ ] EIGRP: edge has 2 neighbours and the `/21` summary via both
- [ ] PAgP `Po1(SU)`; HSRP split between CC1 and CC2
- [ ] All PCs get addresses from SRV-JLM-DHCP
- [ ] Internet works through dynamic NAT; inbound blocked by CBAC

## If something doesn't work
| Problem | Fix |
|---|---|
| EIGRP neighbours don't form | AS number (100) different; key-chain name or key-string typo; link not in `network` |
| Routes look like `10.0.0.0/8` | `no auto-summary` missing |
| PCs get `169.254.x.x` | `ip helper-address` missing on the SVI; snooping uplinks not trusted; option-82 line missing; server pool typo |
| PCs get an address from the wrong pool | Pool's *Default Gateway* doesn't match the VLAN's gateway — the server matches pools by the relay address |
| `ip summary-address` refused | It belongs on the routed uplink Gi1/1/1, not on an SVI |
| No internet from Jerusalem | ISP-B's route to 200.2.10.0/29 (step 12, Mission 5b); NAT inside/outside; pool exhausted |
| A 3650 has every link red | Its power supply is missing (Mission 2) |

---

## 🏋 Exercises

### Exercise A — CC1 dies ⭐⭐
- 🎯 **Task:** find out what survives the loss of one collapsed-core switch.
- 📁 **Start from:** your `step13.pkt` — **File → Save** first.
- 👣 **Steps:**
  1. Delete **SWL3-JLM-CC1** (Delete tool → click CC1).
  2. Fast Forward Time a few times.
  3. Test: (a) a Sales PC `ipconfig /release` then `ipconfig /renew`; (b) a QA PC
     `ping 200.1.100.11`; (c) PC-JLM-ADMIN-01 `ssh -l admin 10.20.0.12`.
  4. **File → Open** your saved file (answer **No** to saving) to get CC1 back.
- ✅ **Done when:** you know which of (a), (b), (c) works and **why**, and you found the one
  service that is affected.

<details><summary>💡 Hint</summary>

Which jobs did CC1 do that CC2 also does? Look at the NTP lines of the other devices.
</details>
<details><summary>✔ Answer</summary>

All three work. CC2 takes over HSRP for every VLAN (`.1` stays alive), CC2 also relays DHCP (the helper
is on both), STP moves traffic to the CC2 uplinks, and the edge still reaches 10.20.8.0/21 through
CC2. Only NTP is affected (CC1 was the branch NTP server) — a single point of failure to discuss in
the book (fix: `ntp server 10.20.5.2` as a second server everywhere, with CC2 also syncing to Haifa).
</details>

### Exercise B — Forget the relay ⭐
- 🎯 **Task:** prove that DHCP across VLANs needs `ip helper-address`.
- 📁 **Start from:** `step13.pkt` → **Save As** `step13-exB.pkt`.
- 👣 **Steps:**
  1. On **CC1 and CC2**: `configure terminal` → `interface vlan 44` → `no ip helper-address 10.20.1.10` → `end`.
  2. PC-JLM-QA-01 → `ipconfig /release` → `ipconfig /renew`.
  3. Switch to **Simulation** mode (M21), filter **DHCP**, renew again and watch where the packet stops.
- ✅ **Done when:** you can say where the Discover stopped and why.

<details><summary>✔ Answer</summary>

The PC gets `169.254.x.x`. The Discover is a broadcast — it reaches the SVI of VLAN 44 and stops
there, because routers (and L3 switches) never forward broadcasts. Without the helper, nobody
turns it into a unicast to 10.20.1.10.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

This is the example's **second branch**. Build **your** second site from plan parts C–E:

- [ ] Open `my-project/step12.pkt` → **Save As** `my-project/step13.pkt`
- [ ] Write the address table first (like 1.7)
- [ ] Place (power supplies!) and cable the site
- [ ] The two collapsed-core switches; check PAgP, HSRP, EIGRP neighbours
- [ ] Access switches, server switch, servers (DHCP pools!)
- [ ] PCs get addresses through the relay
- [ ] Edge router: EIGRP + summary, dynamic NAT, CBAC; the test table of Mission 16

| In the example | In your project — take it from |
|---|---|
| Collapsed core on 2 L3 switches | the ministry requires collapsed core in at least one site (req. 5) — plan part E |
| `10.20.0.0/16`, VLANs 41–46, 99, 100, 130 | plan parts C and D (site 2) |
| EIGRP AS 100, summary `10.20.8.0/21` | your AS number; the summary that covers **your** department subnets |
| PAgP port-channel | plan part E (EtherChannel row) |
| DHCP server + `ip helper-address` | plan part E (DHCP row) |
| Dynamic NAT pool `JLM-POOL`, CBAC `JLM-FW` | your pool (your public block), your names |

This site must **not** be a copy of your HQ (ministry rule ג): different design model, IGP,
NAT type and firewall method.

**Check:** `show ip eigrp neighbors`; `show ip route` on the edge shows your summary; PCs get
DHCP from the server; internet browsing uses the dynamic pool.

💾 Save as `my-project/step13.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★1 Which metric parameters does EIGRP use? What is its algorithm called?**

<details><summary>Answer</summary>

Bandwidth and delay by default (load and reliability can be added with the K values). The algorithm is **DUAL** (Diffusing Update Algorithm).

**Show it:** `show ip protocols` → K values
</details>

**2. ★1 What are the Hello and Hold times of EIGRP? To which address are updates sent?**

<details><summary>Answer</summary>

On fast links Hello 5 s, Hold 15 s. Updates go to the multicast address **224.0.0.10**.

**Show it:** `show ip eigrp neighbors` → Hold column
</details>

**3. ★1 Explain Feasible Distance and Reported Distance, Successor and Feasible Successor, and the feasibility condition.**

<details><summary>Answer</summary>

Reported Distance = the neighbor's own cost to the destination. Feasible Distance = the best total cost from this router. Successor = the neighbor of the best route (in the routing table). Feasible Successor = a backup neighbor that meets the **feasibility condition**: its RD is lower than the current FD. The condition guarantees the backup path has no loop, so it can be used immediately.

**Show it:** `show ip eigrp topology`
</details>

**4. ★2 Which three tables does EIGRP keep, and which commands show them?**

<details><summary>Answer</summary>

Neighbor table `show ip eigrp neighbors`, topology table `show ip eigrp topology`, routing table `show ip route eigrp`.

**Show it:** those three commands
</details>

**5. ★2 What is `passive-interface` and where do you configure it?**

<details><summary>Answer</summary>

Under `router eigrp`: the interface's network is still advertised, but no hellos are sent on it, so no neighbor can form there — used on user-facing interfaces/SVIs. This project uses `passive-interface default` and then `no passive-interface` only on the links to neighbors.

**Show it:** `show ip protocols` → Passive Interface(s)
</details>

**6. ★2 What are EIGRP's three administrative distances?**

<details><summary>Answer</summary>

Internal 90, external (redistributed) 170, summary 5.

**Show it:** `show ip route` → [90/…]
</details>

**7. ★2 How do you do unequal-cost load balancing in EIGRP?**

<details><summary>Answer</summary>

`variance N` under `router eigrp`: routes through feasible successors whose metric is up to N times the best are also used, in proportion to their cost.

**Show it:** —
</details>

**8. ★2 How do you connect two different routing protocols, e.g. OSPF and EIGRP?**

<details><summary>Answer</summary>

Redistribution on a router that runs both: under `router eigrp` → `redistribute ospf 1 metric …`, and under `router ospf` → `redistribute eigrp 100 subnets`.

**Show it:** —
</details>

**9. ◆ How did you summarize and secure EIGRP?**

<details><summary>Answer</summary>

Summary on the interface toward the edge: `ip summary-address eigrp 100 <network> <mask>` (req. 22.15). Security: a key chain and `ip authentication mode eigrp 100 md5` (req. 22.18).

**Show it:** `show ip route` on the edge router → one summary
</details>

**10. ★2 What does `ip helper-address` mean? Why is it used, and on which interface?**

<details><summary>Answer</summary>

DHCP Discover is a broadcast and doesn't cross a router. `ip helper-address <server IP>` on the gateway interface **of the clients' VLAN** (the SVI) forwards their DHCP broadcasts as unicast to the DHCP server in another subnet.

**Show it:** `show running-config interface vlan <x>`
</details>

**11. ★1 Show me the DHCP configuration on the dedicated server and explain the fields.**

<details><summary>Answer</summary>

Services → DHCP: one pool per VLAN — pool name, default gateway (`.1`), DNS server, start IP, subnet mask, maximum users. The server answers relayed requests according to the gateway address in the request.

**Show it:** Server → Services → DHCP
</details>

**12. ◆ What is CBAC?**

<details><summary>Answer</summary>

Context-Based Access Control (`ip inspect`): the router remembers sessions that started inside and opens the return traffic for them only; everything else from outside is blocked by the ACL (req. 22.13).

**Show it:** `show running-config` → `ip inspect` lines on the edge router
</details>

**13. ◆ PAgP vs LACP?**

<details><summary>Answer</summary>

Both negotiate an EtherChannel. PAgP is Cisco-only (`desirable`/`auto`); LACP is the open standard (`active`/`passive`). This branch uses PAgP so it differs from HQ.

**Show it:** `show etherchannel summary` → Protocol
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

In a copy of your file, remove EIGRP from one collapsed-core switch and the `ip helper-address`
lines. Without this page, configure them again from your plan (AS, networks, passive
interfaces, summary, authentication, helper). Target: **15 minutes**. Then show the three EIGRP
tables and a PC getting an address from the DHCP server.

---

## 👩‍🏫 Teacher notes
- **Lesson 1:** collapsed core vs 3-tier (cost vs scale); EIGRP vs OSPF table; Missions 1–4
  (build and cable). Check both power supplies.
- **Lesson 2:** Missions 5–8 — CC1 and CC2 (long — split the class in pairs: one types CC1, the
  other CC2, then swap and compare).
- **Lesson 3:** Missions 9–12 — access switches, server switch, servers; DHCP relay demo in
  Simulation mode (filter **DHCP**: the broadcast stops at the SVI and continues as unicast to
  10.20.1.10).
- **Lesson 4:** Missions 13–16 — edge router (EIGRP, dynamic NAT, CBAC); tests; the 7-PC NAT
  limit test; exercises.
- **Common student questions**
  - *"Why not copy Haifa's design?"* — The ministry forbids identical branches, and real companies
    choose the design that fits each site's size.
  - *"How does the DHCP server know which pool?"* — The relay puts the SVI's address (e.g. 10.20.10.2)
    in the request; the server picks the pool whose gateway is in that subnet.
- **Typical errors:** PAgP on one side and LACP on the other; forgetting `no auto-summary`; summary
  on the SVI; helper missing on CC2; forgetting the option-82 line on the access switches.
