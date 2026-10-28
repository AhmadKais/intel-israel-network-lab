# Step 12 — The WAN: 3 internet providers, BGP, NAT and a DSL backup

> ⏱ 3–4 lessons · 🎯 ministry requirements 1, 3, 4, 9, 10 (static), 22.12, 22.17, 23.1, 23.2, 23.3 · 📁 start from: `my-work/step11.pkt` · 💾 save as: `my-work/step12.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 11 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## 🗺 Your missions in this step
**Lesson 1 — build and ISP-A**
- [ ] **Mission 1** — Open step 11, save as step 12, draw the INTERNET area (3 min)
- [ ] **Mission 2** — Place the WAN devices (10 min)
- [ ] **Mission 3** — Cable the WAN (20 min)
- [ ] **Mission 4** — ISP-A: R-ISPA-BR and R-ISPA-PE (20 min)

**Lesson 2 — ISP-B, ISP-C and BGP**
- [ ] **Mission 5** — ISP-B: R-ISPB-BR and R-ISPB-PE (15 min)
- [ ] **Mission 6** — ISP-C: R-ISPC-BR and R-ISPC-PE (15 min)
- [ ] **Mission 7** — Check BGP and read the BGP table (15 min)
- [ ] **Mission 8** — Internet servers and the home PC (15 min)

**Lesson 3 — Haifa on the internet**
- [ ] **Mission 9** — R-HFA-EDGE: WAN link and routing (10 min)
- [ ] **Mission 10** — R-HFA-EDGE: PAT so employees can browse (10 min)
- [ ] **Mission 11** — R-HFA-EDGE: static NAT + ASA holes, so the internet sees our website (15 min)

**Lesson 4 — the backup line**
- [ ] **Mission 12** — The DSL modem and the cloud (15 min)
- [ ] **Mission 13** — Redundancy tests (15 min)
- [ ] **Exercises** A and B
- [ ] **🛠 Your own project**

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

### 1.7 Address plan of the WAN
Keep this table open — every mission takes its numbers from here.

| Router | Gi0/0 | Gi0/1 | Gi0/2 |
|---|---|---|---|
| R-ISPA-BR | 200.1.0.1/30 | 200.0.12.1/30 | 200.0.13.1/30 |
| R-ISPA-PE | 200.1.0.2/30 | 200.1.1.1/30 → Haifa | 200.1.100.1/24 → servers |
| R-ISPB-BR | 200.2.0.1/30 | 200.0.12.2/30 | 200.0.23.1/30 |
| R-ISPB-PE | 200.2.0.2/30 | 200.2.1.1/30 → Jerusalem | 200.2.9.1/30 → DSL (Haifa backup) |
| R-ISPC-BR | 200.3.0.1/30 | 200.0.13.2/30 | 200.0.23.2/30 |
| R-ISPC-PE | 200.3.0.2/30 | 200.3.1.1/30 → Kiryat Gat | 200.3.50.1/24 → home users |
| R-HFA-EDGE | 10.10.7.1/30 (ASA) | **200.1.1.2/30** (ISP-A) | **200.2.9.2/30** (DSL) |

### 1.8 Before you type on any new router
- A new router asks `Would you like to enter the initial configuration dialog? [yes/no]:` →
  type **`no`**, Enter, then Enter again → `Router>`.
- Router ports start **shut** — every interface block ends with `no shutdown`.
- ISP routers belong to "other companies", so they get only the basics: name, secret, banner
  (the ministry wants banners on **all** routers).

---

## Mission 1 — Open step 11 and save as step 12 (how? → M1)
**File → Open** → `my-work/step11.pkt` → **File → Save As** → `my-work/step12.pkt`.
Draw a big ellipse on the right labelled `INTERNET / WAN` (M19 — choose the **ellipse** tool
instead of the rectangle).

---

## Mission 2 — Place the WAN devices (how? → M2, M4)
| Where in the device bar | Model | Display Name |
|---|---|---|
| Network Devices → Routers | **2911** ×6 | `R-ISPA-BR`, `R-ISPA-PE`, `R-ISPB-BR`, `R-ISPB-PE`, `R-ISPC-BR`, `R-ISPC-PE` |
| Network Devices → Switches | **2960** ×2 | `SW-ISPA-NET`, `SW-ISPC-HOME` |
| End Devices | **Server** ×3 | `SRV-ISP-DNS`, `SRV-ISP-WEB1`, `SRV-ISP-WEB2` |
| End Devices | **PC** | `PC-HOME-01` |
| Network Devices → **WAN Emulation** | **DSL-Modem** | `DSL-HFA` |
| Network Devices → **WAN Emulation** | **Cloud** | `CLOUD-DSL` |

💡 Arrange them like the drawing above: ISP-A on the left, ISP-B in the middle, ISP-C on the right.
A tidy drawing saves you from cabling mistakes.

✅ 14 new devices, all named.

---

## Mission 3 — Cable the WAN (how? → M5)
Router ↔ router = **Cross-Over**. Router/server/PC ↔ switch = **Straight-Through**.
Leave rows 12–14 (DSL) for Mission 12.

| # | Cable | Click first | Port | Click second | Port |
|---|---|---|---|---|---|
| 1 | Cross-Over | R-ISPA-BR | Gi0/0 | R-ISPA-PE | Gi0/0 |
| 2 | Cross-Over | R-ISPB-BR | Gi0/0 | R-ISPB-PE | Gi0/0 |
| 3 | Cross-Over | R-ISPC-BR | Gi0/0 | R-ISPC-PE | Gi0/0 |
| 4 | Cross-Over | R-ISPA-BR | Gi0/1 | R-ISPB-BR | Gi0/1 |
| 5 | Cross-Over | R-ISPA-BR | Gi0/2 | R-ISPC-BR | Gi0/1 |
| 6 | Cross-Over | R-ISPB-BR | Gi0/2 | R-ISPC-BR | Gi0/2 |
| 7 | Cross-Over | R-ISPA-PE | Gi0/1 | **R-HFA-EDGE** | **Gi0/1** |
| 8 | Straight | R-ISPA-PE | Gi0/2 | SW-ISPA-NET | Gi0/1 |
| 9a | Straight | SRV-ISP-DNS | Fa0 | SW-ISPA-NET | Fa0/1 |
| 9b | Straight | SRV-ISP-WEB1 | Fa0 | SW-ISPA-NET | Fa0/2 |
| 9c | Straight | SRV-ISP-WEB2 | Fa0 | SW-ISPA-NET | Fa0/3 |
| 10 | Straight | R-ISPC-PE | Gi0/2 | SW-ISPC-HOME | Gi0/1 |
| 11 | Straight | PC-HOME-01 | Fa0 | SW-ISPC-HOME | Fa0/1 |

✅ 13 cables. The router ends are **red** — normal, router ports are shut until you configure them.
Check every label (M6).

---

## Mission 4 — ISP-A: R-ISPA-BR and R-ISPA-PE

### 4a — R-ISPA-BR: basics and interfaces
👉 **Where:** **R-ISPA-BR** → CLI → `no` to the dialog → `Router>`.

⌨️ Type:
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
```
🔍 **Check now:** `do show ip interface brief` → Gi0/0, Gi0/1, Gi0/2 with their addresses.
(Gi0/1 and Gi0/2 stay `down` until the other ISPs are configured — that's fine.)

### 4b — R-ISPA-BR: OSPF inside and BGP outside
⌨️ Type:
```
ip route 200.1.0.0 255.255.0.0 null0
router ospf 10
 router-id 1.1.1.1
 network 200.1.0.0 0.0.0.3 area 0
 default-information originate always
 exit
router bgp 100
 bgp router-id 1.1.1.1
 neighbor 200.0.12.2 remote-as 200
 neighbor 200.0.13.2 remote-as 300
 network 200.1.0.0 mask 255.255.0.0
 end
copy running-config startup-config
```
| Command | In simple English |
|---|---|
| `ip route 200.1.0.0 255.255.0.0 null0` | Placeholder route so the block exists in the table |
| `default-information originate always` | "Tell my OSPF neighbours: send unknown traffic to me" |
| `router bgp 100` | "Start BGP; I am AS 100" |
| `neighbor 200.0.12.2 remote-as 200` | "The router at 200.0.12.2 is in AS 200 — be BGP partners" |
| `network 200.1.0.0 mask 255.255.0.0` | "Announce my block to the world" |

### 4c — R-ISPA-PE
👉 **Where:** **R-ISPA-PE** → CLI → `no` → `Router>`.

⌨️ Type:
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
| Command | In simple English |
|---|---|
| `ip route 200.1.10.0 255.255.255.248 200.1.1.2` | "Intel Haifa's public static-NAT block is behind the customer's router" |
| `redistribute static subnets` | "Share my static routes (customer blocks) inside OSPF" |

Press **Fast Forward Time**.

✅ `%OSPF-5-ADJCHG: Process 10, Nbr 1.1.1.1 ... from LOADING to FULL`.
🔍 **Check now:** on R-ISPA-PE `ping 200.1.0.1` → `!!!!!`; `show ip route` → `O*E2 0.0.0.0/0`
(the default route from the BR).

---

## Mission 5 — ISP-B: R-ISPB-BR and R-ISPB-PE
Same pattern as ISP-A. Differences: AS **200**, block **200.2.0.0/16**, router-IDs **2.2.2.x**.

### 5a — R-ISPB-BR
👉 **Where:** **R-ISPB-BR** → CLI → `no` → `Router>`.

⌨️ Type:
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
Press **Fast Forward Time** a few times.

✅ `%BGP-5-ADJCHANGE: neighbor 200.0.12.1 Up` — your **first BGP partnership** (ISP-A ↔ ISP-B).

### 5b — R-ISPB-PE
👉 **Where:** **R-ISPB-PE** → CLI → `no` → `Router>`.

⌨️ Type:
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
> `ip route 200.2.10.0 ...` points to Jerusalem's dynamic-NAT pool (step 13). Gi0/1 and Gi0/2
> stay down until steps 12 (DSL) and 13 — that's fine.

---

## Mission 6 — ISP-C: R-ISPC-BR and R-ISPC-PE
AS **300**, block **200.3.0.0/16**, router-IDs **3.3.0.x**.

### 6a — R-ISPC-BR
👉 **Where:** **R-ISPC-BR** → CLI → `no` → `Router>`.

⌨️ Type:
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

### 6b — R-ISPC-PE (dual-stack: IPv4 + IPv6 toward its customers)
👉 **Where:** **R-ISPC-PE** → CLI → `no` → `Router>`.

⌨️ Type:
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
> The IPv6 lines prepare ISP-C for Kiryat Gat in step 14. Type them now; you'll use them later.

Press **Fast Forward Time** several times (BGP is slow to start).

---

## Mission 7 — Check BGP and read the BGP table
👉 **Where:** **R-ISPA-BR** (prompt `R-ISPA-BR#`).

### 7a — Are the partners talking?
⌨️ `show ip bgp summary`
```
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
200.0.12.2      4   200      10      10        4    0    0 00:02:10        2
200.0.13.2      4   300      10      10        4    0    0 00:02:05        2
```
✅ The **last column is a number** → the partnership is up.
❌ `Active` or `Idle` = not up → ping the neighbour's address first. If the ping fails: cable or
interface problem. If the ping works: wrong `neighbor` IP or AS number.

Do the same on **R-ISPB-BR** and **R-ISPC-BR**. Tick (2 numbers each):
- [ ] ISPA-BR  - [ ] ISPB-BR  - [ ] ISPC-BR

### 7b — Read the BGP table
⌨️ `show ip bgp`
```
   Network          Next Hop            Metric LocPrf Weight Path
*> 200.1.0.0/16     0.0.0.0                  0         32768 i
*> 200.2.0.0/16     200.0.12.2               0             0 200 i
*  200.3.0.0/16     200.0.12.2                             0 200 300 i
*>                  200.0.13.2               0             0 300 i
```
How to read it:
- `*>` = the **best** path (the one used).
- `Path` = the list of ASes on the way. `i` at the end = "came from inside that AS".
- `200.3.0.0` has **two** paths: direct (`300`) and through ISP-B (`200 300`). The shorter one wins;
  the longer one is the backup.

📝 In your notebook: "To reach ISP-C, ISP-A goes ______; if that link breaks it goes ______."

---

## Mission 8 — Internet servers and the home PC

### 8a — IP settings (how? → M14)
Desktop → IP Configuration → **Static**:

| Device | IP | Mask | Gateway | DNS |
|---|---|---|---|---|
| SRV-ISP-DNS | 200.1.100.10 | 255.255.255.0 | 200.1.100.1 | 200.1.100.10 |
| SRV-ISP-WEB1 | 200.1.100.11 | 255.255.255.0 | 200.1.100.1 | 200.1.100.10 |
| SRV-ISP-WEB2 | 200.1.100.12 | 255.255.255.0 | 200.1.100.1 | 200.1.100.10 |
| PC-HOME-01 | 200.3.50.10 | 255.255.255.0 | 200.3.50.1 | 200.1.100.10 |

### 8b — Services (how? → M20)
- **SRV-ISP-DNS** → Services → DNS **On** → add A records:

  | Name | Address |
  |---|---|
  | `www.isp-web1.com` | 200.1.100.11 |
  | `www.isp-web2.com` | 200.1.100.12 |
  | `www.intel.lab` | **200.1.10.3** ← the **public** address of our website! |

  Switch the other services **Off**.
- **SRV-ISP-WEB1** → Services → HTTP → `index.html` → **(edit)** → `<h1>ISP Web Server 1</h1>` → **Save**.
- **SRV-ISP-WEB2** → the same with `<h1>ISP Web Server 2</h1>`.
- `SW-ISPA-NET` and `SW-ISPC-HOME`: just named — no configuration needed.

🔍 **Check now:** PC-HOME-01 → **Web Browser** → `http://www.isp-web1.com` → **Go** →
"ISP Web Server 1". ✅ The internet works **across 3 ASes** (home PC in AS 300, server in AS 100).

❌ **If it fails:** `ping 200.1.100.11` from PC-HOME-01. Ping fails → BGP (Mission 7). Ping works
but the name fails → DNS record or the PC's DNS setting.

---

## Mission 9 — R-HFA-EDGE: WAN link and routing
👉 **Where:** **R-HFA-EDGE** → CLI → log in (`admin` / `Intel#Adm1n`) → `R-HFA-EDGE#`.

⌨️ Type:
```
configure terminal
interface gi0/1
 description PRIMARY to R-ISPA-PE
 ip address 200.1.1.2 255.255.255.252
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 200.1.1.1
```
🔍 **Check now:** `do ping 200.1.100.11` → `!!!!!` — the edge router itself reaches the internet.

But try from an **Engineering PC**: `ping 200.1.100.11` → ❌ timed out.
🧠 **Why?** The packet leaves with source `10.10.16.x` — a private address. The internet has no
route back to 10.x. We need NAT.

---

## Mission 10 — R-HFA-EDGE: PAT so employees can browse
👉 **Where:** R-HFA-EDGE, prompt `R-HFA-EDGE(config)#`.

### 10a — Mark the inside and the outside
⌨️ Type:
```
interface gi0/0
 ip nat inside
interface gi0/1
 ip nat outside
 exit
```

### 10b — Who may be translated, and to what
⌨️ Type:
```
ip access-list extended NAT-HFA
 remark --- remote-access VPN users are NOT translated (step 15) ---
 deny ip 10.10.0.0 0.0.255.255 172.16.250.0 0.0.0.255
 permit ip 10.10.0.0 0.0.255.255 any
 exit
ip nat inside source list NAT-HFA interface gi0/1 overload
```
| Command | In simple English |
|---|---|
| `ip nat inside` / `ip nat outside` | Which side of the router is private / public |
| `ip nat inside source list NAT-HFA interface gi0/1 overload` | "Translate everyone in the list to Gi0/1's address, using ports" = **PAT** |

🔍 **Check now:**
1. From an ENG PC: `ping 200.1.100.11` → ✅ replies now.
2. On R-HFA-EDGE: `do show ip nat translations` → a line like
   `icmp 200.1.1.2:5   10.10.16.11:5   200.1.100.11:5 ...` — inside address on the right, the
   shared public address on the left.
3. ENG PC → Web Browser → `http://www.isp-web1.com` → ✅.

---

## Mission 11 — Static NAT + ASA holes: the internet sees our website
🧠 **Why:** PAT only lets **us** start conversations. A visitor on the internet can't start one
toward 10.10.200.13 — so we give the web server a permanent public address.

### 11a — On R-HFA-EDGE
⌨️ Type:
```
ip nat inside source static 10.10.200.13 200.1.10.3
ip nat inside source static udp 10.10.200.14 123 200.1.10.4 123
 end
copy running-config startup-config
```
| Command | In simple English |
|---|---|
| `ip nat inside source static 10.10.200.13 200.1.10.3` | "This server is always this public address" = **static NAT** |
| `... static udp 10.10.200.14 123 200.1.10.4 123` | Publish only port 123 (NTP) of the NTP server |

### 11b — On ASA-HFA-FW1: open the holes
👉 **Where:** ASA → `enable` → password `Intel#En4ble` → `ASA-HFA-FW1#`.

⌨️ Type:
```
configure terminal
access-list OUTSIDE_IN extended permit tcp any host 10.10.200.13 eq www
access-list OUTSIDE_IN extended permit udp any host 10.10.200.14 eq ntp
end
write memory
```
> The ASA sees the **real** server address — the edge router already translated it back.

🔍 **Check now:**

| From | Do | Expected |
|---|---|---|
| PC-HOME-01 | Web Browser → `http://www.intel.lab` | ✅ "Intel Israel - Intranet" — via 200.1.10.3 |
| PC-HOME-01 | `ping 10.10.16.1` | ❌ private addresses don't exist on the internet |
| LT-HFA-GUEST-01 | Web Browser → `http://www.isp-web2.com` | ✅ guests have internet now (step 10) |
| R-HFA-EDGE | `show ip nat translations` | PAT lines + the two static lines (`200.1.10.3`, `200.1.10.4`) |

❌ **If the home PC gets no page:** check in this order: the ISP DNS record (`200.1.10.3`);
`ip route 200.1.10.0 ...` + `redistribute static subnets` on R-ISPA-PE; the ASA `www` line.

---

## Mission 12 — The DSL modem and the cloud
🧠 **Why:** a second line through a **different** provider. If ISP-A's link dies, Haifa still
has a way out.

### 12a — Cables (careful: the phone cable!)
| # | Cable | Click first | Port | Click second | Port |
|---|---|---|---|---|---|
| 12 | Straight | R-HFA-EDGE | **Gi0/2** | DSL-HFA | **Port 1** (Ethernet) |
| 13 | **Phone** (the thin black line in Connections) | DSL-HFA | **Port 0** | CLOUD-DSL | **Modem4** |
| 14 | Straight | CLOUD-DSL | **Ethernet6** | R-ISPB-PE | **Gi0/2** |

### 12b — Tell the cloud to connect the modem to ISP-B
👉 CLOUD-DSL → **Config** tab → left list **DSL** → *From Port* **Modem4**, *To Port*
**Ethernet6** → **Add**.

✅ A line `Modem4 ↔ Ethernet6` appears in the table.

### 12c — Edge router: the backup interface and the floating static route
👉 **Where:** R-HFA-EDGE → `configure terminal`.

⌨️ Type:
```
interface gi0/2
 description BACKUP via DSL to R-ISPB-PE
 ip address 200.2.9.2 255.255.255.252
 ip nat outside
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 200.2.9.1 10
 end
copy running-config startup-config
```
| Command | In simple English |
|---|---|
| `ip route 0.0.0.0 0.0.0.0 200.2.9.1 10` | Backup default route (floating static, distance 10) |

🔍 **Check now:**
1. `ping 200.2.9.1` → `!!!!!` (the DSL line works).
2. `show ip route static` → **only** `S* 0.0.0.0/0 [1/0] via 200.1.1.1` — the backup is hidden.
   That's correct: it waits.

❌ **If the DSL link is red:** the cloud mapping (12b) is missing, or the phone cable is on the
wrong modem port.

---

## Mission 13 — Redundancy tests

### 13a — The BGP triangle
1. On **R-ISPA-BR**: `configure terminal` → `interface gi0/2` → `shutdown` → `end`.
2. Fast Forward Time (about a minute). `show ip bgp` → 200.3.0.0/16 now only via `200 300`.
3. PC-HOME-01 → Web Browser → `http://www.intel.lab` → still ✅ (the long way round).
4. Back: `configure terminal` → `interface gi0/2` → `no shutdown` → `end`.

### 13b — The DSL backup
1. On R-HFA-EDGE: `ping 200.2.0.1 repeat 1000` (stop later with **Ctrl+Shift+6**).
2. Break the primary line: `configure terminal` → `interface gi0/1` → `shutdown` → `end`.
3. `show ip route static` → `S* 0.0.0.0/0 [10/0] via 200.2.9.1` — the backup took over through
   the modem.
4. `configure terminal` → `interface gi0/1` → `no shutdown` → `end` → the primary returns
   (`[1/0] via 200.1.1.1`).

> Limitation: PAT is tied to Gi0/1, so **users** don't browse over DSL — see Exercise B.

💾 `copy running-config startup-config` on all routers, `write memory` on the ASA → **File → Save**.

---

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
| DSL link red | Cloud's DSL mapping missing (12b), or the phone cable is on the wrong modem port |
| A router link red after configuring | `no shutdown` missing, or Straight-Through used between two routers (must be Cross-Over) |

---

## 🏋 Exercises

### Exercise A — Wrong AS number ⭐⭐
- 🎯 **Task:** see what a BGP typo looks like and find it with `show` commands only.
- 📁 **Start from:** `step12.pkt` → **Save As** `step12-exA.pkt`.
- 👣 **Steps:**
  1. On R-ISPB-BR: `configure terminal` → `router bgp 200` → `no neighbor 200.0.23.2 remote-as 300`
     → `neighbor 200.0.23.2 remote-as 333` → `end`.
  2. Fast Forward Time. On R-ISPB-BR: `show ip bgp summary`. On R-ISPC-BR: the same.
  3. Fix it back.
- ✅ **Done when:** you can say which column showed the problem and what message appeared.

<details><summary>💡 Hint</summary>

Look at the last column of `show ip bgp summary` and at the console messages
(`%BGP-3-NOTIFICATION`).
</details>
<details><summary>✔ Answer</summary>

The neighbour stays in `Active`/`Idle` instead of a number. R-ISPC-BR says it is AS 300, R-ISPB-BR
expects 333 → the session is refused ("bad remote AS"). Both sides must agree on **IP and AS**.
The triangle still works through ISP-A, so the internet doesn't break — that's redundancy.
</details>

### Exercise B — Users browse over DSL too ⭐⭐⭐
- 🎯 **Task:** make **users** survive the primary link failure too, by adding a second PAT rule
  for Gi0/2.
- 📁 **Start from:** `step12.pkt` → **Save As** `step12-exB.pkt`.
- 👣 **Steps:** shut Gi0/1, try to browse from a PC (fails). Find a way to have two PAT rules.
- ✅ **Done when:** with Gi0/1 shut, an ENG PC opens `http://www.isp-web1.com` — or you wrote in
  the book why PT refused.

<details><summary>💡 Hint</summary>

Two `ip nat inside source list` rules with the same ACL clash — use **route-maps** that match
the ACL **and** the exit interface.
</details>
<details><summary>✔ Answer</summary>

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

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

- [ ] Open `my-project/step11.pkt` → **Save As** `my-project/step12.pkt`
- [ ] Write your WAN address table first (like 1.7) with your AS numbers and blocks
- [ ] Place and cable the 6 ISP routers, servers, home PC
- [ ] Configure the 3 ISPs; all BGP partnerships show a number
- [ ] Internet servers + DNS (with your website's **public** address)
- [ ] Edge router: WAN link, default route, PAT, static NAT; ASA holes
- [ ] DSL backup + floating static; both redundancy tests

| In the example | In your project — take it from |
|---|---|
| ISP AS numbers 100 / 200 / 300 | your own AS numbers |
| Public blocks `200.1/2/3.0.0/16`, inter-ISP `200.0.0.0/16` | plan part C (your ISP blocks) |
| Static NAT for the web and NTP servers, PAT for browsing | plan part E (NAT row — static at one site, dynamic at another, overload at the third — req. 10) |
| DSL backup with a floating static route | plan part E (WAN backup row) |
| Static routing from the site to the WAN | keep — required by the ministry (req. 22.17) |

**Check:** `show ip bgp summary` shows every neighbor established; an inside PC opens an internet
web page; `show ip nat translations` shows your addresses.

💾 Save as `my-project/step12.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★1 What is the role of NAT? What are the 3 types and how do they differ?**

<details><summary>Answer</summary>

NAT translates private addresses to public ones (and back), so private networks can reach the internet and public addresses are saved. **Static**: one private ↔ one public, permanent — for a server that the internet must reach. **Dynamic**: private addresses take a free address from a pool of public ones. **Overload (PAT)**: many private addresses share one public address, separated by port numbers.

**Show it:** `show running-config | include nat`, `show ip nat translations`
</details>

**2. ★1 What is PAT?**

<details><summary>Answer</summary>

Port Address Translation (NAT overload): many inside hosts share one public IP; the router tells their sessions apart by the source port number.

**Show it:** `show ip nat translations` → the same public IP with different ports
</details>

**3. ★1 Show the NAT configuration in your project.**

<details><summary>Answer</summary>

`ip nat inside` / `ip nat outside` on the interfaces; `ip nat inside source static …` for each published server; the ACL of the inside addresses + `ip nat inside source list … interface … overload` for browsing.

**Show it:** `show running-config`, `show ip nat translations`
</details>

**4. ★2 Which command shows the BGP routing table?**

<details><summary>Answer</summary>

`show ip bgp` (the BGP table); `show ip bgp summary` (the neighbors and their state).

**Show it:** on an ISP router
</details>

**5. ★2 What is the difference between an IGP and an EGP? What is an Autonomous System?**

<details><summary>Answer</summary>

An AS is a network under one administration with one routing policy, identified by an AS number. An IGP (OSPF, EIGRP) routes **inside** an AS; an EGP (BGP) routes **between** ASes.

**Show it:** `show ip bgp summary` → AS numbers
</details>

**6. ★2 What is the difference between iBGP and eBGP?**

<details><summary>Answer</summary>

eBGP: between routers in **different** ASes (administrative distance 20). iBGP: between routers in the **same** AS (administrative distance 200).

**Show it:** `show ip bgp summary`
</details>

**7. ★2 Which BGP attribute prevents loops?**

<details><summary>Answer</summary>

AS_PATH: the list of ASes the route passed through. A router that sees its own AS number in the path rejects the route.

**Show it:** `show ip bgp` → Path column
</details>

**8. ★3 Which attribute decides the exit point out of the AS? What are MED and Weight?**

<details><summary>Answer</summary>

LOCAL_PREF decides which router is the **exit** from your AS (higher wins, shared inside the AS). MED tells a **neighbor AS** which entry into your AS you prefer (lower wins). Weight is Cisco-only, local to one router, and checked first (higher wins).

**Show it:** —
</details>

**9. ◆ Why is routing from the site to the WAN static and not through the IGP? What is a floating static route?**

<details><summary>Answer</summary>

So the internal routing information is not exposed to the provider, and the provider can't inject routes into the company (req. 22.17). A floating static route is a backup route with a higher administrative distance (e.g. `… 10`): it is used only when the main route disappears — here, the DSL backup line.

**Show it:** `show ip route static`; shut the main link and look again
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

In a copy of your file, remove the NAT lines from the edge router. Without this page, write them
again from your plan (inside/outside, static NAT for your server, PAT for browsing). Target:
**10 minutes**. Then browse from a PC and open your server from the internet side, and show
`show ip nat translations`. Explain one BGP neighbor with `show ip bgp summary`.

---

## 👩‍🏫 Teacher notes
- **Lesson 1:** the internet as a network of ASes; BGP vs OSPF (international vs city roads);
  Missions 1–4 (build, cable, ISP-A).
- **Lesson 2:** Missions 5–8 — ISP-B and ISP-C → BGP checks (`show ip bgp`, reading the AS path) →
  servers. Celebrate the first cross-AS web page.
- **Lesson 3:** NAT types with the switchboard comparison → Missions 9–11. Let students **see** the
  failed ping in Mission 9 before adding NAT.
- **Lesson 4:** Missions 12–13 (DSL + cloud + floating static, redundancy tests) → exercises.
- **Show NAT in Simulation mode:** ping from a PC to 200.1.100.11, click the packet before and after
  R-HFA-EDGE — the source address changes from 10.10.16.x to 200.1.1.2.
- **Common student questions**
  - *"Why does BGP take so long?"* — It's designed for stability, not speed. Press Fast Forward.
  - *"Why not run OSPF to the ISP?"* — We'd be telling the internet our internal map — a security
    and stability problem. One static default route is enough.
- **Typical errors:** cross-over vs straight cables between routers; mistyped AS numbers; forgetting
  `ip nat inside` on the inside interface.
