# Step 09 — Buildings B & C (you build them) + OSPF virtual link

## Goal
This is your **first solo build**. Using the patterns from steps 02–08, build buildings B
and C. Then solve a real OSPF problem: building C (area 3) is connected only to building B
(area 2) — **not** to area 0. OSPF forbids that… unless you build a **virtual link**.

Ministry requirements: **8, 22.16, 22.15, 20.1, 20.2 (6 departments)**

> Try every block yourself first. Each answer is hidden in a `<details>` block — open it
> only to check, or when you are really stuck.

## Part 1 — Building B (area 2): copy the pattern

| Item | Value |
|---|---|
| Devices | `SWL3-HFA-B-DIST1`, `SWL3-HFA-B-DIST2` (3650 + PSU), `SW-HFA-B-ACC1`, `SW-HFA-B-ACC2` (2960) |
| VTP domain | `HFA-B`, DIST1 server |
| VLANs | 40 HR, 50 MARKETING, 60 PRINTERS-B, 99 MGMT-B, 998, 999 |
| Access ports | Fa0/1-10 → 40 · Fa0/11-20 → 50 · Fa0/21 → 60 · Fa0/22-24 parking |
| STP/HSRP primary | DIST1: 40, 60, 99 · DIST2: 50 |
| Subnets | see [design.md](../../docs/design.md) §3 (10.10.32.0 – 10.10.35.0) |
| Mgmt IPs | DIST1 .2, DIST2 .3, ACC1 10.10.35.11, ACC2 10.10.35.12 |
| Loopbacks | DIST1 10.10.5.21, DIST2 10.10.5.22 |
| DHCP | split scope on DIST1/DIST2 for VLANs 40 and 50 |

Cabling (same shape as building A):

| From | To | Subnet |
|---|---|---|
| B-ACC1 Gi0/1 / Gi0/2 | B-DIST1 Gi1/0/1 / B-DIST2 Gi1/0/1 | trunk |
| B-ACC2 Gi0/1 / Gi0/2 | B-DIST1 Gi1/0/2 / B-DIST2 Gi1/0/2 | trunk |
| B-DIST1 Gi1/0/23-24 | B-DIST2 Gi1/0/23-24 | LACP Po1 |
| B-DIST1 Gi1/1/1 | CORE1 Gi1/0/3 | 10.10.4.16/30 (.17 / .18) |
| B-DIST1 Gi1/1/2 | CORE2 Gi1/0/3 | 10.10.4.20/30 (.21 / .22) |
| B-DIST2 Gi1/1/1 | CORE1 Gi1/0/4 | 10.10.4.24/30 (.25 / .26) |
| B-DIST2 Gi1/1/2 | CORE2 Gi1/0/4 | 10.10.4.28/30 (.29 / .30) |

PCs: 2 per department for now (`PC-HFA-HR-01/02`, `PC-HFA-MKT-01/02`), a printer
`PRN-HFA-B-01` (10.10.34.10), DHCP for PCs.

**Checklist for each switch** — tick as you go:
- [ ] hardening (hostname, secret, encryption, banner, console)
- [ ] VTP, VLANs, access ports, parking, trunks (native 998, allowed list, nonegotiate)
- [ ] Rapid-PVST, root primary/secondary, PortFast + BPDU Guard, Root Guard, LACP
- [ ] SVIs + HSRP + `ip routing` (DIST) / VLAN 99 IP + default gateway (ACC)
- [ ] DHCP split scope (DIST), DHCP snooping + port-security (ACC)
- [ ] SSH, VTY ACL, AAA, Syslog — and add the 4 new devices to SRV-HFA-AAA
- [ ] routed uplinks with MD5, Loopback0, OSPF

<details><summary>Answer — B-DIST1 OSPF & uplinks (the new part)</summary>

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
interface vlan 99
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
router ospf 1
 router-id 10.10.5.21
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface gi1/1/2
 no passive-interface vlan 99
 network 10.10.0.0 0.0.15.255 area 0
 network 10.10.32.0 0.0.15.255 area 2
 area 2 range 10.10.32.0 255.255.240.0
