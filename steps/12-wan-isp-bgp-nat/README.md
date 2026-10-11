# Step 12 — The WAN: 3 internet providers, BGP, NAT and a DSL backup

> ⏱ 3–4 lessons · 🎯 ministry requirements 1, 3, 4, 9, 10 (static), 22.12, 22.17, 23.1, 23.2, 23.3 · 📁 start from: `my-work/step11.pkt` · 💾 save as: `my-work/step12.pkt`

## What you will have at the end
A small **internet**: three providers (ISPs) that exchange routes with **BGP**, each running
**OSPF** inside. Haifa is connected: employees browse the internet (**PAT**), the internet can
open Intel's website (**static NAT**), and a **DSL modem** is a backup line.

```
          AS 100 (ISP-A)                AS 200 (ISP-B)               AS 300 (ISP-C)
   [R-ISPA-PE]─OSPF─[R-ISPA-BR]══eBGP══[R-ISPB-BR]─OSPF─[R-ISPB-PE]
     │      │             ╲               ╱                │     │
     │   internet          ╲════eBGP════[R-ISPC-BR]─OSPF─[R-ISPC-PE]
     │   servers            (eBGP B↔C too)               │      │
   R-HFA-EDGE ·········· DSL modem ····· cloud ──────────┘      home users
   (Haifa)          (backup line to ISP-B)           (Jerusalem — step 13) (Kiryat Gat — step 14)
```

---

## Part 1 — Understand it first (simple English)

### 1.1 What "the internet" really is
The internet is thousands of networks owned by different companies (ISPs, universities, Google…),
all connected. Each such network is an **AS — Autonomous System**, with a number (here 100, 200, 300).

- **Inside** an AS, routers use an internal protocol (**OSPF** or EIGRP) — like a city's own road signs.
- **Between** ASes, they use **BGP** — the protocol of the internet — like international agreements
  between countries about which roads lead where.

The ministry asks for exactly this: a WAN running BGP (req. 1, 3), with OSPF/EIGRP inside the
providers (req. 4).

### 1.2 Our three providers
Each ISP has two routers:
- **BR (border router)** — talks **BGP** with the other ISPs.
- **PE (provider edge)** — connects the **customers** (our branches). Talks only OSPF inside its ISP.

| ISP | AS | Its address block | Customer |
|---|---|---|---|
| ISP-A | 100 | 200.1.0.0/16 | Haifa (+ the internet servers) |
| ISP-B | 200 | 200.2.0.0/16 | Jerusalem (+ Haifa's DSL backup) |
| ISP-C | 300 | 200.3.0.0/16 | Kiryat Gat (+ home users) |

The three BRs are connected in a **triangle** — if one link breaks, BGP uses the other way round
(redundancy between branches, req. 23.1).

### 1.3 BGP in simple words
- BGP neighbours are written **by hand**: `neighbor 200.0.12.2 remote-as 200` = "the router at
  that address belongs to AS 200 — let's talk".
- Each ISP **announces its own block**: `network 200.1.0.0 mask 255.255.0.0`.
- BGP only announces a block that **exists exactly** in the routing table. So we add a "placeholder"
  route to **Null0** (a black hole): `ip route 200.1.0.0 255.255.0.0 null0`. Real, more specific
  routes inside the block still win, because the router always prefers the **longest match**.

### 1.4 Customers use static routes
Our edge routers don't run any routing protocol toward the ISP. They use **one static default
route**: "everything I don't know → the ISP". This way we **never** tell the internet about our
internal networks (req. 22.17). The ISP, in turn, has a static route toward the customer's
public addresses.

### 1.5 NAT — private inside, public outside
Our 10.x addresses are **private**: they don't exist on the internet. **NAT** (Network Address
Translation) on the edge router swaps them for **public** addresses as packets leave.

