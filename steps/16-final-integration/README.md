# Step 16 — Final integration, scaling & the project book

> ⏱ 3–4 lessons · 🎯 ministry requirements 20.2, 22.19 + the project book · 📁 start from: `my-work/step15.pkt` · 💾 save as: `my-work/final.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 15 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## 🗺 Your missions in this step
**Lesson 1 — scale**
- [ ] **Mission 1** — Open step 15 and save as final (2 min)
- [ ] **Mission 2** — Add PCs to one department, carefully (15 min)
- [ ] **Mission 3** — Scale every other department (the rest of the lesson — in pairs)

**Lesson 2 — email**
- [ ] **Mission 4** — Create the mailboxes (20 min)
- [ ] **Mission 5** — Configure the email clients (20 min)
- [ ] **Mission 6** — Send and receive test mails (10 min)

**Lesson 3 — prove it**
- [ ] **Mission 7** — The final hardening sweep (20 min)
- [ ] **Mission 8** — The 24 end-to-end tests, with screenshots (the rest of the lesson)

**Lesson 4 — the book and the defence**
- [ ] **Mission 9** — Complete the project book (homework + lesson)
- [ ] **Mission 10** — Defend it: the final questions out loud
- [ ] **🛠 Your own project**

## What you will have at the end
A **submission-ready** project: every department has at least 8 PCs, every department has two
email users, every requirement is tested with a screenshot, and the project book is drafted.

---

## Part 1 — Understand it first (simple English)

### 1.1 Why scale at the end?
The ministry requires **6 departments × at least 8 PCs** in every branch (req. 20.2). Building with
2 PCs per department made the lab fast and readable. Because everything is automatic now (DHCP,
SLAAC, port-security sticky), adding PCs is just drag, cable, rename.

### 1.2 How email works in Packet Tracer
- The **email server** keeps mailboxes (`eng1@intel.lab`) — like a post office with P.O. boxes.
- The PC's **email client** sends mail to the server (SMTP) and collects its own mail (POP3).
- The client finds the server by **name** (`mail.intel.lab`) → that's why DNS had to work first.

Haifa and Jerusalem use the DMZ mail server (Jerusalem through the VPN). Kiryat Gat uses its own IPv6
mail server (`kg.intel.lab`).

### 1.3 Testing = proof
The examiner grades what you can **show**. Every requirement needs a test you can run live and a
screenshot in the book.

### 1.4 Port ranges and names for the new PCs
| Branch | Department | Switches · ports | New PCs |
|---|---|---|---|
| Haifa | Engineering | A-ACC1 / A-ACC2 · Fa0/3-10 | PC-HFA-ENG-04 … 08 |
| Haifa | Finance | A-ACC1 / A-ACC2 · Fa0/13-20 | PC-HFA-FIN-04 … 08 |
| Haifa | HR | B-ACC1 / B-ACC2 · Fa0/2-10 | PC-HFA-HR-03 … 08 |
| Haifa | Marketing | B-ACC1 / B-ACC2 · Fa0/12-20 | PC-HFA-MKT-03 … 08 |
| Haifa | R&D | C-ACC1 / C-ACC2 · Fa0/2-10 | PC-HFA-RND-03 … 08 |
| Haifa | IT | C-ACC1 / C-ACC2 · Fa0/12-20 | PC-HFA-IT-03 … 08 |
| Jerusalem | Sales / Logistics | F1-ACC1 · Fa0/3-10 / Fa0/13-20 | PC-JLM-SAL-03…08 / PC-JLM-LOG-03…08 |
| Jerusalem | Production / QA | F2-ACC1 · Fa0/3-10 / Fa0/13-20 | PC-JLM-PRD-… / PC-JLM-QA-… |
| Jerusalem | Security / Admin | F3-ACC1 · Fa0/3-10 / Fa0/13-20 | PC-JLM-SEC-… / PC-JLM-ADM-… |
| Kiryat Gat | Fab-Ops / Quality | KG-ACC1 · Fa0/3-10 / Fa0/13-20 | PC-KG-FAB-… / PC-KG-QUA-… |
| Kiryat Gat | Maintenance / Safety | KG-ACC2 · Fa0/3-10 / Fa0/13-20 | PC-KG-MNT-… / PC-KG-SAF-… |
| Kiryat Gat | Planning / Training | KG-ACC3 · Fa0/3-10 / Fa0/13-20 | PC-KG-PLN-… / PC-KG-TRN-… |

Totals: 48 PCs per branch, **144** user PCs.

> ⚠️ Plug each new PC **only** into a port in its department's range. A port outside the range
> is in another VLAN (or the parking VLAN 999 and shut) — the PC gets the wrong address or none.

---

## Mission 1 — Open step 15 and save as final (how? → M1)
**File → Open** → `my-work/step15.pkt` → **File → Save As** → `my-work/final.pkt`.

> PT gets slow with ~250 devices. **Save often**, close other programs, and stay in **Realtime** mode.

---

## Mission 2 — Add PCs to one department, carefully
🧠 Do one department slowly with all the checks. After that, the rest is repetition.

👉 **Example:** Haifa **Engineering** — needs PC-HFA-ENG-04 … 08 (5 new PCs).

👣 **Steps:**
1. Hold **Ctrl**, click the **PC** icon (End Devices), then click **5 times** next to SW-HFA-A-ACC1
   → press **Esc** (M3).
2. Rename them `PC-HFA-ENG-04` … `PC-HFA-ENG-08` (M4).
3. Cable each (**Straight-Through**, PC **FastEthernet0**) to the next free Engineering port
   (M5). Split them between the two switches:

   | PC | Switch | Port |
   |---|---|---|
   | PC-HFA-ENG-04 | SW-HFA-A-ACC1 | Fa0/3 |
   | PC-HFA-ENG-05 | SW-HFA-A-ACC2 | Fa0/3 |
   | PC-HFA-ENG-06 | SW-HFA-A-ACC1 | Fa0/4 |
   | PC-HFA-ENG-07 | SW-HFA-A-ACC2 | Fa0/4 |
   | PC-HFA-ENG-08 | SW-HFA-A-ACC1 | Fa0/5 |

4. Each PC → **Desktop → IP Configuration → DHCP** (M16).
5. Each PC → **Command Prompt** → `ping 10.10.16.1` (the gateway) — so port-security learns its MAC.

✅ **You should see:** each PC has a `10.10.16.x` address and the ping replies.

❌ **A PC has `169.254.x.x`:** it's on a port outside the range, or the port is shut (parking).
Check with `show vlan brief` on the switch.

---

## Mission 3 — Scale every other department
Repeat Mission 2 for every row of table 1.4. **Kiryat Gat:** in step 4 click **IPv6 → Automatic**
instead of DHCP, and in step 5 ping the DNS server `2001:DB8:30:100::10`.

Tick when the department has **8** PCs that all have addresses:

**Haifa:** - [ ] Engineering  - [ ] Finance  - [ ] HR  - [ ] Marketing  - [ ] R&D  - [ ] IT
**Jerusalem:** - [ ] Sales  - [ ] Logistics  - [ ] Production  - [ ] QA  - [ ] Security  - [ ] Admin
**Kiryat Gat:** - [ ] Fab-Ops  - [ ] Quality  - [ ] Maintenance  - [ ] Safety  - [ ] Planning  - [ ] Training

> 💡 Work in pairs: each pair scales one branch, then you swap files and check each other's ticks.
> 💡 Tidy screenshots: **Logical view** → select a building's devices → right-click → **Cluster**
> groups them into one icon (double-click to open).

💾 **File → Save** after **every** department.

---

## Mission 4 — Create the mailboxes (req. 22.19)
👉 **Where:** **SRV-HFA-DMZ-MAIL** → **Services** → **EMAIL** → *User Setup* area.

For each user: type **User** and **Password** → click **+** (Add). All passwords: `Mail#2026`.

