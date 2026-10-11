# Step 09 — Buildings B & C + the OSPF virtual link

> ⏱ 4 lessons · 🎯 ministry requirements 8, 22.16, 22.15, 20.1, 20.2 · 📁 start from: `my-work/step08.pkt` · 💾 save as: `my-work/step09.pkt`

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

> **How to use this step in class:** buildings B and C repeat everything from steps 02–08.
> Stronger students can try each device **before** looking at its configuration below — it's
> the best revision of the course. Everyone else types it from the guide.

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

You'll **see** this happen first, then fix it.

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

---

## Part 2 — Prepare
**File → Open** → `my-work/step08.pkt` → **File → Save As** → `my-work/step09.pkt`.

---

## Part 3 — Build (click by click)

### 3.1 Building B devices (in the "BUILDING B" rectangle)
| Model | Display Name |
|---|---|
| 3650-24PS (+ **AC-POWER-SUPPLY**!) | `SWL3-HFA-B-DIST1`, `SWL3-HFA-B-DIST2` |
| 2960 | `SW-HFA-B-ACC1`, `SW-HFA-B-ACC2` |
| PC ×4 | `PC-HFA-HR-01`, `PC-HFA-HR-02`, `PC-HFA-MKT-01`, `PC-HFA-MKT-02` |
| Printer | `PRN-HFA-B-01` |

### 3.2 Building C devices (in the "BUILDING C" rectangle)
| Model | Display Name |
|---|---|
| 3650-24PS (+ **AC-POWER-SUPPLY**!) | `SWL3-HFA-C-DIST1`, `SWL3-HFA-C-DIST2` |
| 2960 | `SW-HFA-C-ACC1`, `SW-HFA-C-ACC2` |
| PC ×4 | `PC-HFA-RND-01`, `PC-HFA-RND-02`, `PC-HFA-IT-01`, `PC-HFA-IT-02` |
| Printer | `PRN-HFA-C-01` |

### 3.3 Cables — building B (switch ↔ switch: **Cross-Over**; end device ↔ switch: **Straight-Through**)
| Click first | Port | Click second | Port |
|---|---|---|---|
| SW-HFA-B-ACC1 | Gi0/1 | SWL3-HFA-B-DIST1 | Gi1/0/1 |
| SW-HFA-B-ACC1 | Gi0/2 | SWL3-HFA-B-DIST2 | Gi1/0/1 |
| SW-HFA-B-ACC2 | Gi0/1 | SWL3-HFA-B-DIST1 | Gi1/0/2 |
| SW-HFA-B-ACC2 | Gi0/2 | SWL3-HFA-B-DIST2 | Gi1/0/2 |
| SWL3-HFA-B-DIST1 | Gi1/0/23 | SWL3-HFA-B-DIST2 | Gi1/0/23 |
| SWL3-HFA-B-DIST1 | Gi1/0/24 | SWL3-HFA-B-DIST2 | Gi1/0/24 |
| SWL3-HFA-B-DIST1 | **Gi1/1/1** | SWL3-HFA-CORE1 | **Gi1/0/3** |
| SWL3-HFA-B-DIST1 | **Gi1/1/2** | SWL3-HFA-CORE2 | **Gi1/0/3** |
| SWL3-HFA-B-DIST2 | **Gi1/1/1** | SWL3-HFA-CORE1 | **Gi1/0/4** |
| SWL3-HFA-B-DIST2 | **Gi1/1/2** | SWL3-HFA-CORE2 | **Gi1/0/4** |
| PC-HFA-HR-01 | Fa0 | SW-HFA-B-ACC1 | Fa0/1 |
| PC-HFA-HR-02 | Fa0 | SW-HFA-B-ACC2 | Fa0/1 |
| PC-HFA-MKT-01 | Fa0 | SW-HFA-B-ACC1 | Fa0/11 |
| PC-HFA-MKT-02 | Fa0 | SW-HFA-B-ACC2 | Fa0/11 |
| PRN-HFA-B-01 | Fa0 | SW-HFA-B-ACC1 | Fa0/21 |