| Type | How it works | Comparison | Where |
|---|---|---|---|
| **Static NAT** | One private address ↔ one public address, permanently | A direct phone line to one office | Haifa: the web server `10.10.200.13 ↔ 200.1.10.3` |
| **Dynamic NAT** | Private addresses get a public one from a **pool**, one each, while in use | A shared pool of outside lines | Jerusalem (step 13) |
| **PAT (overload)** | Everyone shares **one** public address; the router tells them apart by **port number** | A company switchboard: one public number, many extensions | Haifa for browsing; Kiryat Gat (step 14) |

The ministry wants all three, one per branch (req. 10). **Haifa's method is static NAT**; it
also uses PAT so employees can browse (req. 22.12).

Static NAT is the reason the internet can reach our website: a visitor opens
`http://www.intel.lab` → the internet's DNS says `200.1.10.3` → the packet reaches our edge →
NAT swaps it to `10.10.200.13` → the ASA lets it through to the DMZ.

We also publish the NTP server on `200.1.10.4` (only port 123) for Kiryat Gat in step 14.

### 1.6 DSL backup and the floating static route
The ministry wants an ISP reached through a **DSL modem** (req. 23.2). We use it as Haifa's
**backup** line, through a different provider (ISP-B).

A **floating static route** is a backup route with a worse "trust score" (administrative distance):
`ip route 0.0.0.0 0.0.0.0 200.2.9.1 10`. The normal route has distance 1; this one has 10, so it
**stays hidden** while the primary works and appears automatically if the primary disappears.

---

## Part 2 — Prepare
**File → Open** → `my-work/step11.pkt` → **File → Save As** → `my-work/step12.pkt`.
Draw a big cloud-shaped area (ellipse) on the right labelled `INTERNET / WAN`.

---

## Part 3 — Build (click by click)

### 3.1 Devices
| Where | Model | Display Name |
|---|---|---|
| Network Devices → Routers | **2911** ×6 | `R-ISPA-BR`, `R-ISPA-PE`, `R-ISPB-BR`, `R-ISPB-PE`, `R-ISPC-BR`, `R-ISPC-PE` |
| Network Devices → Switches | **2960** ×2 | `SW-ISPA-NET`, `SW-ISPC-HOME` |
| End Devices | **Server** ×3 | `SRV-ISP-DNS`, `SRV-ISP-WEB1`, `SRV-ISP-WEB2` |
| End Devices | **PC** | `PC-HOME-01` |
| Network Devices → **WAN Emulation** | **DSL-Modem** | `DSL-HFA` |
| Network Devices → **WAN Emulation** | **Cloud** | `CLOUD-DSL` |

### 3.2 Cables
Router ↔ router = **Cross-Over**. Router/server/PC ↔ switch = **Straight-Through**.

| # | Click first | Port | Click second | Port | Cable |
|---|---|---|---|---|---|
| 1 | R-ISPA-BR | Gi0/0 | R-ISPA-PE | Gi0/0 | Cross-Over |
| 2 | R-ISPB-BR | Gi0/0 | R-ISPB-PE | Gi0/0 | Cross-Over |
| 3 | R-ISPC-BR | Gi0/0 | R-ISPC-PE | Gi0/0 | Cross-Over |
| 4 | R-ISPA-BR | Gi0/1 | R-ISPB-BR | Gi0/1 | Cross-Over |
| 5 | R-ISPA-BR | Gi0/2 | R-ISPC-BR | Gi0/1 | Cross-Over |
| 6 | R-ISPB-BR | Gi0/2 | R-ISPC-BR | Gi0/2 | Cross-Over |
| 7 | R-ISPA-PE | Gi0/1 | **R-HFA-EDGE** | **Gi0/1** | Cross-Over |
| 8 | R-ISPA-PE | Gi0/2 | SW-ISPA-NET | Gi0/1 | Straight |
| 9 | SRV-ISP-DNS / WEB1 / WEB2 | Fa0 | SW-ISPA-NET | Fa0/1 / Fa0/2 / Fa0/3 | Straight |
| 10 | R-ISPC-PE | Gi0/2 | SW-ISPC-HOME | Gi0/1 | Straight |
| 11 | PC-HOME-01 | Fa0 | SW-ISPC-HOME | Fa0/1 | Straight |
| 12 | R-HFA-EDGE | **Gi0/2** | DSL-HFA | **Port 1** (Ethernet) | Straight |
| 13 | DSL-HFA | **Port 0** (phone) | CLOUD-DSL | **Modem4** | **Phone** (in Connections) |
| 14 | CLOUD-DSL | **Ethernet6** | R-ISPB-PE | **Gi0/2** | Straight |

