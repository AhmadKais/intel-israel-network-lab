# Step 05 — Inter-VLAN routing, HSRP & DHCP

> ⏱ 2–3 lessons · 🎯 ministry requirements 22.10, 17, 20.7, 22.11, 6 · 📁 start from: `my-work/step04.pkt` · 💾 save as: `my-work/step05.pkt`

## What you will have at the end
- The two distribution switches also work as **routers**: departments can finally talk to
  each other.
- Every VLAN has a gateway address (`.1`) that **keeps working even if one distribution
  switch dies** (HSRP).
- PCs get their IP settings **automatically** (DHCP), from **both** distribution switches.

```
                    VLAN 10 gateway = 10.10.16.1  (virtual — shared by both)
                            ╱                         ╲
   SWL3-HFA-A-DIST1  10.10.16.2  ACTIVE         SWL3-HFA-A-DIST2  10.10.16.3  STANDBY
   DHCP: hands out .11 – .130                   DHCP: hands out .131 – .254
```

---

## Part 1 — Understand it first (simple English)

### 1.1 Why departments still can't talk
Engineering (`10.10.16.0/24`, VLAN 10) and Finance (`10.10.17.0/24`, VLAN 20) are different
rooms **and** different streets. To cross between streets you need a **router**. Our
distribution switches are Layer-3 switches — they *have* a router inside; we just haven't
switched it on.

### 1.2 SVI — the door from each room into the router
An **SVI** (Switched Virtual Interface, typed as `interface vlan 10`) is a virtual router
port **inside** the switch, one per VLAN. Picture each room (VLAN) having a door into a
central hallway (the router). The SVI's IP address is that door's address.

- `ip routing` — switches the router brain on (without it the doors exist but nobody walks
  through).
- Once on, the switch routes between all its SVIs automatically = **inter-VLAN routing**.

### 1.3 HSRP — two receptionists, one phone number
Each PC has **one** default gateway address. If that gateway is DIST1 and DIST1 dies, the
whole VLAN is cut off — even though DIST2 is fine.

**HSRP** (Hot Standby Router Protocol) solves this. Like two receptionists sharing **one
phone number**: one answers (**Active**), the other listens and takes over if the first
leaves (**Standby**).

| | DIST1 | DIST2 | Shared "virtual" |
|---|---|---|---|
| VLAN 10 IP | 10.10.16.2 | 10.10.16.3 | **10.10.16.1** ← PCs use this |

- **Priority** decides who is Active (higher wins; default 100). We use 110 for the
  preferred switch, 90 for the other.
- **Preempt** — when the preferred switch comes back after a failure, it takes the role back.
  Without preempt it would stay Standby forever.
- **`standby version 2`** — the newer version (more groups, IPv6 support). Both sides must match.
- We number each HSRP **group** like its VLAN (VLAN 10 → group 10): easy to read.

**Match HSRP with Spanning Tree.** In step 04 DIST1 became boss (STP root) of VLAN 10. Now
DIST1 is also HSRP Active of VLAN 10. Then a frame from a PC goes straight up to the switch
that routes it — the shortest path. (DIST2 does the same for VLAN 20 and 120.)

### 1.4 DHCP — automatic addresses
Typing an IP on 50 PCs is slow and full of mistakes. A **DHCP server** hands out IP settings
automatically — like a hotel reception giving each guest a free room number, plus the
Wi-Fi password (gateway) and the phone directory (DNS).

The conversation has 4 messages — remember **DORA**:

| | Message | Meaning |
|---|---|---|
| D | **Discover** (PC → everyone) | "Is there a DHCP server here?" |
| O | **Offer** (server → PC) | "You can have 10.10.16.11" |
| R | **Request** (PC → server) | "Yes please, I'll take 10.10.16.11" |
| A | **Acknowledge** (server → PC) | "It's yours. Gateway .1, DNS 10.10.200.11" |

