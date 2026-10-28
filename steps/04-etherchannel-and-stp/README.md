# Step 04 — EtherChannel & Spanning Tree

> ⏱ 2 lessons · 🎯 ministry requirements 14, 15, 22.14, 6, 13 · 📁 start from: `my-work/step03.pkt` · 💾 save as: `my-work/step04.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 03 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## 🗺 Your missions in this step
- [ ] **Mission 1** — Open step 03 and save as step 04 (2 min)
- [ ] **Mission 2** — Two new cables between DIST1 and DIST2 (5 min)
- [ ] **Mission 3** — DIST1: Rapid STP + boss of its VLANs (5 min)
- [ ] **Mission 4** — DIST1: the EtherChannel + Root Guard (10 min)
- [ ] **Mission 5** — DIST2: the same, mirrored (15 min)
- [ ] **Mission 6** — ACC1 and ACC2: Rapid STP, PortFast, BPDU Guard (10 min)
- [ ] **Mission 7** — Check the EtherChannel and the bosses (10 min)
- [ ] **Mission 8** — Break test 1: pull an uplink (10 min)
- [ ] **Mission 9** — Break test 2: pull an EtherChannel cable (5 min)
- [ ] **Mission 10** — Break test 3: plug in a rogue switch (10 min)
- [ ] **Exercise** — the hidden priority
- [ ] **🛠 Your own project**

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

## Mission 1 — Open step 03 and save as step 04 (how? → M1)
**File → Open** → `my-work/step03.pkt` → **File → Save As** → `my-work/step04.pkt`.

✅ The title bar shows `step04.pkt`.

---

## Mission 2 — Two new cables between DIST1 and DIST2
Both **Copper Cross-Over** (switch ↔ switch) (how? → M5):

| # | Click first | Port | Click second | Port |
|---|---|---|---|---|
| 1 | SWL3-HFA-A-DIST1 | **GigabitEthernet1/0/23** | SWL3-HFA-A-DIST2 | **GigabitEthernet1/0/23** |
| 2 | SWL3-HFA-A-DIST1 | **GigabitEthernet1/0/24** | SWL3-HFA-A-DIST2 | **GigabitEthernet1/0/24** |

✅ Two dashed lines between DIST1 and DIST2, labelled `Gi1/0/23` and `Gi1/0/24` at both ends.
Don't worry if they stay orange — we configure them now.

---

## Mission 3 — DIST1: Rapid STP + boss of its VLANs
👉 **Where:** **SWL3-HFA-A-DIST1** → CLI → **Enter** → banner → `Password:` `Intel#Con` →
`enable` → `Intel#En4ble`. Prompt: `SWL3-HFA-A-DIST1#` (how? → M9).

⌨️ Type:
```
configure terminal
spanning-tree mode rapid-pvst
spanning-tree vlan 10,30,99,110 root primary
spanning-tree vlan 20,120 root secondary
```
✅ No error messages; prompt still `SWL3-HFA-A-DIST1(config)#`.
🔍 **Check now:** `do show spanning-tree vlan 10` → near the top: `This bridge is the root`.

---

## Mission 4 — DIST1: the EtherChannel + Root Guard
👉 **Where:** still DIST1, prompt `(config)#`.

### 4a — The two member cables: identical trunk settings + bundle 1
⌨️ Type:
```
interface range gi1/0/23 - 24
 description Po1 member to SWL3-HFA-A-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,99,110,120,998
 switchport nonegotiate
 channel-group 1 mode active
 no shutdown
 exit
```
✅ After `channel-group 1 mode active` you see `Creating a port-channel interface Port-channel 1`.

### 4b — The logical link that the bundle creates
⌨️ Type:
```
interface port-channel 1
 description EtherChannel to SWL3-HFA-A-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,99,110,120,998
 switchport nonegotiate
 exit
```

### 4c — Root Guard on the ports facing the access switches
⌨️ Type:
```
interface range gi1/0/1 - 2
 spanning-tree guard root
 end
copy running-config startup-config
```
✅ `[OK]`. The bundle is not up yet — DIST2 is still missing its half (Mission 5).