### 3.3 Configure the cloud
CLOUD-DSL → **Config** tab → left list **DSL** → *From Port* **Modem4**, *To Port* **Ethernet6** →
**Add**. Now the cloud passes everything from the modem to ISP-B's router.

### 3.4 Address plan of the WAN
| Router | Gi0/0 | Gi0/1 | Gi0/2 |
|---|---|---|---|
| R-ISPA-BR | 200.1.0.1/30 | 200.0.12.1/30 | 200.0.13.1/30 |
| R-ISPA-PE | 200.1.0.2/30 | 200.1.1.1/30 → Haifa | 200.1.100.1/24 → servers |
| R-ISPB-BR | 200.2.0.1/30 | 200.0.12.2/30 | 200.0.23.1/30 |
| R-ISPB-PE | 200.2.0.2/30 | 200.2.1.1/30 → Jerusalem | 200.2.9.1/30 → DSL (Haifa backup) |
| R-ISPC-BR | 200.3.0.1/30 | 200.0.13.2/30 | 200.0.23.2/30 |
| R-ISPC-PE | 200.3.0.2/30 | 200.3.1.1/30 → Kiryat Gat | 200.3.50.1/24 → home users |
| R-HFA-EDGE | 10.10.7.1/30 (ASA) | **200.1.1.2/30** (ISP-A) | **200.2.9.2/30** (DSL) |

---

## Part 4 — Configure

> On every new router: answer **`no`** to the "initial configuration dialog".
> ISP routers belong to "other companies", so they get only the basics: name, secret, banner
> (the ministry wants banners on **all** routers).

### 4.1 R-ISPA-BR (AS 100)
```
enable
configure terminal
hostname R-ISPA-BR
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  ISP-A (AS 100) - AUTHORIZED ACCESS ONLY  *
#
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
! placeholder so BGP can announce our /16
ip route 200.1.0.0 255.255.0.0 null0
! OSPF inside AS 100
router ospf 10
 router-id 1.1.1.1
 network 200.1.0.0 0.0.0.3 area 0
 ! tell the PE: "send me what you don't know"
 default-information originate always
 exit
! BGP with the other providers
router bgp 100
 bgp router-id 1.1.1.1
 neighbor 200.0.12.2 remote-as 200
 neighbor 200.0.13.2 remote-as 300
 network 200.1.0.0 mask 255.255.0.0
 end
copy running-config startup-config
```

### 4.2 R-ISPA-PE
```
enable
configure terminal
hostname R-ISPA-PE
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  ISP-A (AS 100) - AUTHORIZED ACCESS ONLY  *
#
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
! Intel Haifa's public static-NAT block is behind the customer's router
ip route 200.1.10.0 255.255.255.248 200.1.1.2
router ospf 10
 router-id 1.1.1.2
 passive-interface gi0/1
 passive-interface gi0/2
 network 200.1.0.0 0.0.0.3 area 0
 network 200.1.1.0 0.0.0.3 area 0
 network 200.1.100.0 0.0.0.255 area 0
 redistribute static subnets
 end
copy running-config startup-config
```