```
B-DIST2: Lo0 10.10.5.22, Gi1/1/1 10.10.4.25, Gi1/1/2 10.10.4.29, router-id 10.10.5.22.

**Core side** — CORE1:
```
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
router ospf 1
 no passive-interface gi1/0/3
 no passive-interface gi1/0/4
```
CORE2: Gi1/0/3 10.10.4.22, Gi1/0/4 10.10.4.30.
</details>

<details><summary>Answer — B-DIST1 SVIs + DHCP</summary>

```
ip routing
interface vlan 40
 ip address 10.10.32.2 255.255.255.0
 standby version 2
 standby 40 ip 10.10.32.1
 standby 40 priority 110
 standby 40 preempt
 no shutdown
interface vlan 50
 ip address 10.10.33.2 255.255.255.0
 standby version 2
 standby 50 ip 10.10.33.1
 standby 50 priority 90
 standby 50 preempt
 no shutdown
interface vlan 60
 ip address 10.10.34.2 255.255.255.0
 standby version 2
 standby 60 ip 10.10.34.1
 standby 60 priority 110
 standby 60 preempt
 no shutdown
interface vlan 99
 ip address 10.10.35.2 255.255.255.0
 standby version 2
 standby 99 ip 10.10.35.1
 standby 99 priority 110
 standby 99 preempt
 no shutdown
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
```
</details>

### Verify building B
`show ip route ospf` on CORE1 → `O IA 10.10.32.0/20` via B-DIST1 and B-DIST2. A Finance PC
in building A pings an HR PC in building B. ✅

---

## Part 2 — Building C (area 3) and the virtual link

| Item | Value |
|---|---|
| Devices | `SWL3-HFA-C-DIST1/2`, `SW-HFA-C-ACC1/2` |
| VTP domain | `HFA-C` |
| VLANs | 70 RND, 80 IT, 90 PRINTERS-C, 99 MGMT-C |
| Primary | DIST1: 70, 90, 99 · DIST2: 80 |
| Loopbacks | **10.10.63.1 / 10.10.63.2** (inside area 3's /20 — on purpose) |
| PCs | `PC-HFA-RND-01/02`, `PC-HFA-IT-01/02`, `PRN-HFA-C-01` (10.10.50.10) |

**Building C does not touch the core.** Its uplinks go to building B:

| From | To | Subnet (area 2) |
|---|---|---|
| C-DIST1 Gi1/1/1 | B-DIST1 Gi1/1/3 | 10.10.36.0/30 (B .1 / C .2) |
| C-DIST1 Gi1/1/2 | B-DIST2 Gi1/1/3 | 10.10.36.4/30 (B .5 / C .6) |
| C-DIST2 Gi1/1/1 | B-DIST1 Gi1/1/4 | 10.10.36.8/30 (B .9 / C .10) |
| C-DIST2 Gi1/1/2 | B-DIST2 Gi1/1/4 | 10.10.36.12/30 (B .13 / C .14) |

### Concepts — why a virtual link?
OSPF rule: **every area must touch area 0**, because inter-area routes are only passed
*through* the backbone. Area 3 touches only area 2 → C-DIST1/2 are ABRs that are not
connected to area 0 → **area 3's routes never reach the rest of the company**, and
building C can't reach anything outside area 2.

A **virtual link** is a logical area-0 tunnel *through* a transit area (area 2), between
two ABRs:
- one end on a router in area 0 + area 2 → **B-DIST1/2** (router-IDs 10.10.5.21 / .22)
- the other end on a router in area 2 + area 3 → **C-DIST1/2** (router-IDs 10.10.63.1 / .2)

Syntax: `area <transit-area> virtual-link <router-ID of the other end>` — the **router-ID**,
not an interface IP! We build all 4 combinations, so losing any single switch keeps area 3
connected.

### Step A — build building C *without* the virtual link first
Build everything (same checklist as building B). On C-DIST1:
```
interface loopback 0
 ip address 10.10.63.1 255.255.255.255
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
interface vlan 99
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest
router ospf 1
 router-id 10.10.63.1
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface gi1/1/2
 no passive-interface vlan 99
 network 10.10.36.0 0.0.0.15 area 2
 network 10.10.48.0 0.0.15.255 area 3
 area 3 range 10.10.48.0 255.255.240.0