Settings in a DHCP **pool**:
- `network` — which subnet the pool serves
- `default-router` — the gateway (we give the HSRP **virtual** `.1`, so it survives failures)
- `dns-server` — the name server (it arrives in step 11)
- `ip dhcp excluded-address` — addresses the server must **never** hand out

**Split scope (DHCP redundancy).** Both distribution switches run DHCP for the same subnets,
but hand out **different halves**, so they never give the same address twice, and if one
dies the other still serves:

| Range | Used for |
|---|---|
| .1 – .10 | Reserved: gateway (.1), DIST1 (.2), DIST2 (.3), printers, devices |
| .11 – .130 | DIST1 gives these out |
| .131 – .254 | DIST2 gives these out |

> In Haifa, DHCP runs on the **routing devices** (the L3 switches). The ministry wants DHCP
> on routers in one branch and on a server in another (req. 22.11) — Jerusalem uses a server
> (step 13).

---

## Part 2 — Prepare
**File → Open** → `my-work/step04.pkt` → **File → Save As** → `my-work/step05.pkt`.

---

## Part 3 — Build
Nothing to build — no new devices or cables in this step. Everything happens in the CLI.

---

## Part 4 — Configure (type every line)

### 4.1 SWL3-HFA-A-DIST1 — routing + gateways (Active for 10, 30, 99, 110)
```
enable
configure terminal
! switch the router brain on
ip routing

interface vlan 10
 description GW ENGINEERING
 ip address 10.10.16.2 255.255.255.0
 standby version 2
 standby 10 ip 10.10.16.1
 standby 10 priority 110
 standby 10 preempt
 no shutdown
interface vlan 20
 description GW FINANCE
 ip address 10.10.17.2 255.255.255.0
 standby version 2
 standby 20 ip 10.10.17.1
 standby 20 priority 90
 standby 20 preempt
 no shutdown
interface vlan 30
 description GW PRINTERS-A
 ip address 10.10.18.2 255.255.255.0
 standby version 2
 standby 30 ip 10.10.18.1
 standby 30 priority 110
 standby 30 preempt
 no shutdown
interface vlan 99
 description GW MGMT-A
 ip address 10.10.19.2 255.255.255.0
 standby version 2
 standby 99 ip 10.10.19.1
 standby 99 priority 110
 standby 99 preempt
 no shutdown
interface vlan 110
 description GW WIFI-EMP
 ip address 10.10.20.2 255.255.255.0
 standby version 2
 standby 110 ip 10.10.20.1
 standby 110 priority 110
 standby 110 preempt
 no shutdown
interface vlan 120
 description GW WIFI-GUEST
 ip address 10.10.21.2 255.255.255.0
 standby version 2
 standby 120 ip 10.10.21.1
 standby 120 priority 90
 standby 120 preempt
 no shutdown
 exit
```
You'll see messages like `%HSRP-6-STATECHANGE: Vlan10 Grp 10 state Standby -> Active`.

DHCP on DIST1 — it hands out `.11 – .130`, so it excludes `.1 – .10` and `.131 – .254`:
```
ip dhcp excluded-address 10.10.16.1 10.10.16.10
ip dhcp excluded-address 10.10.16.131 10.10.16.254
ip dhcp excluded-address 10.10.17.1 10.10.17.10
ip dhcp excluded-address 10.10.17.131 10.10.17.254
ip dhcp excluded-address 10.10.20.1 10.10.20.10
ip dhcp excluded-address 10.10.20.131 10.10.20.254
ip dhcp excluded-address 10.10.21.1 10.10.21.10
ip dhcp excluded-address 10.10.21.131 10.10.21.254

ip dhcp pool HFA-ENGINEERING
 network 10.10.16.0 255.255.255.0
 default-router 10.10.16.1
 dns-server 10.10.200.11
 domain-name intel.lab
ip dhcp pool HFA-FINANCE
 network 10.10.17.0 255.255.255.0
 default-router 10.10.17.1
 dns-server 10.10.200.11
 domain-name intel.lab
ip dhcp pool HFA-WIFI-EMP
 network 10.10.20.0 255.255.255.0
 default-router 10.10.20.1
 dns-server 10.10.200.11
 domain-name intel.lab
ip dhcp pool HFA-WIFI-GUEST
 network 10.10.21.0 255.255.255.0
 default-router 10.10.21.1
 dns-server 200.1.100.10
 end
copy running-config startup-config
```
> Guests get the **internet provider's** DNS (200.1.100.10), never our internal one.
> Both DNS servers are built later (steps 11–12) — until then names won't resolve; IPs work.

