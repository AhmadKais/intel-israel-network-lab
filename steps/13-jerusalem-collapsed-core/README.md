# Step 13 — Jerusalem: collapsed core, EIGRP, DHCP server, dynamic NAT, CBAC

## Goal
Build the second branch with a **different design** from Haifa (req. ג — branches must not
be copies): the core and distribution layers are **collapsed** into one pair of L3
switches; routing is **EIGRP** with summarization; DHCP comes from a **dedicated server**;
internet access uses **dynamic NAT**; the edge router is a **CBAC** firewall.

Ministry requirements: **5, 10 (dynamic), 17 (HSRP on L3 switches), 20.x, 22.11, 22.13,
22.14 (PAgP), 22.15, 22.18**

## Concepts
- **Collapsed core** — in a medium site, a separate core layer is overkill. Two L3 switches
  do both jobs: they are the VLAN gateways (distribution) **and** the backbone (core).
  Cheaper, fewer hops, still fully redundant.
  ```
            R-JLM-EDGE
            /        \           routed /30s, EIGRP
       SWL3-JLM-CC1 ══ SWL3-JLM-CC2     PAgP EtherChannel
        /   |   \   \  /  /  |  \
     F1-ACC F2-ACC F3-ACC SRV          trunks, each to BOTH CCs
  ```
- **EIGRP** — Cisco's advanced distance-vector protocol: fast convergence (feasible
  successors = precomputed backup paths), simple config.
  - `no auto-summary` — otherwise EIGRP summarizes at classful boundaries (10.0.0.0/8!).
  - **Manual summary** on the uplink: `ip summary-address eigrp 100 10.20.8.0 255.255.248.0`
    → the edge sees one `/21` for all six departments.
  - **MD5 authentication** with a **key chain**.
