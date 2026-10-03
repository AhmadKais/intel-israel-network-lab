# Step 12 — The WAN: 3 ISPs, BGP, NAT and a DSL backup

## Goal
Build "the internet": three providers that exchange routes with **BGP**, each running
**OSPF** inside its own network. Connect Haifa: users browse with **PAT**, the internet
reaches the DMZ web server through **static NAT**, and a **DSL** line is a backup.

Ministry requirements: **1, 3, 4, 9, 10 (static), 22.12, 22.17, 23.1, 23.2, 23.3**

## Concepts
- **AS (Autonomous System)** — a network under one administration (an ISP). Inside an AS
  you run an **IGP** (OSPF/EIGRP); *between* ASes you run **BGP** — the protocol of the
  internet. eBGP neighbours are configured **by hand** (`neighbor X remote-as Y`).
- Our WAN, inside each ISP, two routers:
  - **BR** (border) — speaks eBGP to the other ISPs, OSPF inward.
  - **PE** (provider edge) — connects the customers, OSPF only.
  ```
           AS 100                    AS 200                   AS 300
   [ISPA-PE]──OSPF──[ISPA-BR]══eBGP══[ISPB-BR]──OSPF──[ISPB-PE]
       │                 ╲                ╱                │
     Haifa                ╲══eBGP══[ISPC-BR]──OSPF──[ISPC-PE]   Jerusalem
                                                  │
                                            Kiryat Gat, home users
  ```
- BGP `network 200.1.0.0 mask 255.255.0.0` only advertises a prefix that is **exactly** in
  the routing table → we add `ip route 200.1.0.0 255.255.0.0 null0` (a "black-hole"
  aggregate; real, more specific routes still win).
- **Customers use static routes** toward the ISP — never a routing protocol (req. 22.17:
  never leak internal routes to the internet). The ISP has a static route back to the
  customer's public addresses and redistributes it into its OSPF.
- **NAT on R-HFA-EDGE**
  - **PAT (overload)**: all of HQ shares the router's public IP `200.1.1.2` for browsing.
  - **Static NAT**: `200.1.10.3 ↔ 10.10.200.13` — the DMZ web server is reachable from the
    internet. `200.1.10.4:123 ↔ 10.10.200.14:123` — public NTP for Kiryat Gat (step 14).
  - HQ's "NAT method" for req. 10 = **static**. (Jerusalem = dynamic, Kiryat Gat = overload.)