### 4.3 R-ISPB-BR (AS 200)
```
enable
configure terminal
hostname R-ISPB-BR
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  ISP-B (AS 200) - AUTHORIZED ACCESS ONLY  *
#
interface gi0/0
 description to R-ISPB-PE
 ip address 200.2.0.1 255.255.255.252
 no shutdown
interface gi0/1
 description eBGP to R-ISPA-BR (AS 100)
 ip address 200.0.12.2 255.255.255.252
 no shutdown
interface gi0/2
 description eBGP to R-ISPC-BR (AS 300)
 ip address 200.0.23.1 255.255.255.252
 no shutdown
 exit
ip route 200.2.0.0 255.255.0.0 null0
router ospf 10
 router-id 2.2.2.1
 network 200.2.0.0 0.0.0.3 area 0
 default-information originate always
 exit
router bgp 200
 bgp router-id 2.2.2.1
 neighbor 200.0.12.1 remote-as 100
 neighbor 200.0.23.2 remote-as 300
 network 200.2.0.0 mask 255.255.0.0
 end
copy running-config startup-config
```

### 4.4 R-ISPB-PE
```
enable
configure terminal
hostname R-ISPB-PE
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  ISP-B (AS 200) - AUTHORIZED ACCESS ONLY  *
#
interface gi0/0
 description to R-ISPB-BR
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
 end
copy running-config startup-config
```

### 4.5 R-ISPC-BR (AS 300)
```
enable
configure terminal
hostname R-ISPC-BR
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  ISP-C (AS 300) - AUTHORIZED ACCESS ONLY  *
#
interface gi0/0
 description to R-ISPC-PE
 ip address 200.3.0.1 255.255.255.252
 no shutdown
interface gi0/1
 description eBGP to R-ISPA-BR (AS 100)
 ip address 200.0.13.2 255.255.255.252
 no shutdown
interface gi0/2
 description eBGP to R-ISPB-BR (AS 200)
 ip address 200.0.23.2 255.255.255.252
 no shutdown
 exit
ip route 200.3.0.0 255.255.0.0 null0
router ospf 10
 router-id 3.3.0.1
 network 200.3.0.0 0.0.0.3 area 0
 default-information originate always
 exit
router bgp 300
 bgp router-id 3.3.0.1
 neighbor 200.0.13.1 remote-as 100
 neighbor 200.0.23.1 remote-as 200
 network 200.3.0.0 mask 255.255.0.0
 end
copy running-config startup-config
```

### 4.6 R-ISPC-PE (dual-stack: IPv4 + IPv6 toward its customers)
```
enable
configure terminal
hostname R-ISPC-PE
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  ISP-C (AS 300) - AUTHORIZED ACCESS ONLY  *
#
ipv6 unicast-routing
interface gi0/0
 description to R-ISPC-BR
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
 ipv6 address FE80::1 link-local
 no shutdown
 exit
router ospf 10
 router-id 3.3.0.2
 passive-interface gi0/1
 passive-interface gi0/2
 network 200.3.0.0 0.0.0.3 area 0
 network 200.3.1.0 0.0.0.3 area 0
 network 200.3.50.0 0.0.0.255 area 0
 end
copy running-config startup-config
```

### 4.7 Internet servers and the home PC
**IP settings** (Desktop → IP Configuration → Static):

| Device | IP | Mask | Gateway | DNS |
|---|---|---|---|---|
| SRV-ISP-DNS | 200.1.100.10 | 255.255.255.0 | 200.1.100.1 | 200.1.100.10 |
| SRV-ISP-WEB1 | 200.1.100.11 | 255.255.255.0 | 200.1.100.1 | 200.1.100.10 |
| SRV-ISP-WEB2 | 200.1.100.12 | 255.255.255.0 | 200.1.100.1 | 200.1.100.10 |
| PC-HOME-01 | 200.3.50.10 | 255.255.255.0 | 200.3.50.1 | 200.1.100.10 |

- **SRV-ISP-DNS** → Services → DNS **On** → A records:
  `www.isp-web1.com → 200.1.100.11`, `www.isp-web2.com → 200.1.100.12`,
  **`www.intel.lab → 200.1.10.3`** (the **public** address of our website!). Other services Off.
- **SRV-ISP-WEB1** → Services → HTTP → edit `index.html` → `<h1>ISP Web Server 1</h1>` → Save.
- **SRV-ISP-WEB2** → the same with `<h1>ISP Web Server 2</h1>`.
- `SW-ISPA-NET` and `SW-ISPC-HOME`: just rename them — no configuration needed.

