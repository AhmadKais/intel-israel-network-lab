# Design — Intel Israel network (source of truth)

Every step in this repo uses the numbers on this page. If you change something here,
change it everywhere.

## 1. Naming convention

```
<ROLE>-<SITE>-<AREA>-<FUNCTION><INDEX>
```

| Part | Values |
|---|---|
| ROLE | `R` router · `SW` Layer-2 switch · `SWL3` Layer-3 switch · `ASA` firewall · `AP` access point · `SRV` server · `PC` · `LT` laptop · `PRN` printer |
| SITE | `HFA` Haifa (HQ) · `JLM` Jerusalem · `KG` Kiryat Gat · `ISPA` / `ISPB` / `ISPC` providers |
| AREA | building (`A`/`B`/`C`) or floor (`F1`/`F2`/`F3`) — optional |
| FUNCTION | `ACC` access · `DIST` distribution · `CORE` core · `CC` collapsed core · `EDGE` internet edge · `BR` ISP border · `PE` ISP provider-edge |

Examples: `SWL3-HFA-A-DIST1`, `SW-JLM-F2-ACC1`, `R-KG-EDGE`, `PC-HFA-ENG-01`, `SRV-HFA-DMZ-WEB`.

Domain name everywhere: **`intel.lab`**

---

## 2. Address blocks

| Site | IPv4 | IPv6 | IGP | NAT type |
|---|---|---|---|---|
| Haifa HQ | `10.10.0.0/16` | — | OSPFv2 multi-area + virtual link | **Static** (+ PAT for browsing) |
| Jerusalem | `10.20.0.0/16` | — | EIGRP AS 100 | **Dynamic** (pool) |
| Kiryat Gat | `10.30.0.0/16` (legacy only) | `2001:DB8:30::/48` | OSPFv3 | **Overload (PAT)** |
| VPN | `172.16.12.0/30` GRE tunnel · `172.16.250.0/24` remote-access pool | | | |
| ISP-A (AS 100) | `200.1.0.0/16` | | OSPF 10 inside, eBGP outside | |
| ISP-B (AS 200) | `200.2.0.0/16` | | OSPF 10 inside, eBGP outside | |
| ISP-C (AS 300) | `200.3.0.0/16` | `2001:DB8:C::/48` | OSPF 10 inside, eBGP outside | |
| Inter-ISP links | `200.0.0.0/16` | | | |

Each site gets **consecutive** blocks from the 10.0.0.0/8 master block (requirement 22.4).

---

## 3. Haifa HQ — `10.10.0.0/16`

The /16 is split into /20s so that **each OSPF area is one summarizable block**:

| OSPF area | Block | What lives there |
|---|---|---|
| Area 0 | `10.10.0.0/20` | core switches, server farm, P2P links, loopbacks |
| Area 1 | `10.10.16.0/20` | Building A |
| Area 2 | `10.10.32.0/20` | Building B (+ the B↔C links) |
| Area 3 | `10.10.48.0/20` | Building C — **not connected to area 0 → virtual link through area 2** |
| (none) | `10.10.200.0/24` | DMZ behind the ASA |
| (none) | `10.10.7.0/30` | ASA outside ↔ edge router |

### VLANs — Building A (area 1)

| VLAN | Name | Subnet | Gateway (HSRP VIP) | DIST1 | DIST2 | HSRP/STP primary |
|---|---|---|---|---|---|---|
| 10 | ENGINEERING | 10.10.16.0/24 | .1 | .2 | .3 | DIST1 |
| 20 | FINANCE | 10.10.17.0/24 | .1 | .2 | .3 | DIST2 |
| 30 | PRINTERS-A | 10.10.18.0/24 | .1 | .2 | .3 | DIST1 |
| 99 | MGMT-A | 10.10.19.0/24 | .1 | .2 | .3 | DIST1 |
| 110 | WIFI-EMP | 10.10.20.0/24 | .1 | .2 | .3 | DIST1 |
| 120 | WIFI-GUEST | 10.10.21.0/24 | .1 | .2 | .3 | DIST2 |
| 998 | NATIVE | — (unused, native on trunks) | | | | |
| 999 | PARKING | — (unused ports, shut down) | | | | |

### VLANs — Building B (area 2)

