# Step 01 — Plan the network (on paper first!)

> ⏱ 2 lessons · 📁 start from: a new file · 💾 save as: `my-work/step01.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 00 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## Goal
Before you drag a single device, you understand **why** every address in
[docs/design.md](../../docs/design.md) is what it is. A network built without a plan
becomes impossible to troubleshoot — and the project book (30% of the grade) is mostly
this plan.

Ministry requirements: **22.1, 22.4, 19, 20.1, 20.2**

---

## Understand it first (simple English)

### 1. Why plan at all?
Building a network without a plan is like building a city without street names: it works
at first, then nobody can find anything. With a plan:
- you can **read** an address and know where it is (`10.10.17.x` = Haifa, building A, Finance);
- routers can **summarize** (one route for a whole building instead of ten);
- the project book almost writes itself.

### 2. Private addresses — the ones companies use inside
Three ranges are reserved for use **inside** organisations. Anyone may use them, and they
are never routed on the internet:

| Range | Size |
|---|---|
| `10.0.0.0/8` | ~16 million addresses ← **we use this one** (room for everything) |
| `172.16.0.0/12` | ~1 million |
| `192.168.0.0/16` | ~65 thousand (home routers) |

To reach the internet, these private addresses are translated to public ones by **NAT**
(step 12). Think of a company switchboard: inside you have extension 254, outside everyone
sees the company's one public phone number.

### 3. What does "/24" mean?
An IPv4 address is **32 bits** (32 ones and zeros), written as 4 numbers of 8 bits each:
```
10      .10      .16      .25
00001010.00001010.00010000.00011001
```
The **prefix** (`/24`, `/20`, `/30`…) says **how many bits from the left are the network
part**. The rest are for hosts.

| Prefix | Mask | Network part | Addresses | Usable hosts |
|---|---|---|---|---|
| /16 | 255.255.0.0 | first 2 numbers | 65 536 | 65 534 |
| /20 | 255.255.240.0 | 2½ numbers | 4 096 | 4 094 |
| /24 | 255.255.255.0 | first 3 numbers | 256 | **254** |
| /29 | 255.255.255.248 | | 8 | 6 |
| /30 | 255.255.255.252 | | 4 | **2** |
| /32 | 255.255.255.255 | everything | 1 | 1 (one device) |

**Formula:** addresses = 2^(32 − prefix). Usable = that − 2.
Why −2? The **first** address is the *network address* (the name of the street, e.g.
`10.10.16.0`) and the **last** is the *broadcast address* (`10.10.16.255` — "send to
everyone on this street"). Neither can be given to a device.

### 4. The trick that makes subnetting easy — think in "/24 blocks"
Every **step of 1** in the prefix **halves** the size. So, counted in /24s:

| Prefix | How many /24s it contains | Starts at a 3rd number that is a multiple of… |
|---|---|---|
| /24 | 1 | 1 |
| /23 | 2 | 2 |
| /22 | 4 | 4 |
| /21 | 8 | 8 |
| /20 | 16 | 16 |

So the /20s inside `10.10.0.0/16` start at **0, 16, 32, 48, 64 …** in the third number.
That is exactly why Haifa's buildings are at `10.10.16.x`, `10.10.32.x`, `10.10.48.x`.

### 5. Hierarchical addressing — like a postal address
```
Country  -> City      -> Neighbourhood  -> Street        -> House
10.0.0.0/8  10.10.0.0/16  10.10.16.0/20    10.10.17.0/24   10.10.17.25
(company)   (Haifa)       (building A)     (Finance VLAN)  (one PC)
```
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

**Why does this matter? → Route summarization.** A post office in Jerusalem doesn't need a
list of every street in Haifa — it only needs "everything for Haifa → send to Haifa". In
the same way, the router at building A's exit can tell the rest of the company **one**
route, `10.10.16.0/20`, instead of every /24 inside. Smaller routing tables, and if one
VLAN in building A goes up and down, nobody outside the building notices
(requirement 22.15).

### 6. A few words you will meet later (preview — no need to master them yet)
| Word | Simple meaning | Step |
|---|---|---|
| **VLAN** | Cutting one physical switch into several separate "virtual switches" — one per department | 02 |
| **OSPF area** | A group of routers that share a detailed map; between areas they share only summaries | 08 |
| **HSRP** | Two routers share one "virtual" gateway address, so if one dies the other answers | 05 |
| **DMZ** | A separate zone for servers the internet may reach | 11 |

### 7. Subnet sizes we use, and why
| Prefix | Hosts | Used for | Why this size |
|---|---|---|---|
| /24 | 254 | every VLAN | plenty for a department, and easy to read: 3rd number = which VLAN |
| /30 | 2 | link between two routers | a cable has exactly 2 ends — don't waste addresses |
| /32 | 1 | loopbacks | a "virtual interface" that never goes down — the device's ID |
| /29 | 6 | small public blocks from the ISP | public addresses are expensive |

### 8. Gateway convention (memorise it — you'll use it everywhere)
```
.1   = HSRP virtual gateway (what PCs use)
.2   = DIST1 / CC1 real IP
.3   = DIST2 / CC2 real IP
.4–.10    reserved (servers, printers, network devices)
.11–.254  DHCP for users
```
A rule like this means you never have to look up a gateway — it's always `.1`.

### 9. Naming
`SWL3-HFA-A-DIST1` = **L3 switch**, **Haifa**, building **A**, **distribution #1**.
Anyone reading `show cdp neighbors` immediately knows what is cabled where. The ministry
requires logical names (22.1). Full rules: [design.md §1](../../docs/design.md).

### 10. VLAN planning rules we use
- One VLAN per department (security + less broadcast noise).
- Printers in their own VLAN (you can protect them separately later).
- **VLAN 99** = management only. Never put users in it.
- **VLAN 998** = "native" VLAN on trunks, with no ports and no IP (blocks an attack called
  VLAN hopping — explained in step 03).
- **VLAN 999** = parking: every unused port goes here and is shut down.

---

## The Haifa plan you will build in steps 02–09
Keep this page open while you work — every number you type in the next steps comes from here.

### Naming rule
```
<ROLE>-<SITE>-<AREA>-<FUNCTION><NUMBER>
```
| Part | Values |
|---|---|
| ROLE | `R` router · `SW` Layer-2 switch · `SWL3` Layer-3 switch · `ASA` firewall · `AP` access point · `SRV` server · `PC` · `LT` laptop · `PRN` printer |
| SITE | `HFA` Haifa · `JLM` Jerusalem · `KG` Kiryat Gat |
| AREA | building `A` / `B` / `C` (or floor `F1`…) |
| FUNCTION | `ACC` access · `DIST` distribution · `CORE` core · `EDGE` internet edge |

PCs: `PC-HFA-ENG-01` = PC, Haifa, Engineering, number 01.

### Haifa building A — the devices of steps 02–07
```
                 SWL3-HFA-A-DIST1 ══(Po1)══ SWL3-HFA-A-DIST2
                      │      ╲              ╱      │
                      │        ╲          ╱        │
                      │          ╲      ╱          │
                 SW-HFA-A-ACC1      ╳       SW-HFA-A-ACC2
                 (Gi0/1 → DIST1, Gi0/2 → DIST2 on both)
                   │ │ │                    │ │ │
                 PCs, printer, admin PC     PCs, printer, 2 WiFi APs