### 4.8 R-HFA-EDGE — the WAN side, NAT and the DSL backup
Log in (`admin` / `Intel#Adm1n`):
```
enable
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
! floating static = backup (distance 10)
ip route 0.0.0.0 0.0.0.0 200.2.9.1 10

! ---------- PAT: all of HQ browses as 200.1.1.2 ----------
ip access-list extended NAT-HFA
 remark --- remote-access VPN users are NOT translated (step 15) ---
 deny ip 10.10.0.0 0.0.255.255 172.16.250.0 0.0.0.255
 permit ip 10.10.0.0 0.0.255.255 any
 exit
ip nat inside source list NAT-HFA interface gi0/1 overload

! ---------- static NAT: publish the web server and NTP ----------
ip nat inside source static 10.10.200.13 200.1.10.3
ip nat inside source static udp 10.10.200.14 123 200.1.10.4 123
 end
copy running-config startup-config
```

### 4.9 ASA — open the holes for the website and NTP
```
enable
configure terminal
access-list OUTSIDE_IN extended permit tcp any host 10.10.200.13 eq www
access-list OUTSIDE_IN extended permit udp any host 10.10.200.14 eq ntp
end
write memory
```
(The ASA sees the **real** server address — the edge router already translated it back.)

### 4.10 What the new commands mean
| Command | In simple English |
|---|---|
| `router bgp 100` | "Start BGP; I am AS 100" |
| `neighbor 200.0.12.2 remote-as 200` | "The router at 200.0.12.2 is in AS 200 — be BGP partners" |
| `network 200.1.0.0 mask 255.255.0.0` | "Announce my block to the world" |
| `ip route 200.1.0.0 255.255.0.0 null0` | Placeholder route so the block exists in the table |
| `default-information originate always` | "Tell my OSPF neighbours: send unknown traffic to me" |
| `redistribute static subnets` | "Share my static routes (customer blocks) inside OSPF" |
| `ip nat inside` / `ip nat outside` | Which side of the router is private / public |
| `ip nat inside source list NAT-HFA interface gi0/1 overload` | "Translate everyone in the list to Gi0/1's address, using ports" = **PAT** |
| `ip nat inside source static 10.10.200.13 200.1.10.3` | "This server is always this public address" = **static NAT** |
| `ip route 0.0.0.0 0.0.0.0 200.2.9.1 10` | Backup default route (floating static) |

---

## Part 5 — Check your work

### 5.1 BGP
Press **Fast Forward Time** several times (BGP is slow to start). On **R-ISPA-BR**:
```
show ip bgp summary
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
200.0.12.2      4   200      10      10        4    0    0 00:02:10        2
200.0.13.2      4   300      10      10        4    0    0 00:02:05        2
```
✅ The last column is a **number** → the partnership is up. (`Active` or `Idle` = not up.)
```
show ip bgp
   Network          Next Hop            Metric LocPrf Weight Path
*> 200.1.0.0/16     0.0.0.0                  0         32768 i
*> 200.2.0.0/16     200.0.12.2               0             0 200 i
*  200.3.0.0/16     200.0.12.2                             0 200 300 i
*>                  200.0.13.2               0             0 300 i
```
`*>` = best path. The `Path` column is the list of ASes on the way. 200.3.0.0 has two paths:
direct (300) and through ISP-B (200 300) — the backup.

On **R-ISPA-PE**: `show ip route` → `O*E2 0.0.0.0/0` (from the BR) and
`S 200.1.10.0/29 via 200.1.1.2`.

### 5.2 Internet from Haifa
| From | Do | Expected |
|---|---|---|
| an ENG PC | Web Browser → `http://www.isp-web1.com` | ✅ "ISP Web Server 1" |
| an ENG PC | `ping 200.1.100.12` | ✅ |
| LT-HFA-GUEST-01 | Web Browser → `http://www.isp-web2.com` | ✅ guests have internet |
| R-HFA-EDGE | `show ip nat translations` | lines like `icmp 200.1.1.2:5 10.10.16.11:5 …` (PAT) + the two static lines |