> No pool for VLAN 30 (printers) and VLAN 99 (management): those devices get **fixed** addresses.

### 4.2 SWL3-HFA-A-DIST2 — routing + gateways (Active for 20, 120)
```
enable
configure terminal
ip routing

interface vlan 10
 description GW ENGINEERING
 ip address 10.10.16.3 255.255.255.0
 standby version 2
 standby 10 ip 10.10.16.1
 standby 10 priority 90
 standby 10 preempt
 no shutdown
interface vlan 20
 description GW FINANCE
 ip address 10.10.17.3 255.255.255.0
 standby version 2
 standby 20 ip 10.10.17.1
 standby 20 priority 110
 standby 20 preempt
 no shutdown
interface vlan 30
 description GW PRINTERS-A
 ip address 10.10.18.3 255.255.255.0
 standby version 2
 standby 30 ip 10.10.18.1
 standby 30 priority 90
 standby 30 preempt
 no shutdown
interface vlan 99
 description GW MGMT-A
 ip address 10.10.19.3 255.255.255.0
 standby version 2
 standby 99 ip 10.10.19.1
 standby 99 priority 90
 standby 99 preempt
 no shutdown
interface vlan 110
 description GW WIFI-EMP
 ip address 10.10.20.3 255.255.255.0
 standby version 2
 standby 110 ip 10.10.20.1
 standby 110 priority 90
 standby 110 preempt
 no shutdown
interface vlan 120
 description GW WIFI-GUEST
 ip address 10.10.21.3 255.255.255.0
 standby version 2
 standby 120 ip 10.10.21.1
 standby 120 priority 110
 standby 120 preempt
 no shutdown
 exit
```
DHCP on DIST2 — it hands out `.131 – .254`, so it excludes `.1 – .130`:
```
ip dhcp excluded-address 10.10.16.1 10.10.16.130
ip dhcp excluded-address 10.10.17.1 10.10.17.130
ip dhcp excluded-address 10.10.20.1 10.10.20.130
ip dhcp excluded-address 10.10.21.1 10.10.21.130

ip dhcp pool HFA-ENGINEERING
 network 10.10.16.0 255.255.255.0
 default-router 10.10.16.1
 dns-server 10.10.200.11
 domain-name intel.lab
ip dhcp pool HFA-FINANCE
 network 10.10.17.0 255.255.255.0
 default-router 10.10.17.1
 dns-server 10.10.200.11
 domain-name intel.lab
ip dhcp pool HFA-WIFI-EMP
 network 10.10.20.0 255.255.255.0
 default-router 10.10.20.1
 dns-server 10.10.200.11
 domain-name intel.lab
ip dhcp pool HFA-WIFI-GUEST
 network 10.10.21.0 255.255.255.0
 default-router 10.10.21.1
 dns-server 200.1.100.10
 end
copy running-config startup-config
```

### 4.3 Access switches — a management address
Access switches are Layer-2: they don't route. But we want to reach them over the network
(SSH in step 07), so each gets **one** IP address in the management VLAN 99, and a
gateway (like a PC).

**SW-HFA-A-ACC1:**
```
enable
configure terminal
interface vlan 99
 description MANAGEMENT
 ip address 10.10.19.11 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.10.19.1
end
copy running-config startup-config
```
**SW-HFA-A-ACC2:** the same with `ip address 10.10.19.12 255.255.255.0`.

### 4.4 PCs → DHCP
For every **user PC** (ENG-01/02/03, FIN-01/02/03): click it → **Desktop** → **IP
Configuration** → select **DHCP**. After a few seconds you see `DHCP request successful` and
the fields fill in by themselves. (If you see `DHCP failed. APIPA is being used`, press
**Fast Forward Time** and click **DHCP** again.)