| Users | Department |
|---|---|
| `eng1`, `eng2` | Haifa Engineering |
| `fin1`, `fin2` | Haifa Finance |
| `hr1`, `hr2` | Haifa HR |
| `mkt1`, `mkt2` | Haifa Marketing |
| `rnd1`, `rnd2` | Haifa R&D |
| `it1`, `it2` | Haifa IT |
| `sal1`, `sal2`, `log1`, `log2`, `prd1`, `prd2` | Jerusalem Sales, Logistics, Production |
| `qa1`, `qa2`, `sec1`, `sec2`, `adm1`, `adm2` | Jerusalem QA, Security, Admin |

✅ 24 users in the list.

👉 Then **SRV-KG-MAIL** (domain `kg.intel.lab`): `fab1`, `fab2`, `qua1`, `qua2`, `mnt1`, `mnt2`,
`saf1`, `saf2`, `pln1`, `pln2`, `trn1`, `trn2` — same password.

✅ 12 users in the list.

❌ **Typed a user wrong?** Select it → click **−** (Remove) → add it again.

---

## Mission 5 — Configure the email clients
👉 **Where:** the **-01** and **-02** PC of every department → **Desktop → Email** →
**Configure Mail** (first time it opens by itself).

| Field | Haifa / Jerusalem example | Kiryat Gat example |
|---|---|---|
| Your Name | `Engineering 1` | `Fab Ops 1` |
| Email Address | `eng1@intel.lab` | `fab1@kg.intel.lab` |
| Incoming Mail Server | `mail.intel.lab` | `mail.kg.intel.lab` |
| Outgoing Mail Server | `mail.intel.lab` | `mail.kg.intel.lab` |
| User Name | `eng1` | `fab1` |
| Password | `Mail#2026` | `Mail#2026` |

