# The real project — from your proposal to exam day

> 🏁 This is the path to the project you **really** submit and show the examiner: **your** company,
> **your** values. Do the 27 missions (P1–P27) in order. Tick each box only when its ✅ is true.

**The steps teach you. This page gets you finished.** Steps 00–16 teach each topic on the Intel
example. This page says what to do with that knowledge, in what order, what the ministry and your
own proposal demand **beyond** the example, and how you know you are done.

## What you must deliver

The ministry's guidelines (questionnaire 735918) ask for two things, then a practical exam:

| # | What | Details | Mission |
|---|---|---|---|
| 1 | **The simulation file** | Your company's network with 3 branches, built in Packet Tracer | P5–P19 |
| 2 | **The project book** (ספר הפרויקט) | **30% of the final grade.** Only what you **really built** in the simulator | P20 |
| 3 | **The practical exam** | Present your project, answer questions, explain **why** you built it this way, do a practical task | P21–P25 |

The practical task is set by the examiner. The guidelines say it can be **adding components, from
one department up to a whole branch**, with working communication, routing and security. The
examiner may also **cause a fault** and ask you to find and fix it. P22–P24 train exactly this.

## What "finished" means

⛔ **The Intel files in `my-work/` are practice. You never hand them in.** Each step builds only a piece.
Your project is finished only when `final.pkt` holds **everything** in these two tables, P19 is fully
ticked, and P26 passes.

**1. Count your devices.** These are the numbers when you follow the steps and P11. A different
number is fine only if your teacher approved it in your plan. Departments and PCs may never be fewer.

| Device | Site 1 | Site 2 | Site 3 | ISPs | Total |
|---|---|---|---|---|---|
| Routers | 1 | 1 | 1 | 6 | 9 |
| ASA firewall | 1 | — | — | — | 1 |
| Layer-3 switches | 8 | 2 | 2 | — | 12 |
| Layer-2 switches | 8 | 4 | 4 | 2 | 18 |
| Access points | 2 | 2 | 2 | — | 6 |
| Servers | 8 | 3 | 4 | 4 | 19 |
| Department PCs (6 × 8) | 48 | 48 | 48 | — | 144 |
| Other PCs | 1 admin | 1 admin | 1 admin, 2 IPv4 | 1 home | 6 |
| Laptops | 3 | 2 | 2 | — | 7 |
| Printers | 3 | 3 | 2 | — | 8 |
| DSL modem and cloud | — | — | — | 1 + 1 | 2 |

**2. Check what runs where.**

| What | Site 1 | Site 2 | Site 3 |
|---|---|---|---|
| Layout | 3 buildings, access–distribution–core | 3 floors, collapsed core | 3 access switches, 2 L3 switches |
| Addresses | IPv4 | IPv4 | IPv6, plus one small IPv4 VLAN |
| Routing | OSPF areas, virtual link, MD5 | EIGRP, summary, MD5 | OSPFv3 |
| Gateway redundancy | HSRP | HSRP | HSRP for IPv6 |
| EtherChannel | LACP, plus core (P13) | PAgP | — |
| DHCP | On the L3 switches | Dedicated server + relay | SLAAC + stateless DHCPv6 |
| NAT | Static + PAT | Dynamic | Overload |
| Firewall | ASA, inside/outside/DMZ | CBAC | IPv6 filter on the edge |
| Servers | 6 DMZ + AAA + Syslog | DHCP, AAA, Syslog | DNS, Mail, AAA, Syslog |
| WiFi | Employees + guests | Employees (P11) | Employees (P11) |
| Internet link | ISP + DSL backup | ISP | ISP |
| VPN | Ends both tunnels + remote access | Site-to-site | Site-to-site (P14) |
| Layer-2 protection | Port security, snooping, DAI | Port security, snooping, DAI | Port security |
| Management | SSH, AAA, management VLAN | SSH, AAA, management VLAN | SSH, AAA, management VLAN |
| Time and logs | Main NTP, Syslog | Branch NTP, Syslog | Branch NTP, Syslog |

**Between the sites:** 3 ISPs with BGP between them, OSPF inside each ISP, ISP DNS and web servers.

## The rules

- ⛔ Two students may not submit the same topic.
- ⛔ You may not copy one branch to the others. Each branch uses different features.
- ⛔ No Intel values in your project: no `Intel#…` password, no `10.10.x.x` from the example, no
  `HFA`, `JLM` or `KG`. P17 checks this.
- ⛔ The book describes only what is really in your file. If you did not build it, it is not in the book.

## Your folders

```
my-project/
  plan.md            your plan (P3)
  progress.md        your progress table
  step02.pkt …       one saved file per step
  final.pkt          the project you submit
  screenshots/       every test screenshot (P18)
```

> ⚠️ **Keep the book outside the course folder.** Its cover page has your ID number, and this
> folder may be shared. The same goes for your proposal form.

## The signs on this page

`<ANGLE BRACKETS>` = a value from **your** plan. Never type the brackets.
Example: if your plan says the VLAN is 60, then `vlan <V>` means you type `vlan 60`.

| Word | Means |
|---|---|
| **Site 1** | Your main branch (HQ): 3-tier, OSPF, ASA, DMZ. In the example: Haifa |
| **Site 2** | Your collapsed-core branch: EIGRP, DHCP server. In the example: Jerusalem |
| **Site 3** | Your IPv6 branch: OSPFv3. In the example: Kiryat Gat |

## All the missions

**Part A — Lock the design**
- [ ] **P1** — Check your proposal against the ministry
- [ ] **P2** — Match your proposal to what you will build
- [ ] **P3** — Fill in your plan and check it
- [ ] **P4** — Get your plan approved

**Part B — Build the network, step by step**
- [ ] **P5** — Set up your file and your working routine
- [ ] **P6** — Site 1 LAN (steps 02–09)
- [ ] **P7** — WiFi, firewall, internet (steps 10–12)
- [ ] **P8** — Site 2 (step 13)
- [ ] **P9** — Site 3 (step 14)
- [ ] **P10** — VPN (step 15)

**Part C — What the example doesn't have, and you must**
- [ ] **P11** — Access points in sites 2 and 3
- [ ] **P12** — Dynamic ARP Inspection
- [ ] **P13** — EtherChannel between your core switches
- [ ] **P14** — Site-to-site VPN to site 3

**Part D — Finish and prove**
- [ ] **P15** — Scale to 8 PCs per department
- [ ] **P16** — Email
- [ ] **P17** — Hardening sweep, and no Intel left
- [ ] **P18** — The tests, with screenshots
- [ ] **P19** — Prove every ministry requirement

**Part E — The book**
- [ ] **P20** — Write the project book

**Part F — Rehearse the exam**
- [ ] **P21** — Your opening presentation
- [ ] **P22** — Practical task: add a department
- [ ] **P23** — Practical task: add a branch
- [ ] **P24** — Practical task: find the fault
- [ ] **P25** — The questions

**Part G — Submit**
- [ ] **P26** — Freeze the file and do the cold start
- [ ] **P27** — Hand in, and exam day

---

# Part A — Lock the design

## P1 — Check your proposal against the ministry
🧠 **Why:** your proposal is what you promised. Every problem found now saves weeks later.

👣 **Steps.** First do [step 00](../steps/00-getting-started), including its **🛠 Your own project** list.
Then open your proposal and tick each box:

- [ ] 3 branches in different places, one defined as the **main** branch
- [ ] **6 different departments** in **every** branch, at least **8 PCs** in each
- [ ] One branch is collapsed core, one branch is IPv6, and no branch is a copy of another
- [ ] Each branch is 3 buildings, 3 floors, or one large floor
- [ ] All three NAT types are there: static, dynamic, overload, one per branch

✅ **Done when:** every box is ticked. If one isn't, tell your teacher before you go on.

