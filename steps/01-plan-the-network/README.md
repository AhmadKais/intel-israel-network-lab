# Step 01 — Plan the network (on paper first!)

## Goal
Before you drag a single device, you understand **why** every address in
[docs/design.md](../../docs/design.md) is what it is. A network built without a plan
becomes impossible to troubleshoot — and the project book (30% of the grade) is mostly
this plan.

Ministry requirements: **22.1, 22.4, 19, 20.1, 20.2**

## Concepts

### 1. Hierarchical addressing
One master block, split into consecutive pieces, split again:

```
10.0.0.0/8                          master block (whole company)
 ├── 10.10.0.0/16   Haifa HQ
 │    ├── 10.10.0.0/20   OSPF area 0  (core)      10.10.0.0  – 10.10.15.255
 │    ├── 10.10.16.0/20  OSPF area 1  (bldg A)    10.10.16.0 – 10.10.31.255
 │    ├── 10.10.32.0/20  OSPF area 2  (bldg B)    10.10.32.0 – 10.10.47.255
 │    └── 10.10.48.0/20  OSPF area 3  (bldg C)    10.10.48.0 – 10.10.63.255
 ├── 10.20.0.0/16   Jerusalem
 └── 10.30.0.0/16   Kiryat Gat (legacy IPv4 only)
```

**Why /20 per area?** Because then the ABR (the switch between area 1 and area 0) can
advertise *one* route — `10.10.16.0/20` — instead of every /24 inside it. That is
**route summarization** (req. 22.15). Smaller routing tables, and a flapping VLAN in
building A doesn't make every router in the company recalculate.

### 2. Subnet sizes
| Prefix | Hosts | Used for |
|---|---|---|
| /24 | 254 | every VLAN (simple to read: 3rd octet = which VLAN) |
| /30 | 2 | point-to-point links between two routers/L3 switches |
| /32 | 1 | loopbacks (router-ID, management) |
| /29 | 6 | small public blocks from the ISP |

### 3. Gateway convention (memorise it — you'll use it everywhere)
```
.1   = HSRP virtual gateway (what PCs use)
.2   = DIST1 / CC1 real IP
.3   = DIST2 / CC2 real IP
.4–.10    reserved (servers, printers, network devices)
.11–.254  DHCP for users
```

### 4. Naming
`SWL3-HFA-A-DIST1` = **L3 switch**, **Haifa**, building **A**, **distribution #1**.
Anyone reading `show cdp neighbors` immediately knows what is cabled where.

### 5. VLAN planning rules we use
- One VLAN per department (security + smaller broadcast domains).
- Printers in their own VLAN (you can ACL them later).
- **VLAN 99** = management only. Never put users in it.
- **VLAN 998** = native VLAN on trunks, has no ports and no IP (defeats VLAN-hopping).
- **VLAN 999** = parking: every unused port goes here and is shut down.

## Do it — exercises (paper or a spreadsheet)

Do these **without looking** at design.md, then compare.

1. Haifa building B is area 2. Which /20 is it? What are its first and last addresses?
2. HR is the first VLAN in building B. Which /24 does it get? What is the HSRP gateway?
3. You need 13 point-to-point /30 links. How many addresses does that use? Do they fit in
   `10.10.4.0/24`?
4. Jerusalem has 6 department subnets `10.20.10.0/24`…`10.20.15.0/24`. What is the
   **smallest** single prefix that covers all six?
5. Write the hostname for: the 2nd access switch on floor 3 in Jerusalem; the web server
   in Haifa's DMZ; the 5th PC in Kiryat Gat's QUALITY department.

<details><summary>Answers</summary>

1. `10.10.32.0/20` → 10.10.32.0 – 10.10.47.255
2. `10.10.32.0/24`, gateway `10.10.32.1`
3. 13 × 4 = 52 addresses → yes, 10.10.4.0 – 10.10.4.51
4. 10 = 0000 1010, 15 = 0000 1111. They share the first 5 bits only if we start at 8:
   `10.20.8.0/21` covers 8–15. (`/22` at .8 covers only 8–11, `/22` at .12 covers 12–15.)
5. `SW-JLM-F3-ACC2`, `SRV-HFA-DMZ-WEB`, `PC-KG-QUA-05`
</details>

## Draw it
Draw (draw.io / paper) the **logical** topology of Haifa building A as you imagine it:
2 access switches, 2 distribution L3 switches, 2 core L3 switches. Every access switch
has **two** uplinks (one to each distribution switch). Why two? → if one dist switch dies,
the building keeps working. That is the core of requirement 6 (redundancy).

## Checkpoint
- [ ] You did the 5 exercises
- [ ] You read every table in design.md once
- [ ] You can explain to someone why VLAN 998 exists

## Challenge
Kiryat Gat uses IPv6 with "VLAN number = 4th hextet". Write the /64 of VLAN 35 and the
address of DIST2 in it. How many /64s fit in the /48?

<details><summary>Answer</summary>

`2001:DB8:30:35::/64`, DIST2 = `2001:DB8:30:35::3`. A /48 holds 2^(64-48) = **65,536** /64s.
</details>