Keep **static**: the printer (10.10.18.10, gw 10.10.18.1) and the admin PC (10.10.19.100,
gw 10.10.19.1).

### 4.5 What the new commands mean
| Command | In simple English |
|---|---|
| `ip routing` | "Switch on the router inside this switch" |
| `interface vlan 10` | "The router's door into room 10" (SVI) |
| `ip address 10.10.16.2 255.255.255.0` | This door's own address |
| `standby version 2` | Use HSRP version 2 |
| `standby 10 ip 10.10.16.1` | "In HSRP group 10, we share the phone number .1" |
| `standby 10 priority 110` | "I prefer to be the one who answers" (higher wins) |
| `standby 10 preempt` | "When I come back, I take my job back" |
| `ip dhcp excluded-address A B` | "Never give out the addresses from A to B" |
| `ip dhcp pool NAME` | Start a list of settings to hand out |
| `network` / `default-router` / `dns-server` | Which subnet / which gateway / which DNS the PCs get |
| `ip default-gateway 10.10.19.1` | On a Layer-2 switch: where to send its own management traffic |

---

## Part 5 — Check your work

### 5.1 HSRP
On **DIST1**: `show standby brief`
```
                     P indicates configured to preempt.
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Vl10        10   110 P Active   local           10.10.16.3      10.10.16.1
Vl20        20   90  P Standby  10.10.17.3      local           10.10.17.1
Vl30        30   110 P Active   local           10.10.18.3      10.10.18.1
Vl99        99   110 P Active   local           10.10.19.3      10.10.19.1
Vl110       110  110 P Active   local           10.10.20.3      10.10.20.1
Vl120       120  90  P Standby  10.10.21.3      local           10.10.21.1
```
✅ Active for 10/30/99/110, Standby for 20/120. DIST2 shows the opposite.

### 5.2 Routing table
On **DIST1**: `show ip route`
```
C    10.10.16.0/24 is directly connected, Vlan10
C    10.10.17.0/24 is directly connected, Vlan20
...
```
`C` = **C**onnected: the switch knows every VLAN because it has a door (SVI) in each.

### 5.3 DHCP
On **DIST1**: `show ip dhcp binding` → addresses between .11 and .130 with the PCs' MACs.
On **DIST2**: `show ip dhcp binding` → maybe some .131+ addresses (whoever answered first).

On a PC → Command Prompt → `ipconfig /all`:
```
   IPv4 Address......: 10.10.16.11
   Subnet Mask.......: 255.255.255.0
   Default Gateway...: 10.10.16.1
   DHCP Servers......: 10.10.16.2
   DNS Servers.......: 10.10.200.11
```

### 5.4 Pings
| From | Type | Expected |
|---|---|---|
| an ENG PC | `ping 10.10.16.1` | ✅ its gateway |
| an ENG PC | `ping` a FIN PC's address | ✅ **inter-VLAN routing works!** |
| an ENG PC | `ping 10.10.18.10` (printer) | ✅ |
| an ENG PC | `tracert` a FIN PC | first hop `10.10.16.2` (DIST1 — the Active for VLAN 10) |
| PC-HFA-ADMIN-01 | `ping 10.10.19.11` and `10.10.19.12` | ✅ the access switches' management IPs |

### 5.5 Failover test — HSRP in action
1. On an **ENG PC**: `ping -t` a **FIN PC**.
2. Click **DIST1** → **Physical** → click its **power button** (switch it off).
3. Watch the ping: a few timeouts, then replies continue — DIST2 took over `.1`.
4. On DIST2: `show standby brief` → now **Active** for every group.
5. While DIST1 is off, on another PC: `ipconfig /release` then `ipconfig /renew` → you get a
   **.131+** address — from DIST2. Split scope works!
6. Switch DIST1 back on (power button). Wait for it to boot (Fast Forward Time).
   `show standby brief` on DIST1 → Active again for 10/30/99/110 — that's **preempt**.