---

## P2 — Match your proposal to what you will build
🧠 **Why:** the proposals promise some things the Intel example doesn't have. The examiner compares
your book with your file. Nothing you promised may silently disappear.

👣 **Steps.** **Part H** of your plan already lists these six rows. The middle column below is already
decided for the standard proposal. Check each line against **your own** proposal, then fill in part H.

| What the proposal says | What you do | Why |
|---|---|---|
| Access points in every branch ("נקודות אלחוט") | **Build it:** P11 | Ministry 20.6: every branch includes access points. The example has them only in Haifa |
| Dynamic ARP Inspection ("Dynamic ARP Inspection") | **Build it:** P12 | You promised it |
| EtherChannel between the core switches ("EtherChannel בין מתגי הליבה") | **Build it:** P13 | You promised it. The example has it between distribution switches |
| Site 3 has a site-to-site VPN to the main branch | **Build it:** P14, for the small IPv4 VLAN | Ministry 21: the DMZ must be reachable through VPN from the other branches |
| Site 3 uses a dedicated DHCP server ("DHCP על שרת ייעודי") | **Don't build. Your teacher approved this change:** SLAAC + stateless DHCPv6 on the two L3 switches | Ministry 22.11 lets you choose for the third branch. The server would sit in another VLAN, and Packet Tracer (checked in 9.0.1) has no `ipv6 dhcp relay` command to reach it |
| An eBGP connection from every branch, and a double BGP connection at the main branch ("חיבור eBGP מכל סניף") | **Don't build. Your teacher approved this change:** static routes from each branch to its ISP. BGP runs only **between the ISPs** | Ministry 22.17: routing from a branch to the WAN must be static. The same proposal says this too, in its WAN section. The main branch's second connection is the DSL backup |

✍️ **Your job:** your teacher already approved the two "Don't build" changes, so write them in
chapter 6 of your book. If your proposal has any other line that isn't in the steps, add it to
the table and ask your teacher.

> 💡 The proposal's background also lists company systems, such as cameras or cloud services. That
> describes the company, and it isn't a promise to build them. Don't write them in the book as built.

✅ **Done when:** part H of your plan is filled in and every line has a decision.

---

## P3 — Fill in your plan and check it
🧠 **Why:** every value you type for the next months comes from this plan. A mistake here is a
mistake everywhere.

👣 **Steps:**
1. Do the **🛠 Your own project** section of [step 01](../steps/01-plan-the-network) (copy the template
   to `my-project/plan.md`, fill in parts A–G).
2. Fill in **part I** (the extra values), the last part of the template.
3. Check the plan against this list. Tick each one:

**Addresses**
- [ ] All IPv4 blocks come from one master block, and the sites are consecutive
- [ ] No two blocks overlap
- [ ] Site 1 has a block for area 0 and one /20 for each of 3 buildings
- [ ] Every VLAN of a building sits **inside** that building's /20
- [ ] Links and loopbacks sit in the area-0 block, not in a building's block
- [ ] Site 2's 6 departments fit in **one** summary. A /21 covers 8 /24s, and it must start at a
      multiple of 8 (like `.8.0` or `.16.0`, not `.10.0`). Check: in the example, `10.20.10.0`–`.15.0`
      fit in `10.20.8.0/21`
- [ ] Every VLAN has a gateway `.1` and real addresses `.2` / `.3`
- [ ] Site 3 has a /48, one /64 per VLAN, and only one small IPv4 VLAN (for NAT overload)

**The virtual link**
- [ ] One of your site 1 buildings connects **only** to another building, not to the core. Without
      this there is no reason for a virtual link, and the ministry requires one (req. 8, 22.16)

**VLANs and names**
- [ ] A VLAN table for each site: 6 departments + MGMT + NATIVE + PARKING + what that site needs
      (printers, servers, WiFi)
- [ ] No device, PC or port is in VLAN 1. Your VLAN numbers are not 1 and not 1002–1005
- [ ] For each VLAN, one switch is primary, and that switch is both the STP root and the HSRP active
- [ ] Every name follows your pattern and shows the device type and its place (req. 22.1)

**Passwords and text**
- [ ] None of your passwords is an `Intel#…` password
- [ ] The banner text is in English and has no `#`
- [ ] You can read every password without guessing (`I` or `l`? `0` or `O`?)

4. Redo exercises 3.1–3.4 of step 01 with **your** blocks.

✅ **Done when:** every box is ticked and parts A–I are full.

---

## P4 — Get your plan approved
Show your teacher `my-project/plan.md`, including part H.

✅ **Done when:** the approval line at the bottom of the plan is signed. **Don't start P5 before this.**

---

# Part B — Build the network, step by step

## P5 — Set up your file and your working routine
🧠 **Why:** a fixed routine stops the two usual disasters: a lost file and an unnoticed mistake.

**The routine, for every step from 02 to 15:**
1. Do the step on the **Intel example** in `my-work/`, and pass its 🧪 self-test.
2. Open your previous file → **Save As** `my-project/stepNN.pkt`.
3. Do the step's **🛠 Your own project** list, with values from your plan.
4. Read that step's traps in P6–P10 below **before** you type.
5. Run the step's **🧪 self-test again**, with your values (its last table tells you what to swap).
6. Save: `copy running-config startup-config` on every device you changed, then **File → Save**.
7. Tick columns 2 and 3 of `my-project/progress.md`.
8. Copy every table you made in this step into the end of part I of your plan. Your book needs them.
9. Write this step's topics in your book now, from the table in P20. Leaving the book to the end is
   the most common way to fail the deadline.

> ⚠️ Never go on with a failing test. Fix it now: a mistake found after 2 minutes takes 2 minutes.

> ⚠️ **Packet Tracer version.** The course uses 9.x. A file saved in a newer version doesn't open in
> an older one, so check early that the exam computer has 9.x too, the same or newer than yours.

✅ **Done when:** `my-project/step01.pkt` opens, and you know the routine.

---

## P6 — Site 1 LAN (steps 02–09)
Follow the routine in P5, one step at a time. These are the traps that appear when you change the values.