| VLAN | Name | Subnet | Primary |
|---|---|---|---|
| 40 | HR | 10.10.32.0/24 | DIST1 |
| 50 | MARKETING | 10.10.33.0/24 | DIST2 |
| 60 | PRINTERS-B | 10.10.34.0/24 | DIST1 |
| 99 | MGMT-B | 10.10.35.0/24 | DIST1 |

### VLANs — Building C (area 3)

| VLAN | Name | Subnet | Primary |
|---|---|---|---|
| 70 | RND | 10.10.48.0/24 | DIST1 |
| 80 | IT | 10.10.49.0/24 | DIST2 |
| 90 | PRINTERS-C | 10.10.50.0/24 | DIST1 |
| 99 | MGMT-C | 10.10.51.0/24 | DIST1 |

> VLAN 99 exists in every building but is a **different subnet** in each one. That is fine:
> each building is its own Layer-2 island (its own VTP domain), joined to the others only
> by routed links.

**6 departments in HQ:** Engineering, Finance, HR, Marketing, R&D, IT.

### Core / server farm (area 0)

| VLAN | Name | Subnet | VIP | CORE1 | CORE2 | ASA inside |
|---|---|---|---|---|---|---|
| 100 | SERVERS | 10.10.1.0/24 | .1 | .2 | .3 | **.254** |

| Server | IP |
|---|---|
| SRV-HFA-SYSLOG | 10.10.1.10 |
| SRV-HFA-AAA (TACACS+ / RADIUS) | 10.10.1.11 |

### Loopbacks (router-IDs)