### 3.4 Cables — building C (its uplinks go to **building B**)
| Click first | Port | Click second | Port |
|---|---|---|---|
| SW-HFA-C-ACC1 | Gi0/1 | SWL3-HFA-C-DIST1 | Gi1/0/1 |
| SW-HFA-C-ACC1 | Gi0/2 | SWL3-HFA-C-DIST2 | Gi1/0/1 |
| SW-HFA-C-ACC2 | Gi0/1 | SWL3-HFA-C-DIST1 | Gi1/0/2 |
| SW-HFA-C-ACC2 | Gi0/2 | SWL3-HFA-C-DIST2 | Gi1/0/2 |
| SWL3-HFA-C-DIST1 | Gi1/0/23 | SWL3-HFA-C-DIST2 | Gi1/0/23 |
| SWL3-HFA-C-DIST1 | Gi1/0/24 | SWL3-HFA-C-DIST2 | Gi1/0/24 |
| SWL3-HFA-C-DIST1 | **Gi1/1/1** | SWL3-HFA-B-DIST1 | **Gi1/1/3** |
| SWL3-HFA-C-DIST1 | **Gi1/1/2** | SWL3-HFA-B-DIST2 | **Gi1/1/3** |
| SWL3-HFA-C-DIST2 | **Gi1/1/1** | SWL3-HFA-B-DIST1 | **Gi1/1/4** |
| SWL3-HFA-C-DIST2 | **Gi1/1/2** | SWL3-HFA-B-DIST2 | **Gi1/1/4** |
| PC-HFA-RND-01 | Fa0 | SW-HFA-C-ACC1 | Fa0/1 |
| PC-HFA-RND-02 | Fa0 | SW-HFA-C-ACC2 | Fa0/1 |
| PC-HFA-IT-01 | Fa0 | SW-HFA-C-ACC1 | Fa0/11 |
| PC-HFA-IT-02 | Fa0 | SW-HFA-C-ACC2 | Fa0/11 |
| PRN-HFA-C-01 | Fa0 | SW-HFA-C-ACC1 | Fa0/21 |

### 3.5 Address plan of this step
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

### 3.6 The final management guest list
From now on, network devices may be managed from **all three MGMT VLANs** and the **IT
department**. All new devices get this ACL; in Part 7 you update the old ones:
```
ip access-list standard MGMT-ONLY
 permit 10.10.19.0 0.0.0.255
 permit 10.10.35.0 0.0.0.255
 permit 10.10.51.0 0.0.0.255
 permit 10.10.49.0 0.0.0.255
 deny any log
```

---

## Part 4 — Configure building B

Log in to each new switch with **Enter** (no passwords yet). When a block contains
`crypto key generate rsa`, answer **`1024`**.

### 4.1 SWL3-HFA-B-DIST1 — complete
```
enable
configure terminal
! ===== hardening =====
hostname SWL3-HFA-B-DIST1
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
! ===== VTP server of building B + VLANs =====
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
! ===== Spanning Tree =====
spanning-tree mode rapid-pvst
spanning-tree vlan 40,60,99 root primary
spanning-tree vlan 50 root secondary
! ===== trunks to the access switches + Root Guard =====
interface range gi1/0/1 - 2
 description TRUNK to access switch
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 40,50,60,99,998
 switchport nonegotiate
 spanning-tree guard root
 no shutdown
! ===== EtherChannel to B-DIST2 =====
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
! ===== unused =====
interface range gi1/0/3 - 22
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
! ===== gateways (SVIs + HSRP) =====
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
! ===== DHCP split scope: this switch gives .11-.130 =====
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
! ===== routed uplinks: to the core (area 0) and to building C (area 2) =====
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
! ===== OSPF: ABR between area 0 and area 2 =====
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
! ===== management: SSH, ACL, AAA, Syslog =====
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
(`1024`)
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

### 4.2 SWL3-HFA-B-DIST2 — complete
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
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
(`1024`)
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

### 4.3 SW-HFA-B-ACC1 — complete
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
! ===== uplinks =====
interface range gi0/1 - 2
 description TRUNK uplink to B-DIST1 / B-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 40,50,60,99,998
 switchport nonegotiate
 ip dhcp snooping trust
 no shutdown
! ===== users =====
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
! ===== printer =====
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
! ===== unused =====
interface range fa0/22 - 24
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
! ===== DHCP snooping =====
ip dhcp snooping
ip dhcp snooping vlan 40,50,60,99
no ip dhcp snooping information option
! ===== management =====
interface vlan 99
 description MANAGEMENT
 ip address 10.10.35.11 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.10.35.1
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
(`1024`)
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

### 4.4 SW-HFA-B-ACC2 — complete
Exactly the same as 4.3, with **two changes**:
- `hostname SW-HFA-B-ACC2`
- `ip address 10.10.35.12 255.255.255.0` under `interface vlan 99`

(Type the whole block again from 4.3 with these two lines changed.)

### 4.5 CORE1 and CORE2 — open the ports toward building B
**SWL3-HFA-CORE1:**
```
enable
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
**SWL3-HFA-CORE2:** the same, with `10.10.4.22` on Gi1/0/3 (description `to SWL3-HFA-B-DIST1 Gi1/1/2`)
and `10.10.4.30` on Gi1/0/4 (description `to SWL3-HFA-B-DIST2 Gi1/1/2`).