| Step | Trap in your own project |
|---|---|
| 02 | The banner ends at the first `#`. If your text contains a `#`, the banner breaks |
| 03 | The trunk's allowed list must include **every** VLAN of that building **and** the native VLAN. A VLAN you forget works nowhere, with no error message |
| 03 | A VTP domain name is case-sensitive. Each building has its own domain |
| 04 | STP root and HSRP active must be the **same** switch for each VLAN (your plan's "primary" column) |
| 05 | DIST1 and DIST2 hand out **different** ranges of each subnet. Overlapping ranges cause duplicate addresses |
| 05 | The DNS server doesn't exist yet. Put your planned DMZ DNS address in the pools now, so you don't have to change them all in P7 |
| 06 | `ip dhcp snooping vlan` must list your own VLANs. `ip dhcp snooping trust` goes on the uplinks only. Do Mission 6e: remove the rogue server's MAC |
| 07 | The ACL mask is a **wildcard**: for a /24 it is `0.0.0.255`, not `255.255.255.0` |
| 08 | `network` and `area … range` use your /20. `network` takes a wildcard (`0.0.15.255`), `area … range` takes a mask (`255.255.240.0`). Mixing them is the most common mistake here |
| 09 | `area <transit> virtual-link` takes the other side's **router-ID**, not an interface address. Build it on **all four** ABRs |
| 09 | `area … range` goes on **all four** ABRs too. Add your new devices to the AAA server and to every `MGMT-ONLY` list |

✅ **Done when:** step 09's self-test passes with your values, and `my-project/step09.pkt` is saved.

---

## P7 — WiFi, firewall, internet (steps 10–12)
| Step | Trap in your own project |
|---|---|
| 10 | The SSID and key are case-sensitive, on the AP and on each laptop |
| 10 | `GUEST-IN` keeps its 3 private-range `deny` lines as they are (10/8, 172.16/12, 192.168/16). Only the DNS server address changes. Apply it `in` on **both** gateways |
| 11 | Check the ASA's ports: 1 = outside, 2 = inside, 3 = DMZ |
| 11 | The ASA's route to your inside block uses your site 1 /16, and a **mask**, not a wildcard. The ASA's ACLs also use masks |
| 11 | Every server has **only its own** service on. All 6 DMZ servers are required (DNS, Email, Web, NTP, FTP, AAA) |
| 12 | BGP's `network … mask` must **exactly** match a route. That is why each ISP has a route to `null0` |
| 12 | The ISP's DNS record for your website holds your **public** address, not the private one |
| 12 | The public block for static NAT must be routed to you by your ISP (`ip route` + `redistribute static subnets`) |

✅ **Done when:** step 12's self-test passes, and an inside PC opens the ISP's web pages.

---

## P8 — Site 2 (step 13)
| Trap in your own project |
|---|
| This site must not look like site 1: collapsed core, EIGRP, PAgP, DHCP server, dynamic NAT, CBAC |
| `ip helper-address` goes on **every** user SVI, on **both** collapsed-core switches |
| On the DHCP server, click **Add** for a new pool. **Save** overwrites the selected pool |
| `ip summary-address eigrp` uses your /21 (or whatever your plan's summary is), on the uplink of **both** switches |
| `WAN-IN` uses **your** public addresses. Don't leave the example's lines |
| The NAT pool holds your own public addresses. They must be routed to you by the ISP |

✅ **Done when:** step 13's self-test passes.

---

## P9 — Site 3 (step 14)
| Trap in your own project |
|---|
| VTP is **transparent** here: create each VLAN on **each** switch that uses it |
| A long IPv6 address is easy to mistype. Use one simple rule, like "the VLAN number is the 4th group" |
| Servers and the admin PC have a static gateway `FE80::1`. If your Packet Tracer refuses it, see the step's troubleshooting table |
| The edge router needs IPv6 routes **and** IPv4 routes, because the small IPv4 VLAN uses PAT |
| Your ISP needs a route to your /48. Without it, replies never come back |

✅ **Done when:** step 14's self-test passes.

---

## P10 — VPN (step 15)
| Trap in your own project |
|---|
| The licence needs a **reload** on each edge router. `copy running-config startup-config` first |
| The two sides are mirrors. Draw your own table like section 1.7 of the step before you type |
| Policy, key and transform set must be **identical** on both sides. Compare letter by letter |
| Don't forget the ASA lines for site 2 and for the remote users |
| `NAT-…` on the site 1 edge must **skip** your remote-access pool (the `deny` line) |
| `dst` and `src` in `show crypto isakmp sa` may be swapped. That's fine |

✅ **Done when:** step 15's self-test passes, and `my-project/step15.pkt` is saved. Save a copy now:
**File → Save As** `my-project/step15-backup.pkt`. Then open `step15.pkt` again. Part C changes your
network, and this copy is the way back if something goes wrong.

---

# Part C — What the example doesn't have, and you must

Everything here is new: no step teaches it click by click. Read each mission to the end before you
start. Do it on `my-project/step15.pkt`, and **File → Save** after each mission.

> ⚠️ The commands of P12–P14 were checked against the command list of Packet Tracer 9.0.1, but
> they were not run on a full network. If Packet Tracer refuses a command, **stop**, keep your last
> saved file, and tell your teacher. Then record the change in part H and in chapter 6 of your book.

## Recipe — add a new VLAN to a site
🧠 You need this recipe twice: in P11 (a WiFi VLAN) and in P22 (a new department at the exam).

**Before you start, choose and write down:** the VLAN number `<V>`, its name `<NAME>`, and its
subnet. `<NET>` is the first 3 numbers of the subnet (for `10.50.33.0/24`, `<NET>` is `10.50.33`).
The subnet must sit inside the block of that building or site. Then no routing change is needed.

**The other signs:**

| Sign | Means | Take it from |
|---|---|---|
| `<TRUNKS>` on an L3 switch | The trunk ports to the access switches, and in site 3 also the link to the other DIST. **Not** the port-channel members: the `interface port-channel 1` line covers them | Your cabling table |
| `<TRUNKS>` on an access switch | Its uplink ports | Your cabling table |
| `<PORTS>` | Free user ports on that access switch | Your port plan |
| `<SITE>` | The site code | Plan part A |
| `<DNS>` | The DNS address your other pools use | Plan part D |
| `<DHCP-SERVER>` | Site 2's DHCP server address | Plan part D |
| `<PREFIX>` | The first 4 groups of the new /64, like `2001:DB8:30:60` | Plan part C |
| `<V6-POOL>` | The name of your stateless DHCPv6 pool from step 14 | Your step 14 config |

### Recipe A — site 1 (3-tier)
👉 **Where:** the building's DIST1 (the VTP server, and the primary for this VLAN).
```
configure terminal
vlan <V>
 name <NAME>
 exit
interface range <TRUNKS>
 switchport trunk allowed vlan add <V>
interface port-channel 1
 switchport trunk allowed vlan add <V>
 exit
spanning-tree vlan <V> root primary
interface vlan <V>
 description GW <NAME>
 ip address <NET>.2 255.255.255.0
 standby version 2
 standby <V> ip <NET>.1
 standby <V> priority 110
 standby <V> preempt
 no shutdown
 exit
ip dhcp excluded-address <NET>.1 <NET>.10
ip dhcp excluded-address <NET>.131 <NET>.254
ip dhcp pool <SITE>-<NAME>
 network <NET>.0 255.255.255.0
 default-router <NET>.1
 dns-server <DNS>
 end
copy running-config startup-config
```
👉 **Where:** DIST2. The same, with these differences: **no** `vlan` lines (VTP brings the VLAN),
`root secondary`, address `<NET>.3`, `priority 90`, and only one exclusion:
`ip dhcp excluded-address <NET>.1 <NET>.130`.

Then do **"On each access switch"** below.

### Recipe B — site 2 (collapsed core)
👉 **Where:** CC1 (the VTP server).
```
configure terminal
vlan <V>
 name <NAME>
 exit
interface range <TRUNKS>
 switchport trunk allowed vlan add <V>
interface port-channel 1
 switchport trunk allowed vlan add <V>
 exit
spanning-tree vlan <V> root primary
interface vlan <V>
 description GW <NAME>
 ip address <NET>.2 255.255.255.0
 ip helper-address <DHCP-SERVER>
 standby version 2
 standby <V> ip <NET>.1
 standby <V> priority 110
 standby <V> preempt
 no shutdown
 end
copy running-config startup-config
```
👉 **Where:** CC2. The same, with these differences: **no** `vlan` lines, `root secondary`, address
`<NET>.3`, `priority 90`.

👉 **Where:** your DHCP server → **Services** → **DHCP**. Fill in Pool Name, Default Gateway
`<NET>.1`, DNS Server, Start IP Address `<NET>.11`, Subnet Mask, Max Users `240` → click **Add**.

Then do **"On each access switch"** below.

### On each access switch that has this VLAN (recipes A and B)
Pick free ports. `<PORTS>` is an example like `fa0/23 - 24`. They are parked, so this also wakes them up.
```
configure terminal
interface range <TRUNKS>
 switchport trunk allowed vlan add <V>
interface range <PORTS>
 description <NAME> users
 switchport mode access
 switchport access vlan <V>
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 ip dhcp snooping limit rate 10
 no cdp enable
 no shutdown
 exit
ip dhcp snooping vlan <V>
 end
copy running-config startup-config
```
> There is no `vlan` line here on purpose. These switches are VTP clients, so the VLAN arrives by
> itself from the VTP server. Check with `show vlan brief` before you type.

### Recipe C — site 3 (IPv6)
VTP is transparent here, so **each** switch needs the VLAN. There is no DHCP snooping.

👉 **Where:** DIST1.
```
configure terminal
vlan <V>
 name <NAME>
 exit
interface range <TRUNKS>
 switchport trunk allowed vlan add <V>
 exit
spanning-tree vlan <V> root primary
interface vlan <V>
 description GW <NAME>
 ipv6 address <PREFIX>::2/64
 ipv6 ospf 1 area 0
 ipv6 nd other-config-flag
 ipv6 dhcp server <V6-POOL>
 standby version 2
 standby <V> ipv6 FE80::1
 standby <V> priority 110
 standby <V> preempt
 no shutdown
 end
copy running-config startup-config
```
👉 **Where:** DIST2. The same, with these differences: `root secondary`, address `<PREFIX>::3/64`,
`priority 90`.

👉 **Where:** each access switch that has this VLAN.
```
configure terminal
vlan <V>
 name <NAME>
 exit
interface range <TRUNKS>
 switchport trunk allowed vlan add <V>
interface range <PORTS>
 description <NAME> users
 switchport mode access
 switchport access vlan <V>
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no cdp enable
 no shutdown
 end
copy running-config startup-config
```

### Check the recipe
| # | On | Type / do | Must see | If not |
|---|---|---|---|---|
| 1 | Both L3 switches | `show standby brief` | The new VLAN: Active on the first, Standby on the second | `show interfaces trunk`: is `<V>` allowed and active on the port-channel? |
| 2 | Access switch | `show vlan brief` | Your ports in `<V>` | Recipe, "On each access switch" |
| 3 | Access switch | `show interfaces trunk` | `<V>` in the allowed list | `switchport trunk allowed vlan add <V>` |
| 4 | A new PC on one of those ports | DHCP (sites 1, 2) or IPv6 Automatic (site 3) | An address in the new subnet, gateway `.1` or `FE80::1` | Rows 1–3. Sites 1–2: `show ip dhcp snooping` must list `<V>` |
| 5 | That PC | `ping` a server in another VLAN | ✅ replies | Routing: is the subnet inside the planned block? |

---

## P11 — Access points in sites 2 and 3
🧠 **Why:** ministry requirement 20.6 says **every** branch includes access points. Guests remain
only in site 1 (req. 18). In sites 2 and 3, the access points are for employees.

**You place:** a WiFi VLAN, 2 access points, 2 laptops, in **each** of the two sites.

### P11a — The WiFi VLAN
On the L3 switches, do the **Recipe** with `<NAME>` = `WIFI`: recipe **B** for site 2 (CC1, CC2 and the
DHCP server), recipe **C** for site 3 (DIST1 and DIST2).

**Don't** do the recipe's access-switch block, because AP ports are set up differently (P11c). Type this:

👉 **Where:** site 2, each access switch with an AP.
```
configure terminal
interface range <TRUNKS>
 switchport trunk allowed vlan add <V>
 exit
ip dhcp snooping vlan <V>
 end
copy running-config startup-config
```

👉 **Where:** site 3, each access switch with an AP.
```
configure terminal
vlan <V>
 name <NAME>
 exit
interface range <TRUNKS>
 switchport trunk allowed vlan add <V>
 end
copy running-config startup-config
```

### P11b — Place and cable
1. **Network Devices** → **Wireless Devices** → drag **AccessPoint-PT**, twice, one near each of two
   access switches. Rename them with your naming pattern (M4).
2. Cable with **Copper Straight-Through**: AP **Port 0** → a free port on the switch (M5).
3. Drag 2 **Laptop-PT**. Swap the card to **WPC300N**: power off → remove the Ethernet module →
   drag **WPC300N** in → power on. Same as step 10, Mission 3.

### P11c — The AP ports
👉 **Where:** each access switch with an AP. `<AP-PORT>` is the port you just cabled.
```
configure terminal
interface <AP-PORT>
 description <AP name>
 switchport mode access
 switchport access vlan <V>
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 30
 switchport port-security violation restrict
 no cdp enable
 no shutdown
 end
copy running-config startup-config
```
On **site 2** only, add this under the same interface, before `end`: `ip dhcp snooping limit rate 20`.

### P11d — Security
On each AP → **Config** → **Port 1**, and on each laptop → **Config** → **Wireless0**:

| Field | Value |
|---|---|
| SSID | Your SSID for that site (plan part I) |
| 2.4 GHz Channel (AP only) | `1` on the first AP, `6` on the second |
| Authentication | **WPA2-PSK** |
| PSK Pass Phrase | Your key (plan part F) |
| Encryption Type | **AES** |
| IP Configuration (laptop only) | Site 2: **DHCP**. Site 3: IPv6 **Automatic** |

### P11e — Physical view
In the **Physical** tab, drag the APs and laptops into their building, next to each other (M26).

| # | From | Type | Must see | If not |
|---|---|---|---|---|
| 1 | Workspace | Look | A dashed radio line from each laptop to an AP | SSID, key or encryption differs by one letter |
| 2 | Site 2 laptop | `ipconfig` | An address in the WiFi subnet, gateway `.1` | Recipe check, rows 1–4 |
| 3 | Site 3 laptop | `ipconfig` | An address in your WiFi prefix, gateway `FE80::1` | Recipe check, rows 1–4 |
| 4 | Each laptop | `ping` a server of its own site | ✅ replies | Recipe check, row 5 |

✅ **Done when:** all 4 rows pass. Write the new devices into part G of your plan and your book.

---

## P12 — Dynamic ARP Inspection
🧠 **Why:** your proposal promises it. DAI drops fake ARP answers (ARP poisoning). It checks each
ARP packet against the DHCP snooping binding table.

**Where it applies:** the access switches of sites 1 and 2. Site 3 has no ARP on its user VLANs
(IPv6 uses Neighbor Discovery), so write that in your book as the reason it isn't there.

**Which VLANs:** only department VLANs, because only devices with a DHCP address are in the binding
table. **Not** the MGMT VLAN or the printer VLAN: their static devices would be cut off.

👣 **Steps:** do the exercise at the end of [step 06](../steps/06-layer2-security) on the example first. Then:

1. Every PC of those VLANs on that switch must use **DHCP**. Check:
   `show ip dhcp snooping binding` shows each PC.
2. 👉 **Where:** each access switch. `<DEPT-VLANS>` = the department VLANs of that switch.
   ```
   configure terminal
   ip arp inspection vlan <DEPT-VLANS>
   interface range <TRUNKS>
    ip arp inspection trust
    end
   copy running-config startup-config
   ```

| # | On | Type / do | Must see | If not |
|---|---|---|---|---|
| 1 | Access switch | `show ip arp inspection` | Your VLANs enabled | Step 2 |
| 2 | A PC | `ping` its gateway | ✅ replies | Uplinks not trusted, or the PC isn't in the binding table → `ipconfig /renew` |
| 3 | The same PC → IP Configuration | **Static**, a free address in its subnet, then `ping` the gateway | ❌ fails — **correct!** The address isn't in the binding table | DAI isn't on for that VLAN |
| 4 | The same PC | Back to **DHCP** → `ping` the gateway | ✅ replies again | Do it again, then `ipconfig /renew` |

> ⚠️ **Do P26's cold start after this.** The binding table may be empty after you reopen the file.
> Then the PCs lose their network until `ipconfig /renew`. If that happens, tell your teacher before
> the exam, and agree what to do.

✅ **Done when:** all 4 rows pass on one switch, and DAI is on every access switch of sites 1 and 2.

---

## P13 — EtherChannel between your core switches
🧠 **Why:** your proposal promises it. The two core switches have one routed link. You turn it into a
bundle of two, so a cable can fail with no break in OSPF.

This bundle is **Layer 3**: it has an IP address, like a routed port. In the example the link is
Gi1/0/23, so the bundle uses Gi1/0/22 and Gi1/0/23. Use your own ports, and make sure the second port is free.

**Save first:** **File → Save**.

### P13a — Add the second cable
**Copper Cross-Over** between the two core switches, on the second port (M5).

### P13b — CORE1
👉 **Where:** your CORE1. `<LINK-IP1>` is CORE1's address on the core-to-core /30 (plan).
```
configure terminal
interface gi1/0/23
 no ip address
 no ip ospf authentication
 no ip ospf message-digest-key 1 md5 <OSPF-KEY>
 exit
interface port-channel 2
 description EtherChannel to CORE2
 no switchport
 ip address <LINK-IP1> 255.255.255.252
 ip ospf message-digest-key 1 md5 <OSPF-KEY>
 ip ospf authentication message-digest
 no shutdown
 exit
interface range gi1/0/22 - 23
 no switchport
 channel-group 2 mode active
 no shutdown
 exit
router ospf 1
 passive-interface gi1/0/23
 no passive-interface port-channel 2
 end
copy running-config startup-config
```

### P13c — CORE2
The same, with `<LINK-IP2>` (CORE2's address) and the description `to CORE1`.

Press **Fast Forward Time** a few times.

| # | On | Type | Must see | If not |
|---|---|---|---|---|
| 1 | Both cores | `show etherchannel summary` | `Po2(RU)`, both members `(P)`. **R** means Layer 3 | `(SD)` / `(I)`: `no switchport` missing on a member, or the other side isn't done |
| 2 | Both cores | `show ip ospf neighbor` | The other core is `FULL`, on `Port-channel2` | MD5 key, or `no passive-interface port-channel 2` |
| 3 | Workspace | Delete one of the two cables, then `show ip ospf neighbor` | Still `FULL` | Row 1 |
| 4 | Repair | Put the cable back. Fast Forward Time | Row 1 matches again | Same port number on both sides? |

❌ **Packet Tracer refuses `no switchport` on the port-channel, or the bundle never comes up:**
**File → Open** your saved file (answer **No** to saving). Your EtherChannel at the distribution
layer already meets ministry requirement 22.14. Tell your teacher, and record the change in part H.

✅ **Done when:** all 4 rows pass, or your teacher approved the change.

---

## P14 — Site-to-site VPN to site 3
🧠 **Why:** your proposal promises it, and ministry requirement 21 wants the DMZ reachable through VPN
from the other branches.

**What it can and can't do.** The tunnel carries **IPv4**. Your small IPv4 VLAN reaches the DMZ through
it. The IPv6 PCs can't, because the DMZ is IPv4 and Packet Tracer has no NAT64. Write this in
chapter 6 of your book: it shows you understand your network.

**Values, from plan part I:**

| Value | Means | In an example like Intel's |
|---|---|---|
| `<PUB1>` | Site 1 edge's public address | 200.1.1.2 |
| `<PUB3>` | Site 3 edge's public address | 200.3.1.2 |
| `<T3-1>` / `<T3-3>` | The two ends of the new tunnel /30 | 172.16.13.1 / .2 |
| `<LEGACY>` | The IPv4 VLAN's network | 10.30.199.0 |
| `<HQ-NET>` / `<HQ-MASK>` | Site 1's block | 10.10.0.0 / 255.255.0.0 |
| `<KEY3>` | A new pre-shared key | plan part F |

### P14a — Licence on the site 3 edge
Do step 15, Mission 3, on your site 3 edge router (the licence, save, reload).

### P14b — Site 1 edge
👉 **Where:** your site 1 edge router.
```
configure terminal
interface tunnel 1
 description GRE to site 3 (encrypted by IPsec)
 ip address <T3-1> 255.255.255.252
 tunnel source gi0/1
 tunnel destination <PUB3>
 exit
ip route <LEGACY> 255.255.255.0 <T3-3>
crypto isakmp key <KEY3> address <PUB3> no-xauth
ip access-list extended S2S-GRE-3
 permit gre host <PUB1> host <PUB3>
 exit
crypto map VPN-MAP 20 ipsec-isakmp
 set peer <PUB3>
 set transform-set S2S-SET
 match address S2S-GRE-3
 end
copy running-config startup-config
```
> Sequence **20** goes between site 2 (10) and the remote users (65535). The crypto map is already on
> Gi0/1, so don't apply it again. Use your own name if yours isn't `VPN-MAP` or `S2S-SET`.

### P14c — Site 3 edge (the mirror)
👉 **Where:** your site 3 edge router.
```
configure terminal
interface tunnel 0
 description GRE to site 1 (encrypted by IPsec)
 ip address <T3-3> 255.255.255.252
 tunnel source gi0/1
 tunnel destination <PUB1>
 exit
ip route <HQ-NET> <HQ-MASK> <T3-1>
crypto isakmp policy 10
 encryption aes 256
 hash sha
 authentication pre-share
 group 5
 lifetime 86400
 exit
crypto isakmp key <KEY3> address <PUB1> no-xauth
crypto ipsec transform-set S2S-SET esp-aes 256 esp-sha-hmac
ip access-list extended S2S-GRE
 permit gre host <PUB3> host <PUB1>
 exit
crypto map VPN-MAP 10 ipsec-isakmp
 set peer <PUB1>
 set transform-set S2S-SET
 match address S2S-GRE
 exit
interface gi0/1
 crypto map VPN-MAP
 end
copy running-config startup-config
```

### P14d — The ASA hole
👉 **Where:** the ASA.
```
configure terminal
access-list OUTSIDE_IN extended permit ip <LEGACY> 255.255.255.0 <HQ-NET> <HQ-MASK>
end
write memory
```

| # | From | Type | Must see | If not |
|---|---|---|---|---|
| 1 | Site 3 edge | `ping <T3-1>` | `!!!!!` (the first may fail) | Routes, `tunnel destination` |
| 2 | Site 1 edge | `show crypto isakmp sa` | Lines for **both** branches, `QM_IDLE` | Policy or key differs. Compare letter by letter |
| 3 | The IPv4 PC of site 3 | `ping` your DMZ web server's private address | ✅ replies (the first may time out). **Private address, through the VPN** | The ASA line (P14d), or the route on site 1's edge |
| 4 | The same PC → Web Browser | `http://` + that address | Your website | Row 3 |
| 5 | The same PC | `ping` the ISP's web server | ✅ still replies (PAT still works) | PAT list on the site 3 edge |
| 6 | Site 2 PC | `ping` a site 1 PC | ✅ still replies | P14b: sequence number, or a typo in the crypto map |

✅ **Done when:** all 6 rows pass. **File → Save As** `my-project/step-extras.pkt`.

---

# Part D — Finish and prove

## P15 — Scale to 8 PCs per department
Open `my-project/step-extras.pkt` → **Save As** `my-project/final.pkt`.

Write your own table like section 1.4 of [step 16](../steps/16-final-integration), then do its
Missions 2–3 with your values.

- [ ] 6 departments × 8 PCs in site 1 (48 PCs)
- [ ] 6 departments × 8 PCs in site 2 (48 PCs)
- [ ] 6 departments × 8 PCs in site 3 (48 PCs)
- [ ] Each PC has an address, and pinged once, so port-security learned its MAC
- [ ] Sites 1–2, if you did P12: each new PC uses DHCP, so it is in the binding table
- [ ] Each new PC is in its building in the Physical view
- [ ] Your file matches table 1 of **What "finished" means**: count every row

✅ **Done when:** you counted 8 in every department, and each new port is in the right VLAN.

---

## P16 — Email
Do Missions 4–6 of step 16 with your domain, your users, your password.

- [ ] 2 email users per department, in every branch (req. 22.19). Sites 1 and 2 use the DMZ mail server. Site 3 uses its own IPv6 mail server and its own domain
- [ ] The client is set on 2 PCs in every department
- [ ] A mail sent between two departments arrives
- [ ] A mail from site 1 to a site 2 user arrives, through the VPN
- [ ] A mail between two departments of site 3 arrives
- [ ] The user table is copied to chapter 15 of your book

---

## P17 — Hardening sweep, and no Intel left
**1.** Do Mission 7 of step 16: the sweep on every device. Don't forget to give the ASA a banner.

**2.** The Intel sweep. On **every** router and L3 switch, and on one access switch per building:
```
show running-config | include Intel|intel|HFA|JLM|KG
```
If PT refuses the filter, read `show running-config` with **Space**.

✅ **Must see:** nothing, unless the word is really part of your own names. While you are there, read
the addresses too: `10.10.x.x`, `10.20.x.x`, `10.30.x.x` and `200.x.x.x` are the example's. They are a
mistake unless your plan really uses them.

**3.** Check by eye: every device name, DNS record, SSID, email domain, banner, DHCP pool name, and
web page. Open your website: the page must show **your** company, not Intel's.

✅ **Done when:** no Intel value is left, and every device is saved
(`copy running-config startup-config`, `write memory` on the ASA).

---

## P18 — The tests, with screenshots
🧠 **Why:** the examiner grades what you can **show**, and the screenshots go in your book.

**1.** Do the 24 tests of [step 16](../steps/16-final-integration), Mission 8, on `final.pkt` with your
values. Save each screenshot as `my-project/screenshots/test-01.png`, and so on.

> ⚠️ Tests 13–17 break things. **File → Save** first, and reopen the saved file afterwards.

**2.** Add these 4 tests for what you built in Part C:

| # | From | Do | Proves |
|---|---|---|---|
| 25 | One laptop in site 2 and one in site 3 | `ipconfig`, then `ping` a server of its site | Access points in every branch (20.6) |
| 26 | An access switch | `show ip arp inspection`, then P12 row 3 | DAI |
| 27 | Both cores | `show etherchannel summary` + `show ip ospf neighbor` | L3 EtherChannel |
| 28 | Site 3's IPv4 PC, and `show crypto isakmp sa` on the site 1 edge | `ping` your DMZ web server | Site-to-site VPN to site 3, DMZ through VPN (21) |

**3.** Run the whole-network self-test: [step 16's self-test](../steps/16-final-integration/self-test.md), with your values.

✅ **Done when:** all 28 tests pass, you have every screenshot, and the step 16 self-test is fully ticked.

---

## P19 — Prove every ministry requirement
🧠 **Why:** this is your last check that nothing is missing. "T5" means test 5 of P18.

Tick each row only when you **showed** it on your own file. Don't tick from memory.

| Req. | What | Proof on your file | ✔ |
|---|---|---|---|
| 1, 3 | BGP joins the branches through the WAN | T23 | ☐ |
| 2 | VPN between branches, private addresses | T5, T28 | ☐ |
| 4 | OSPF or EIGRP inside the ISP's AS | `show ip route` on an ISP router: `O` routes | ☐ |
| 5 | Collapsed core on L3 switches | Site 2: no separate core, CC1 `show ip route` | ☐ |
| 6, 23.1 | Redundancy, layers 2 and 3, inside and between branches | T13, T14, T15 | ☐ |
| 7 | One branch fully IPv6 | T8, `show ipv6 route` | ☐ |
| 8, 22.16 | Multi-area OSPF with virtual link | T1, T21 | ☐ |
| 9 | Internet | T2 | ☐ |
| 10, 22.12 | Static, dynamic, overload NAT. Browsing + static to internal web | T3, T7, T9 | ☐ |
| 11, 12 | inside / outside / DMZ on an ASA | `show nameif`, step 11 Mission 13 tests | ☐ |
| 13 | Port security, SSH, attack protection | T11, T12, T16, T17, T26 | ☐ |
| 14 | STP, manual root, PortFast | `show spanning-tree vlan <V>` on the primary | ☐ |
| 15 | BPDU Guard + Root Guard | T17, `show running-config interface <downlink>` | ☐ |
| 16 | DHCP snooping | T16, `show ip dhcp snooping` | ☐ |
| 17 | HSRP in every branch | T24 | ☐ |
| 18 | WiFi: guests + employees, secured | T4 | ☐ |
| 19 | One main branch | Your book, chapter 7 | ☐ |
| 20.1 | 3 buildings / floors, access–distribution–core | Physical view of each site | ☐ |
| 20.2 | 6 departments × 8 PCs in each branch | P15 | ☐ |
| 20.3–20.5 | Switches, routers, printers in each branch | Logical view of each site | ☐ |
| 20.6 | Access points in each branch | T4, T25 | ☐ |
| 20.7 | DHCP in each branch | `ipconfig /all` on a PC of each site | ☐ |
| 20.8 | NTP synced to the main NTP | T18 | ☐ |
| 20.9 | Syslog server + logging on all devices | T19 in sites 1–2, the Syslog server's log in site 3, P17 | ☐ |
| 20.10 | AAA server | T11, the server's AAA tab in each site. Site 3: the note in step 14, Mission 12 | ☐ |
| 21 | 6 DMZ servers, reachable through VPN | T6, T28 | ☐ |
| 22.1–22.3 | Hostnames, banner, hashed passwords | P17 | ☐ |
| 22.4 | Hierarchical addressing from one master block | Your plan, part C | ☐ |
| 22.5 | SSH/AAA, management VLAN, other addresses blocked | T11, T12 | ☐ |
| 22.6–22.8 | VLANs, trunks, VTP | `show vlan brief`, `show interfaces trunk`, `show vtp status` | ☐ |
| 22.9 | Port security | `show port-security` | ☐ |
| 22.10 | Inter-VLAN routing | T1 | ☐ |
| 22.11 | DHCP on routers/L3 at one site, on a server at another | Site 1: `show ip dhcp binding`. Site 2: `ipconfig /all` shows the server | ☐ |
| 22.13 | ACLs, CBAC | T4, T12, step 13 self-test D3–D4 | ☐ |
| 22.14 | EtherChannel | T20, T27 | ☐ |
| 22.15, 22.18 | Summarization, passive, routing password | T22, `show ip protocols`, B6 of step 08's self-test | ☐ |
| 22.17 | Static routing toward the WAN | `show ip route static` on each edge | ☐ |
| 22.19 | Email on 2 PCs per department per branch | P16 | ☐ |
| 23.2 | ISP reached by DSL modem | T14 | ☐ |
| 23.3 | ISP DNS + 2 web servers | T2. Site 3: your IPv4 VLAN and the IPv6 web server. No NAT64 in PT: chapter 6 | ☐ |
| 23.4 | Remote-access VPN | T10 | ☐ |

✅ **Done when:** every row is ticked.

---

# Part E — The book

## P20 — Write the project book
🧠 **Why:** the book is **30% of your grade**, and the examiner will ask about every line.

**How to work:** start from [docs/project-book-draft-he.md](../docs/project-book-draft-he.md), which has the
ministry's chapters in order. Replace **all** the Intel content with yours, in your own words. Copied
text you can't explain will hurt you at the exam.

| Ch. | Chapter | What goes in | Taken from |
|---|---|---|---|
| 1 | דף שער | School, project name, "פרויקט גמר הנדסאים", "מגמת תקשוב", your name, ID, school year, mentor | You |
| 2 | תוכן עניינים | Automatic, from the word processor. Update it last | Word processor |
| 3 | מבוא | What the project is, its goals, why it matters | Proposal |
| 4 | אודות החברה | The real company and its sites, an organization chart, and how many people work in each department | Proposal |
| 5 | דרישות הארגון מהרשת | What the company needs, in plain words | Proposal, P19 |
| 6 | בחירות וחלופות | Each choice, the alternatives, and **why**. Include all of part H, step 16's list of simulator limits, and IP Source Guard (Packet Tracer 9.0.1 has no such command) | Plan E, H |
| 7 | סניפים ומחלקות | Every branch, its 6 departments, and what each department does | Plan A |
| 8 | טופולוגיה פיזית | Physical view: all branches together, then one per branch. The ministry recommends **realistic building drawings** | Step 16 Mission 9a |
| 9 | טופולוגיה לוגית | Logical view: all branches together, then one per branch + the WAN | Step 16 Mission 9a |
| 10 | חלוקת ציוד | How many of each device, per branch | Your file |
| 11 | מפרט ציוד | The models (2960, 3650, 2911, ASA 5506-X, AccessPoint-PT…) and why | Plan |
| 12 | טבלת שמות | Every device name, **with the meaning of each abbreviation** | Plan B |
| 13 | טבלאות VLAN / IP | All VLANs, subnets, gateways, fixed addresses, links | Plan C, D, I |
| 14 | טבלת סיסמאות | Every password, per device type | Plan F |
| 15 | טבלת אימיילים | Every email user | P16 |
| 16 | פקודות | See below | Your file |
| 17 | ביבליוגרפיה | The sources you really used | You |

### Chapter 16 — commands
This is the biggest part of the book. The ministry's sample contents page gives every technology its
own section, split by branch, and runs to about 218 pages.

**Every topic in the table gets the same 4 parts:**
1. **What it is and why you need it,** in your own words. Sources: part 1 of the step, and its 🎓 answers.
2. **Advantages and disadvantages,** where the topic has them.
3. **The commands,** with one line explaining each. The steps have these tables.
4. **One sub-section per branch that has it:** your real configuration, then a screenshot of the proof.

| # | Topic | Theory to include | Sites | Proof | Learn in |
|---|---|---|---|---|---|
| 1 | Hostnames, banners, passwords | What a banner is, hashing | 1, 2, 3 | `show running-config` | 02 |
| 2 | VLAN | Types, advantages, disadvantages | 1, 2, 3 | `show vlan brief`, port table | 02 |
| 3 | Trunk | 802.1Q tag, native VLAN | 1, 2, 3 | `show interfaces trunk` | 03 |
| 4 | VTP | Modes, password | 1, 2, 3 | `show vtp status` | 03 |
| 5 | STP | How loops form, root election, port roles | 1, 2, 3 | `show spanning-tree vlan <V>` | 04 |
| 6 | PortFast, BPDU Guard, Root Guard | The attack each one stops | 1, 2, 3 | `show running-config interface <port>` | 04 |
| 7 | EtherChannel | LACP against PAgP | 1, 2 | `show etherchannel summary` | 04, 13, P13 |
| 8 | Layer-2 against layer-3 switch | Differences, advantages, disadvantages | — | — | 05 |
| 9 | Routing and inter-VLAN routing | What a router does, SVI, route choice | 1, 2, 3 | `show ip route`, `show ipv6 route` | 05 |
| 10 | HSRP | Priority, preempt, what happens on failure | 1, 2, 3 | `show standby brief` | 05, 14 |
| 11 | DHCP | DORA, relay and `ip helper-address` | 1, 2, 3 | `show ip dhcp binding`, `ipconfig /all` | 05, 13, 14 |
| 12 | DHCP snooping | Starvation and spoofing attacks | 1, 2 | `show ip dhcp snooping` | 06 |
| 13 | Dynamic ARP Inspection | ARP poisoning | 1, 2 | `show ip arp inspection` | P12 |
| 14 | Port security | Violation modes, sticky | 1, 2, 3 | `show port-security` | 06 |
| 15 | CDP, VLAN hopping, `no ip domain-lookup` | The risk of each, parking VLAN | 1, 2, 3 | `show running-config` | 02, 06 |
| 16 | SSH and the management VLAN | SSH against Telnet | 1, 2, 3 | SSH test, refused test | 07 |
| 17 | ACL | How they work, types, wildcard | 1, 2, 3 | `show access-lists` | 07, 10, 11 |
| 18 | AAA | RADIUS against TACACS+ | 1, 2, 3 | Login test, server's AAA tab | 08 |
| 19 | Syslog | Severity levels | 1, 2, 3 | The server's log | 08 |
| 20 | NTP | Main server, branch sync | 1, 2, 3 | `show ntp status` | 08, 11 |
| 21 | OSPF | Areas, neighbor states, LSA, LSDB, MD5, passive, summary, virtual link | 1 | `show ip ospf neighbor`, `show ip ospf virtual-links` | 08, 09 |
| 22 | EIGRP | Neighbor conditions, tables, summary | 2 | `show ip eigrp neighbors` | 13 |
| 23 | IPv6 and OSPFv3 | Address types, SLAAC, stateless DHCPv6 | 3 | `show ipv6 ospf neighbor` | 14 |
| 24 | BGP | Message types, neighbor process, tables | ISPs | `show ip bgp summary` | 12 |
| 25 | NAT | Terms, 3 types | 1, 2, 3 | `show ip nat translations` | 12, 13, 14 |
| 26 | ASA and DMZ | Security levels, zones | 1 | `show nameif`, `show access-list` | 11 |
| 27 | CBAC | What it inspects | 2 | `show ip inspect sessions` | 13 |
| 28 | VPN | IPsec, GRE, remote access | 1, 2, 3 | `show crypto isakmp sa`, `show crypto ipsec sa` | 15, P14 |
| 29 | WiFi | WPA2, guest isolation | 1, 2, 3 | Laptop `ipconfig` and ping | 10, P11 |
| 30 | HTTP, DNS, FTP, email | What each service does | 1, 3, ISPs | Browser, `nslookup`, mail screenshots | 11, 12, 14, 16 |
| 31 | Redundancy and DSL backup | What fails, what takes over | 1 | T13, T14, T15 | 12 |

**End chapter 16 with:** all the `show` commands you used, the full `show running-config` of each main
device, and `show ip route` of each main router.

👣 **The final book check:**
- [ ] All 31 topics are there, each with a sub-section for every branch that has it
- [ ] Every value in the book matches your file: open the file and compare 10 random values
- [ ] No "Intel", no example address, no "טיוטה", no ✍️ left
- [ ] Every figure and table has a caption and is on the contents page
- [ ] Your teacher read it. Ask whether your college needs a declaration page (הצהרת הסטודנט) too

✅ **Done when:** your teacher approved it. Keep it **outside** the course folder.

---

# Part F — Rehearse the exam

## P21 — Your opening presentation
The ministry says you present your work, give the background, talk about yourself and the company, and
briefly explain your technologies. Prepare **3 minutes**, without reading:

1. Who you are, and the company you chose (30 seconds).
2. The 3 branches: where, what each does, what's different in each (1 minute). Open the Physical view.
3. The main technologies, and why you chose them (1 minute). Open the Logical view.
4. One live proof: for example, `ping -t` while you fail a link (30 seconds).

✅ **Done when:** you did it twice in front of a classmate, in under 3 minutes.

---

## P22 — Practical task: add a department
🧠 The ministry says this may happen: *"add components, from a department up to a whole branch"*.

**Drill:** on a **copy** of your file (**Save As** `my-project/drill.pkt`), add a department to each of
your 3 sites, with the **Recipe** in Part C. Time yourself. Work from memory first, and say each
command out loud.

- [ ] Site 1 (recipe A) done in under 20 minutes
- [ ] Site 2 (recipe B) done in under 20 minutes
- [ ] Site 3 (recipe C) done in under 20 minutes

> 💡 A 2960 has only 24 ports. If the examiner asks for 8 PCs and your switch has no free ports, you
> need a **new access switch**: hardening, VTP, trunks to both L3 switches, management address, SSH,
> and add it to the AAA server. Steps 02, 03 and 07 have all of it.

**And know what you did NOT need to change, and why:**
- OSPF: the subnet is inside the building's /20, so `network` and `area … range` already cover it.
- EIGRP: `network` covers the whole site. A subnet outside your /21 appears as one extra route, and
  that's fine. Inside the /21, the summary already covers it.
- OSPFv3 in site 3: no change on the edge, because the SVI joins by itself with `ipv6 ospf 1 area 0`.
- ASA and NAT: the rules cover the whole block.
- The VPN: the routes cover the whole block.

> ⚠️ Answer "why no change?" only if it's true: if the new subnet is **outside** the planned block, those
> things **do** need changing. That's why you choose the subnet first.

---

## P23 — Practical task: add a branch
This is the biggest task the ministry mentions. You don't need to finish a full site at the exam. You
must show that you know the **order**, and get a small one working. On your `drill.pkt`:

| # | Do | Prove |
|---|---|---|
| 1 | Write the new block, VLANs and public address on paper | It doesn't overlap your plan |
| 2 | Place an edge router, 1 L3 switch, 1 access switch, 2 PCs. Cable them | Green lights |
| 3 | Hardening on all 3 | Banner, `enable secret` |
| 4 | VLANs, trunk, access ports | `show vlan brief` |
| 5 | SVI + DHCP (a pool on the L3 switch) | The PCs get addresses |
| 6 | Routing inside: a default route from the L3 switch to the edge, a static route back on the edge | PC pings the edge |
| 7 | WAN: connect the edge to an ISP router. Default route on the edge. Add the new public /30 on the ISP so BGP or its IGP knows it | Edge pings the ISP |
| 8 | NAT overload on the edge | PC reaches the ISP's web server |
| 9 | Security: SSH + VTY ACL + port security | SSH from the right place only |

✅ **Done when:** you did it once in under 60 minutes, and you can say why each step comes in that order.

---

## P24 — Practical task: find the fault
🧠 The examiner breaks something and asks you to find it. The method beats guessing.

**The method.** Always in this order, from the bottom up:
1. **Who can't reach what?** Test it yourself. Try one hop closer until something works.
2. **Cable and port:** `show ip interface brief`, `show interfaces status`.
3. **VLAN and trunk:** `show vlan brief`, `show interfaces trunk`.
4. **Address and gateway:** `ipconfig /all`, `show standby brief`.
5. **Routing:** `show ip route`, `show ip ospf neighbor`, `show ip eigrp neighbors`, `show ip bgp summary`.
6. **Filters and NAT:** `show access-lists`, `show ip nat translations`, `show access-list` on the ASA.
7. Say what you found **before** you fix it. Fix it. Test again.

**Drill.** On `drill.pkt`, a classmate makes one fault from this table without telling you which. You
find it, fix it, and swap. Reopen the file between faults.

| # | The fault | What you notice | Found with |
|---|---|---|---|
| 1 | A user port in the wrong VLAN | The PC gets another department's address | `show vlan brief` |
| 2 | `switchport trunk allowed vlan remove <V>` | That VLAN's PCs get `169.254…` | `show interfaces trunk` |
| 3 | `no ip dhcp snooping trust` on an uplink | PCs get `169.254…` | `show ip dhcp snooping` |
| 4 | `no standby <V> preempt` on the primary | After a repair, the primary stays Standby | `show standby brief` |
| 5 | A wrong OSPF MD5 key on one link | A neighbour disappears | `show ip ospf neighbor` |
| 6 | `passive-interface` on an uplink | A neighbour disappears | `show ip ospf interface` / `show running-config \| section ospf` |
| 7 | `no area <N> range …` on the ABRs | Many routes on the core instead of one | `show ip route ospf` |
| 8 | A wrong router-ID in `area … virtual-link` | The far building is cut off | `show ip ospf virtual-links` |
| 9 | A line removed from the ASA's ACL | The home PC can't reach your website | `show access-list` |
| 10 | The static NAT line removed | The same symptom as 9 | `show ip nat translations` |
| 11 | A wrong BGP `neighbor` on an ISP | One ISP's prefixes vanish | `show ip bgp summary` |
| 12 | A wrong pre-shared key | The VPN fails, internet still works | `show crypto isakmp sa` |
| 13 | A second MAC on a printer port | The port is down | `show port-security interface fa0/x` |
| 14 | A wrong default gateway in a DHCP pool | Pings work inside the VLAN only | `ipconfig /all`, then the pool |

✅ **Done when:** you found and fixed 10 faults in a row, each in under 10 minutes.

---

## P25 — The questions
The ministry says you must explain **why** you built it this way, and show deep understanding of the
protocols. Read [docs/exam-guide.md](../docs/exam-guide.md).

- [ ] You answered every **🎓 Explain it to the examiner** question of steps 00–16, out loud, in **your**
      project: what it is → why → where in my project → show it
- [ ] You did every step's **🔁 drill** on your own values
- [ ] You can explain every line of part H and of chapter 6
- [ ] Step 16's final rehearsal, twice

---

# Part G — Submit

## P26 — Freeze the file and do the cold start
⛔ Don't start until P15–P20 are done.

🧠 **Why:** a project that works while it's open can fail after a reopen, if something wasn't saved.
This is the last and most important test.

1. Save **every** device: `copy running-config startup-config`, and `write memory` on the ASA.
2. **File → Save**. Close Packet Tracer completely.
3. Open Packet Tracer → open `final.pkt`. Press **Fast Forward Time** until the lights settle (several times).
4. Run this cold-start table:

| # | From | Do | Must see |
|---|---|---|---|
| 1 | Site 1 PC | `ipconfig`, then browse the ISP's website | An address, the page opens |
| 2 | Site 2 PC | `ipconfig`, then `ping` a site 1 PC | ✅ (the first may time out while the VPN builds) |
| 3 | Site 3 PC | Browse your IPv6 web server | The page opens |
| 4 | Home PC | Browse your website, then connect the VPN and `ping` a site 1 server | Both work |
| 5 | Site 1 edge | `show crypto isakmp sa` | Site 2's VPN `QM_IDLE` (after row 2) |
| 6 | CORE1 | `show ip ospf neighbor` | All `FULL` |
| 7 | An ISP border router | `show ip bgp summary` | Every neighbour ends with a number |
| 8 | Admin PC | SSH to a switch | Login works |

❌ **Something fails only after reopening:** that device wasn't saved. Fix it, save it, and repeat
from step 1.

5. Copy the file to a USB drive **and** to a second place. Open that copy once on the **exam
   computer's Packet Tracer version**.

✅ **Done when:** all 8 rows pass after a full restart, and the backups open.

**From now on, don't change `final.pkt`.** Practice only on copies.

---

## P27 — Hand in, and exam day
**Hand in** what your teacher asks for, the way your teacher asks. The ministry's two parts are:

- [ ] `final.pkt`: the simulation file
- [ ] The project book, with its cover page complete

**Bring to the exam:**
- [ ] `final.pkt` on two separate drives
- [ ] Your printed book, or the file
- [ ] Your plan, with all the passwords
- [ ] A calm head: you've already done all of this at least twice

**During the exam:**
- Don't guess. If you don't know, say what you do know, and show it.
- Think out loud: the examiner grades **how** you think too.
- Before any practical task, **Save As** a new name, so the good file stays untouched.
- After a fix, always test. Show the examiner it works.

🏁 **You're done.** Good luck!
