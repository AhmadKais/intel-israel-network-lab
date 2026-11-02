# Step 16 — Self-test (the final check before you submit)

> 🧪 Do this **after** you finish step 16, on your saved file `my-work/final.pkt`.
> It takes about 45 minutes. It is the last check before you hand in the project.

## How to use this test
1. Press **Fast Forward Time** 3–4 times first (M18). With ~250 devices, give PT time.
2. Do the tests **in order**, top to bottom.
3. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
4. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
5. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
6. Every box ticked → the project is ready to submit.

This test checks what step 16 added (Parts A–C) and then runs a short check of the **whole**
network (Part D), because adding 100+ PCs can break things that worked before. The
24 tests of Mission 8 are **not** repeated here. Do them in the step, with screenshots.
"M…" numbers are the basic moves in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — Scaling: 8 PCs per department

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | Each Haifa access switch | `show vlan brief` | The new PCs' ports are in the right VLAN (e.g. ENG-04 on A-ACC1 Fa0/3 → VLAN 10) | Mission 2 table. Wrong port → move the cable |
| ☐ | A2 | 3 new PCs per branch, at random (e.g. PC-HFA-ENG-06, PC-JLM-QA-05, PC-KG-SAF-07) | `ipconfig` | Haifa / Jerusalem: an address in its VLAN, gateway `.1`. Kiryat Gat: `2001:DB8:30:<vlan>:…`, gateway `FE80::1` | `169.254…` → port outside the range or still parked (shutdown) |
| ☐ | A3 | The same 3 PCs | `ping` the gateway / DNS of Mission 3's table | ✅ replies | A2 |
| ☐ | A4 | **Physical** → Navigation | Look under **HFA Data Center** | No `PC-…` name is left there (no PC belongs in the Data Center) | Drag each new PC to its building (Mission 3 table) |
| ☐ | A5 | Count | Each department | **8** PCs | Mission 3 |

## Part B — Email

| ☐ | # | On / From | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | SRV-HFA-DMZ-MAIL → Services → EMAIL | Look | Domain `intel.lab`, **24** users (`eng1` … `adm2`) | Mission 4 |
| ☐ | B2 | SRV-KG-MAIL → Services → EMAIL | Look | Domain `kg.intel.lab`, **12** users (`fab1` … `trn2`) | Mission 4 |
| ☐ | B3 | PC-HFA-ENG-01 → Email | Compose to `sal1@intel.lab` → **Send** | `Send Success` | `nslookup mail.intel.lab` on that PC (DNS record, step 11) |
| ☐ | B4 | PC-JLM-SAL-01 → Email | **Receive** | The message from B3 (Haifa → Jerusalem through the VPN) | VPN up? (step 15 self-test, Part B) |
| ☐ | B5 | PC-KG-FAB-01 → Email | Compose to `qua1@kg.intel.lab` → Send. Then PC-KG-QUA-01 → **Receive** | The message arrives | `mail.kg.intel.lab` AAAA record (step 14) |

## Part C — The hardening sweep

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | Every router and switch (except the expected exceptions in Mission 7) | `show running-config \| include hostname\|banner\|enable secret\|service password\|ip ssh\|logging\|ntp server` | hostname, banner, `enable secret`, `service password-encryption`, `ip ssh version 2`, `logging host`, `ntp server` | Add the missing line, then save |
| ☐ | C2 | The same devices | `show running-config \| section vty` | `transport input ssh` + `access-class` (or `ipv6 access-class` in Kiryat Gat) | Step 07 |
| ☐ | C3 | ASA | `show running-config` → look for `banner` | `banner motd Intel Israel - AUTHORIZED ACCESS ONLY` | Mission 7, ASA note |
| ☐ | C4 | The 6 ISP routers | `show running-config` | hostname, banner, `enable secret` | Step 12 |

## Part D — The whole network still works

