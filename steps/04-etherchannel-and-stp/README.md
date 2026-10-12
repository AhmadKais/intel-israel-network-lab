# Step 04 — EtherChannel & Spanning Tree

> ⏱ 2 lessons · 🎯 ministry requirements 14, 15, 22.14, 6, 13 · 📁 start from: `my-work/step03.pkt` · 💾 save as: `my-work/step04.pkt`

## What you will have at the end
- DIST1 and DIST2 joined by **two cables bundled into one logical link** (an
  **EtherChannel** called `Port-channel1`).
- **Rapid Spanning Tree** on every switch, with **you** deciding which switch is the
  "boss" (root) of each VLAN.
- PC ports that come up instantly (**PortFast**) and shut themselves down if someone plugs in
  a switch (**BPDU Guard**).
- Distribution ports that refuse a fake "boss" (**Root Guard**).

```
        SWL3-HFA-A-DIST1 ══ Gi1/0/23 + Gi1/0/24 (Po1, LACP) ══ SWL3-HFA-A-DIST2
        root: VLAN 10,30,99,110                                 root: VLAN 20,120
             │   ╲                                          ╱   │
             │     ╲──────────────╲          ╱────────────╱     │
           ACC1                     ╲      ╱                 ACC2
                                      ╲  ╱
```

---

## Part 1 — Understand it first (simple English)

### 1.1 The loop problem (again)
Since step 03 our switches are connected in a circle. Redundancy is good — but switches
forward broadcasts out of every port, so in a loop **one broadcast circles forever**,
multiplies, and in a few seconds uses 100% of every link: a **broadcast storm**. The network
freezes.

### 1.2 Spanning Tree (STP) — the traffic police of switching
**STP** finds loops and **blocks** just enough ports to break them. If an active link dies,
it **unblocks** a backup port. You get redundancy *and* no loops.

How it decides, in simple steps:
1. **Elect a boss — the root bridge.** Every switch has a *bridge ID* = a **priority number**
   + its MAC address. **Lowest wins.** Think of choosing a team captain: lowest number is
   captain.
2. **Every other switch finds its best road to the boss** (its *root port*).
3. **On every other link, one side blocks.** That's the orange light you saw in step 03.

**Why choose the boss ourselves?** By default all priorities are equal (32768), so the switch
with the lowest MAC — often an old access switch at the edge — becomes root. Traffic then
takes silly paths. The root must be a strong **distribution** switch.

**PVST+** = **P**er-**V**LAN Spanning Tree: a separate tree for each VLAN. So DIST1 can be boss
of VLAN 10 while DIST2 is boss of VLAN 20 → both distribution switches carry traffic
(**load sharing**).

**Rapid-PVST+** = the modern, fast version: recovers from a failure in about **1 second**
instead of 30–50 seconds. We always use it.

Our plan:

| VLANs | Boss (root primary) | Deputy (root secondary) |
|---|---|---|
| 10, 30, 99, 110 | DIST1 | DIST2 |
| 20, 120 | DIST2 | DIST1 |

(In step 05 the boss of a VLAN will also be its main gateway — so traffic goes the shortest way.)

### 1.3 EtherChannel — two cables, one road
Two cables between DIST1 and DIST2 would normally be a loop, and STP would block one —
wasting it. **EtherChannel** bundles 2–8 cables into **one logical link**. STP sees *one*
link, so nothing is blocked:
- **double the bandwidth**,
- if one cable breaks, the other keeps working — nobody notices.

Comparison: two lanes painted as one wide road.

The two switches agree on the bundle with a protocol:

| Protocol | Who made it | Modes that form a bundle |
|---|---|---|
| **LACP** (we use it in Haifa) | IEEE standard — works with any vendor | `active` ↔ `active`, or `active` ↔ `passive` |
| **PAgP** (we use it in Jerusalem, step 13) | Cisco only | `desirable` ↔ `desirable`, or `desirable` ↔ `auto` |