### 4.6 Building B end devices
- **PCs** (HR-01/02, MKT-01/02): Desktop → IP Configuration → **DHCP**.
- **PRN-HFA-B-01**: Config → FastEthernet0 → IP `10.10.34.10`, mask `255.255.255.0`;
  Config → Settings → Gateway `10.10.34.1`.

### 4.7 Check building B before you continue
- CORE1: `show ip route ospf` → `O IA 10.10.32.0/20` via B-DIST1 **and** B-DIST2 ✅
- HR PC gets `10.10.32.x`; `ping` an Engineering PC in building A ✅
- B-DIST1: `show standby brief`, `show etherchannel summary`, `show ip ospf neighbor` (CORE1, CORE2, B-DIST2)

---

## Part 5 — Configure building C (first WITHOUT the virtual link)

### 5.1 SWL3-HFA-C-DIST1 — complete
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
! ===== loopback inside AREA 3's block =====
interface loopback 0
 ip address 10.10.63.1 255.255.255.255
! ===== uplinks to building B (area 2) =====
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
! ===== OSPF: ABR between area 2 and area 3 (no area 0 here!) =====
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
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
(`1024`)
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

### 5.2 SWL3-HFA-C-DIST2 — complete
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
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
(`1024`)
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

### 5.3 SW-HFA-C-ACC1 — complete
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
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
(`1024`)
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

### 5.4 SW-HFA-C-ACC2 — complete
Same as 5.3 with: `hostname SW-HFA-C-ACC2` and `ip address 10.10.51.12 255.255.255.0`.

### 5.5 Building C end devices
- PCs (RND-01/02, IT-01/02) → **DHCP**.
- PRN-HFA-C-01 → IP `10.10.50.10` / `255.255.255.0`, gateway `10.10.50.1`.

### 5.6 See the problem 🔍
Press **Fast Forward Time** a few times, then:
- **B-DIST1** `show ip ospf neighbor` → C-DIST1 and C-DIST2 are FULL neighbours (area 2 works).
- **CORE1** `show ip route | include 10.10.48` → **nothing**. Area 3 is invisible to the core.
- **PC-HFA-RND-01** → `ping 10.10.1.10` (Syslog server) → ❌ Request timed out.
- **C-DIST1** `show ip route` → no `O IA` routes from areas 0 and 1 — the island is complete.

This is the rule from 1.2 in action. Now fix it.

---

## Part 6 — The virtual links

**SWL3-HFA-B-DIST1** and **SWL3-HFA-B-DIST2** (ABRs 0/2) — type on both:
```
enable
configure terminal
router ospf 1
 area 2 virtual-link 10.10.63.1
 area 2 virtual-link 10.10.63.2
 end
copy running-config startup-config
```
**SWL3-HFA-C-DIST1** and **SWL3-HFA-C-DIST2** (ABRs 2/3) — type on both:
```
enable
configure terminal
router ospf 1
 area 2 virtual-link 10.10.5.21
 area 2 virtual-link 10.10.5.22
 end
copy running-config startup-config
```
Press **Fast Forward Time**. You'll see `%OSPF-5-ADJCHG: Process 1, Nbr 10.10.63.1 on OSPF_VL0 from LOADING to FULL`.