| Device | Loopback0 |
|---|---|
| SWL3-HFA-CORE1 / CORE2 | 10.10.5.1 / 10.10.5.2 |
| SWL3-HFA-A-DIST1 / DIST2 | 10.10.5.11 / 10.10.5.12 |
| SWL3-HFA-B-DIST1 / DIST2 | 10.10.5.21 / 10.10.5.22 |
| SWL3-HFA-C-DIST1 / DIST2 | **10.10.63.1 / 10.10.63.2** (inside area 3's block on purpose) |

### Point-to-point links (/30)

| Link | Subnet | Side 1 | Side 2 | Area |
|---|---|---|---|---|
| A-DIST1 Gi1/1/1 ↔ CORE1 Gi1/0/1 | 10.10.4.0/30 | .1 | .2 | 0 |
| A-DIST1 Gi1/1/2 ↔ CORE2 Gi1/0/1 | 10.10.4.4/30 | .5 | .6 | 0 |
| A-DIST2 Gi1/1/1 ↔ CORE1 Gi1/0/2 | 10.10.4.8/30 | .9 | .10 | 0 |
| A-DIST2 Gi1/1/2 ↔ CORE2 Gi1/0/2 | 10.10.4.12/30 | .13 | .14 | 0 |
| B-DIST1 Gi1/1/1 ↔ CORE1 Gi1/0/3 | 10.10.4.16/30 | .17 | .18 | 0 |
| B-DIST1 Gi1/1/2 ↔ CORE2 Gi1/0/3 | 10.10.4.20/30 | .21 | .22 | 0 |
| B-DIST2 Gi1/1/1 ↔ CORE1 Gi1/0/4 | 10.10.4.24/30 | .25 | .26 | 0 |
| B-DIST2 Gi1/1/2 ↔ CORE2 Gi1/0/4 | 10.10.4.28/30 | .29 | .30 | 0 |
| CORE1 Gi1/0/23 ↔ CORE2 Gi1/0/23 | 10.10.4.48/30 | .49 | .50 | 0 |
| B-DIST1 Gi1/1/3 ↔ C-DIST1 Gi1/1/1 | 10.10.36.0/30 | .1 | .2 | 2 |
| B-DIST2 Gi1/1/3 ↔ C-DIST1 Gi1/1/2 | 10.10.36.4/30 | .5 | .6 | 2 |
| B-DIST1 Gi1/1/4 ↔ C-DIST2 Gi1/1/1 | 10.10.36.8/30 | .9 | .10 | 2 |
| B-DIST2 Gi1/1/4 ↔ C-DIST2 Gi1/1/2 | 10.10.36.12/30 | .13 | .14 | 2 |
| ASA Gi1/1 (outside) ↔ R-HFA-EDGE Gi0/0 | 10.10.7.0/30 | .2 (ASA) | .1 (router) | — |

### DMZ — `10.10.200.0/24` (gateway = ASA dmz `10.10.200.1`)

| Server | IP | Public (static NAT) |
|---|---|---|
| SRV-HFA-DMZ-DNS | 10.10.200.11 | |
| SRV-HFA-DMZ-MAIL | 10.10.200.12 | |
| SRV-HFA-DMZ-WEB | 10.10.200.13 | **200.1.10.3** (all ports) |
| SRV-HFA-DMZ-NTP | 10.10.200.14 | **200.1.10.4** (UDP 123 only) — main NTP server |
| SRV-HFA-DMZ-FTP | 10.10.200.15 | |
| SRV-HFA-DMZ-AAA | 10.10.200.16 | |

### Haifa edge

| Interface | Address | To |
|---|---|---|
| R-HFA-EDGE Gi0/0 | 10.10.7.1/30 (nat inside) | ASA outside |
| R-HFA-EDGE Gi0/1 | 200.1.1.2/30 (nat outside) | R-ISPA-PE (primary) |
| R-HFA-EDGE Gi0/2 | 200.2.9.2/30 (nat outside) | DSL modem → cloud → R-ISPB-PE (backup) |
| Public block routed to HQ | 200.1.10.0/29 | static NAT addresses |

---

## 4. Jerusalem — `10.20.0.0/16` — collapsed core, EIGRP 100

| VLAN | Name | Subnet | VIP | CC1 | CC2 | Primary |
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
| 998 / 999 | NATIVE / PARKING | | | | | |

The six department subnets 10.20.10.0 – 10.20.15.0 sit inside **10.20.8.0/21** → one
EIGRP summary route (requirement 22.15).

| Server | IP |
|---|---|
| SRV-JLM-DHCP | 10.20.1.10 |
| SRV-JLM-SYSLOG | 10.20.1.11 |
| SRV-JLM-AAA | 10.20.1.12 |

| Link | Subnet | Sides |
|---|---|---|
| R-JLM-EDGE Gi0/0 ↔ CC1 Gi1/1/1 | 10.20.4.0/30 | .1 / .2 |
| R-JLM-EDGE Gi0/2 ↔ CC2 Gi1/1/1 | 10.20.4.4/30 | .5 / .6 |
| R-JLM-EDGE Gi0/1 ↔ R-ISPB-PE Gi0/1 | 200.2.1.0/30 | .2 / .1 |
| Dynamic NAT pool | 200.2.10.0/29 | 200.2.10.1 – .6 |
| Loopbacks | CC1 10.20.5.1 · CC2 10.20.5.2 · EDGE 10.20.5.10 | |

Management IPs (VLAN 99): CC1 .2, CC2 .3, SW-JLM-F1-ACC1 .11, F2 .12, F3 .13, SW-JLM-SRV .14.

---

## 5. Kiryat Gat — `2001:DB8:30::/48` — IPv6, OSPFv3

Rule: **the VLAN number is the 4th hextet**. VLAN 31 → `2001:DB8:30:31::/64`.

| VLAN | Name | Prefix | DIST1 | DIST2 | Virtual GW |
|---|---|---|---|---|---|
| 31 | FAB-OPS | 2001:DB8:30:31::/64 | ::2 | ::3 | FE80::1 |
| 32 | QUALITY | 2001:DB8:30:32::/64 | ::2 | ::3 | FE80::1 |
| 33 | MAINTENANCE | 2001:DB8:30:33::/64 | ::2 | ::3 | FE80::1 |
| 34 | SAFETY | 2001:DB8:30:34::/64 | ::2 | ::3 | FE80::1 |
| 35 | PLANNING | 2001:DB8:30:35::/64 | ::2 | ::3 | FE80::1 |
| 36 | TRAINING | 2001:DB8:30:36::/64 | ::2 | ::3 | FE80::1 |
| 99 | MGMT | 2001:DB8:30:99::/64 | ::2 | ::3 | FE80::1 |
| 100 | SERVERS | 2001:DB8:30:100::/64 | ::2 | ::3 | FE80::1 |
| 199 | LEGACY-IPV4 | **10.30.199.0/24** (only IPv4 in the branch — see note) | .1 | — | — |

| Link | IPv6 | IPv4 |
|---|---|---|
| R-KG-EDGE Gi0/0 ↔ DIST1 Gi1/1/1 | 2001:DB8:30:FF01::/64 (::1 / ::2) | 10.30.4.0/30 (.1 / .2) |
| R-KG-EDGE Gi0/2 ↔ DIST2 Gi1/1/1 | 2001:DB8:30:FF02::/64 (::1 / ::2) | — |
| R-KG-EDGE Gi0/1 ↔ R-ISPC-PE Gi0/1 | 2001:DB8:C:1::/64 (::2 / ::1) | 200.3.1.0/30 (.2 / .1) |

| Server | IPv6 |
|---|---|
| SRV-KG-DNS | 2001:DB8:30:100::10 |
| SRV-KG-MAIL | 2001:DB8:30:100::11 |
| SRV-KG-SYSLOG | 2001:DB8:30:100::12 |
| SRV-KG-AAA | 2001:DB8:30:100::13 |

OSPFv3 router-IDs: EDGE 3.3.3.1 · DIST1 3.3.3.2 · DIST2 3.3.3.3.

> **Design note — IPv6 branch vs. NAT requirement.** The ministry asks for one branch that is
> entirely IPv6 (req. 7) *and* NAT on every branch, with overload on one of them (req. 10).
> NAT is an IPv4 thing, so Kiryat Gat keeps exactly one small IPv4 VLAN (199 — legacy badge
> readers / cameras that don't speak IPv6) which is PAT-ed to the internet. All users,
> servers and management are IPv6. **Confirm this interpretation with your mentor.**

---

## 6. WAN

| Device | Gi0/0 | Gi0/1 | Gi0/2 |
|---|---|---|---|
| R-ISPA-BR (AS 100) | 200.1.0.1/30 → ISPA-PE | 200.0.12.1/30 → ISPB-BR | 200.0.13.1/30 → ISPC-BR |
| R-ISPB-BR (AS 200) | 200.2.0.1/30 → ISPB-PE | 200.0.12.2/30 → ISPA-BR | 200.0.23.1/30 → ISPC-BR |
| R-ISPC-BR (AS 300) | 200.3.0.1/30 → ISPC-PE | 200.0.13.2/30 → ISPA-BR | 200.0.23.2/30 → ISPB-BR |
| R-ISPA-PE | 200.1.0.2/30 → BR | 200.1.1.1/30 → **Haifa** | 200.1.100.1/24 → internet servers |
| R-ISPB-PE | 200.2.0.2/30 → BR | 200.2.1.1/30 → **Jerusalem** | 200.2.9.1/30 → DSL cloud → Haifa backup |
| R-ISPC-PE | 200.3.0.2/30 → BR | 200.3.1.1/30 + 2001:DB8:C:1::1/64 → **Kiryat Gat** | 200.3.50.1/24 + 2001:DB8:C:50::1/64 → home users |

| Internet server | IP | DNS name |
|---|---|---|
| SRV-ISP-DNS | 200.1.100.10 | — |
| SRV-ISP-WEB1 | 200.1.100.11 | www.isp-web1.com |
| SRV-ISP-WEB2 | 200.1.100.12 | www.isp-web2.com |
| SRV-ISPC-WEB6 | 2001:DB8:C:50::80 | www6.isp-web.com |
| PC-HOME-01 (teleworker) | 200.3.50.10 | — |

---

## 7. Feature allocation — no branch is a copy (req. ג)

| Feature | Haifa HQ | Jerusalem | Kiryat Gat |
|---|---|---|---|
| Design model | 3-tier, 3 buildings | **Collapsed core** (L3 switches), 3 floors | compact 3-tier |
| Addressing | IPv4 | IPv4 | **IPv6** |
| IGP | **OSPF multi-area + virtual link** | **EIGRP 100** + summarization | **OSPFv3** |
| HSRP | v2 on L3 distribution switches | v2 on collapsed-core L3 switches | HSRP for IPv6 |
| DHCP | **on the L3 switches** (split scopes) | **dedicated server** + ip helper | stateless DHCPv6 + SLAAC (student's choice) |
| NAT | **Static** (web, NTP) + PAT for browsing | **Dynamic** pool | **Overload (PAT)** |
| Firewall | **ASA 5506-X** inside/outside/DMZ | **CBAC** (ip inspect) on edge | ACL on edge |
| EtherChannel | **LACP** dist pairs | **PAgP** collapsed-core pair | — |
| VTP | server/client, one domain per building | server/client | **transparent** |
| Wireless | **Employee + Guest** | — | — |
| VPN | GRE/IPsec hub + **remote-access** server | GRE/IPsec spoke | — |
| WAN backup | **DSL** floating static route | — | — |