```

### Building A VLANs
| VLAN | Name | Subnet | Gateway | DIST1 IP | DIST2 IP | Who is "boss" |
|---|---|---|---|---|---|---|
| 10 | ENGINEERING | 10.10.16.0/24 | 10.10.16.1 | .2 | .3 | DIST1 |
| 20 | FINANCE | 10.10.17.0/24 | 10.10.17.1 | .2 | .3 | DIST2 |
| 30 | PRINTERS-A | 10.10.18.0/24 | 10.10.18.1 | .2 | .3 | DIST1 |
| 99 | MGMT-A | 10.10.19.0/24 | 10.10.19.1 | .2 | .3 | DIST1 |
| 110 | WIFI-EMP | 10.10.20.0/24 | 10.10.20.1 | .2 | .3 | DIST1 |
| 120 | WIFI-GUEST | 10.10.21.0/24 | 10.10.21.1 | .2 | .3 | DIST2 |
| 998 | NATIVE | — no devices, no IP | | | | |
| 999 | PARKING | — unused ports | | | | |

### Building B and C VLANs (step 09)
| VLAN | Name | Subnet | | VLAN | Name | Subnet |
|---|---|---|---|---|---|---|
| 40 | HR | 10.10.32.0/24 | | 70 | RND | 10.10.48.0/24 |
| 50 | MARKETING | 10.10.33.0/24 | | 80 | IT | 10.10.49.0/24 |
| 60 | PRINTERS-B | 10.10.34.0/24 | | 90 | PRINTERS-C | 10.10.50.0/24 |
| 99 | MGMT-B | 10.10.35.0/24 | | 99 | MGMT-C | 10.10.51.0/24 |

The 6 Haifa departments: **Engineering, Finance, HR, Marketing, R&D, IT**.

### Fixed addresses in building A
| Device | Address |
|---|---|
| PRN-HFA-A-01 (printer) | 10.10.18.10 / gw 10.10.18.1 |
| PC-HFA-ADMIN-01 (network admin) | 10.10.19.100 / gw 10.10.19.1 |
| SW-HFA-A-ACC1 management | 10.10.19.11 |
| SW-HFA-A-ACC2 management | 10.10.19.12 |

The full company plan (Jerusalem, Kiryat Gat, the WAN) is in
[docs/design.md](../../docs/design.md); each later step repeats the part it needs.

---

## Do it — exercises (paper or a spreadsheet)

Do these **without looking** at design.md, then compare.

1. Haifa building B is area 2. Which /20 is it? What are its first and last addresses?
2. HR is the first VLAN in building B. Which /24 does it get? What is the HSRP gateway?
3. You need 13 point-to-point /30 links. How many addresses does that use? Do they fit in
   `10.10.4.0/24`?
4. Jerusalem has 6 department subnets `10.20.10.0/24`…`10.20.15.0/24`. What is the
   **smallest** single prefix that covers all six?
5. Write the hostname for: the 2nd access switch on floor 3 in Jerusalem; the web server
   in Haifa's DMZ; the 5th PC in Kiryat Gat's QUALITY department (department code: `QUA`).

<details><summary>Answers (with the reasoning)</summary>

1. The /20s start at 0, 16, **32**, 48 → area 2 = `10.10.32.0/20`. A /20 holds 16 /24s, so it
   runs from `10.10.32.0` to `10.10.47.255` (32 + 16 − 1 = 47).
2. The first /24 inside it: `10.10.32.0/24`. Gateway by our rule = `.1` → `10.10.32.1`.
3. 13 links × 4 addresses = 52 → `10.10.4.0` – `10.10.4.51`. A /24 has 256 → it fits easily.
4. **Easy way:** 6 /24s → round up to the next power of two = **8** /24s → that's a **/21**
   (table in §4). A /21 must start at a multiple of 8 → the 8-block that contains 10–15 is
   **8–15** → `10.20.8.0/21`.
   **Binary way:** 8 = `00001000`, 15 = `00001111`. Numbers 8 to 15 all start with the same
   5 bits `00001`. 16 bits (first two numbers) + 5 = **21** → `/21`.
5. `SW-JLM-F3-ACC2`, `SRV-HFA-DMZ-WEB`, `PC-KG-QUA-05`
</details>

## Draw it
Draw (draw.io / paper) the **logical** topology of Haifa building A as you imagine it:
2 access switches, 2 distribution L3 switches, 2 core L3 switches. Every access switch
has **two** uplinks (one to each distribution switch). Why two? → if one distribution switch
dies, the building keeps working. That is the heart of requirement 6 (redundancy).

## Prepare your project file in Packet Tracer (click by click)
This is the file you will keep building in for the whole project.

1. Open Packet Tracer → **File → New**.
2. Mark the three Haifa buildings on the workspace:
   1. In the common tools, click **Drawing Palette** (the pencil icon). In the window that
      opens, choose **Rectangle** and a light colour.
      (Older PT versions have a **Draw Rectangle** button instead — use that.)
   2. Drag a big rectangle on the left part of the workspace.
   3. Draw two more rectangles next to it. Leave space at the top for the core (step 08) and at
      the right for the firewall and the internet (steps 11–12).
   4. Close the Drawing Palette window.
3. Label them:
   1. Click the **Place Note** tool (the notepad icon).
   2. Click inside the first rectangle and type `HAIFA - BUILDING A (OSPF area 1)`.
   3. Do the same for `BUILDING B (area 2)` and `BUILDING C (area 3)`, and a note at the top:
      `HAIFA HQ - CORE (area 0)`.
4. Press **Esc** to go back to the **Select** arrow.
5. **File → Save As** → `my-work/step01.pkt`.

> Tip: the rectangles are only drawings — they don't do anything technical, but they make the
> big network readable and look good in the project book's screenshots.

## Checkpoint
- [ ] You did the 5 exercises
- [ ] You read every table in design.md once
- [ ] You can explain to someone why VLAN 998 exists
- [ ] You can explain "/24" and "/30" without looking
- [ ] **Your own plan** (`my-project/plan.md`) is filled in and approved by your teacher

## Challenge
Kiryat Gat uses IPv6 with "VLAN number = 4th group". Write the /64 of VLAN 35 and the
address of DIST2 in it. How many /64s fit in the /48?

<details><summary>Answer</summary>

`2001:DB8:30:35::/64`, DIST2 = `2001:DB8:30:35::3`. A /48 holds 2^(64−48) = **65,536** /64s.
(IPv6 is explained properly in step 14 — for now just notice the pattern.)
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

1. Copy [docs/project-plan-template.md](../../docs/project-plan-template.md) to
   `my-project/plan.md`.
2. Fill in parts **A–F**. Take the company, sites, departments and which site gets which
   feature (part E) from your **project proposal**. Use the method of this step: a /16 per site from one master block,
   a /20 per OSPF area at HQ, a /24 per VLAN, the `.1/.2/.3` gateway rule, the naming pattern.
3. Use **your own** numbers and names. Do not copy `10.10.x.x`, VLAN 10/20/30 or the Intel
   department names.
4. Check yourself: redo exercises 1–4 of this step with **your** blocks.
5. **Show the plan to your teacher. Do not start step 02 until it is approved.**
6. Prepare your Packet Tracer file exactly as in "Prepare your project file", but with your
   building and site names in the notes.

💾 Save as `my-project/step01.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★1 What is a subnet mask?**