| ☐ | # | From / On | Type / do | Must see | If not → run this self-test again |
|---|---|---|---|---|---|
| ☐ | D1 | CORE1 | `show ip ospf neighbor` | Every neighbour `FULL` | Step 08, Part B |
| ☐ | D2 | CORE1 | `show ip route ospf` | Three `O IA` routes: `10.10.16.0/20`, `10.10.32.0/20`, `10.10.48.0/20` | Step 09, Part D |
| ☐ | D2b | CORE1 | `show ip route static` | `S* 0.0.0.0/0 [1/0] via 10.10.1.254` (the ASA) | Step 11, Part B |
| ☐ | D3 | B-DIST1 | `show ip ospf virtual-links` | 2 links, both `up` | Step 09, Part C |
| ☐ | D4 | SWL3-HFA-A-DIST1, SWL3-JLM-CC1, SWL3-KG-DIST1 | `show standby brief` | Active / Standby exactly as in steps 05, 13, 14 | Steps 05 / 13 / 14 |
| ☐ | D5 | A-DIST1 and CC1 | `show etherchannel summary` | `Po1(SU)`, both members `(P)`: LACP in Haifa, PAgP in Jerusalem | Steps 04 / 13 |
| ☐ | D6 | R-JLM-EDGE | `show ip route eigrp` | `D 10.20.8.0/21` via two paths | Step 13, Part C |
| ☐ | D7 | R-ISPA-BR | `show ip bgp` | 3 prefixes: `200.1.0.0/16`, `200.2.0.0/16`, `200.3.0.0/16` | Step 12, Part A |
| ☐ | D8 | R-HFA-EDGE | `show crypto isakmp sa` (after a Jerusalem PC pings Haifa) | `QM_IDLE` with `200.2.1.2` | Step 15, Part B |
| ☐ | D9 | One device per branch: SW-HFA-A-ACC1, SWL3-JLM-CC1, SWL3-KG-DIST1 | `show ntp status` | Synchronized | Steps 11 / 14 / 15 |
| ☐ | D10 | SRV-HFA-SYSLOG and SRV-JLM-SYSLOG → Services → SYSLOG | Look | Messages with timestamps | Steps 08 / 13 |
| ☐ | D11 | PC-HFA-ENG-01 | `ping` PC-HFA-IT-01's address | ✅ (area 1 → 0 → 2 → 3) | Step 09 |
| ☐ | D12 | PC-HOME-01 → Web Browser | `http://www.intel.lab` | **Intel Israel - Intranet** | Step 12, Part D |
| ☐ | D13 | LT-HFA-GUEST-01 | `ping 10.10.1.10` | ❌ fails — **correct!** | Step 10, Part C |
| ☐ | D14 | PC-KG-FAB-01 → Web Browser | `http://www6.isp-web.com` | **IPv6 works!** | Step 14, Part C |
| ☐ | D15 | PC-HFA-RND-01 | `ssh -l admin 10.10.19.11` | ❌ refused — **correct!** | Step 09, Part E |

## Part E — Ready to submit

| ☐ | # | Check | Must be true |
|---|---|---|---|
| ☐ | E1 | Every device | Saved: `copy running-config startup-config` (`write memory` on the ASA) → `[OK]` |
| ☐ | E2 | **File → Save** | Title shows `final.pkt`. **No** break from Mission 8 tests 13–17 is left in it (A-DIST1 exists, R-HFA-EDGE Gi0/1 and R-ISPA-BR Gi0/2 are up, no rogue devices) |
| ☐ | E3 | `screenshots` folder | 24 test screenshots + the Physical and Logical screenshots of Mission 9a |
| ☐ | E4 | [docs/requirements-matrix.md](../../docs/requirements-matrix.md) | Every row ✅ |
| ☐ | E5 | Project book | Every chapter filled, in **your** words (Mission 9) |
| ☐ | E6 | Mission 10 | You answered all 5 questions out loud without notes |

---

## 🛠 Your own project
This is the test that matters most. Run **every** step's self-test once more on your **own**
final file, with your values (each self-test ends with a "replace" table), then this one.
Your project is ready when every box in every self-test is ticked.
