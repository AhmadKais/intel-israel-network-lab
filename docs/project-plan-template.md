# Your project plan — template

This repo teaches you with **one worked example** (Intel Israel). Your final project is
**your own company**, with your own sites, departments and numbers. This page is the plan for
**your** network.

**How to use it**
1. Copy this file to `my-project/plan.md`.
2. Fill it in **during step 01** (parts A–E and G), on paper first if you like. Parts F, H and I
   come from missions P2–P3 of the [final project guide](../final-project/README.md).
3. Show it to your teacher **before step 02**. Do not start building until it is approved.
4. From step 02 on, every value you type in **your** project comes from this plan. When a
   step shows an Intel value (a name, a VLAN, an address, a password), look up the matching
   value in your plan and type that instead.
5. If you change something here, change it everywhere in your project.

The example's full plan is [docs/design.md](design.md). Use it as a model for the
**structure**, never for the **values**.

---

## The rules (ministry questionnaire 735918)

- **One topic per student.** Two students may not submit the same company or the same plan.
- **No branch is a copy of another branch.** Each site must use a different design model and
  different features (see part E).
- **Every ministry requirement must appear in your project.** Each step lists the requirements
  it covers in its first line (🎯). How you number and name things is your choice; that the
  feature is there is not.

### What must be different from the example — and what you may keep

| Must be **yours** | You **may keep** from the example |
|---|---|
| Company name, domain name, banner text | The 3-site structure (HQ + 2 branches) |
| Site names and site codes | The naming **pattern** `<ROLE>-<SITE>-<AREA>-<FUNCTION><NUMBER>` |
| Departments, their VLAN numbers and VLAN names | The **method**: a /16 per site, a /20 per OSPF area, a /24 per VLAN, /30 for links, /32 for loopbacks |
| Every IP address and subnet | The gateway convention (`.1` virtual gateway, `.2`/`.3` real IPs) |
| Every password and key | VLAN roles: a management VLAN, an unused "native" VLAN, a "parking" VLAN |
| Hostnames, SSIDs, email users | Device models (2960, 3650, 2911, ASA 5506-X, AccessPoint-PT) |
| Which site gets which feature (part E) | The commands themselves |

---

## A. The company

> Parts A and E come from your **project proposal** — copy them from there. If your plan and
> your proposal disagree, ask your teacher which one to change.

| Field | Your answer |
|---|---|
| Company name | |
| What the company does (1–2 sentences, for the project book) | |
| Domain name (e.g. `mycompany.lab`) | |
| Banner text (English, no `#` character) | |

| Site | City | Code (2–4 letters) | Role | Design model | Buildings / floors |
|---|---|---|---|---|---|
| 1 | | | HQ (main branch) | 3-tier | |
| 2 | | | Branch | | |
| 3 | | | Branch (the IPv6 site) | | |

Departments — the ministry asks for **6 different departments in every site, with at least
8 PCs in each department** (requirement 20.2):

| Site | Departments (name + short code, e.g. Engineering = `ENG`) |
|---|---|
| 1 | |
| 2 | |
| 3 | |

## B. Naming

| Part | Your values |
|---|---|
| ROLE | `R` router · `SW` L2 switch · `SWL3` L3 switch · `ASA` firewall · `AP` access point · `SRV` server · `PC` · `LT` laptop · `PRN` printer (you may keep these) |
| SITE | your 3 site codes: |
| AREA | buildings / floors: |
| FUNCTION | `ACC` · `DIST` · `CORE` · `CC` · `EDGE` (you may keep these) |

Three example hostnames from **your** network:
1.
2.
3.

## C. Address blocks

| Site | IPv4 block | IPv6 block | Routing protocol (IGP) | NAT type |
|---|---|---|---|---|
| 1 (HQ) | | — | | |
| 2 | | — | | |
| 3 | (small legacy block only) | | | |
| ISP 1 / 2 / 3 | | | | |

Blocks must be **consecutive** and must come from **one master block** (requirement 22.4).

