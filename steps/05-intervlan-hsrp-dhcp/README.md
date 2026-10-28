# Step 05 — Inter-VLAN routing, HSRP & DHCP

> ⏱ 2–3 lessons · 🎯 ministry requirements 22.10, 17, 20.7, 22.11, 6 · 📁 start from: `my-work/step04.pkt` · 💾 save as: `my-work/step05.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 04 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## 🗺 Your missions in this step
- [ ] **Mission 1** — Open step 04 and save as step 05 (2 min)
- [ ] **Mission 2** — DIST1: switch on routing + the first gateway (VLAN 10) (10 min)
- [ ] **Mission 3** — DIST1: the other five gateways (15 min)
- [ ] **Mission 4** — DIST2: the six gateways, mirrored (20 min)
- [ ] **Mission 5** — Check HSRP and test inter-VLAN routing with fixed addresses (10 min)
- [ ] **Mission 6** — DHCP on DIST1 (15 min)
- [ ] **Mission 7** — DHCP on DIST2 (10 min)
- [ ] **Mission 8** — Management address on the access switches (10 min)
- [ ] **Mission 9** — Switch the user PCs to DHCP and test (15 min)
- [ ] **Mission 10** — Failover test: kill a gateway, keep pinging (15 min)
- [ ] **Exercises** A and B
- [ ] **🛠 Your own project**

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

The full plan for this step:

| VLAN | Subnet | DIST1 IP | DIST2 IP | Virtual `.1` | DIST1 priority | DIST2 priority | Active |
|---|---|---|---|---|---|---|---|
| 10 | 10.10.16.0/24 | .2 | .3 | 10.10.16.1 | **110** | 90 | DIST1 |
| 20 | 10.10.17.0/24 | .2 | .3 | 10.10.17.1 | 90 | **110** | DIST2 |
| 30 | 10.10.18.0/24 | .2 | .3 | 10.10.18.1 | **110** | 90 | DIST1 |
| 99 | 10.10.19.0/24 | .2 | .3 | 10.10.19.1 | **110** | 90 | DIST1 |
| 110 | 10.10.20.0/24 | .2 | .3 | 10.10.20.1 | **110** | 90 | DIST1 |
| 120 | 10.10.21.0/24 | .2 | .3 | 10.10.21.1 | 90 | **110** | DIST2 |

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

## Mission 1 — Open step 04 and save as step 05 (how? → M1)
**File → Open** → `my-work/step04.pkt` → **File → Save As** → `my-work/step05.pkt`.

Nothing to build in this step — no new devices or cables. Everything happens in the CLI.

---

## Mission 2 — DIST1: switch on routing + the first gateway (VLAN 10)
🧠 **Why:** do **one** VLAN first and check it. If it works, the other five are the same pattern.

👉 **Where:** **SWL3-HFA-A-DIST1** → CLI → log in (`Intel#Con`, `enable`, `Intel#En4ble`) →
`SWL3-HFA-A-DIST1#`.

⌨️ Type:
```
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
 end
```
Press **Fast Forward Time** (M18).

