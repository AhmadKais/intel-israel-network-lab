# Step 08 — Core layer, OSPF, server farm, AAA & Syslog

## Goal
Two **core** L3 switches join building A with **routed** (not switched) links. **OSPF**
carries the routes: core = **area 0**, building A = **area 1**, summarized to one /20.
OSPF is authenticated (MD5) and only speaks where it must (passive interfaces). A small
server farm gets a **Syslog** server and an **AAA (TACACS+)** server, and every device
logs to Syslog and authenticates admins against AAA.

Ministry requirements: **8 (start), 20.9, 20.10, 22.5, 22.15, 22.18, 20.1 (core tier), 6**

## Concepts
- **Routed links between layers** — distribution ↔ core are `no switchport` /30 links. No
  STP between layers, both uplinks active at once (ECMP), failover in < 1 s.
- **OSPF areas** — an area is a group of routers that share the full topology map.
  **Area 0** (backbone) connects all others. A router in two areas is an **ABR**.
  A-DIST1/2 are ABRs (area 0 uplinks + area 1 SVIs).
- **Summarization at the ABR** — `area 1 range 10.10.16.0 255.255.240.0` → the core sees
  one route `O IA 10.10.16.0/20` instead of six /24s.
- **Passive interface** — OSPF advertises the subnet but sends **no hellos** there. On user
  VLANs this stops an attacker running OSPF from a PC and injecting routes.
  We use `passive-interface default` and open only the interfaces toward other routers.
- **MD5 authentication** — neighbours must share a key, otherwise no adjacency.
- **Loopback0** — always-up interface; we use its IP as the OSPF **router-ID** and as the
  source of AAA/Syslog packets (stable identity).
- **AAA** — Authentication (who are you), Authorization (what may you do), Accounting
  (what did you do). With `aaa new-model` + TACACS+ every admin login is checked by the
  central server; `local` is the fallback if the server is unreachable.

## Build

| Device | Name | Note |
|---|---|---|
| 3650-24PS | `SWL3-HFA-CORE1`, `SWL3-HFA-CORE2` | power supplies! |
| 2960-24TT | `SW-HFA-SRV` | server-farm switch |
| Server-PT | `SRV-HFA-SYSLOG` 10.10.1.10 · `SRV-HFA-AAA` 10.10.1.11 | gw 10.10.1.1 |

| From | To |
|---|---|
| A-DIST1 Gi1/1/1 | CORE1 Gi1/0/1 |
| A-DIST1 Gi1/1/2 | CORE2 Gi1/0/1 |
| A-DIST2 Gi1/1/1 | CORE1 Gi1/0/2 |
| A-DIST2 Gi1/1/2 | CORE2 Gi1/0/2 |
| CORE1 Gi1/0/23 | CORE2 Gi1/0/23 |
| CORE1 Gi1/0/24 | SW-HFA-SRV Gi0/1 |
| CORE2 Gi1/0/24 | SW-HFA-SRV Gi0/2 |
| SRV-HFA-SYSLOG | SW-HFA-SRV Fa0/1 |
| SRV-HFA-AAA | SW-HFA-SRV Fa0/2 |

## Configure

### 1. SWL3-HFA-CORE1
Basic hardening + SSH block first (steps 02 & 07). Then:
```
configure terminal
ip routing
vtp mode transparent
spanning-tree mode rapid-pvst
vlan 100
 name SERVERS
 exit

interface loopback 0
 ip address 10.10.5.1 255.255.255.255

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
interface gi1/0/24
 description to SW-HFA-SRV Gi0/1
 switchport mode access
 switchport access vlan 100
 spanning-tree portfast
 no shutdown

interface vlan 100
 description GW SERVERS
 ip address 10.10.1.2 255.255.255.0
 standby version 2
 standby 100 ip 10.10.1.1
 standby 100 priority 110
 standby 100 preempt
 no shutdown

router ospf 1
 router-id 10.10.5.1
 passive-interface default
 no passive-interface gi1/0/1
 no passive-interface gi1/0/2
 no passive-interface gi1/0/23
 network 10.10.0.0 0.0.15.255 area 0
end
wr
```

> `network 10.10.0.0 0.0.15.255 area 0` = "every interface whose IP is in 10.10.0.0/20 is
> in area 0". That catches the /30 links, Loopback0 and VLAN 100 in one line — the payoff
> of planning per-area blocks in step 01.

### 2. SWL3-HFA-CORE2
Same as CORE1 with: Loopback 10.10.5.2 · Gi1/0/1 `10.10.4.6` (→ A-DIST1 Gi1/1/2) ·
Gi1/0/2 `10.10.4.14` (→ A-DIST2 Gi1/1/2) · Gi1/0/23 `10.10.4.50` · VLAN 100 `10.10.1.3`
priority **90** · `router-id 10.10.5.2`.

### 3. SWL3-HFA-A-DIST1 — uplinks + OSPF
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
! DIST1 and DIST2 also peer with each other inside area 1, over the MGMT VLAN
interface vlan 99
 ip ospf message-digest-key 1 md5 Intel#Ospf
 ip ospf authentication message-digest

router ospf 1
 router-id 10.10.5.11
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface gi1/1/2
 no passive-interface vlan 99
 network 10.10.0.0 0.0.15.255 area 0
 network 10.10.16.0 0.0.15.255 area 1
 area 1 range 10.10.16.0 255.255.240.0