- **DSL backup** — a second path through a DSL modem to ISP-B, used via a **floating static
  route** (administrative distance 10 > 1, so it's only installed if the primary dies).

## Build

| Device | Names |
|---|---|
| 2911 ×6 | `R-ISPA-BR`, `R-ISPA-PE`, `R-ISPB-BR`, `R-ISPB-PE`, `R-ISPC-BR`, `R-ISPC-PE` |
| 2960 ×2 | `SW-ISPA-NET` (internet servers), `SW-ISPC-HOME` (home users) |
| Server-PT ×3 | `SRV-ISP-DNS` 200.1.100.10, `SRV-ISP-WEB1` .11, `SRV-ISP-WEB2` .12 (gw 200.1.100.1) |
| PC | `PC-HOME-01` 200.3.50.10/24, gw 200.3.50.1, DNS 200.1.100.10 |
| DSL-Modem-PT | `DSL-HFA` |
| Cloud-PT | `CLOUD-DSL` |

Cabling — see [design.md §6](../../docs/design.md#6-wan):

| From | To |
|---|---|
| ISPA-BR Gi0/0 | ISPA-PE Gi0/0 |
| ISPB-BR Gi0/0 | ISPB-PE Gi0/0 |
| ISPC-BR Gi0/0 | ISPC-PE Gi0/0 |
| ISPA-BR Gi0/1 | ISPB-BR Gi0/1 |
| ISPA-BR Gi0/2 | ISPC-BR Gi0/1 |
| ISPB-BR Gi0/2 | ISPC-BR Gi0/2 |
| ISPA-PE Gi0/1 | **R-HFA-EDGE Gi0/1** |
| ISPA-PE Gi0/2 | SW-ISPA-NET Gi0/1 (+ 3 servers on Fa0/1-3) |
| ISPC-PE Gi0/2 | SW-ISPC-HOME Gi0/1 (+ PC-HOME-01 on Fa0/1) |
| R-HFA-EDGE Gi0/2 | DSL-HFA Ethernet port |
| DSL-HFA phone port | CLOUD-DSL **Modem4** (phone cable) |
| CLOUD-DSL **Ethernet6** | ISPB-PE Gi0/2 |

**CLOUD-DSL**: Config → *DSL* → map **Modem4 ↔ Ethernet6** → Add.

(ISPB-PE Gi0/1 → Jerusalem and ISPC-PE Gi0/1 → Kiryat Gat get connected in steps 13 & 14.)

## Configure

### ISP-A — AS 100
```
! ===== R-ISPA-BR =====
hostname R-ISPA-BR
interface gi0/0
 description to R-ISPA-PE
 ip address 200.1.0.1 255.255.255.252
 no shutdown
interface gi0/1
 description eBGP to R-ISPB-BR (AS 200)
 ip address 200.0.12.1 255.255.255.252
 no shutdown
interface gi0/2
 description eBGP to R-ISPC-BR (AS 300)
 ip address 200.0.13.1 255.255.255.252
 no shutdown
 exit
! aggregate for BGP to advertise
ip route 200.1.0.0 255.255.0.0 null0

router ospf 10
 router-id 1.1.1.1
 network 200.1.0.0 0.0.0.3 area 0
 ! tell the PE: "send everything you don't know to me"
 default-information originate always
router bgp 100
 bgp router-id 1.1.1.1
 neighbor 200.0.12.2 remote-as 200
 neighbor 200.0.13.2 remote-as 300
 network 200.1.0.0 mask 255.255.0.0

! ===== R-ISPA-PE =====
hostname R-ISPA-PE
interface gi0/0
 description to R-ISPA-BR
 ip address 200.1.0.2 255.255.255.252
 no shutdown
interface gi0/1
 description customer: Intel Haifa
 ip address 200.1.1.1 255.255.255.252
 no shutdown
interface gi0/2
 description internet services LAN
 ip address 200.1.100.1 255.255.255.0
 no shutdown
 exit
! Intel Haifa's public static-NAT block lives behind the customer router
ip route 200.1.10.0 255.255.255.248 200.1.1.2
router ospf 10
 router-id 1.1.1.2
 passive-interface gi0/1
 passive-interface gi0/2
 network 200.1.0.0 0.0.0.3 area 0
 network 200.1.1.0 0.0.0.3 area 0
 network 200.1.100.0 0.0.0.255 area 0
 redistribute static subnets
```

### ISP-B — AS 200
```
! ===== R-ISPB-BR =====
hostname R-ISPB-BR
interface gi0/0
 ip address 200.2.0.1 255.255.255.252
 no shutdown
interface gi0/1
 ip address 200.0.12.2 255.255.255.252
 no shutdown
interface gi0/2
 ip address 200.0.23.1 255.255.255.252
 no shutdown
 exit
ip route 200.2.0.0 255.255.0.0 null0
router ospf 10
 router-id 2.2.2.1
 network 200.2.0.0 0.0.0.3 area 0
 default-information originate always
router bgp 200
 bgp router-id 2.2.2.1
 neighbor 200.0.12.1 remote-as 100
 neighbor 200.0.23.2 remote-as 300
 network 200.2.0.0 mask 255.255.0.0

! ===== R-ISPB-PE =====
hostname R-ISPB-PE
interface gi0/0
 ip address 200.2.0.2 255.255.255.252
 no shutdown
interface gi0/1
 description customer: Intel Jerusalem (step 13)
 ip address 200.2.1.1 255.255.255.252
 no shutdown
interface gi0/2
 description DSL customer: Intel Haifa backup
 ip address 200.2.9.1 255.255.255.252
 no shutdown
 exit
! Jerusalem's dynamic-NAT pool (step 13)
ip route 200.2.10.0 255.255.255.248 200.2.1.2
router ospf 10
 router-id 2.2.2.2
 passive-interface gi0/1
 passive-interface gi0/2
 network 200.2.0.0 0.0.0.3 area 0
 network 200.2.1.0 0.0.0.3 area 0
 network 200.2.9.0 0.0.0.3 area 0
 redistribute static subnets
```

### ISP-C — AS 300 (dual-stack toward its customers)
```
! ===== R-ISPC-BR =====
hostname R-ISPC-BR
interface gi0/0
 ip address 200.3.0.1 255.255.255.252
 no shutdown
interface gi0/1
 ip address 200.0.13.2 255.255.255.252
 no shutdown
interface gi0/2
 ip address 200.0.23.2 255.255.255.252
 no shutdown
 exit
ip route 200.3.0.0 255.255.0.0 null0
router ospf 10
 router-id 3.3.0.1
 network 200.3.0.0 0.0.0.3 area 0
 default-information originate always
router bgp 300
 bgp router-id 3.3.0.1
 neighbor 200.0.13.1 remote-as 100
 neighbor 200.0.23.1 remote-as 200
 network 200.3.0.0 mask 255.255.0.0

! ===== R-ISPC-PE =====
hostname R-ISPC-PE
ipv6 unicast-routing
interface gi0/0
 ip address 200.3.0.2 255.255.255.252
 no shutdown
interface gi0/1
 description customer: Intel Kiryat Gat (step 14)
 ip address 200.3.1.1 255.255.255.252
 ipv6 address 2001:DB8:C:1::1/64
 no shutdown
interface gi0/2
 description home users
 ip address 200.3.50.1 255.255.255.0
 ipv6 address 2001:DB8:C:50::1/64
 no shutdown
 exit
router ospf 10
 router-id 3.3.0.2
 passive-interface gi0/1
 passive-interface gi0/2
 network 200.3.0.0 0.0.0.3 area 0
 network 200.3.1.0 0.0.0.3 area 0
 network 200.3.50.0 0.0.0.255 area 0
```
Add `enable secret`, banner and `no ip domain-lookup` on the ISP routers too — the ministry
asks for banners on **all** routers.

### Internet servers
- **SRV-ISP-DNS**: DNS On, records `www.isp-web1.com → 200.1.100.11`,
  `www.isp-web2.com → 200.1.100.12`, **`www.intel.lab → 200.1.10.3`** (the public address!).
- **SRV-ISP-WEB1 / WEB2**: different `index.html` text ("ISP Web 1", "ISP Web 2").

### R-HFA-EDGE — WAN, NAT, DSL backup
```
configure terminal
interface gi0/0
 ip nat inside
interface gi0/1
 description PRIMARY to R-ISPA-PE
 ip address 200.1.1.2 255.255.255.252
 ip nat outside
 no shutdown
interface gi0/2
 description BACKUP via DSL to R-ISPB-PE
 ip address 200.2.9.2 255.255.255.252
 ip nat outside
 no shutdown
 exit

! ---------- static routing toward the WAN (req. 22.17) ----------
ip route 0.0.0.0 0.0.0.0 200.1.1.1
! floating static: only used if the primary disappears
ip route 0.0.0.0 0.0.0.0 200.2.9.1 10

! ---------- PAT: all of HQ browses as 200.1.1.2 ----------
ip access-list extended NAT-HFA
 remark --- VPN destinations are NOT translated (step 15) ---
 deny ip 10.10.0.0 0.0.255.255 172.16.250.0 0.0.0.255
 permit ip 10.10.0.0 0.0.255.255 any
 exit
ip nat inside source list NAT-HFA interface gi0/1 overload

! ---------- static NAT: publish the DMZ web + NTP servers ----------
ip nat inside source static 10.10.200.13 200.1.10.3
ip nat inside source static udp 10.10.200.14 123 200.1.10.4 123
end
wr
```

### ASA — let the internet reach the web server and NTP
```
configure terminal
access-list OUTSIDE_IN extended permit tcp any host 10.10.200.13 eq www
access-list OUTSIDE_IN extended permit udp any host 10.10.200.14 eq ntp
end
write memory
```
(The ASA sees the **real** server IP — the edge router already un-NATed it.)

## Verify

```
R-ISPA-BR# show ip bgp summary
Neighbor     V  AS  MsgRcvd MsgSent  ... State/PfxRcd
200.0.12.2   4  200   ...                 1          <- a number = Established
200.0.13.2   4  300   ...                 1
R-ISPA-BR# show ip bgp
*> 200.1.0.0/16   0.0.0.0        0  32768 i
*> 200.2.0.0/16   200.0.12.2     0      0 200 i
*> 200.3.0.0/16   200.0.13.2     0      0 300 i
*  200.3.0.0/16   200.0.12.2            0 200 300 i      <- backup path via AS 200
R-ISPA-PE# show ip route
O*E2 0.0.0.0/0 ...                  <- from BR (default-information originate)
S    200.1.10.0/29 [1/0] via 200.1.1.2
```

| From | Test | Expected |
|---|---|---|
| ENG PC | browser `http://www.isp-web1.com` | ✅ "ISP Web 1" |
| ENG PC | `ping 200.1.100.12` | ✅ |
| R-HFA-EDGE | `show ip nat translations` | `icmp 200.1.1.2:x  10.10.16.x:x ...` (PAT) and the two static lines |
| PC-HOME-01 | browser `http://www.intel.lab` | ✅ intranet page via **200.1.10.3** (static NAT) |
| PC-HOME-01 | `ping 10.10.16.1` | ❌ private addresses don't exist on the internet |
| LT-HFA-GUEST-01 | browser `http://www.isp-web2.com` | ✅ guests have internet |

### Redundancy tests
1. **BGP**: shut the ISPA-BR ↔ ISPC-BR link. `show ip bgp` on ISPA-BR → 200.3.0.0/16 now via
   AS 200 (path `200 300`). Home PC still reaches www.intel.lab. Unshut.
2. **DSL backup**: `ping -t 200.2.0.1` from R-HFA-EDGE, then shut Gi0/1. `show ip route`
   → default now via 200.2.9.1 [10/0]. Pings to ISP-B continue through the DSL modem.
   No shut Gi0/1 → the primary returns. (Limitation: PAT is bound to Gi0/1, so *users*
   don't browse over DSL — see the challenge.)

## Checkpoint
`my-work/step12.pkt` + commit.

## Common mistakes
- BGP neighbour stuck in **Active/Idle** → wrong neighbour IP or `remote-as`, or the link
  is down. BGP needs a working IP path first.
- Prefix not in `show ip bgp` → the `network ... mask` doesn't **exactly** match a route in
  the table (forgot the null0 route, or typo in the mask).
- Home PC can't reach www.intel.lab → missing `ip route 200.1.10.0 ...` on ISPA-PE, missing
  `redistribute static subnets`, or the ASA ACL line for www.
- Everything translated except the web server → `ip nat inside` missing on Gi0/0.

## Challenge
Make **users** survive the primary link failure too: add a second PAT rule for Gi0/2.
Hint: two `ip nat inside source list` rules with the same ACL clash — you need
**route-maps** matching the outgoing interface.

<details><summary>Answer</summary>

```
route-map NAT-PRIMARY permit 10
 match ip address NAT-HFA
 match interface gi0/1
route-map NAT-BACKUP permit 10
 match ip address NAT-HFA
 match interface gi0/2
no ip nat inside source list NAT-HFA interface gi0/1 overload
ip nat inside source route-map NAT-PRIMARY interface gi0/1 overload
ip nat inside source route-map NAT-BACKUP interface gi0/2 overload
```
Packet Tracer's support for `ip nat ... route-map` depends on the version. If it's
rejected, write in the book that the DSL backup serves the router itself and that
route-map NAT is the real-world solution.
</details>