All cables in a bundle must have **identical settings** (speed, trunk, native VLAN, allowed VLANs).

### 1.4 PortFast — skip the waiting room for PCs
A normal switch port waits ~30 s before forwarding, to check it's not creating a loop.
A PC is never a loop, and waiting makes PCs fail to get an address from DHCP. **PortFast**
lets **end-device ports** forward immediately.
⚠️ Only on ports to end devices — never on links between switches.

### 1.5 BPDU Guard — "a PC port must never hear a switch"
Switches talk STP with messages called **BPDUs**. PCs never send them. So if a BPDU arrives on
a PortFast port, someone plugged a **switch** in there (a mistake or an attack).
**BPDU Guard** immediately **shuts the port down** (state: *err-disabled*). An admin has to
turn it back on.

### 1.6 Root Guard — "no one below me may become boss"
An attacker could connect a switch with priority 0 and become the root → all traffic flows
through him. **Root Guard** on the distribution ports facing the access switches says: "if a
better boss appears through *this* port, block the port". Protects the root election.

---

## Part 2 — Prepare
**File → Open** → `my-work/step03.pkt` → **File → Save As** → `my-work/step04.pkt`.

---

## Part 3 — Build (click by click)
Two new cables, both **Copper Cross-Over** (switch ↔ switch):

| Click first | Port | Click second | Port |
|---|---|---|---|
| SWL3-HFA-A-DIST1 | **GigabitEthernet1/0/23** | SWL3-HFA-A-DIST2 | **GigabitEthernet1/0/23** |
| SWL3-HFA-A-DIST1 | **GigabitEthernet1/0/24** | SWL3-HFA-A-DIST2 | **GigabitEthernet1/0/24** |

Don't worry if they stay orange — we configure them now.

---

## Part 4 — Configure (type every line)

### 4.1 SWL3-HFA-A-DIST1
Log in (console `Intel#Con`, enable `Intel#En4ble`):
```
enable
configure terminal

! ---------- Spanning Tree: rapid mode + choose the boss ----------
spanning-tree mode rapid-pvst
spanning-tree vlan 10,30,99,110 root primary
spanning-tree vlan 20,120 root secondary

! ---------- EtherChannel members: identical trunk settings + bundle 1 ----------
interface range gi1/0/23 - 24
 description Po1 member to SWL3-HFA-A-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,99,110,120,998
 switchport nonegotiate
 channel-group 1 mode active
 no shutdown
 exit

! ---------- the logical link that the bundle creates ----------
interface port-channel 1
 description EtherChannel to SWL3-HFA-A-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,99,110,120,998
 switchport nonegotiate
 exit

! ---------- Root Guard on the ports facing the access switches ----------
interface range gi1/0/1 - 2
 spanning-tree guard root
 end
copy running-config startup-config
```
After `channel-group 1 mode active` you see `Creating a port-channel interface Port-channel 1` — good.

### 4.2 SWL3-HFA-A-DIST2
Same structure, but DIST2 is boss of VLAN 20 and 120:
```
enable
configure terminal

spanning-tree mode rapid-pvst
spanning-tree vlan 20,120 root primary
spanning-tree vlan 10,30,99,110 root secondary

interface range gi1/0/23 - 24
 description Po1 member to SWL3-HFA-A-DIST1
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,99,110,120,998
 switchport nonegotiate
 channel-group 1 mode active
 no shutdown
 exit

interface port-channel 1
 description EtherChannel to SWL3-HFA-A-DIST1
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,99,110,120,998
 switchport nonegotiate
 exit

interface range gi1/0/1 - 2
 spanning-tree guard root
 end
copy running-config startup-config
```

### 4.3 SW-HFA-A-ACC1
```
enable
configure terminal
spanning-tree mode rapid-pvst

! ports to end devices (PCs, printer, admin PC): PortFast + BPDU Guard
interface range fa0/1 - 22
 spanning-tree portfast
 spanning-tree bpduguard enable
 end
copy running-config startup-config
```
PT prints a long `%Warning: portfast should only be enabled on ports connected to a single
host...` — that's a reminder, not an error. It's correct here.

