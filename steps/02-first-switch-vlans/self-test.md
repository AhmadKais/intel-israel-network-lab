# Step 02 — Self-test

> 🧪 Do this **after** you finish step 02, on your saved file `my-work/step02.pkt`.
> It takes about 15 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Do the tests **in order**, top to bottom.
2. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
3. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
4. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
5. Every box ticked → step 02 works. Go to step 03.

Type the commands at the `SW-HFA-A-ACC1#` prompt (after the login in A1–A3). "M…" numbers
are the basic moves in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — Hardening

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | SW-HFA-A-ACC1 → CLI | Type `exit` until you see `Press RETURN to get started.` → **Enter** | The banner `Intel Israel - AUTHORIZED ACCESS ONLY`, then `Password:` | No banner → Mission 5c. No `Password:` → Mission 5d (`line con 0` → `password` → `login`) |
| ☐ | A2 | ACC1 | `Intel#Con` → **Enter** | `SW-HFA-A-ACC1>` | Keyboard on English? Caps Lock off? → Mission 5d |
| ☐ | A3 | ACC1 | `enable` → `Intel#En4ble` | `SW-HFA-A-ACC1#` | Mission 5b (`enable secret`) |
| ☐ | A4 | ACC1 | `show running-config` (Space = next page) | `enable secret 5 $1$…` (hashed) · under `line con 0`: `password 7 …` (scrambled) · `no ip domain-lookup` | Clear-text console password → `service password-encryption` is missing (Mission 5b) |
| ☐ | A5 | ACC1 | Type a wrong word, e.g. `shwo` → **Enter** | `% Unknown command…` or `Translating "shwo"…` that ends **at once** (no 30-second wait) | A long wait → `no ip domain-lookup` is missing |

## Part B — VLANs and ports

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | ACC1 | `show vlan brief` | Exactly as in Mission 7c: 10 ENGINEERING (Fa0/1–10), 20 FINANCE (Fa0/11–20), 30 PRINTERS-A (Fa0/21), 99 MGMT-A (Fa0/22), 110 WIFI-EMP, 120 WIFI-GUEST, 998 NATIVE, 999 PARKING (Fa0/23, Fa0/24). VLAN 1 has **only** Gig0/1, Gig0/2 | Wrong port → Mission 7. Name `VLAN0010` → Mission 6 |
| ☐ | B2 | ACC1 | `show interfaces status` | Fa0/23 and Fa0/24 are `disabled` (shut down) | Mission 7b — `shutdown` is missing |
| ☐ | B3 | Workspace | Hover over each cable at the switch | ENG-01 Fa0/1, ENG-02 Fa0/2, FIN-01 Fa0/11, FIN-02 Fa0/12, printer Fa0/21, ADMIN-01 Fa0/22 | Mission 4 — delete the cable (M7) and redo it |
| ☐ | B4 | Each PC + printer | Hover over it | ENG-01 `10.10.16.101`, ENG-02 `.16.102`, FIN-01 `10.10.17.101`, FIN-02 `.17.102`, ADMIN-01 `10.10.19.100`, printer `10.10.18.10` | Mission 8 |

## Part C — Who can talk to whom

| ☐ | # | From | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | PC-HFA-ENG-01 | `ping 10.10.16.102` | ✅ replies (same VLAN) | B1, B3, B4 |
| ☐ | C2 | PC-HFA-FIN-01 | `ping 10.10.17.102` | ✅ replies (same VLAN) | B1, B3, B4 |
| ☐ | C3 | PC-HFA-ENG-01 | `ping 10.10.17.101` | ❌ fails — **correct!** Different VLAN, no router yet | If it replies, a FIN PC is in VLAN 10 → B1 |

## Part D — Saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | D1 | ACC1 | `show startup-config` | The config, with `hostname SW-HFA-A-ACC1` near the top. **Not** `startup-config is not present` | Mission 7d: `copy running-config startup-config` |
| ☐ | D2 | **File → Save** | — | Title shows `step02.pkt` | M1 |

---

## 🛠 Your own project
Run the same test on your own first access switch. Replace:

| In the test | Use from your plan |
|---|---|
| `SW-HFA-A-ACC1` | your access-switch hostname |
| `Intel Israel - AUTHORIZED ACCESS ONLY` | your banner text |
| `Intel#Con`, `Intel#En4ble` | your console password and enable secret |
| VLANs 10/20/30/99/110/120/998/999 and their port ranges | your VLAN table and port plan |
| `10.10.16.x`, `10.10.17.x` … | your PC addresses |

Your project passes when A1–A5, B1–B4, C1–C3 and D1–D2 all match with **your** values.