| Command | In simple English |
|---|---|
| `area 2 virtual-link 10.10.63.1` | "Build a piece of area 0 **through area 2**, to the router whose router-ID is 10.10.63.1" |

---

## Part 7 — Update the older devices

### 7.1 Final management ACL on the 7 older devices
On **ACC1, ACC2, DIST1, DIST2 (building A), CORE1, CORE2, SW-HFA-SRV**:
```
enable
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

Check: `show access-lists MGMT-ONLY` → 4 permits, then the deny.

### 7.2 Register the 8 new devices on SRV-HFA-AAA
SRV-HFA-AAA → **Services → AAA → Network Configuration** → add (secret `Intel#Tac`, type **Tacacs**):

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

---

## Part 8 — Check your work

### 8.1 Virtual links
On **B-DIST1**: `show ip ospf virtual-links`
```
Virtual Link OSPF_VL0 to router 10.10.63.1 is up
  Transit area 2, via interface GigabitEthernet1/1/3, Cost of using 1
  ...
Virtual Link OSPF_VL1 to router 10.10.63.2 is up
```

### 8.2 The core now sees every building as one route
On **CORE1**: `show ip route ospf`
```
O IA  10.10.16.0/20 ...   <- building A (area 1)
O IA  10.10.32.0/20 ...   <- building B (area 2)
O IA  10.10.48.0/20 ...   <- building C (area 3) — arrived through the virtual link!
```

### 8.3 Pings and SSH
| From | Type | Expected |
|---|---|---|
| PC-HFA-RND-01 | `ping 10.10.1.10` | ✅ (failed before the virtual link) |
| PC-HFA-IT-01 | `ping` an Engineering PC in building A | ✅ |
| PC-HFA-HR-01 | `ping` an R&D PC | ✅ |
| PC-HFA-IT-01 | `ssh -l admin 10.10.19.11` (ACC1 in building A) | ✅ IT may manage devices |
| PC-HFA-RND-01 | `ssh -l admin 10.10.51.11` | ❌ refused (not on the guest list) |

---

## Part 9 — Save
`copy running-config startup-config` on all changed switches → **File → Save**.

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

## Challenge
Remove the area 2 summary on B-DIST1 and B-DIST2 (`router ospf 1` → `no area 2 range 10.10.32.0 255.255.240.0`).
Look at CORE1's routing table. How many more lines do you see, and why are the `10.10.36.x`
links among them? Put the summary back.

<details><summary>Answer</summary>

Without the summary, every area-2 network is announced separately as `O IA`: the 4 VLANs
(40, 50, 60, 99) **and** the four /30 links to building C — about 8 lines instead of 1. The /30
links are area-2 networks too. With the summary they're hidden inside 10.10.32.0/20 — that's
why we numbered them 10.10.36.x and not 10.10.4.x.
</details>

---

## 👩‍🏫 Teacher notes
- **Lesson 1:** revision game — students say which step each block of B-DIST1's config comes
  from (hardening = 02, VTP = 03, STP/EtherChannel = 04, SVIs/HSRP/DHCP = 05, …). Then build
  and cable both buildings.
- **Lesson 2:** configure building B; check 4.7.
- **Lesson 3:** configure building C; **stop at 5.6** and let the class discover the problem.
  Ask: "OSPF neighbours are FULL — so why doesn't the core see area 3?" Lead them to the
  "every area must touch area 0" rule.
- **Lesson 4:** virtual links → checks → final ACL → challenge.
- **Common student questions**
  - *"Why not just connect building C to the core?"* — In a real campus that's the better design!
    We build it this way because the ministry requires a virtual link (22.16). Say so in the book.
  - *"Why use the router-ID and not an IP address?"* — The virtual link is between two *routers*,
    not two interfaces. OSPF identifies routers by router-ID.
- **Typical errors:** missing power supplies (4 new 3650s!); wrong /30 side on the B↔C links;
  forgetting to update the ACL on the old devices.