---

## Mission 5 — DIST2: the same, mirrored
👉 **Where:** **SWL3-HFA-A-DIST2** → CLI → log in → `SWL3-HFA-A-DIST2#`.

DIST2 is boss of VLAN 20 and 120, deputy for the others — the opposite of DIST1.

### 5a — STP
⌨️ Type:
```
configure terminal
spanning-tree mode rapid-pvst
spanning-tree vlan 20,120 root primary
spanning-tree vlan 10,30,99,110 root secondary
```

### 5b — EtherChannel and Root Guard
⌨️ Type:
```
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
Press **Fast Forward Time** 2–3 times (M18).

✅ You see messages like `%LINK-5-CHANGED: Interface Port-channel1, changed state to up`, and the
two DIST1–DIST2 cables turn green.
🔍 **Check now:** `show etherchannel summary` → a line `1  Po1(SU)  LACP  Gig1/0/23(P) Gig1/0/24(P)`.
❌ `Po1(SD)` → see "If something doesn't work" (first row).

---

## Mission 6 — ACC1 and ACC2: Rapid STP, PortFast, BPDU Guard

### 6a — SW-HFA-A-ACC1
👉 **Where:** ACC1 → CLI → log in → `SW-HFA-A-ACC1#`.

⌨️ Type:
```
configure terminal
spanning-tree mode rapid-pvst

! ports to end devices (PCs, printer, admin PC): PortFast + BPDU Guard
interface range fa0/1 - 22
 spanning-tree portfast
 spanning-tree bpduguard enable
 end
copy running-config startup-config
```
✅ PT prints a long `%Warning: portfast should only be enabled on ports connected to a single
host...` — that's a reminder, not an error. It's correct here.

### 6b — SW-HFA-A-ACC2
👉 **Where:** ACC2 → CLI → log in → `SW-HFA-A-ACC2#`.

⌨️ Type:
```
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

### What the new commands mean
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

## Mission 7 — Check the EtherChannel and the bosses

### 7a — The EtherChannel
👉 On **DIST1** (`#` prompt): `show etherchannel summary`

✅ You should see:
```
Flags:  D - down        P - bundled in port-channel
        S - Layer2      U - in use
...
Group  Port-channel  Protocol    Ports
------+-------------+-----------+--------------------------------
1      Po1(SU)           LACP   Gig1/0/23(P) Gig1/0/24(P)
```
`Po1(SU)` = Layer-2, **in use**. Both ports `(P)` = bundled.
❌ `(SD)` or `(I)`/`(s)` next to the ports → see the troubleshooting table.

### 7b — Who is the boss?
👉 On **DIST1**: `show spanning-tree vlan 10`

✅ You should see:
```
VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    24586
             Address     000A.F3C4.1101
             This bridge is the root            <- DIST1 is boss of VLAN 10
```
👉 On **DIST2**: `show spanning-tree vlan 20` → ✅ `This bridge is the root`.

### 7c — The view from an access switch
👉 On **ACC1**: `show spanning-tree vlan 10`

✅ You should see (your cost numbers may differ):
```
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- -----------
Gi0/1            Root FWD 4         128.25   P2p        <- road to the boss (DIST1)
Gi0/2            Altn BLK 4         128.26   P2p        <- backup road, blocked
Fa0/1            Desg FWD 19        128.1    P2p Edge   <- "Edge" = PortFast
```
👉 Now `show spanning-tree vlan 20` on ACC1.
✅ This time **Gi0/2** (toward DIST2) is the Root port. That's the load sharing: VLAN 10 goes up
via DIST1, VLAN 20 via DIST2.

👉 `show spanning-tree summary` → ✅ `Switch is in rapid-pvst mode`.

---

## Mission 8 — Break test 1: pull an uplink
🧠 **Why:** prove the redundancy really works. The best way to learn is to break things on purpose.