<details><summary>Answer</summary>

It marks which part of an IP address is the **network** and which part is the **host**. Bits set to 1 in the mask are the network part. Example: `255.255.255.0` → the first 3 numbers are the network.

**Show it:** Any PC → IP Configuration; your plan's VLAN table.
</details>

**2. ★1 What is a prefix?**

<details><summary>Answer</summary>

The short way to write the mask: the number of network bits. `/24` = `255.255.255.0`, `/30` = `255.255.255.252`. Addresses in the subnet = 2^(32 − prefix); usable = that − 2.

**Show it:** Your plan, part C/D.
</details>

**3. ◆ Why −2 usable hosts? How many hosts in a /24, a /29, a /30?**

<details><summary>Answer</summary>

The first address is the network address and the last is the broadcast address; neither can be given to a device. /24 → 254, /29 → 6, /30 → 2.

**Show it:** Your /30 link table: exactly two devices per link.
</details>

**4. ★1 (NAT section) What are the private address ranges of each class?**

<details><summary>Answer</summary>

Class A `10.0.0.0/8`, class B `172.16.0.0/12` (172.16–172.31), class C `192.168.0.0/16`. They are not routed on the internet; NAT translates them to public addresses.

**Show it:** Your master block in plan part C.
</details>

**5. ★3 Show me your IP address tables and explain why you planned this structure.**