→ **Save**.

> PC **-01** of a department uses user **1**, PC **-02** uses user **2** (e.g. PC-HFA-ENG-02 →
> `eng2`).

Tick each branch when all its 12 clients are set: - [ ] Haifa  - [ ] Jerusalem  - [ ] Kiryat Gat

---

## Mission 6 — Send and receive test mails
1. On **PC-HFA-ENG-01** → Email → **Compose** → To `sal1@intel.lab`, Subject `Test`, any text → **Send**.
   ✅ "Send Success".
2. On **PC-JLM-SAL-01** → Email → **Receive** → the message arrives (Haifa → Jerusalem through
   the VPN). ✅
3. In Kiryat Gat: from `fab1` send to `qua1@kg.intel.lab` → **Receive** on PC-KG-QUA-01. ✅

❌ **"Server not found" / "Receive Mail Failure":** the PC can't resolve `mail.intel.lab`. Test
`nslookup mail.intel.lab` on that PC. Jerusalem: is the VPN up (step 15)?

📝 Copy the full user list into the book's **email table** (טבלת אימיילים).

---

## Mission 7 — The final hardening sweep
👉 **Where:** **every** router and switch (SSH from PC-HFA-IT-01 is fastest).

⌨️ On each device:
```
show running-config | include hostname|banner|enable secret|service password|ip ssh|logging|ntp server
```
✅ **Each must show:** a logical hostname, a banner, `enable secret`, `service password-encryption`,
`ip ssh version 2`, a `logging host`, an `ntp server`.

⌨️ Then:
```
show running-config | section vty
```
✅ `transport input ssh` + `access-class` (or `ipv6 access-class` in Kiryat Gat).

> If PT doesn't accept the `|` filter, type `show running-config` and scroll with **Space**.

Fix what's missing, then save **every** device (`copy run start`, `write memory` on the ASA).

| Branch | Devices to sweep | Done |
|---|---|---|
| Haifa | 15 switches + SW-HFA-DMZ + R-HFA-EDGE + ASA | - [ ] |
| Jerusalem | CC1, CC2, F1/F2/F3-ACC1, SW-JLM-SRV, R-JLM-EDGE | - [ ] |
| Kiryat Gat | DIST1, DIST2, ACC1/2/3, SW-KG-SRV, R-KG-EDGE | - [ ] |
| ISPs | 6 routers: hostname + banner + secret only | - [ ] |

---

## Mission 8 — The 24 end-to-end tests (screenshot every ✅)
**How to take a screenshot for the book:** on Linux press **PrtSc** (or **Shift+PrtSc** to select an area);
on Windows **Win+Shift+S** (M22). Save as `test-01.png`, `test-02.png` … in a `screenshots` folder.