```
On B-DIST1 add the downlinks to C (area 2):
```
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
router ospf 1
 no passive-interface gi1/1/3
 no passive-interface gi1/1/4
```
(`10.10.36.x` is inside `10.10.32.0/20`, so the existing `network ... area 2` line already
puts these links in area 2.)

**Now observe the problem:**
- `SWL3-HFA-CORE1# show ip route | include 10.10.48` → **nothing**. Area 3 is invisible.
- RND PC → `ping 10.10.1.10` (Syslog server) → ❌

### Step B — add the virtual links
```
! B-DIST1 and B-DIST2 (ABR 0/2):
router ospf 1
 area 2 virtual-link 10.10.63.1
 area 2 virtual-link 10.10.63.2

! C-DIST1 and C-DIST2 (ABR 2/3):
router ospf 1
 area 2 virtual-link 10.10.5.21
 area 2 virtual-link 10.10.5.22
```

## Verify
```
SWL3-HFA-B-DIST1# show ip ospf virtual-links
Virtual Link OSPF_VL0 to router 10.10.63.1 is up
  Transit area 2, via interface GigabitEthernet1/1/3, Cost of using 1
  ...
SWL3-HFA-CORE1# show ip route ospf
O IA 10.10.16.0/20 ...    <- area 1 summary
O IA 10.10.32.0/20 ...    <- area 2 summary
O IA 10.10.48.0/20 ...    <- area 3 summary — arrived via the virtual link!
```
RND PC → `ping 10.10.1.10` ✅ · IT PC → `ping` an Engineering PC in building A ✅

## Part 3 — final management ACL
Now that all buildings and the IT department exist, replace `MGMT-ONLY` on **every Haifa
device** with the final version:
```
no ip access-list standard MGMT-ONLY
ip access-list standard MGMT-ONLY
 remark --- MGMT VLANs of buildings A, B, C + IT department ---
 permit 10.10.19.0 0.0.0.255
 permit 10.10.35.0 0.0.0.255
 permit 10.10.51.0 0.0.0.255
 permit 10.10.49.0 0.0.0.255
 deny any log
```
Test: `PC-HFA-IT-01` can SSH to any switch; `PC-HFA-RND-01` cannot.

## Checkpoint
`my-work/step09.pkt` + commit. Haifa's LAN is **complete**: 3 buildings, 6 departments,
3-tier, OSPF areas 0/1/2/3 with a virtual link.

## Common mistakes
- Virtual link configured with an **interface IP** instead of the router-ID → never comes up.
- Virtual link stuck down → transit area typo (`area 3 virtual-link` instead of `area 2`).
- Area 0 authentication configured at area level (`area 0 authentication message-digest`)
  → the virtual link (which *is* area 0) also needs the key. We used interface-level auth
  to avoid this trap.

## Challenge
Remove the area-2 summary on B-DIST1/2 (`no area 2 range ...`). Look at CORE1's routing
table. How many more lines do you see, and why are the `10.10.36.x` links among them?
Put the summary back.

<details><summary>Answer</summary>

Without `area 2 range`, every area-2 subnet (40, 50, 60, 99 /24s and the four /30 B↔C
links) is advertised individually as `O IA` — about 8 lines instead of 1. The /30 links are
area-2 networks too, so they leak until the summary covers them (they are inside
10.10.32.0/20 — that's why we numbered them 10.10.36.x and not 10.10.4.x).
</details>
