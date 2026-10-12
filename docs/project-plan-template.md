# Your project plan — template

This repo teaches you with **one worked example** (Intel Israel). Your final project is
**your own company**, with your own sites, departments and numbers. This page is the plan for
**your** network.

**How to use it**
1. Copy this file to `my-project/plan.md`.
2. Fill it in **during step 01** (parts A–E), on paper first if you like.
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

### HQ areas / buildings

| OSPF area | Block (/20) | What lives there |
|---|---|---|
| 0 | | core, servers, links, loopbacks |
| 1 | | |
| 2 | | |
| 3 | | |

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

---

Teacher approval: ☐ approved ☐ needs changes — date: ______