### 4.4 SW-HFA-A-ACC2
```
enable
configure terminal
spanning-tree mode rapid-pvst

! PCs, printer and the two access-point ports (an AP is an end device for STP)
interface range fa0/1 - 23
 spanning-tree portfast
 spanning-tree bpduguard enable
 end
copy running-config startup-config
```
Press **Fast Forward Time** a few times.

### 4.5 What the new commands mean
| Command | In simple English |
|---|---|
| `spanning-tree mode rapid-pvst` | "Use the fast, one-tree-per-VLAN version of STP" |
| `spanning-tree vlan 10 root primary` | "Make me the boss of VLAN 10" (sets a low priority: 24576) |
| `spanning-tree vlan 20 root secondary` | "Make me the deputy of VLAN 20" (28672) — takes over if the boss dies |
| `channel-group 1 mode active` | "Put this cable into bundle number 1, and actively ask the other side using LACP" |
| `interface port-channel 1` | The logical link that represents the whole bundle |
| `spanning-tree portfast` | "This port goes to an end device — skip the waiting" |
| `spanning-tree bpduguard enable` | "If a switch talks on this port, shut it down" |
| `spanning-tree guard root` | "Never accept a new boss through this port" |

---

## Part 5 — Check your work

### 5.1 The EtherChannel
On **DIST1**:
```
show etherchannel summary
```
```
Flags:  D - down        P - bundled in port-channel
        S - Layer2      U - in use
...
Group  Port-channel  Protocol    Ports
------+-------------+-----------+--------------------------------
1      Po1(SU)           LACP   Gig1/0/23(P) Gig1/0/24(P)
```
✅ `Po1(SU)` = Layer-2, **in use**. Both ports `(P)` = bundled.
If you see `(SD)` or `(I)`/`(s)` next to the ports, see the troubleshooting table.

### 5.2 Who is the boss?
On **DIST1**: `show spanning-tree vlan 10`
```
VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    24586
             Address     000A.F3C4.1101
             This bridge is the root            <- ✅ DIST1 is boss of VLAN 10
```
On **DIST2**: `show spanning-tree vlan 20` → `This bridge is the root` ✅

On **ACC1**: `show spanning-tree vlan 10`
```
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- -----------
Gi0/1            Root FWD 4         128.25   P2p        <- road to the boss (DIST1)
Gi0/2            Altn BLK 4         128.26   P2p        <- backup road, blocked
Fa0/1            Desg FWD 19        128.1    P2p Edge   <- "Edge" = PortFast
```
Now `show spanning-tree vlan 20` on ACC1 → this time **Gi0/2** (toward DIST2) is the Root port.
That's the load sharing: VLAN 10 goes up via DIST1, VLAN 20 via DIST2.

`show spanning-tree summary` → `Switch is in rapid-pvst mode` ✅

### 5.3 Break things on purpose (the best way to learn)
**Test 1 — redundancy**
1. PC-HFA-ENG-01 → Command Prompt → `ping -t 10.10.16.103` (continuous ping).
2. Click the **Delete** tool (X) → click the cable **ACC1 Gi0/1 ↔ DIST1**.
3. Watch the ping: 1–2 timeouts, then replies again — traffic moved to the Gi0/2 road.
4. Press **Ctrl+C** in the Command Prompt to stop. Put the cable back (Cross-Over,
   ACC1 Gi0/1 → DIST1 Gi1/0/1).

**Test 2 — the EtherChannel survives a broken cable**
1. Delete one of the two DIST1 ↔ DIST2 cables.
2. `show etherchannel summary` → Po1 still `(SU)` with one member.
3. Put the cable back (same ports) → after a moment both members are `(P)` again.

