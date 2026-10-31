# Cities, buildings and the two views — the guide

This page answers four questions that the steps only touch on:

1. [How do I divide the network between cities and buildings?](#part-1--dividing-the-network-between-cities-and-buildings)
2. [What is the difference between the Logical and the Physical view?](#part-2--the-logical-view-and-the-physical-view)
3. [How do I go from the logical map to the physical places, and back?](#part-3--from-logical-to-physical-and-back)
4. [How do I move devices from one place or network to a new city?](#part-4--moving-devices-to-a-new-city-or-a-new-network)

The click-by-click moves are in [basic-moves.md](basic-moves.md): **M24** (switch view), **M25**
(make a city / building / closet), **M26** (move a device), **M27** (cluster), **M28** (cable
into a cluster).

---

## Part 1 — Dividing the network between cities and buildings

### 1.1 The idea in one picture
A big company is divided like a postal address — from big to small:

```
Company  ->  City (site)  ->  Building / floor  ->  Department (VLAN)  ->  One device
10.0.0.0/8   10.10.0.0/16     10.10.16.0/20         10.10.16.0/24          10.10.16.25
```

Every level gets **one block of addresses**, and that block is **cut out of the level above it**.
Then a router at the exit of a building (or a city) can tell the rest of the company **one
short line** — "everything in 10.10.16.0/20 is behind me" — instead of a long list. That's
**summarization** (requirement 22.15).

### 1.2 The recipe — do it in this order
**Step 1 — Count.** Write down, for each city: how many buildings (or floors), and which
departments sit in each building.

**Step 2 — One block per city.** Take your private master block (e.g. `10.0.0.0/8`) and give
each city the next free /16:

| City | Block |
|---|---|
| City 1 (HQ) | 10.**10**.0.0/16 |
| City 2 | 10.**20**.0.0/16 |
| City 3 | 10.**30**.0.0/16 |

> 💡 Jump by 10 (10, 20, 30) — it's easy to read, and it leaves room for new cities later.
> The ministry wants the blocks to come **in order** from one master block (requirement 22.4).

**Step 3 — One block per building, inside the city's block.** A /16 has 256 "/24 blocks".
Cut it into **equal pieces**, one per building, plus one piece for the core / server room:

| Pieces you need | Size of each piece | Each piece holds |
|---|---|---|
| up to 4 | /18 | 64 × /24 |
| up to 8 | /19 | 32 × /24 |
| up to **16** | **/20** | **16 × /24** ← what Haifa uses |

A /20 always starts at a **multiple of 16** in the third number: `.0`, `.16`, `.32`, `.48` …

**Step 4 — One /24 per department (VLAN), inside the building's block.** Count up from the
start of the building's block. Keep the order the same in every building (users first, then
printers, then management) — then you can guess any address.

**Step 5 — Small things.** Router-to-router links (/30), loopbacks (/32) and servers go in the
**core** piece (Haifa: `10.10.0.0/20`), never inside a building's piece.

**Step 6 — Check.** For every building: does every /24 really sit **inside** the building's
block? (Exercise 3.1 of step 01 shows how.) If yes, the building can be summarized in one line.

### 1.3 The worked example — Intel Israel
```
10.0.0.0/8                                  the whole company
├── Haifa (HQ)          10.10.0.0/16
│    ├── Core / servers   10.10.0.0/20      links, loopbacks, servers      (OSPF area 0)
│    ├── Building A       10.10.16.0/20     VLAN 10 = .16, 20 = .17 …      (OSPF area 1)
│    ├── Building B       10.10.32.0/20     VLAN 40 = .32, 50 = .33 …      (OSPF area 2)
│    ├── Building C       10.10.48.0/20     VLAN 70 = .48, 80 = .49 …      (OSPF area 3)
│    └── DMZ              10.10.200.0/24    behind the firewall
├── Jerusalem           10.20.0.0/16        one building, 3 floors
│    ├── MGMT / servers / printers   10.20.0.0 – 10.20.2.0 (/24 each)
│    ├── Links + loopbacks           10.20.4.0, 10.20.5.0
│    └── 6 departments               10.20.10.0 – 10.20.15.0  → summary 10.20.8.0/21
└── Kiryat Gat          2001:DB8:30::/48    IPv6: VLAN 31 → 2001:DB8:30:31::/64
                        (+ 10.30.0.0/16 for the one legacy IPv4 VLAN)
```

Two different ways to divide, both correct:
- **Haifa** has 3 separate **buildings** → one **/20 per building** (one OSPF area each).
- **Jerusalem** is **one building with 3 floors** and a collapsed core → the floors don't get
  their own blocks; instead the 6 departments sit next to each other so they fit in **one /21**.

The full tables are in [design.md](design.md).

### 1.4 Which devices go where?
| Place | Devices | Intel example |
|---|---|---|
| **Main building** of a city (the "data center" / server room) | edge router, firewall, core switches, server switch + servers | Haifa: R-HFA-EDGE, ASA-HFA-FW1, CORE1/2, SW-HFA-SRV, DMZ |
| **Every building** | its distribution switches (2) + its access switches | Haifa building B: B-DIST1/2, B-ACC1/2 |
| **Every floor** (a branch with floors) | one access switch per floor | Jerusalem: SW-JLM-F1/F2/F3-ACC1 |
| **Desks** in the building | PCs, printers, access points | PC-HFA-HR-01 in building B |
| **The internet** (not ours) | provider routers, internet servers, the home PC | R-ISPA-BR, SRV-ISP-WEB1, PC-HOME-01 |

**Rule of thumb:** a device belongs to the building whose users it serves. A switch that
connects buildings (core) belongs to the main building.

### 1.5 Fill-in table for your own project
Copy this into `my-project/plan.md` (it is part G of the plan template):

| City (site) | Its block | Building / floor | Its block | Devices there |
|---|---|---|---|---|
| | | | | |
| | | | | |

---

## Part 2 — The Logical view and the Physical view
Packet Tracer shows the **same devices** in two views (switch with the tabs at the top-left of
the workspace — M24):

| | **Logical view** | **Physical view** |
|---|---|---|
| Answers | **Who is connected to whom?** | **Where is each device?** |
| Looks like | A network map: icons and cables | A real map: cities → buildings → wiring closets → racks |
| You do here | Place, cable, configure, test — all the steps | Put each device in its real place |
| Project book | Chapter 9 — logical topology | Chapter 8 — physical topology |
| Grouping | **Clusters** (M27) | **Cities, buildings, closets** (M25) |

**Important:** a device exists **once**. Moving it in one view never deletes it, and never
changes its cables, addresses or commands.

The levels of the Physical view:
```
Intercity (the country map)
 └── City            e.g. Haifa
      └── Building     e.g. HFA Building B          (PCs and printers stand here)
           └── Wiring closet   e.g. Main Wiring Closet   (switches and routers in racks)
```

At the start, everything is in **Home City → Corporate Office**: switches and routers in its
**Main Wiring Closet**, PCs and printers in the building itself.

---

## Part 3 — From logical to physical, and back

### 3.1 Logical → Physical (build the real places)
Do it once the site is built and working (the steps tell you when: steps 09, 12, 13, 14).

1. **Make a table** first — every device and its place. For Intel:

   | City | Building | Closet | Devices |
   |---|---|---|---|
   | Haifa | HFA Data Center (renamed *Corporate Office*) | Main Wiring Closet | CORE1/2, SW-HFA-SRV, servers, ASA, R-HFA-EDGE, SW-HFA-DMZ, DMZ servers |
   | Haifa | HFA Building A | Closet A | A-DIST1/2, A-ACC1/2 (+ PCs, printer, APs, laptops in the building) |
   | Haifa | HFA Building B | Closet B | B-DIST1/2, B-ACC1/2 (+ PCs, printer) |
   | Haifa | HFA Building C | Closet C | C-DIST1/2, C-ACC1/2 (+ PCs, printer) |
   | Internet | ISP-A / ISP-B / ISP-C / Home | one closet each | the provider routers, internet servers, DSL modem, cloud, PC-HOME-01 |
   | Jerusalem | JLM Office | Main closet | CC1, CC2, R-JLM-EDGE, SW-JLM-SRV, servers, the 3 floor switches (+ PCs, printers) |
   | Kiryat Gat | KG Fab | Main closet | DIST1/2, ACC1/2/3, R-KG-EDGE, SW-KG-SRV, servers (+ PCs, printers) |

2. **Make the places** (M25): rename *Home City* → `Haifa`, make the new cities, buildings and
   closets.
3. **Move the devices** (M26), one building at a time — use **Navigation** and drag, it's
   faster.
4. **Check in the Logical view:** ping across the site still works (it must — nothing changed).
5. Take the Physical screenshots for chapter 8 (M22): Intercity, each city, one closet.

### 3.2 Physical → Logical (read the real places as a map)
You get a file, or the examiner points at a building in the Physical view and asks "what is
in there and how is it connected?":

1. In the Physical view, open the building / closet → write down the device names.
2. Click **Navigation** → the tree shows **all** places and devices at once. Copy it as a list.
3. Click the **Logical** tab → find those names on the map (in a big network: open each
   cluster).
4. Follow the cables of each device in the Logical view → now you can draw the building's part
   of the logical map.
5. A new device placed **in the Physical view** also appears in the Logical view — on the
   logical map, near the top-left. Drag it to the right place and cable it there.

### 3.3 Keep the two views in step
- Same names for the same groups: cluster **JERUSALEM** (Logical) ↔ city **Jerusalem**
  (Physical).
- When you add a device later (e.g. 8 PCs per department in step 16), place it in the Logical
  view as always, then move it to its building in the Physical view (M26).

---

## Part 4 — Moving devices to a new city or a new network
There are **two different kinds of "move"** — know which one you need:

| Kind | What changes | What stays | How |
|---|---|---|---|
| **Move a place** (physical) | only the location in the Physical view | cables, addresses, config | M26 |
| **Move to another network** (logical) | the switch it's plugged into, its VLAN, its IP address | the device itself | the 8 steps in 4.2 |

### 4.1 Opening a new city and moving its devices there (physical)
Example: you just built Jerusalem (step 13). All its devices are still in Home City / Haifa.

1. Physical view → **Intercity** → **New City** → rename it `Jerusalem` (M25).
2. Click it → **New Building** → rename `JLM Office`.
3. Click the building → **New Closet** → rename `JLM Main Closet`.
4. Go back to where the Jerusalem devices are now. **Navigation** → drag every `…-JLM-…`
   switch and router into **Jerusalem → JLM Office → JLM Main Closet**, and every Jerusalem
   PC, printer and server into **Jerusalem → JLM Office** (M26).
5. Go to **Intercity**. ✅ A line now joins Haifa, Jerusalem and Internet — those are the WAN
   links between the cities.
6. Logical view: `ping` from a Jerusalem PC to the internet still works.

❌ Link down after the move → cable length effects are on — see M26 ❌.

**Moving a whole logical group (cluster) into a new cluster:** Logical view → **Move Object**
→ click the device → choose the new cluster in the menu (M27).

### 4.2 Moving a device to a different network (logical)
Example: PC-HFA-ENG-08 moves from Haifa Engineering to **Jerusalem Sales**.

1. **File → Save** first.
2. Logical view: delete its cable (M7).
3. Rename it to the new site's naming: `PC-JLM-SAL-09` (M4).
4. Drag it near the new switch, SW-JLM-F1-ACC1.
5. Cable it (**Straight-Through**, **FastEthernet0**) to a **free port in the Sales range**
   (Fa0/1–10 on SW-JLM-F1-ACC1). Not sure which port is free? On the switch:
   `show interfaces status` → pick a port that says `notconnect` in VLAN 41.
6. PC → **Desktop → IP Configuration** → click **Static**, then **DHCP** again (this forces a
   new address).
7. ✅ The address is now `10.20.10.x` and the gateway `10.20.10.1`. `ping 10.20.10.1` → replies.
8. Physical view: move it to **Jerusalem → JLM Office** (M26).

❌ **`169.254.x.x`:** the port is not in VLAN 41 (wrong range) or is shut (parking VLAN 999).\
❌ **Still no address on a port that had another PC before:** port-security remembers the
old PC (sticky MAC) and drops the new one. Use a port that never had a device (`notconnect` in
`show interfaces status`).

> **Moving a switch to another network** is a bigger job: its VTP domain, VLANs, management
> address, gateway, uplinks and the guest list (ACL) all change. Treat it as a **new** switch:
> erase it (`erase startup-config` → `reload`) and configure it with the new site's block from
> its step.

---

## Checklist — is everything covered in your project?
- [ ] Plan part C: one block per city, one per building/floor, one /24 per department (Part 1)
- [ ] Plan part G: the table "which device in which city / building / closet" (1.5)
- [ ] Physical view: one city per site + Internet, buildings, closets, all devices moved (Part 3)
- [ ] Logical view still works after the moves (ping)
- [ ] Screenshots: Physical (chapter 8) and Logical with clusters (chapter 9)
- [ ] You can explain to the examiner why each city and building has its own block (summarization)
