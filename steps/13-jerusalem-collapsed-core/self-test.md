# Step 13 — Self-test

> 🧪 Do this **after** you finish step 13, on your saved file `my-work/step13.pkt`.
> It takes about 30 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Press **Fast Forward Time** 3–4 times first (M18).
2. Do the tests **in order**, top to bottom.
3. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
4. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
5. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
6. Every box ticked → step 13 works. Go to step 14.

Log in everywhere with `admin` / `Intel#Adm1n`. Part E **breaks things on purpose** and repairs
them. "M…" numbers are the basic moves in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — The collapsed core

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | CC1 | `show etherchannel summary` | `Po1(SU)  PAgP  Gi1/0/23(P) Gi1/0/24(P)` | Mission 5b / 8a (`channel-group 1 mode desirable` on both) |
| ☐ | A2 | CC1 | `show standby brief` | **Active** for Vl41, Vl43, Vl45, Vl99, Vl100, Vl130 · **Standby** for Vl42, Vl44, Vl46 | Mission 6 |
| ☐ | A3 | CC2 | `show standby brief` | Active for Vl42, Vl44, Vl46 · Standby for the other 6 | Mission 8b |
| ☐ | A4 | CC1 | `show spanning-tree vlan 41` | `This bridge is the root` | Mission 5b |
| ☐ | A5 | SW-JLM-F1-ACC1 | `show vlan brief` | Fa0/1–10 in 41 · Fa0/11–20 in 42 · Fa0/21 in 130 · Fa0/22 in 99 | Mission 9a |

## Part B — DHCP from the server, through the relay

| ☐ | # | On / From | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | SRV-JLM-DHCP → Services → DHCP | Look | **On**, **7** pools: serverPool + JLM-SALES, -LOGISTICS, -PRODUCTION, -QA, -SECURITY, -ADMIN. Each gateway = `.1` of its VLAN | Mission 11d (Add, not Save) |
| ☐ | B2 | PC-JLM-SAL-01 | `ipconfig /all` | `10.20.10.11` or higher · gateway `10.20.10.1` · DHCP server `10.20.1.10` | `169.254…` → `ip helper-address 10.20.1.10` on the SVI, snooping trust on uplinks, option-82 line (troubleshooting table) |
| ☐ | B3 | One PC of each department (LOG, PRD, QA, SEC, ADM) | `ipconfig` | `10.20.11.x`, `10.20.12.x`, `10.20.13.x`, `10.20.14.x`, `10.20.15.x` | Wrong subnet → that pool's Default Gateway is wrong |
| ☐ | B4 | PC-JLM-SAL-01 | `ping 10.20.13.11` (a QA PC. Check its address first) | ✅ replies (inter-VLAN routing) | Part A |

## Part C — EIGRP and the edge router

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | R-JLM-EDGE | `show ip eigrp neighbors` | 2 neighbours: `10.20.4.2` (Gi0/0) and `10.20.4.6` (Gi0/2) | AS not `100`, key-chain / key-string typo, link not in `network` |
| ☐ | C2 | R-JLM-EDGE | `show ip route eigrp` | `D 10.20.8.0/21` with **two** paths (via `10.20.4.2` and `10.20.4.6`). **No** separate `10.20.10.0/24` … `10.20.15.0/24` | Six /24s → `ip summary-address eigrp 100 10.20.8.0 255.255.248.0` missing on a CC's Gi1/1/1 |
| ☐ | C3 | CC1 | `show ip route \| include 0.0.0.0` | `D*EX 0.0.0.0/0 [170/…] via 10.20.4.1` | `redistribute static` on R-JLM-EDGE (Mission 13b) |
| ☐ | C4 | PC-JLM-SAL-01 | `tracert 200.1.100.11` | `10.20.10.2` (or `10.20.10.1`) → `10.20.4.1` → `200.2.1.1` → … → `200.1.100.11` | — |

## Part D — Dynamic NAT and CBAC

| ☐ | # | On / From | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | D1 | PC-JLM-SAL-01 | `ping 200.1.100.11` | ✅ replies | NAT inside/outside; ISP-B's route to `200.2.10.0/29` (step 12, Mission 5b) |
| ☐ | D2 | R-JLM-EDGE | `show ip nat translations` | `--- 200.2.10.x  10.20.10.x ---`: one **pool** address per PC (no ports, no `overload`) | Mission 14 |
| ☐ | D3 | R-JLM-EDGE, **while** a PC runs `ping -t 200.1.100.11` | `show ip inspect sessions` | Sessions are listed | Mission 15a: `ip inspect JLM-FW out` on Gi0/1 |
| ☐ | D4 | PC-HOME-01 | `ping 200.2.1.2` | ❌ fails — **correct!** CBAC drops traffic nobody asked for | `ip access-group WAN-IN in` on Gi0/1 |
| ☐ | D5 | PC-JLM-ADMIN-01 | `ssh -l admin 10.20.0.12` | `SW-JLM-F2-ACC1#` | Mission 9b (management), AAA client list (Mission 11c) |
| ☐ | D6 | A Sales PC | `ssh -l admin 10.20.0.12` | ❌ refused — **correct!** | The ACL permits too much |

## Part E — Prove it: the dynamic-NAT limit (then clean up)

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | E1 | R-JLM-EDGE | `clear ip nat translation *` | — | — |
| ☐ | E2 | 7 **different** department PCs, one after the other | `ping -t 200.1.100.11` on each | PCs 1–6 reply. The **7th**: `Request timed out` | Only 6 addresses in the pool (`200.2.10.1`–`.6`). If the 7th works, `overload` was added by mistake |
| ☐ | E3 | R-JLM-EDGE | `show ip nat statistics` | All 6 pool addresses in use | — |
| ☐ | E4 | Clean up | **Ctrl+C** on all 7 PCs → on R-JLM-EDGE `clear ip nat translation *` | — | — |

## Part F — Physical view and saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | F1 | **Physical** → Intercity | Look | 3 cities: Haifa, Jerusalem, Internet, with lines between them | Mission 17a |
| ☐ | F2 | **Navigation** | Look under **Jerusalem → JLM Office** | `JLM Main Closet`: R-JLM-EDGE, CC1, CC2, SW-JLM-SRV · `Floor 1/2/3 Closet`: one ACC each · the building: 3 servers, 13 PCs, 3 printers. **No** `-JLM-` name left under Haifa | Mission 17b |
| ☐ | F3 | **Logical** tab | PC-JLM-SAL-01 → `ping 200.1.100.11` | ✅ still replies | Cable Length Effects must be **off** |
| ☐ | F4 | Every Jerusalem device | `copy running-config startup-config` | `[OK]` | — |
| ☐ | F5 | **File → Save** | — | Title shows `step13.pkt` | M1 |

---

## 🛠 Your own project
Run the same test in your own project. Replace:

| In the test | Use from your plan |
|---|---|
| VLANs 41–46, 99, 100, 130 and `10.20.x.x` | your branch's VLANs and subnets |
| Summary `10.20.8.0/21`, AS `100` | your summary and EIGRP AS |
| DHCP server `10.20.1.10`, its pools | your DHCP server and pools |
| Pool `200.2.10.1`–`.6` (6 addresses → the 7th PC fails) | your pool. The PC that fails is "pool size + 1" |

Your project passes when every row matches with **your** values. C2 is the most important
routing test: the edge router must see your departments as **one** summary, through **two** paths.