1. Click **PC-HFA-ENG-01** → Command Prompt → `ping -t 10.10.16.103` (a ping that never stops).
   Leave the window open.
2. Toolbar: click the **Delete** tool (X) → click the cable **ACC1 Gi0/1 ↔ DIST1** (M7).
3. Watch the ping window.

✅ 1–2 `Request timed out`, then `Reply from` again — traffic moved to the Gi0/2 road.

4. Click in the Command Prompt and press **Ctrl+C** to stop the ping.
5. Put the cable back: **Copper Cross-Over**, ACC1 **Gi0/1** → DIST1 **Gi1/0/1**.

---

## Mission 9 — Break test 2: pull an EtherChannel cable
1. Delete one of the two DIST1 ↔ DIST2 cables (M7).
2. On DIST1: `show etherchannel summary`.
   ✅ Po1 is still `(SU)`, now with one member.
3. Put the cable back (Cross-Over, same ports as Mission 2) → Fast Forward Time.
4. `show etherchannel summary` again.
   ✅ Both members are `(P)` again.

---

## Mission 10 — Break test 3: plug in a rogue switch
1. Drag a new **2960** next to ACC1 (leave its name `Switch…`).
2. Connect it with **Copper Cross-Over**: new switch **Fa0/1** → ACC1 **Fa0/9**.
3. Watch ACC1's CLI.

✅ `%SPANTREE-2-BLOCK_BPDUGUARD: Received BPDU on port Fa0/9 with BPDU Guard enabled. Disabling port.`

4. On ACC1: `show interfaces status`.
   ✅ Fa0/9 shows `err-disabled`. The dot on that cable is red.
5. Delete the rogue switch (M7).
6. Recover the port on ACC1:
```
configure terminal
interface fa0/9
 shutdown
 no shutdown
 end
```
7. Save all 4 switches (`copy running-config startup-config`) → **File → Save**.

---

## Checkpoint
- [ ] `show etherchannel summary` → `Po1(SU)`, both ports `(P)` — on DIST1 and DIST2
- [ ] DIST1 is root for 10/30/99/110, DIST2 for 20/120
- [ ] Access ports show `Edge` (PortFast) in `show spanning-tree`
- [ ] You did the 3 break tests

## If something doesn't work
| Problem | Fix |
|---|---|
| `Po1(SD)` or ports `(s)` / `(I)` | The two cables are configured differently, or the other switch isn't configured yet. On **both** switches: `interface range gi1/0/23 - 24` → `no channel-group 1` → `exit` → `no interface port-channel 1`, then retype Mission 4 / 5b |
| One switch uses `active`, other `desirable` | Mixed LACP/PAgP — use `active` on both |
| Wrong switch is root | Check `show run \| include spanning-tree vlan` — maybe you typed `primary` on both for the same VLAN |
| PCs on ACC1 lose connectivity after Root Guard | `show spanning-tree vlan 10` on DIST1 — the port to the access switch is blocked by Root Guard because an access switch has a lower priority than DIST1/2. Remove any `spanning-tree vlan ... priority` from the access switches |
| Fa0/9 stays red after removing the rogue switch | You need `shutdown` + `no shutdown` on it |

---

## 🏋 Exercise — The hidden priority ⭐⭐
- 🎯 **Task:** `spanning-tree vlan 10 root primary` is a shortcut. Find out which **number** it
  actually set, and explain why that number is a multiple of 4096.
- 📁 **Start from:** your `step04.pkt`.
- 👣 **Steps:**
  1. On DIST1 run `show run | include priority`.
  2. Write down the priority for VLAN 10 and for VLAN 20.
  3. Run `show spanning-tree vlan 10` and look at **Bridge ID → Priority**. Why is it not exactly
     the number from step 2?
- ✅ **Done when:** you have both numbers and a one-sentence explanation of the difference.

<details><summary>💡 Hint 1</summary>

Subtract the number in step 2 from the number in step 3. What do you get? Does it remind you
of something?
</details>
<details><summary>💡 Hint 2</summary>