<details><summary>Answer</summary>

Hierarchical: one master block → one /16 per site (consecutive, req. 22.4) → one /20 per OSPF area at HQ → one /24 per VLAN, /30 per link, /32 per loopback. Why: you can read where an address is, routers can **summarize** one route per area/site, the network can grow, and the project book follows the same tables.

**Show it:** `my-project/plan.md` parts C and D; on a distribution switch `show ip route` shows one summary per building (from step 08).
</details>

**6. ◆ What is route summarization and why does your plan make it possible?**

<details><summary>Answer</summary>

Advertising one route that covers many subnets (e.g. one /20 for a whole building). It works only because each building/site got one **contiguous** block. Smaller routing tables, and a VLAN going up and down inside a building is not seen outside it.

**Show it:** From step 08: `show ip route` on the core.
</details>

**7. ◆ What is your gateway convention, and why have one?**

<details><summary>Answer</summary>

`.1` = the HSRP virtual gateway that PCs use, `.2`/`.3` = the real addresses of the two switches, `.4–.10` reserved, `.11+` DHCP. With one rule you never need to look up a gateway.

**Show it:** Plan part D.
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

Close the plan. On a blank page, the examiner may give you numbers and ask you to calculate.
Practise until each takes under 2 minutes, without a calculator:
1. The /20 of OSPF area N inside your HQ /16, and its first and last address.
2. The smallest summary that covers a list of your /24s (e.g. your 6 department subnets).
3. How many /30 links fit in one /24, and the addresses of link number 5.
Then explain your plan's tables out loud, top to bottom, in 3 minutes.

---

## 👩‍🏫 Teacher notes
- **Lesson plan (~2 × 90 min):** lesson 1 = private ranges, prefixes, the "/24 blocks" table,
  network & broadcast addresses (lots of board practice). Lesson 2 = the hierarchy diagram,
  summarization with the post-office comparison, gateway rule, naming, exercises.
- **Teach the "/24 blocks" table before any binary.** Most students get subnetting from
  "each step halves the size" much faster than from binary. Bring in binary only for the
  "why" (exercise 4, binary way).
- **Common student questions**
  - *"Why not use 192.168.x like at home?"* — too small for 3 branches with room to grow;
    10/8 lets each site have its own /16.
  - *"Why waste a /24 on a department of 8 PCs?"* — readability and growth; private
    addresses are free. Public ones (/29) we use carefully.
  - *"Why /30 for links?"* — a link has 2 ends → 2 usable addresses is exactly enough.
- **Project-design tip for students:** each student must have their **own** company,
  cities and address plan (two students may not submit the same project). They can keep
  the same *structure* (/16 per site, /20 per area) but must change the numbers and names.