**Test 3 — BPDU Guard catches a rogue switch**
1. Drag a new **2960** next to ACC1 (leave it named `Switch…`).
2. Connect it with **Copper Cross-Over**: new switch **Fa0/1** → ACC1 **Fa0/9**.
3. Watch ACC1's CLI: `%SPANTREE-2-BLOCK_BPDUGUARD: Received BPDU on port Fa0/9 with BPDU Guard enabled. Disabling port.`
4. `show interfaces status err-disabled` → Fa0/9 is listed. The link light is red.
5. Delete the rogue switch. Recover the port:
```
configure terminal
interface fa0/9
 shutdown
 no shutdown
 end
```

---

## Part 6 — Save
`copy running-config startup-config` on all 4 switches → **File → Save**.

## Checkpoint
- [ ] `show etherchannel summary` → `Po1(SU)`, both ports `(P)` — on DIST1 and DIST2
- [ ] DIST1 is root for 10/30/99/110, DIST2 for 20/120
- [ ] Access ports show `Edge` (PortFast) in `show spanning-tree`
- [ ] You did the 3 break tests

## If something doesn't work
| Problem | Fix |
|---|---|
| `Po1(SD)` or ports `(s)` / `(I)` | The two cables are configured differently, or the other switch isn't configured yet. Reset both members on **both** switches: `default interface range gi1/0/23 - 24`, then retype 4.1/4.2 |
| One switch uses `active`, other `desirable` | Mixed LACP/PAgP — use `active` on both |
| Wrong switch is root | Check `show run \| include spanning-tree vlan` — maybe you typed `primary` on both for the same VLAN |
| PCs on ACC1 lose connectivity after Root Guard | `show spanning-tree inconsistentports` — an access switch has a lower priority than DIST1/2. Remove any `spanning-tree vlan ... priority` from access switches |
| Fa0/9 stays red after removing the rogue switch | You need `shutdown` + `no shutdown` on it |

## Challenge
`spanning-tree vlan 10 root primary` is a shortcut. Run `show run | include priority` on
DIST1: what priority did it actually set? Why is that number a multiple of 4096?

<details><summary>Answer</summary>

`spanning-tree vlan 10 priority 24576` (secondary = 28672). The priority field has 16 bits,
but the last 12 bits are used to carry the VLAN number ("extended system ID"), so the
priority itself can only change in steps of 2^12 = **4096**. That's why `show spanning-tree`
displays 24586 for VLAN 10: 24576 + 10.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

| In the example | In your project — take it from |
|---|---|
| Port-channel between the two distribution switches, LACP | plan part E (EtherChannel row: which site uses LACP) |
| Root bridge per VLAN: DIST1 for 10,30,99,110 — DIST2 for 20,120 | plan part D (the "primary" column — STP root must match the HSRP primary) |
| PortFast / BPDU guard on the user ports | your access ports from the port plan |

**Check:** `show etherchannel summary` shows `(SU)` and `(P)`; `show spanning-tree vlan <yours>`
shows the right switch as root for each of your VLANs.

💾 Save as `my-project/step04.pkt`.

---

## 👩‍🏫 Teacher notes
- **Lesson 1 (90 min):** draw a loop on the board and "pass a broadcast" around it until the
  class sees the storm; STP election as choosing a team captain; root/designated/blocked
  ports; PVST load sharing → students configure STP on all 4 switches → check 5.2.
- **Lesson 2 (90 min):** EtherChannel (two lanes = one road), LACP vs PAgP → cables + config →
  PortFast/BPDU Guard/Root Guard → the 3 break tests (students love test 3).
- **Common student questions**
  - *"Why not just use one cable and no STP?"* — One cable = one point of failure. Companies
    pay for redundancy; STP makes it safe.
  - *"Why is the blocked port wasted?"* — It's a spare tyre. And with PVST load sharing it is
    *not* wasted: it blocks only some VLANs and forwards others.
  - *"Why `active` on both sides?"* — `passive` + `passive` never forms a bundle (both wait).
    `active` + `active` is the safe choice.
- **Typical errors:** configuring the Port-channel but forgetting the members (or the
  opposite); `root primary` on both switches for the same VLAN; PortFast on uplinks.
