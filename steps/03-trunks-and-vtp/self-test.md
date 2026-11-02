# Step 03 — Self-test

> 🧪 Do this **after** you finish step 03, on your saved file `my-work/step03.pkt`.
> It takes about 20 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Press **Fast Forward Time** 2–3 times first (M18).
2. Do the tests **in order**, top to bottom.
3. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
4. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
5. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
6. Every box ticked → step 03 works. Go to step 04.

Type the `show` commands at the `#` prompt (console password `Intel#Con`, then `enable` →
`Intel#En4ble`). "M…" numbers are the basic moves in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — Devices and cables

| ☐ | # | Type / do | Must see | If not |
|---|---|---|---|---|
| ☐ | A1 | Look at DIST1 and DIST2 | Both are on (lights). The ports have green/orange dots, not dark | Power supply missing → Mission 2b |
| ☐ | A2 | Look at the 4 uplinks | **Dashed** lines (Cross-Over). One or more dots may be orange (STP blocking — correct) | Red dot / solid line → Straight-Through by mistake. Mission 4a |
| ☐ | A3 | Hover over the uplinks | ACC1 Gi0/1 → DIST1 Gi1/0/1 · ACC1 Gi0/2 → DIST2 Gi1/0/1 · ACC2 Gi0/1 → DIST1 Gi1/0/2 · ACC2 Gi0/2 → DIST2 Gi1/0/2 | Mission 4a. `Gi1/1/1` is the wrong port |

## Part B — VTP

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | DIST1 | `show vtp status` | `VTP version running : 2` · `VTP Domain Name : HFA-A` · `VTP Operating Mode : Server` · write down the **Configuration Revision** | Mission 5b |
| ☐ | B2 | DIST2 | `show vtp status` | Domain `HFA-A` · Mode `Client` · **same revision** as B1 | Mission 6b |
| ☐ | B3 | ACC1 | `show vtp status` | Domain `HFA-A` · Mode `Client` · same revision | Mission 7 |
| ☐ | B4 | ACC2 | `show vtp status` | Domain `HFA-A` · Mode `Client` · same revision | Mission 8b |
| ☐ | B5 | ACC2 | `show vlan brief` | All 8 VLANs (10, 20, 30, 99, 110, 120, 998, 999) with their names. You never typed them on ACC2 | Domain, password or version differ. On both switches compare `show vtp status` and `show vtp password` (case-sensitive) |
| ☐ | B6 | ACC2 | `configure terminal` → `vlan 60` | `VTP VLAN configuration not allowed when device is in CLIENT mode.` Then type `end` | ACC2 isn't a client → Mission 8b |

## Part C — Trunks

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | DIST1 | `show interfaces trunk` | `Gig1/0/1` and `Gig1/0/2` · Mode `on` · `802.1q` · `trunking` · Native vlan `998` · allowed `10,20,30,99,110,120,998` | Mission 5d |
| ☐ | C2 | DIST2 | `show interfaces trunk` | The same as C1 | Mission 6c |
| ☐ | C3 | ACC1 | `show interfaces trunk` | `Gig0/1` and `Gig0/2` · `trunking` · native `998` · same allowed list | Mission 7 |
| ☐ | C4 | ACC2 | `show interfaces trunk` | `Gig0/1` and `Gig0/2` · `trunking` · native `998` · same allowed list | Mission 8b |
| ☐ | C5 | Any switch | Watch the CLI for 1 minute (Fast Forward Time) | **No** `%CDP-4-NATIVE_VLAN_MISMATCH` messages | One side of a trunk is missing `native vlan 998` |

## Part D — Pings across the trunks

| ☐ | # | From | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | D1 | PC-HFA-ENG-01 (ACC1) | `ping 10.10.16.103` (ENG-03 on ACC2) | ✅ replies | Part C. Is ENG-03 on ACC2 Fa0/1 in VLAN 10 (`show vlan brief` on ACC2)? Its IP → Mission 4c |
| ☐ | D2 | PC-HFA-FIN-01 (ACC1) | `ping 10.10.17.103` (FIN-03 on ACC2) | ✅ replies | The same, for VLAN 20 / Fa0/11 |
| ☐ | D3 | PC-HFA-ENG-01 | `ping 10.10.17.103` | ❌ fails — **correct!** No router yet | If it replies, FIN-03 is in VLAN 10 by mistake |

## Part E — VTP really works (add a VLAN, then remove it)

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | E1 | DIST1 | `configure terminal` → `vlan 50` → `name TEST` → `end` | — | — |
| ☐ | E2 | ACC2 | `show vlan brief` | `50 TEST` is in the list | Part B |
| ☐ | E3 | DIST1 | `configure terminal` → `no vlan 50` → `end` | — | — |
| ☐ | E4 | ACC2 | `show vlan brief` | VLAN 50 is **gone** | Fast Forward Time, check again |

## Part F — Saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | F1 | Each of the 4 switches | `copy running-config startup-config` → **Enter** | `[OK]` | — |
| ☐ | F2 | **File → Save** | — | Title shows `step03.pkt` | M1 |

---

## 🛠 Your own project
Run the same test in your own project. Replace:

| In the test | Use from your plan |
|---|---|
| Domain `HFA-A`, password `Intel#Vtp` | your VTP domain and password |
| Native VLAN `998`, allowed list `10,20,30,99,110,120,998` | your native VLAN and your VLAN list |
| Switch names and ports | your building's switches and uplink ports |
| `10.10.16.103`, `10.10.17.103` | the addresses of your PCs on the second access switch |

Your project passes when every row matches with **your** values. E1–E4 can use any free VLAN
number. Remove it afterwards.