> **How to divide:** one block per city, one block per building (or floor) inside it, one /24
> per department inside that. The recipe, step by step: [sites-and-views.md, Part 1](sites-and-views.md#part-1--dividing-the-network-between-cities-and-buildings).

### HQ areas / buildings

| OSPF area | Block (/20) | What lives there |
|---|---|---|
| 0 | | core, servers, links, loopbacks |
| 1 | | |
| 2 | | |
| 3 | | |

### Branches — floors / buildings

| Site | Building or floor | Departments there | Block (or the summary that covers them) |
|---|---|---|---|
| 2 | | | |
| 2 | | | |
| 3 | | | |
| 3 | | | |

## D. VLANs (one table per site; copy it 3 times)

Site: ______

| VLAN | Name | Subnet | Gateway (.1) | DIST1 / CC1 (.2) | DIST2 / CC2 (.3) | HSRP/STP primary |
|---|---|---|---|---|---|---|
| | | | | | | |
| | | | | | | |
| | | | | | | |
| | | | | | | |
| | | | | | | |
| | | | | | | |
| | MGMT | | | | | |
| | NATIVE | — no devices, no IP | | | | |
| | PARKING | — unused ports | | | | |

Access-switch port plan (same on every access switch of the site):

| Ports | VLAN | Use |
|---|---|---|
| | | |

Fixed addresses (printers, admin PC, switch management IPs, servers):

| Device | Address / gateway |
|---|---|
| | |

## E. Feature allocation — no branch is a copy

Fill one feature per cell. Every row must be implemented somewhere; across a row, the sites
should differ wherever the ministry asks for it. Compare with section 7 of
[design.md](design.md).

| Feature | Site 1 (HQ) | Site 2 | Site 3 |
|---|---|---|---|
| Design model | | | |
| Addressing (IPv4 / IPv6) | | | |
| IGP | | | |
| HSRP | | | |
| DHCP (where it runs) | | | |
| NAT (static / dynamic / overload) | | | |
| Firewall (ASA / CBAC / ACL) | | | |
| EtherChannel (LACP / PAgP) | | | |
| VTP mode | | | |
| Wireless | | | |
| VPN role | | | |
| WAN backup | | | |

## F. Passwords

Make your own — **never reuse the example's `Intel#...` passwords**. Keep this table in the
project book.

| Purpose | Value |
|---|---|
| `enable secret` | |
| Console line | |
| Local admin user | |
| TACACS+ key | |
| VTP password | |
| OSPF MD5 key | |
| EIGRP key | |
| WiFi — employees / guests | |
| VPN keys | |
| Email users | |

## G. Physical layout — which device in which city, building and closet

You will build this in Packet Tracer's **Physical view** (steps 09, 12, 13, 14 — moves M24–M26).
How to fill it: [sites-and-views.md, Part 3](sites-and-views.md#part-3--from-logical-to-physical-and-back).

| City | Building | Wiring closet | Network devices (switches, routers, firewall) | End devices (PCs, printers, servers, APs) |
|---|---|---|---|---|
| (site 1) | (main building / data center) | | | |
| (site 1) | | | | |
| (site 1) | | | | |
| (site 2) | | | | |
| (site 3) | | | | |
| Internet | (one per provider) | | | |

---

## H. My proposal against what I build

Mission [P2](../final-project/README.md). One row for each promise. Add a row for anything else your
proposal promises that the steps do not build. The two "change" rows need your teacher's approval.

| Promise in the proposal | In my proposal? | Standard decision | My decision | Teacher ✔ |
|---|---|---|---|---|
| Access points in every branch | ☐ | Build: P11 | | |
| Dynamic ARP Inspection | ☐ | Build: P12 | | |
| EtherChannel between the core switches | ☐ | Build: P13 | | |
| Site 3 site-to-site VPN to the main branch | ☐ | Build: P14 | | |
| Site 3 dedicated DHCP server | ☐ | Change: SLAAC + stateless DHCPv6 | | |
| eBGP from every branch, dual BGP at the main branch | ☐ | Change: static routes to the ISP; BGP only between ISPs | | |
| | ☐ | | | |

---

## I. The extra values

The steps and missions ask for these. Decide them now, so that nothing is invented halfway through.

| Value | Site 1 | Site 2 | Site 3 |
|---|---|---|---|
| Public block | | | |
| Edge router's public address | | | |
| Public addresses for static NAT (Web, others) | | — | — |
| NAT pool (first–last) | — | | — |
| Routing protocol ID | OSPF process: | EIGRP AS: | OSPFv3 process: |
| WiFi VLAN number | | | |
| WiFi network or prefix | | | |
| SSID | | | |
| AP names and switch ports | | | |

| Value | Mine |
|---|---|
| BGP AS number of each ISP | |
| DSL backup link (network and both addresses) | |
| Site 1 ↔ site 2 tunnel `/30` | |
| Site 1 ↔ site 3 tunnel `/30` (`<T3-1>` / `<T3-3>`) | |
| Site 3 IPv4 VLAN and network (`<LEGACY>`) | |
| Remote-access address pool | |
| Remote-access group name and user | |
| Site 3 VPN key (`<KEY3>`) | |
| DNS names of your servers and the ISP's servers | |
| Email domain | |

---

Teacher approval: ☐ approved ☐ needs changes — date: ______