### 5.3 The internet reaches Intel's website
| From | Do | Expected |
|---|---|---|
| PC-HOME-01 | Web Browser → `http://www.intel.lab` | ✅ "Intel Israel - Intranet" — via 200.1.10.3 |
| PC-HOME-01 | `ping 10.10.16.1` | ❌ private addresses don't exist on the internet |

### 5.4 Redundancy tests
1. **BGP triangle:** shut the link ISPA-BR ↔ ISPC-BR (on R-ISPA-BR: `interface gi0/2` → `shutdown`).
   After a minute (Fast Forward), `show ip bgp` → 200.3.0.0/16 now only via `200 300`. PC-HOME-01
   still opens www.intel.lab ✅. `no shutdown` again.
2. **DSL backup:** on R-HFA-EDGE `ping 200.2.0.1 repeat 1000` (stop with Ctrl+Shift+6), and shut Gi0/1.
   `show ip route` → `S* 0.0.0.0/0 [10/0] via 200.2.9.1` — the backup took over through the modem.
   `no shutdown` Gi0/1 → the primary returns.
   (Limitation: PAT is tied to Gi0/1, so **users** don't browse over DSL — see the challenge.)

---

## Part 6 — Save
`copy running-config startup-config` on all routers, `write memory` on the ASA → **File → Save**.

## Checkpoint
- [ ] 6 BGP partnerships up (2 per BR)
- [ ] Haifa PCs browse both ISP web servers
- [ ] The home PC opens www.intel.lab
- [ ] NAT translations visible on R-HFA-EDGE
- [ ] DSL backup route appears when the primary is down

## If something doesn't work
| Problem | Fix |
|---|---|
| BGP neighbour stuck in **Active/Idle** | Wrong neighbour IP or AS number, or the link is down (`show ip interface brief`). Ping the neighbour first |
| A block missing from `show ip bgp` | The `network ... mask` doesn't **exactly** match a route — check the null0 route and the mask |
| Haifa can't browse but the edge router can | `ip nat inside` missing on Gi0/0, or the NAT ACL doesn't match 10.10.x |
| Home PC can't open www.intel.lab | ISP DNS record (must be 200.1.10.3); `ip route 200.1.10.0 ...` + `redistribute static subnets` on R-ISPA-PE; ASA ACL line for www |
| DSL link red | Cloud's DSL mapping missing (3.3), or the phone cable is on the wrong modem port |

## Challenge
Make **users** survive the primary link failure too, by adding a second PAT rule for Gi0/2.
Hint: two `ip nat inside source list` rules with the same ACL clash — use **route-maps**.

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
Packet Tracer's support for `ip nat ... route-map` depends on the version. If it's refused,
write in the book that the DSL backup serves the router itself, and that route-map NAT is the
real-world solution.
</details>

---

## 👩‍🏫 Teacher notes
- **Lesson 1:** the internet as a network of ASes; BGP vs OSPF (international vs city roads);
  build and cable (14 cables!) → configure ISP-A.
- **Lesson 2:** ISP-B and ISP-C → BGP checks (`show ip bgp`, reading the AS path) → the BGP triangle test.
- **Lesson 3:** NAT types with the switchboard comparison → edge router NAT → ASA holes → tests 5.2–5.3.
- **Lesson 4:** DSL modem + cloud + floating static → backup test → challenge.
- **Show NAT in Simulation mode:** ping from a PC to 200.1.100.11, click the packet before and after
  R-HFA-EDGE — the source address changes from 10.10.16.x to 200.1.1.2.
- **Common student questions**
  - *"Why does BGP take so long?"* — It's designed for stability, not speed. Press Fast Forward.
  - *"Why not run OSPF to the ISP?"* — We'd be telling the internet our internal map — a security
    and stability problem. One static default route is enough.
- **Typical errors:** cross-over vs straight cables between routers; mistyped AS numbers; forgetting
  `ip nat inside` on the inside interface.