7. Stop the ping with **Ctrl+C**.

---

## Part 6 — Save
`copy running-config startup-config` on all 4 switches → **File → Save**.

## Checkpoint
- [ ] `show standby brief` correct on both DIST switches
- [ ] User PCs get DHCP addresses with gateway `.1`
- [ ] Engineering ↔ Finance ping works
- [ ] Failover test passed (ping survives DIST1 off; preempt returns it)

## If something doesn't work
| Problem | Fix |
|---|---|
| SVI is `down/down` (`show ip interface brief`) | The VLAN doesn't exist on this switch or no port/trunk carries it. `show vlan brief`, `show interfaces trunk` |
| Pings between VLANs fail but each VLAN reaches its gateway | Forgot `ip routing` |
| Both switches say **Active** for the same group | `standby version` differs, or the VLAN isn't carried between them (check Po1 trunk allowed list) |
| PCs get `169.254.x.x` (APIPA) | No DHCP answer: pool `network` typo, all addresses excluded, or the PC's port is in the wrong VLAN. Press Fast Forward Time and retry |
| PC gets an address but no gateway | `default-router` missing in the pool |

## Challenge
Load-sharing question: VLAN 10 traffic goes through DIST1, VLAN 20 through DIST2. Imagine you
made DIST1 the HSRP Active for VLAN 20 **but** DIST2 stays STP root for VLAN 20. Draw the path a
FIN PC on ACC1 takes to reach its gateway. Why is it worse?

<details><summary>Answer</summary>

PC → ACC1 → (STP forwards toward the root DIST2) → DIST2 → Po1 → DIST1 (HSRP Active) →
routed. The frame crosses the EtherChannel for nothing: an extra hop, and the inter-DIST link
carries traffic it shouldn't. Rule: **HSRP Active = STP root**, per VLAN.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

| In the example | In your project — take it from |
|---|---|
| SVIs `.2` / `.3`, virtual gateway `.1` | plan part D (your subnets, same `.1/.2/.3` rule) |
| HSRP group number = VLAN number | keep the rule, with **your** VLAN numbers |
| HSRP primary (priority + preempt) per VLAN | plan part D ("primary" column — same switch as the STP root in step 04) |
| DHCP pools `HFA-ENGINEERING`… and excluded addresses | your pool names `<SITE>-<DEPT>`, your subnets, your reserved range |
| DHCP on the L3 switches | plan part E (DHCP row) — the ministry wants DHCP on routers/L3 at one site and on a server at another (req. 22.11) |

**Check:** PCs set to DHCP get an address from **your** subnet with gateway `.1`; ping between
two of your departments works; shut the primary switch — ping keeps working.

💾 Save as `my-project/step05.pkt`.

---

## 👩‍🏫 Teacher notes
- **Lesson 1 (90 min):** why VLANs can't talk → SVIs as "doors into the router" → `ip routing`
  → configure DIST1 SVIs without HSRP first if the class is slow, test inter-VLAN ping.
- **Lesson 2 (90 min):** HSRP (two receptionists, one phone number) → add HSRP on both →
  `show standby brief` → failover test on the projector.
- **Lesson 3 (60–90 min):** DHCP and DORA (show it in Simulation mode with the **DHCP**
  filter) → split scope → pools on both → switch PCs to DHCP → release/renew test with DIST1 off.
- **Common student questions**
  - *"Why does DHCP give `.1` and not `.2` as gateway?"* — `.2` is DIST1 itself; if DIST1 dies,
    `.2` dies. `.1` moves to whichever switch is alive.
  - *"Why two DHCP servers? Won't they conflict?"* — Not with split scope: they never give the
    same address.
  - *"Why do access switches need an IP?"* — Only to be managed remotely. They don't route.
- **Typical errors:** typo in the HSRP virtual IP on one switch (two different "phone
  numbers"); forgetting `no shutdown` on SVIs; the same priority on both (then the highest IP
  wins — unpredictable for students).