end
wr
```

### 4. SWL3-HFA-A-DIST2
Same, with Loopback `10.10.5.12`, Gi1/1/1 `10.10.4.9` (→ CORE1 Gi1/0/2), Gi1/1/2
`10.10.4.13` (→ CORE2 Gi1/0/2), `router-id 10.10.5.12`.

### 5. SW-HFA-SRV
Hardening + SSH, then:
```
configure terminal
vtp mode transparent
vlan 100
 name SERVERS
 exit
interface range gi0/1 - 2
 description uplinks to CORE1 / CORE2
 switchport mode access
 switchport access vlan 100
interface range fa0/1 - 23
 switchport mode access
 switchport access vlan 100
 spanning-tree portfast
 spanning-tree bpduguard enable
interface fa0/24
 description to ASA-HFA-FW1 inside (step 11)
 switchport mode access
 switchport access vlan 100
interface vlan 100
 ip address 10.10.1.5 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.10.1.1
end
wr
```

### 6. Servers
- **SRV-HFA-SYSLOG**: IP 10.10.1.10/24, gw 10.10.1.1 · *Services → SYSLOG → On*.
- **SRV-HFA-AAA**: IP 10.10.1.11/24, gw 10.10.1.1 · *Services → AAA → On*.
  - *Network Configuration* — add one line per device (ServerType **Tacacs**, Key `Intel#Tac`):

    | Client name | Client IP |
    |---|---|
    | SW-HFA-A-ACC1 | 10.10.19.11 |
    | SW-HFA-A-ACC2 | 10.10.19.12 |
    | SWL3-HFA-A-DIST1 | 10.10.5.11 |
    | SWL3-HFA-A-DIST2 | 10.10.5.12 |
    | SWL3-HFA-CORE1 | 10.10.5.1 |
    | SWL3-HFA-CORE2 | 10.10.5.2 |
    | SW-HFA-SRV | 10.10.1.5 |
  - *User Setup*: user `admin`, password `Intel#Adm1n`.

### 7. AAA + Syslog on **every** Haifa device so far
```
configure terminal
service timestamps log datetime msec
logging trap informational
logging host 10.10.1.10

aaa new-model
tacacs-server host 10.10.1.11 key Intel#Tac
aaa authentication login default group tacacs+ local
! only on L3 switches (they have a Loopback0):
ip tacacs source-interface loopback 0

line vty 0 15
 login authentication default
line console 0
 login authentication default
end
wr
```
Also add the SERVER VLAN gateway to the VTY ACL? **No** — servers don't manage devices.

## Verify

```
SWL3-HFA-CORE1# show ip ospf neighbor
Neighbor ID   Pri State     Dead Time  Address     Interface
10.10.5.11    1   FULL/DR   ...        10.10.4.1   Gi1/0/1
10.10.5.12    1   FULL/DR   ...        10.10.4.9   Gi1/0/2
10.10.5.2     1   FULL/BDR  ...        10.10.4.50  Gi1/0/23
```
```
SWL3-HFA-CORE1# show ip route ospf
O IA  10.10.16.0/20 [110/2] via 10.10.4.1, Gi1/0/1
                    [110/2] via 10.10.4.9, Gi1/0/2       <- ONE summary, TWO paths (ECMP)
O     10.10.5.11/32 ...
```
```
SWL3-HFA-A-DIST1# show ip ospf interface brief    <- Gi1/1/x + Lo0 in area 0, SVIs in area 1
SWL3-HFA-A-DIST1# show ip ospf neighbor           <- CORE1, CORE2 and DIST2 (via Vl99)
```

| Test | Expected |
|---|---|
| ENG-01 → `ping 10.10.1.10` | ✅ user VLAN → dist → core → server |
| ENG-01 → `tracert 10.10.1.10` | 10.10.16.2 → 10.10.4.2 → 10.10.1.10 |
| ADMIN-01 → `ssh -l admin 10.10.19.11` | ✅ — and on SRV-HFA-AAA nothing visible, but… |
| Power off SRV-HFA-AAA, SSH again | ✅ still works — `local` fallback |
| `shutdown` / `no shutdown` an unused port on ACC1 | message appears in SRV-HFA-SYSLOG → Services → SYSLOG |

**Passive-interface test**: in Simulation mode filter on OSPF. Hellos appear only on the
routed links and VLAN 99 — never on VLAN 10/20 toward PCs.

## Checkpoint
`my-work/step08.pkt` + commit.

## Common mistakes
- Neighbours never reach FULL → MD5 key typo on one side, or one side still passive.
- `% Invalid input` on `ip address` under Gi1/1/1 → forgot `no switchport`.
- TACACS login always falls back to local → the client IP in the AAA server doesn't match
  the packet's **source** IP. Without the `source-interface` command, an L3 switch sources
  from its **uplink** IP (e.g. 10.10.4.1). See [pt-gotchas](../../docs/pt-gotchas.md).

## Challenge
Shut CORE1 completely (power off). Prove — with `ping -t` from ENG-01 to the Syslog server
and `show ip route` on A-DIST1 — that HQ survives. What happens to the server-farm
gateway (10.10.1.1)?

<details><summary>Answer</summary>

A-DIST1's two equal-cost routes become one (via CORE2); a few pings are lost while OSPF
detects the dead neighbour (dead timer, or instantly if the link goes down). CORE2
becomes HSRP **Active** for VLAN 100, so the servers' gateway 10.10.1.1 keeps working.
When CORE1 comes back, `preempt` returns VLAN 100 to it.
</details>