| # | From | Do | Proves | Done |
|---|---|---|---|---|
| 1 | PC-HFA-ENG-01 | `ping` PC-HFA-IT-01 | inter-VLAN + OSPF areas 1 → 0 → 2 → 3 (virtual link) | - [ ] |
| 2 | PC-HFA-ENG-01 | browse `http://www.isp-web1.com` | ASA → edge PAT → BGP internet | - [ ] |
| 3 | PC-HOME-01 | browse `http://www.intel.lab` | static NAT + ASA ACL | - [ ] |
| 4 | LT-HFA-GUEST-01 | `ping 10.10.1.10` ❌ / browse `www.isp-web2.com` ✅ | guest isolation | - [ ] |
| 5 | PC-JLM-SAL-01 | `ping 10.10.16.1` | GRE/IPsec VPN, private addressing | - [ ] |
| 6 | PC-JLM-SAL-01 | browse `www.intel.lab` + email to Haifa | DMZ services via VPN | - [ ] |
| 7 | PC-JLM-SAL-01 | `ping 200.1.100.11` + `show ip nat translations` on R-JLM-EDGE | dynamic NAT | - [ ] |
| 8 | PC-KG-FAB-01 | browse `http://www6.isp-web.com` | IPv6 end to end | - [ ] |
| 9 | PC-KG-LEGACY-01 | `ping 200.1.100.11` | PAT (overload) | - [ ] |
| 10 | PC-HOME-01 | VPN client → `ping 10.10.1.10` | remote-access VPN | - [ ] |
| 11 | PC-HFA-IT-01 | `ssh` to devices in Haifa and Jerusalem | SSH + AAA + management ACL | - [ ] |
| 12 | PC-HFA-RND-01 | `ssh` to a switch ❌ | management ACL blocks non-admins | - [ ] |
| 13 | **File → Save**, then delete A-DIST1 (**Delete** tool); re-open the saved file afterwards | `ping -t` from an ENG PC keeps going | HSRP + STP + OSPF redundancy | - [ ] |
| 14 | shut R-HFA-EDGE Gi0/1 | `show ip route` → default via 200.2.9.1 | DSL floating static | - [ ] |
| 15 | shut ISPA-BR ↔ ISPC-BR link | PC-HOME-01 → www.intel.lab still works | BGP redundancy | - [ ] |
| 16 | rogue DHCP server on an access port | PC still gets the real lease | DHCP snooping | - [ ] |
| 17 | rogue switch on a PortFast port | port err-disabled | BPDU Guard | - [ ] |
| 18 | `show ntp status` in each branch | synchronized | NTP hierarchy | - [ ] |
| 19 | SRV-HFA-SYSLOG / SRV-JLM-SYSLOG | messages with timestamps | Syslog | - [ ] |
| 20 | `show etherchannel summary` (HFA + JLM) | `Po1(SU)` LACP / PAgP | EtherChannel | - [ ] |
| 21 | `show ip ospf virtual-links` on B-DIST1 | up | virtual link | - [ ] |
| 22 | `show ip route` on R-JLM-EDGE | `D 10.20.8.0/21` | EIGRP summarization | - [ ] |
| 23 | `show ip bgp` on R-ISPA-BR | 3 prefixes | BGP | - [ ] |
| 24 | `show standby brief` in each branch | Active/Standby | HSRP (incl. IPv6) | - [ ] |

> ⚠️ Tests 13–17 **break** things. **File → Save** before each, and **File → Open** the saved
> file (answer **No** to saving) after each, so the break doesn't stay in your final file.
> Tests 16–17 are the attacks from step 06 — repeat them the same way.

---

## Mission 9 — Complete the project book (ספר הפרויקט — 30% of the grade)
A **Hebrew draft** of the whole book is ready in [`docs/project-book-draft-he.md`](../../docs/project-book-draft-he.md)
— the chapters in the ministry's order, already filled with this project's content. What **you** add:

| Chapter | You add | Done |
|---|---|---|
| 1 Cover page | school, your name, ID, year, mentor | - [ ] |
| 8 Physical topology | PT **Physical** view screenshots | - [ ] |
| 9 Logical topology | PT **Logical** view screenshots — one per branch + WAN | - [ ] |
| 16 Commands | `show run` of one device of each type + the 24 test screenshots | - [ ] |

Also adapt the text to **your** words — the examiner will ask you about every choice.

**Known simulator limitations** (already in the draft, chapter 6 — it shows understanding):
- IPv6-only branch can't reach IPv4-only services (no NAT64 in PT).
- RA Guard / DHCPv6 Guard not available.
- DSL backup carries router traffic only unless route-map NAT is supported.
- Private VLANs for DMZ server isolation not available.
- AAA over IPv6 limited — Kiryat Gat uses local authentication.

---