The priority field is 16 bits; 12 of them carry something else. 2^12 = ?
</details>
<details><summary>✔ Answer</summary>

`spanning-tree vlan 10 priority 24576` (secondary = 28672). The priority field has 16 bits,
but the last 12 bits are used to carry the VLAN number ("extended system ID"), so the
priority itself can only change in steps of 2^12 = **4096**. That's why `show spanning-tree`
displays 24586 for VLAN 10: 24576 + 10.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

- [ ] Open `my-project/step03.pkt` → **Save As** `my-project/step04.pkt`
- [ ] Two Cross-Over cables between your distribution switches
- [ ] Rapid-PVST on every switch; root primary/secondary per **your** VLAN table
- [ ] EtherChannel (the protocol your plan part E gives this site) on both distribution switches
- [ ] Root Guard on the distribution downlinks; PortFast + BPDU Guard on every end-device port
- [ ] Mission 7 checks and the 3 break tests pass in your project

| In the example | In your project — take it from |
|---|---|
| Port-channel between the two distribution switches, LACP | plan part E (EtherChannel row: which site uses LACP) |
| Root bridge per VLAN: DIST1 for 10,30,99,110 — DIST2 for 20,120 | plan part D (the "primary" column — STP root must match the HSRP primary) |
| PortFast / BPDU guard on the user ports | your access ports from the port plan |

**Check:** `show etherchannel summary` shows `(SU)` and `(P)`; `show spanning-tree vlan <yours>`
shows the right switch as root for each of your VLANs.