✅ You see `%LINK-5-CHANGED: Interface Vlan10, changed state to up` and, a few seconds later,
`%HSRP-6-STATECHANGE: Vlan10 Grp 10 state Standby -> Active`.
🔍 **Check now:** `show standby brief` → one line: `Vl10  10  110 P Active  local  unknown  10.10.16.1`.
(`Standby` says `unknown` because DIST2 isn't configured yet — correct.)
❌ The SVI stays `down` → VLAN 10 must exist and be carried on a trunk: `show vlan brief`.

---

## Mission 3 — DIST1: the other five gateways
👉 **Where:** DIST1, prompt `SWL3-HFA-A-DIST1#`.

Same pattern as Mission 2. **Watch the priority:** 110 where DIST1 is Active (30, 99, 110),
**90** where DIST2 will be Active (20, 120) — see the table in 1.3.

⌨️ Type:
```
configure terminal
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
 end
copy running-config startup-config
```
🔍 **Check now:** `show ip interface brief` → at the bottom: Vlan10, 20, 30, 99, 110, 120 with
addresses `10.10.16.2` … `10.10.21.2`, all `up up`.

---

## Mission 4 — DIST2: the six gateways, mirrored
👉 **Where:** **SWL3-HFA-A-DIST2** → CLI → log in → `SWL3-HFA-A-DIST2#`.

Differences from DIST1: the address ends in **`.3`**, and the priorities are the **opposite**
(110 for 20 and 120, 90 for the others).

⌨️ Type:
```
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
 end
copy running-config startup-config
```
Press **Fast Forward Time** 2–3 times.

✅ `%HSRP-6-STATECHANGE` messages: Vl20 and Vl120 become **Active**, the others **Standby**.

---

## Mission 5 — Check HSRP and test inter-VLAN routing

### 5a — HSRP
👉 On **DIST1**: `show standby brief`

✅ You should see:
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
Active for 10/30/99/110, Standby for 20/120. On DIST2 the same command shows the opposite.

❌ Both switches say **Active** for the same group → `standby version` differs, or the VLAN
isn't carried between them (check the Po1 allowed list from step 04).

### 5b — Routing table
👉 On **DIST1**: `show ip route`

✅ You should see lines like:
```
C    10.10.16.0/24 is directly connected, Vlan10
C    10.10.17.0/24 is directly connected, Vlan20
...
```
`C` = **C**onnected: the switch knows every VLAN because it has a door (SVI) in each.

### 5c — The first inter-VLAN ping
The user PCs still have fixed addresses **without a gateway**. Give two of them a gateway
for this test (how? → M14):
- **PC-HFA-ENG-01**: Default Gateway `10.10.16.1`
- **PC-HFA-FIN-01**: Default Gateway `10.10.17.1`

Then from PC-HFA-ENG-01 → Command Prompt → `ping 10.10.17.101`.

✅ Replies — **Engineering can finally talk to Finance!** (The first one or two may time out — ARP.)
❌ Each PC reaches its own gateway (`ping 10.10.16.1`) but not the other VLAN → `ip routing` is
missing on the switch.

---

## Mission 6 — DHCP on DIST1
👉 **Where:** DIST1, prompt `SWL3-HFA-A-DIST1#`.

DIST1 hands out `.11 – .130`, so it excludes `.1 – .10` and `.131 – .254`.

### 6a — Excluded addresses
⌨️ Type:
```
configure terminal
ip dhcp excluded-address 10.10.16.1 10.10.16.10
ip dhcp excluded-address 10.10.16.131 10.10.16.254
ip dhcp excluded-address 10.10.17.1 10.10.17.10
ip dhcp excluded-address 10.10.17.131 10.10.17.254
ip dhcp excluded-address 10.10.20.1 10.10.20.10
ip dhcp excluded-address 10.10.20.131 10.10.20.254
ip dhcp excluded-address 10.10.21.1 10.10.21.10
ip dhcp excluded-address 10.10.21.131 10.10.21.254
```

### 6b — The pools
⌨️ Type:
```
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
✅ After `ip dhcp pool …` the prompt is `(dhcp-config)#`. `[OK]` at the end.

> Guests get the **internet provider's** DNS (200.1.100.10), never our internal one.
> Both DNS servers are built later (steps 11–12) — until then names won't resolve; IPs work.

> No pool for VLAN 30 (printers) and VLAN 99 (management): those devices get **fixed** addresses.

---

## Mission 7 — DHCP on DIST2
👉 **Where:** DIST2, prompt `SWL3-HFA-A-DIST2#`.

DIST2 hands out `.131 – .254`, so it excludes `.1 – .130`. The pools are **identical** to DIST1.

⌨️ Type:
```
configure terminal
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
✅ `[OK]`.

---

## Mission 8 — Management address on the access switches
🧠 **Why:** access switches are Layer-2: they don't route. But we want to reach them over the
network (SSH in step 07), so each gets **one** IP address in the management VLAN 99, and a
gateway (like a PC).

### 8a — SW-HFA-A-ACC1
👉 ACC1 → CLI → log in → `SW-HFA-A-ACC1#`.

⌨️ Type:
```
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

### 8b — SW-HFA-A-ACC2
👉 ACC2 → CLI → log in → `SW-HFA-A-ACC2#`. The same, with **`.12`**:

⌨️ Type:
```
configure terminal
interface vlan 99
 description MANAGEMENT
 ip address 10.10.19.12 255.255.255.0
 no shutdown
 exit
ip default-gateway 10.10.19.1
end
copy running-config startup-config
```
🔍 **Check now:** PC-HFA-ADMIN-01 → Command Prompt → `ping 10.10.19.11` and `ping 10.10.19.12`.
✅ Both reply.

### What the new commands mean
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

## Mission 9 — Switch the user PCs to DHCP and test

### 9a — DHCP on the six user PCs (how? → M16)
For **each** of PC-HFA-ENG-01, ENG-02, ENG-03, FIN-01, FIN-02, FIN-03:
1. Click it → **Desktop** → **IP Configuration**.
2. Select **DHCP** at the top.
3. Wait for `DHCP request successful`.

✅ The fields fill in by themselves: an address `.11`–`.254`, mask `255.255.255.0`, gateway
`.1` of its VLAN, DNS `10.10.200.11`.
❌ `DHCP failed. APIPA is being used` → press **Fast Forward Time**, click **Static** and then
**DHCP** again.

Keep **static**: the printer (10.10.18.10, gw 10.10.18.1) and the admin PC (10.10.19.100,
gw 10.10.19.1).

### 9b — Check the leases
👉 On **DIST1**: `show ip dhcp binding` → ✅ addresses between .11 and .130 with the PCs' MACs.
👉 On **DIST2**: `show ip dhcp binding` → maybe some .131+ addresses (whoever answered first).

👉 On a PC → Command Prompt → `ipconfig /all` → ✅ for example:
```
   IPv4 Address......: 10.10.16.11
   Subnet Mask.......: 255.255.255.0
   Default Gateway...: 10.10.16.1
   DHCP Servers......: 10.10.16.2
   DNS Servers.......: 10.10.200.11
```

### 9c — Pings
Write down the address each PC got (hover over it). Then (how? → M17):

| # | From | Type | Expected |
|---|---|---|---|
| 1 | an ENG PC | `ping 10.10.16.1` | ✅ its gateway |
| 2 | an ENG PC | `ping` a FIN PC's address | ✅ **inter-VLAN routing works!** |
| 3 | an ENG PC | `ping 10.10.18.10` (printer) | ✅ |
| 4 | an ENG PC | `tracert` a FIN PC | first hop `10.10.16.2` (DIST1 — the Active for VLAN 10) |
| 5 | PC-HFA-ADMIN-01 | `ping 10.10.19.11` and `10.10.19.12` | ✅ the access switches' management IPs |

✅ All five as expected.

---

## Mission 10 — Failover test: kill a gateway, keep pinging
🧠 **Why:** prove that HSRP and split-scope DHCP really protect the users.

1. On an **ENG PC**: `ping -t` a **FIN PC's** address (continuous ping). Leave it running.
2. On **DIST1**, shut down its door into VLAN 10:
```
configure terminal
interface vlan 10
 shutdown
 end
```
3. Watch the ping.
   ✅ A few timeouts (up to 10 seconds — the HSRP hold time), then replies continue — DIST2
   took over `.1`.
4. On **DIST2**: `show standby brief`.
   ✅ Now **Active** for group 10 too.
5. On **another ENG PC** → Command Prompt: `ipconfig /release`, then `ipconfig /renew`.
   ✅ You get a **.131+** address — from DIST2, because DIST1 can't hear VLAN 10 now. Split
   scope works!
6. On **DIST1**, open the door again:
```
configure terminal
interface vlan 10
 no shutdown
 end
```
7. Wait a few seconds (Fast Forward Time), then on DIST1 `show standby brief`.
   ✅ DIST1 is **Active** again for group 10 — that's **preempt**.
8. Stop the ping with **Ctrl+C**.
9. Save all 4 switches (`copy running-config startup-config`) → **File → Save**.

---

## Checkpoint
- [ ] `show standby brief` correct on both DIST switches
- [ ] User PCs get DHCP addresses with gateway `.1`
- [ ] Engineering ↔ Finance ping works
- [ ] Failover test passed (ping survives DIST1's VLAN 10 gateway going down; preempt returns it)

## If something doesn't work
| Problem | Fix |
|---|---|
| SVI is `down/down` (`show ip interface brief`) | The VLAN doesn't exist on this switch or no port/trunk carries it. `show vlan brief`, `show interfaces trunk` |
| Pings between VLANs fail but each VLAN reaches its gateway | Forgot `ip routing` |
| Both switches say **Active** for the same group | `standby version` differs, or the VLAN isn't carried between them (check Po1 trunk allowed list) |
| PCs get `169.254.x.x` (APIPA) | No DHCP answer: pool `network` typo, all addresses excluded, or the PC's port is in the wrong VLAN. Press Fast Forward Time and retry |
| PC gets an address but no gateway | `default-router` missing in the pool |
| A typo in the virtual IP (e.g. `10.10.16.11`) | On that SVI: `no standby 10 ip 10.10.16.11` → `standby 10 ip 10.10.16.1` |

---

## 🏋 Exercises

### Exercise A — The wrong boss ⭐⭐
- 🎯 **Task:** explain why the HSRP Active of a VLAN should be the same switch as its STP root.
- 📁 **Start from:** paper (no Packet Tracer needed).
- 👣 **Steps:**
  1. Imagine DIST1 is HSRP Active for VLAN 20, **but** DIST2 stays STP root for VLAN 20.
  2. Draw ACC1, DIST1, DIST2 and Po1.
  3. Draw the path a FIN PC on ACC1 takes to reach its gateway. (Remember from step 04: on ACC1,
     VLAN 20 goes up through the port toward the STP root.)
- ✅ **Done when:** your drawing shows the path, and you can say in one sentence why it is worse.

<details><summary>💡 Hint</summary>

Which uplink of ACC1 forwards VLAN 20? Where does the frame arrive? Is that the switch that
answers for `.1`?
</details>
<details><summary>✔ Answer</summary>

PC → ACC1 → (STP forwards toward the root DIST2) → DIST2 → Po1 → DIST1 (HSRP Active) →
routed. The frame crosses the EtherChannel for nothing: an extra hop, and the inter-DIST link
carries traffic it shouldn't. Rule: **HSRP Active = STP root**, per VLAN.
</details>

### Exercise B — Who gave me this address? ⭐⭐
- 🎯 **Task:** prove which switch gave each PC its address.
- 📁 **Start from:** your `step05.pkt`.
- 👣 **Steps:**
  1. On every user PC run `ipconfig /all` and write down **IPv4 Address** and **DHCP Servers**.
  2. On DIST1 and DIST2 run `show ip dhcp binding`.
  3. Match each PC to a line in one of the two tables.
- ✅ **Done when:** you have a small table "PC → address → server" and every address is in the
  right half (.11–.130 from DIST1, .131–.254 from DIST2).

<details><summary>💡 Hint</summary>

`DHCP Servers` shows `.2` (DIST1) or `.3` (DIST2) of the PC's own VLAN. The binding table shows
the PC's MAC — compare it with `Physical Address` in `ipconfig /all`.
</details>
<details><summary>✔ Answer</summary>

Every PC whose `DHCP Servers` is `10.10.x.2` has an address .11–.130 and appears in DIST1's
binding table; every PC served by `10.10.x.3` has .131+ and appears in DIST2's. If one switch
answered all the PCs, that's fine — the other is the backup (you saw it work in Mission 10).
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

- [ ] Open `my-project/step04.pkt` → **Save As** `my-project/step05.pkt`
- [ ] `ip routing` + one SVI with HSRP on both distribution switches → `show standby brief` OK
- [ ] All the other SVIs; priorities per **your** "primary" column
- [ ] DHCP split scope on both switches (or wherever your plan part E puts DHCP)
- [ ] Management IP + default gateway on every access switch
- [ ] User PCs on DHCP; Mission 9 and Mission 10 tests pass

| In the example | In your project — take it from |
|---|---|
| SVIs `.2` / `.3`, virtual gateway `.1` | plan part D (your subnets, same `.1/.2/.3` rule) |
| HSRP group number = VLAN number | keep the rule, with **your** VLAN numbers |
| HSRP primary (priority + preempt) per VLAN | plan part D ("primary" column — same switch as the STP root in step 04) |
| DHCP pools `HFA-ENGINEERING`… and excluded addresses | your pool names `<SITE>-<DEPT>`, your subnets, your reserved range |
| DHCP on the L3 switches | plan part E (DHCP row) — the ministry wants DHCP on routers/L3 at one site and on a server at another (req. 22.11) |

**Check:** PCs set to DHCP get an address from **your** subnet with gateway `.1`; ping between
two of your departments works; shut the primary switch's SVI (`shutdown`) — ping keeps working.

💾 Save as `my-project/step05.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★2 What is an SVI and why do you use it? Show me where you configured one.**

<details><summary>Answer</summary>

Switch Virtual Interface — a virtual Layer-3 interface for a VLAN (`interface vlan 10`) on a Layer-3 switch. It is the gateway of that VLAN and routes between VLANs inside the switch, fast, without an external router.

**Show it:** `show ip interface brief`, `show running-config interface vlan 10`
</details>

**2. ★1 What is router-on-a-stick? What problem does it solve? Show its commands.**

<details><summary>Answer</summary>

One router interface connected to a trunk and split into sub-interfaces, one per VLAN (`interface g0/0.10` → `encapsulation dot1Q 10` → `ip address …`). It gives inter-VLAN routing with a single router port. **In this project inter-VLAN routing is done with SVIs on Layer-3 switches** (faster, no single-link bottleneck) — say so, and explain router-on-a-stick as the alternative. If your own project uses it somewhere, show it there.

**Show it:** Your SVIs: `show ip interface brief`
</details>

**3. ★1 What is HSRP? What are its advantages?**

<details><summary>Answer</summary>

Hot Standby Router Protocol (Cisco): two or more routers share one **virtual IP and virtual MAC**, which the PCs use as their gateway. If the active router fails, the standby takes over the virtual address — the PCs don't notice. Advantage: gateway redundancy without changing anything on the PCs.

**Show it:** `show standby brief`
</details>

**4. ★2 How does HSRP work? What are the Active and Standby routers?**

<details><summary>Answer</summary>

Routers in the same group send hellos. The **Active** router answers for the virtual IP/MAC and forwards the traffic. The **Standby** router listens to the active's hellos and takes over if they stop. The others stay in Listen.

**Show it:** `show standby brief` → State column
</details>

**5. ★2 What are the hello and hold times of HSRP? What happens when the active fails?**

<details><summary>Answer</summary>

Hello 3 s, hold 10 s. If the standby hears no hello for 10 s it becomes active, takes over the virtual IP and MAC, and a new standby is elected. PCs keep the same gateway.

**Show it:** Shut down the active switch (or its SVI) and ping continuously from a PC.
</details>

**6. ★2 How is the Active router chosen? How do you make a router the Active one?**

<details><summary>Answer</summary>

Highest priority (default 100) wins; on a tie, the highest IP address. Set `standby 10 priority 110` and `standby 10 preempt` on the router that should be active.

**Show it:** `show standby` → Priority, Preemption
</details>

**7. ★2 What does `preempt` mean?**

<details><summary>Answer</summary>

A router with a higher priority takes the active role back as soon as it comes up again. Without preempt, the router that is active stays active even if a better one returns.

**Show it:** `show standby` → "Preemption enabled"
</details>

**8. ★2 What are the HSRP states?**

<details><summary>Answer</summary>

Initial → Learn → Listen → Speak → Standby → Active.

**Show it:** `show standby`
</details>

**9. ★2 Which HSRP group do the routers in the branch belong to? What is a group?**

<details><summary>Answer</summary>

A group = the routers that share one virtual IP. In this project the group number equals the VLAN number (VLAN 10 → group 10), one group per VLAN.

**Show it:** `show standby brief` → Grp column
</details>

**10. ★2 What does `track` do and where do you configure it?**

<details><summary>Answer</summary>

It watches an interface (usually the uplink) and lowers the HSRP priority if it goes down, so the other router (with preempt) becomes active. It is configured on the active router, under the SVI: `standby 10 track <uplink> 20`. **The example doesn't use track** — explain it, and add it to your project if you want the gateway to follow the working uplink.

**Show it:** —
</details>

**11. ★2 If this router fails, what happens to routing out of the branch? Which protocol is responsible?**

<details><summary>Answer</summary>

HSRP moves the gateway to the other switch within the hold time (≤10 s), so the PCs keep sending. The routing protocol (OSPF/EIGRP) removes the failed router's links and reconverges the routes out of the branch.

**Show it:** `show standby brief`, `show ip route` after shutting the switch.
</details>

**12. ★2 What makes HSRP go from Active to Standby?**

<details><summary>Answer</summary>

The router fails or its interface goes down; a tracked interface lowers its priority below the other router's (with preempt); a router with higher priority and preempt comes up; or the priority is changed manually.

**Show it:** —
</details>

**13. ★3 How can HSRP provide redundancy for the LAN and for the WAN?**

<details><summary>Answer</summary>

LAN: two gateways for each VLAN — if one switch dies the other answers. WAN: track the WAN/uplink interface, so the active role moves to the router whose WAN link still works.

**Show it:** —
</details>

**14. ★3 How can you load-balance between the routers of an HSRP setup?**

<details><summary>Answer</summary>

Use several groups with different active routers. In this project DIST1 is active for some VLANs and DIST2 for others, so both switches forward traffic.

**Show it:** `show standby brief` on both distribution switches
</details>

**15. ★1 How do the computers in your network get their IP address?**

<details><summary>Answer</summary>

From DHCP: the PC broadcasts Discover → the server sends Offer → the PC sends Request → the server sends Ack (DORA). The PC gets IP, mask, gateway and DNS.

**Show it:** PC → IP Configuration → DHCP; `ipconfig /all`
</details>

**16. ★1 Show me the table of physical (MAC) address ↔ IP address that devices received from the DHCP server.**

<details><summary>Answer</summary>

On the device that is the DHCP server (here the Layer-3 switch): the binding table.

**Show it:** `show ip dhcp binding`
</details>

**17. ★2 Show me the DHCP lines in the running-config of a Cisco device and explain them.**

<details><summary>Answer</summary>

`ip dhcp excluded-address` (addresses DHCP must not give — gateways, servers), `ip dhcp pool NAME` → `network` (the subnet), `default-router` (the gateway `.1`), `dns-server`.

**Show it:** `show running-config` → scroll (Space) to the `ip dhcp` lines
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

In a copy of your finished step-05 file, delete the SVIs, the HSRP lines and the DHCP pools of **one**
VLAN. Without this page, rebuild them from your plan on both distribution switches.
Target: **10 minutes per VLAN**. Then show the examiner: a PC gets an address by DHCP,
`show standby brief`, and a continuous ping that survives shutting down the active gateway
(`interface vlan <x>` → `shutdown` on the Active switch).

---

## 👩‍🏫 Teacher notes
- **Lesson 1 (90 min):** why VLANs can't talk → SVIs as "doors into the router" → `ip routing`
  → Missions 2–3 (DIST1). Mission 2 deliberately does **one** VLAN and checks it before the rest
  — insist on the check.
- **Lesson 2 (90 min):** HSRP (two receptionists, one phone number) → Mission 4 (DIST2) →
  Mission 5 checks → Mission 10 failover on the projector.
- **Lesson 3 (60–90 min):** DHCP and DORA (show it in Simulation mode with the **DHCP**
  filter) → split scope → Missions 6–9 → students repeat Mission 10 including release/renew.
- **Common student questions**
  - *"Why does DHCP give `.1` and not `.2` as gateway?"* — `.2` is DIST1 itself; if DIST1 dies,
    `.2` dies. `.1` moves to whichever switch is alive.
  - *"Why two DHCP servers? Won't they conflict?"* — Not with split scope: they never give the
    same address.
  - *"Why do access switches need an IP?"* — Only to be managed remotely. They don't route.
- **Typical errors:** typo in the HSRP virtual IP on one switch (two different "phone
  numbers"); forgetting `no shutdown` on SVIs; the same priority on both (then the highest IP
  wins — unpredictable for students); copying DIST1's priorities onto DIST2 unchanged.