- **DHCP relay** — the DHCP server is in VLAN 100; PCs broadcast in VLAN 41..46. Routers
  don't forward broadcasts, so each SVI gets `ip helper-address 10.20.1.10`, which turns
  the broadcast into a unicast to the server. The server picks the pool from the relay
  address (the SVI's IP).
- **Dynamic NAT** — inside addresses are mapped 1:1 to addresses from a **pool**
  (`200.2.10.1–.6`). Only 6 PCs can be on the internet at the same time — a real
  limitation you should write about in the book (that's why PAT exists).
- **CBAC** (`ip inspect`) — a stateful firewall on a router. Inbound ACL denies everything;
  CBAC watches *outgoing* sessions and temporarily opens holes for their replies.

## Build

| Device | Name |
|---|---|
| 2911 | `R-JLM-EDGE` |
| 3650-24PS ×2 (PSU!) | `SWL3-JLM-CC1`, `SWL3-JLM-CC2` |
| 2960 ×4 | `SW-JLM-F1-ACC1`, `SW-JLM-F2-ACC1`, `SW-JLM-F3-ACC1`, `SW-JLM-SRV` |
| Server-PT ×3 | `SRV-JLM-DHCP` 10.20.1.10 · `SRV-JLM-SYSLOG` .11 · `SRV-JLM-AAA` .12 (gw 10.20.1.1, DNS 10.10.200.11) |
| PCs | 2 per department to start (`PC-JLM-SAL-01/02`, `-LOG-`, `-PRD-`, `-QA-`, `-SEC-`, `-ADM-`), `PC-JLM-ADMIN-01` (10.20.0.100, gw .1) |
| Printers | `PRN-JLM-F1-01` 10.20.2.11, `-F2-01` .12, `-F3-01` .13 (gw 10.20.2.1) |

| From | To |
|---|---|
| CC1 Gi1/0/1 / CC2 Gi1/0/1 | F1-ACC1 Gi0/1 / Gi0/2 |
| CC1 Gi1/0/2 / CC2 Gi1/0/2 | F2-ACC1 Gi0/1 / Gi0/2 |
| CC1 Gi1/0/3 / CC2 Gi1/0/3 | F3-ACC1 Gi0/1 / Gi0/2 |
| CC1 Gi1/0/4 / CC2 Gi1/0/4 | SW-JLM-SRV Gi0/1 / Gi0/2 |
| CC1 Gi1/0/23-24 | CC2 Gi1/0/23-24 (PAgP Po1) |
| CC1 Gi1/1/1 | R-JLM-EDGE Gi0/0 |
| CC2 Gi1/1/1 | R-JLM-EDGE Gi0/2 |
| R-JLM-EDGE Gi0/1 | R-ISPB-PE Gi0/1 |

| Floor switch | Fa0/1-10 | Fa0/11-20 | Fa0/21 | Fa0/22 |
|---|---|---|---|---|
| F1 | 41 SALES | 42 LOGISTICS | 130 printer | 99 admin PC (F1 only) |
| F2 | 43 PRODUCTION | 44 QA | 130 printer | parking |
| F3 | 45 SECURITY | 46 ADMIN | 130 printer | parking |

## Configure

### SWL3-JLM-CC1
Hardening first (hostname, secret, banner, …). Then:
```
configure terminal
ip routing
spanning-tree mode rapid-pvst
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

spanning-tree vlan 41,43,45,99,100,130 root primary
spanning-tree vlan 42,44,46 root secondary

! ---------- PAgP EtherChannel to CC2 (Haifa used LACP - variety!) ----------
interface range gi1/0/23 - 24
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 41-46,99,100,130,998
 switchport nonegotiate
 channel-group 1 mode desirable
 no shutdown
interface port-channel 1
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 41-46,99,100,130,998
 switchport nonegotiate

! ---------- downlinks to floors + server switch ----------
interface range gi1/0/1 - 4
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 41-46,99,100,130,998
 switchport nonegotiate
 spanning-tree guard root
 no shutdown
interface range gi1/0/5 - 22
 switchport mode access
 switchport access vlan 999
 shutdown

! ---------- uplink to the edge router ----------
key chain EIGRP-KEYS
 key 1
  key-string Intel#Eigrp
  exit
 exit
interface loopback 0
 ip address 10.20.5.1 255.255.255.255
interface gi1/1/1
 description to R-JLM-EDGE Gi0/0
 no switchport
 ip address 10.20.4.2 255.255.255.252
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 EIGRP-KEYS
 ! one route for all 6 departments
 ip summary-address eigrp 100 10.20.8.0 255.255.248.0
 no shutdown
```
SVIs — **pattern** (do all nine: 41-46, 99, 100, 130):
```
interface vlan 41
 description GW SALES
 ip address 10.20.10.2 255.255.255.0
 ip helper-address 10.20.1.10
 standby version 2
 standby 41 ip 10.20.10.1
 standby 41 priority 110
 standby 41 preempt
 no shutdown
```
| VLAN | CC1 IP | VIP | CC1 priority | helper? |
|---|---|---|---|---|
| 41 | 10.20.10.2 | .1 | 110 | yes |
| 42 | 10.20.11.2 | .1 | 90 | yes |
| 43 | 10.20.12.2 | .1 | 110 | yes |
| 44 | 10.20.13.2 | .1 | 90 | yes |
| 45 | 10.20.14.2 | .1 | 110 | yes |
| 46 | 10.20.15.2 | .1 | 90 | yes |
| 99 | 10.20.0.2 | .1 | 110 | no |
| 100 | 10.20.1.2 | .1 | 110 | no |
| 130 | 10.20.2.2 | .1 | 110 | no (static printers) |

EIGRP:
```
interface vlan 100
 ! CC1 and CC2 also peer over the server VLAN
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 EIGRP-KEYS
router eigrp 100
 eigrp router-id 10.20.5.1
 network 10.20.0.0 0.0.255.255
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface vlan 100
 no auto-summary
end
wr
```

### SWL3-JLM-CC2
Same, mirrored: `vtp mode client` (use the transparent→client trick), STP root primary for
42, 44, 46 and secondary for the rest, `channel-group 1 mode desirable`, Loopback
10.20.5.2, Gi1/1/1 `10.20.4.6/30` (→ EDGE Gi0/2), SVIs `.3` with the opposite priorities,
`eigrp router-id 10.20.5.2`.

### Floor switches & server switch
Exactly your Haifa access-switch pattern (steps 02–07): VTP client domain `JLM`, trunks
(allowed `41-46,99,100,130,998`, native 998), access ports per the table, parking,
PortFast + BPDU Guard, port-security, DHCP snooping (VLANs 41-46, trusted uplinks,
`no ip dhcp snooping information option`), SSH, AAA, Syslog. Management:

| Switch | VLAN 99 IP | default-gateway |
|---|---|---|
| SW-JLM-F1-ACC1 | 10.20.0.11 | 10.20.0.1 |
| SW-JLM-F2-ACC1 | 10.20.0.12 | 10.20.0.1 |
| SW-JLM-F3-ACC1 | 10.20.0.13 | 10.20.0.1 |
| SW-JLM-SRV | 10.20.0.14 (servers on Fa0/1-3, VLAN 100) | 10.20.0.1 |

Jerusalem management ACL (local MGMT VLAN + HQ's IT department over the VPN):
```
ip access-list standard MGMT-ONLY
 permit 10.20.0.0 0.0.0.255
 permit 10.10.49.0 0.0.0.255
 deny any log
```
Syslog → `10.20.1.11`. TACACS+ → `10.20.1.12` (register every JLM device on SRV-JLM-AAA).

### R-JLM-EDGE
```
configure terminal
hostname R-JLM-EDGE
! (+ hardening, SSH, AAA, syslog as usual)

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

! ---------- static default to the ISP, shared with EIGRP ----------
ip route 0.0.0.0 0.0.0.0 200.2.1.1
router eigrp 100
 eigrp router-id 10.20.5.10
 network 10.20.4.0 0.0.0.7
 network 10.20.5.10 0.0.0.0
 redistribute static
 no auto-summary
 exit

! ---------- dynamic NAT ----------
ip nat pool JLM-POOL 200.2.10.1 200.2.10.6 netmask 255.255.255.248
access-list 10 permit 10.20.0.0 0.0.255.255
ip nat inside source list 10 pool JLM-POOL

! ---------- CBAC firewall ----------
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
end
wr
```

### SRV-JLM-DHCP — *Services → DHCP*
One pool per department. Leave the built-in `serverPool` but set it to the server's own
subnet (gateway 10.20.1.1, start 10.20.1.100) so it never hands out wrong addresses.

| Pool | Gateway | DNS | Start IP | Mask | Max users |
|---|---|---|---|---|---|
| JLM-SALES | 10.20.10.1 | 10.10.200.11 | 10.20.10.11 | 255.255.255.0 | 240 |
| JLM-LOGISTICS | 10.20.11.1 | 10.10.200.11 | 10.20.11.11 | 255.255.255.0 | 240 |
| JLM-PRODUCTION | 10.20.12.1 | 10.10.200.11 | 10.20.12.11 | 255.255.255.0 | 240 |
| JLM-QA | 10.20.13.1 | 10.10.200.11 | 10.20.13.11 | 255.255.255.0 | 240 |
| JLM-SECURITY | 10.20.14.1 | 10.10.200.11 | 10.20.14.11 | 255.255.255.0 | 240 |
| JLM-ADMIN | 10.20.15.1 | 10.10.200.11 | 10.20.15.11 | 255.255.255.0 | 240 |

DNS points to Haifa's DMZ DNS — it becomes reachable through the VPN in step 15. Until
then, test with IP addresses.

### NTP
`SWL3-JLM-CC1`: `ntp server 10.10.200.14` (Haifa's main NTP — works after step 15).
All other Jerusalem devices: `ntp server 10.20.5.1` → CC1 is the branch NTP server (req. 20.8).

## Verify

```
R-JLM-EDGE# show ip eigrp neighbors        <- 10.20.4.2 and 10.20.4.6
R-JLM-EDGE# show ip route eigrp
D     10.20.8.0/21 [90/...] via 10.20.4.2, via 10.20.4.6    <- the summary, 2 paths
D     10.20.0.0/24, 10.20.1.0/24, 10.20.2.0/24 ...
SWL3-JLM-CC1# show ip route | include 0.0.0.0
D*EX  0.0.0.0/0 [170/...] via 10.20.4.1                     <- redistributed static default
SWL3-JLM-CC1# show etherchannel summary    <- Po1(SU) PAgP
```

| From | Test | Expected |
|---|---|---|
| any JLM PC | `ipconfig` | address from SRV-JLM-DHCP (`.11+`), gw `.1` |
| PC-JLM-SAL-01 | `ping 200.1.100.11` (ISP web) | ✅ |
| R-JLM-EDGE | `show ip nat translations` | `200.2.10.1  10.20.10.x` (one pool address per PC) |
| PC-JLM-SAL-01 | `tracert 200.1.100.11` | 10.20.10.2 → 10.20.4.1 → 200.2.1.1 → … |
| PC-HOME-01 | `ping 200.2.1.2` | ❌ CBAC/ACL drops unsolicited traffic |
| R-JLM-EDGE | `show ip inspect sessions` | sessions while a PC browses |

**Dynamic-NAT limit test**: make 7 PCs ping the internet at once. The 7th fails —
`show ip nat statistics` shows the pool exhausted. Write this observation in the book.

## Checkpoint
`my-work/step13.pkt` + commit.

## Common mistakes
- EIGRP neighbours don't form → AS number mismatch, key-chain name/key typo, or `network`
  statement missing the link.
- Routes show as `10.0.0.0/8` → forgot `no auto-summary`.
- PCs get 169.254.x.x → helper missing on the SVI, or DHCP snooping without
  `no ip dhcp snooping information option`, or the uplink not trusted.
- `ip summary-address` rejected on the SVI → it belongs on the routed uplink.

## Challenge
Kill CC1 (power off). Which of these still works, and why: DHCP for a SALES PC, browsing
from a QA PC, SSH from PC-JLM-ADMIN-01 to F2-ACC1?

<details><summary>Answer</summary>

All three. CC2 takes over HSRP for every VLAN (so `.1` stays alive), CC2 also relays DHCP
(`ip helper-address` is on both CCs), STP reconverges onto the CC2 uplinks, and the edge
router still has a route to 10.20.8.0/21 through CC2. Only NTP sync is affected, because CC1
was the branch NTP server — a good "single point of failure" to discuss in the book (fix:
`ntp server 10.20.5.2` as a second server on all devices, with CC2 also syncing to Haifa).
</details>