💾 Save as `my-project/step04.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★2 What is the advantage of EtherChannel between two switches?**

<details><summary>Answer</summary>

Several physical links act as one logical link: more bandwidth, and if one cable fails traffic continues on the others with no STP recalculation.

**Show it:** `show etherchannel summary`
</details>

**2. ★2 Two switches are connected with 4 cables of 100 Mbps, but the speed between them is 100, not 400. Why? What is the fix?**

<details><summary>Answer</summary>

STP sees 4 parallel links as a loop and blocks 3 of them; only one forwards. Fix: bundle them into an EtherChannel — STP then sees one logical link and all 4 carry traffic.

**Show it:** `show spanning-tree` before and after the port-channel.
</details>

**3. ★2 Which two protocols can build an EtherChannel bundle?**

<details><summary>Answer</summary>

LACP (IEEE 802.3ad, open standard — modes `active`/`passive`) and PAgP (Cisco — modes `desirable`/`auto`). You can also force it with mode `on` (no protocol).

**Show it:** `show etherchannel summary` → Protocol column
</details>

**4. ★1 What is the role of STP in a LAN?**

<details><summary>Answer</summary>

To prevent Layer-2 loops in a redundant network: it blocks some ports so there is exactly one active path between any two switches, and unblocks a backup if the active path fails.

**Show it:** `show spanning-tree vlan <yours>`
</details>

**5. ★2 Where is STP used in your project and which type?**

<details><summary>Answer</summary>

On every switch, in Rapid-PVST+ mode (`spanning-tree mode rapid-pvst`): one rapid spanning tree per VLAN.

**Show it:** `show spanning-tree summary`
</details>

**6. ★2 Show me which switch is the root bridge. How do you make a non-root switch the root?**

<details><summary>Answer</summary>

The root has the lowest bridge ID (priority + MAC). `show spanning-tree vlan 10` → "This bridge is the root". To make another switch root: `spanning-tree vlan 10 root primary` (or `spanning-tree vlan 10 priority 4096`).

**Show it:** `show spanning-tree vlan <yours>` on both distribution switches
</details>

**7. ★2 Show me the port states. What does Blocking / Forwarding mean?**

<details><summary>Answer</summary>

Forwarding (FWD): the port sends and receives frames. Blocking (BLK, in RSTP "discarding"): it only listens to BPDUs and drops data frames, to break a loop. Port roles: Root, Designated, Alternate/Backup.

**Show it:** `show spanning-tree vlan <yours>`
</details>

**8. ★2 What is a BPDU?**

<details><summary>Answer</summary>

Bridge Protocol Data Unit — the message switches exchange (every 2 s) to elect the root bridge and decide which ports forward or block.

**Show it:** Simulation mode with the STP filter.
</details>

**9. ★2 Which two commands are recommended on access ports, related to STP, and what do they do?**

<details><summary>Answer</summary>

`spanning-tree portfast` — the port goes straight to forwarding (a PC doesn't wait ~30 s). `spanning-tree bpduguard enable` — if a BPDU arrives (someone plugged in a switch), the port is shut down (err-disabled).

**Show it:** `show running-config interface fa0/1`
</details>

**10. ★2 Which versions of STP exist? What is the difference between STP and PVST?**

<details><summary>Answer</summary>

802.1D STP (one tree for all VLANs), PVST+ (Cisco — one tree per VLAN), RSTP 802.1w (fast convergence), Rapid-PVST+ (RSTP per VLAN — what we use), MSTP 802.1s (groups VLANs into instances). PVST lets each VLAN have a different root, so different VLANs can use different links (load sharing).

**Show it:** `show spanning-tree summary` → mode
</details>

**11. ★3 How long does a port take to start forwarding with classic STP? Which states?**

<details><summary>Answer</summary>

Blocking → Listening (15 s) → Learning (15 s) → Forwarding: 30 s for a new port, up to 50 s if it first waits max-age (20 s) after a topology change. Rapid-PVST+ and PortFast make it almost immediate.

**Show it:** —
</details>

**12. ★3 How can STP be used to attack a LAN, and how do you protect it?**

<details><summary>Answer</summary>

An attacker plugs in a switch that sends BPDUs with a better (lower) priority, becomes the root, and traffic flows through it (man-in-the-middle) or the topology keeps changing. Protection: BPDU Guard on access ports, Root Guard on the ports facing down to the access layer.

**Show it:** `show running-config | include guard`
</details>

**13. ★3 What is the difference between BPDU Guard and Root Guard?**

<details><summary>Answer</summary>

BPDU Guard shuts the port if **any** BPDU arrives — for ports with end devices. Root Guard allows BPDUs but blocks the port (root-inconsistent) if a **superior** BPDU arrives — for switch-to-switch ports where a new root must never appear.

**Show it:** `show running-config` on a downlink → `spanning-tree guard root`; `show spanning-tree vlan <yours>`
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

In a copy of your finished step-04 file, remove the port-channel and the root/priority commands.
Without this page: rebuild the EtherChannel between your distribution switches, set the root per
VLAN according to your plan, and add PortFast + BPDU Guard on access ports and Root Guard on
downlinks. Target: **15 minutes**. Then show `show etherchannel summary` and
`show spanning-tree vlan <yours>` and explain which port blocks and why.

---

## 👩‍🏫 Teacher notes
- **Lesson 1 (90 min):** draw a loop on the board and "pass a broadcast" around it until the
  class sees the storm; STP election as choosing a team captain; root/designated/blocked
  ports; PVST load sharing → Missions 1–3 and 5a (STP on the distribution switches), Mission 6
  → Mission 7b–c checks.
- **Lesson 2 (90 min):** EtherChannel (two lanes = one road), LACP vs PAgP → Missions 4, 5b,
  7a → Missions 8–10, the 3 break tests (students love test 3).
- **Common student questions**
  - *"Why not just use one cable and no STP?"* — One cable = one point of failure. Companies
    pay for redundancy; STP makes it safe.
  - *"Why is the blocked port wasted?"* — It's a spare tyre. And with PVST load sharing it is
    *not* wasted: it blocks only some VLANs and forwards others.
  - *"Why `active` on both sides?"* — `passive` + `passive` never forms a bundle (both wait).
    `active` + `active` is the safe choice.
- **Typical errors:** configuring the Port-channel but forgetting the members (or the
  opposite); `root primary` on both switches for the same VLAN; PortFast on uplinks.