## Mission 10 — Defend it: the final questions out loud
The examiner asks "why?" about every choice. Answer **out loud, without notes**, then open the
answers and compare:
1. Why isn't building C's area connected to area 0, and what does the virtual link do?
2. Why doesn't the ASA do NAT in this design?
3. Why GRE over IPsec instead of plain IPsec?
4. Why is the HSRP active always the STP root of the same VLAN?
5. What happens to Jerusalem if R-JLM-EDGE dies? How would you fix it?

<details><summary>✔ Short answers</summary>

1. To meet req. 8/22.16 and show the rule "every area must touch area 0". The virtual link is a logical
   area-0 connection through transit area 2 between B-DIST (ABR 0/2) and C-DIST (ABR 2/3).
2. So private addresses pass the ASA unchanged — one NAT point (the edge) is simpler, and the VPN sees
   the real 10.x addresses.
3. Static NAT on the web server would translate its VPN replies before encryption; traffic through a GRE
   tunnel interface (not a NAT interface) isn't translated. GRE also allows routing protocols through the tunnel later.
4. To keep the path to the gateway short — otherwise frames cross the inter-distribution link for nothing.
5. Jerusalem loses internet and the VPN (single point of failure). Fix: a second edge router + second ISP
   link, HSRP between them, and a second tunnel to Haifa.
</details>

---

## Checkpoint — final
- [ ] `my-work/final.pkt` saved
- [ ] 8+ PCs per department, 6 departments per branch
- [ ] 2 email users per department, mail test passed
- [ ] All 24 tests done with screenshots
- [ ] Every row of [docs/requirements-matrix.md](../../docs/requirements-matrix.md) ✅
- [ ] Project book completed from the draft

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

- [ ] Open `my-project/step15.pkt` → **Save As** `my-project/final.pkt`
- [ ] Write your own port-range table (like 1.4) and scale every department to 8+ PCs
- [ ] Mailboxes and clients: 2 per department per site; the mail tests
- [ ] Hardening sweep on every device
- [ ] The 24 tests on **your** network, with screenshots
- [ ] Your project book

| In the example | In your project — take it from |
|---|---|
| 6 departments × 8 PCs per site | plan part A (your departments — at least 8 PCs each, req. 20.2) |
| Email users and the mail domain | your domain, 2 users per department per site (req. 22.19) |
| End-to-end test plan | the same tests, run on **your** network; screenshot every ✅ |
| Example project book (`docs/project-book-draft-he.md`) | the **structure** only — your book describes **your** company, plan and screenshots |

**Final check:** go through the requirement list in `docs/requirements-matrix.md` and tick
where **your** project implements each one.

💾 Save as `my-project/final.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★3 Show me your IP address tables and explain your plan.**

<details><summary>Answer</summary>

Use your project book: master block → site blocks → areas → VLANs → links/loopbacks, and why (readability, summarization, growth).

**Show it:** `my-project/plan.md`, the project book
</details>

**2. ◆ Walk me through a packet from a PC in a branch to the web server in the DMZ.**

<details><summary>Answer</summary>

PC → its gateway (HSRP `.1`) → the branch routing protocol → the edge router (static default route, NAT) → the WAN / VPN tunnel → HQ edge → ASA (ACL, security levels) → DMZ server. Name the device and the decision at every hop.

**Show it:** `tracert <server IP>` from the PC
</details>

**3. ◆ Which requirement is implemented where?**

<details><summary>Answer</summary>

Go through the ministry list and point to the device and the `show` command for each one. Prepare this list in advance in your project book.

**Show it:** `docs/requirements-matrix.md`, your own version
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

The final rehearsal — do it twice before the exam:
1. Ask a classmate to choose any 5 requirements; for each, show the configuration and a `show`
   command that proves it works, and explain every line — **2 minutes per requirement**.
2. Ask a classmate to break something in a copy of your file (shut an interface, delete a line).
   Find it with `show` commands and fix it — **10 minutes**.

---

## 👩‍🏫 Teacher notes
- **Lesson 1:** Missions 1–3 — scaling (do Mission 2 together on the projector; then pairs: each
  pair scales one branch) + Clusters for clean screenshots.
- **Lesson 2:** Missions 4–6 — email servers and clients; the mail test across the VPN.
- **Lesson 3:** Missions 7–8 — the sweep and the test plan; students run all 24 tests and collect
  screenshots.
- **Lesson 4:** Missions 9–10 — the project book (go through the draft chapter by chapter); mock oral
  exam with the final questions.
- **Before submission check:** two students may not submit the same project — each student's company,
  cities, names and address plan must be their own.
