# Step 09 — Self-test

> 🧪 Do this **after** you finish step 09, on your saved file `my-work/step09.pkt`.
> It takes about 35 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Press **Fast Forward Time** 3–4 times first (M18). OSPF needs a little time.
2. Do the tests **in order**, top to bottom.
3. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
4. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
5. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
6. Every box ticked → step 09 works. Go to step 10.

Log in everywhere with `admin` / `Intel#Adm1n`. Part F **breaks things on purpose** and repairs
them. "M…" numbers are the basic moves in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — Building B

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | B-DIST1 | `show etherchannel summary` | `Po1(SU)`, both members `(P)` | Mission 5c / 7a |
| ☐ | A2 | B-DIST1 | `show standby brief` | **Active** for Vl40, Vl60, Vl99 · **Standby** for Vl50 | Mission 6a |
| ☐ | A3 | B-DIST2 | `show standby brief` | Active for Vl50 · Standby for Vl40, Vl60, Vl99 | Mission 7b |
| ☐ | A4 | B-ACC1 and B-ACC2 | `show vtp status` | Domain `HFA-B`, mode `Client` | Mission 8 |
| ☐ | A5 | B-DIST1 | `show ip ospf neighbor` | CORE1 (`10.10.5.1`), CORE2 (`10.10.5.2`), B-DIST2 (`10.10.5.22`), C-DIST1 (`10.10.63.1`), C-DIST2 (`10.10.63.2`), all `FULL`. C-DIST1/2 may appear twice: once on `Gi1/1/x` and once on `OSPF_VL…` (the virtual link). Both are correct | Core missing → Mission 9 (core ports). C missing → Mission 11d / 12c (MD5 on the 10.10.36.x links) |
| ☐ | A6 | PC-HFA-HR-01 | `ipconfig` | `10.10.32.x`, gateway `10.10.32.1` | Mission 10a. Troubleshooting table |
| ☐ | A7 | PC-HFA-MKT-01 | `ipconfig` | `10.10.33.x`, gateway `10.10.33.1` | Mission 10a |

## Part B — Building C

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | C-DIST1 | `show standby brief` | **Active** for Vl70, Vl90, Vl99 · **Standby** for Vl80 | Mission 11c |
| ☐ | B2 | C-DIST2 | `show standby brief` | Active for Vl80 · Standby for Vl70, Vl90, Vl99 | Mission 12b |
| ☐ | B3 | C-ACC1 and C-ACC2 | `show vtp status` | Domain `HFA-C`, mode `Client` | Mission 13 |
| ☐ | B4 | PC-HFA-RND-01 | `ipconfig` | `10.10.48.x`, gateway `10.10.48.1` | Mission 14a. Domain `HFA-C` typo? Snooping trust on uplinks? |
| ☐ | B5 | PC-HFA-IT-01 | `ipconfig` | `10.10.49.x`, gateway `10.10.49.1` | Mission 14a |

## Part C — The virtual links

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | B-DIST1 | `show ip ospf virtual-links` | `Virtual Link … to router 10.10.63.1 is up` **and** `… to router 10.10.63.2 is up` · `Transit area 2` | `is down` → you typed an interface IP instead of the router-ID, or the area isn't `2` (Mission 15a) |
| ☐ | C2 | B-DIST2 | `show ip ospf virtual-links` | The same two links, both `up` | Mission 15a on B-DIST2 |
| ☐ | C3 | C-DIST1 | `show ip ospf virtual-links` | Links to `10.10.5.21` and `10.10.5.22`, both `up` | Mission 15b |
| ☐ | C4 | C-DIST2 | `show ip ospf virtual-links` | Links to `10.10.5.21` and `10.10.5.22`, both `up` | Mission 15b |

## Part D — The core sees every building as one route

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | D1 | CORE1 | `show ip route ospf` | Exactly **three** building routes, each `O IA`: `10.10.16.0/20`, `10.10.32.0/20`, `10.10.48.0/20` | No `48.0/20` → Part C. Many `10.10.3x.0/24` or `10.10.36.x/30` lines → `area 2 range 10.10.32.0 255.255.240.0` missing on B-DIST1/2 **or** C-DIST1/2. Many `10.10.4x/5x` lines → `area 3 range` missing on C-DIST |
| ☐ | D2 | CORE2 | `show ip route ospf` | The same three `/20` routes | — |

## Part E — Pings and SSH

| ☐ | # | From | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | E1 | PC-HFA-RND-01 | `ping 10.10.1.10` | ✅ replies | Part C |
| ☐ | E2 | PC-HFA-HR-01 | `ping 10.10.1.10` | ✅ replies | Part A |
| ☐ | E3 | PC-HFA-IT-01 | `ping` an Engineering PC in building A | ✅ replies | — |
| ☐ | E4 | PC-HFA-HR-01 | `ping` an R&D PC | ✅ replies | — |
| ☐ | E5 | PC-HFA-IT-01 | `ssh -l admin 10.10.19.11` (A-ACC1) | `SW-HFA-A-ACC1#` (IT may manage) | Mission 16: `show access-lists MGMT-ONLY` on A-ACC1 → 4 permits, **then** the deny |
| ☐ | E6 | PC-HFA-IT-01 | `ssh -l admin 10.10.51.11` (C-ACC1) | `SW-HFA-C-ACC1#` | C-ACC1's MGMT-ONLY (Mission 13a) |
| ☐ | E7 | PC-HFA-RND-01 | `ssh -l admin 10.10.51.11` | ❌ refused — **correct!** R&D isn't on the guest list | The ACL permits too much |
| ☐ | E8 | SRV-HFA-AAA → Services → AAA | Look | **15** clients | Mission 17 |

## Part F — Prove it: lose one B-DIST (then repair)

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | F1 | **File → Save** first | — | — | — |
| ☐ | F2 | PC-HFA-RND-01 | `ping -t 10.10.1.10` (leave it running) | Replies | E1 |
| ☐ | F3 | B-DIST1 | `configure terminal` → `interface range gi1/1/1 - 4` → `shutdown` → `end` | The ping: a few timeouts, then replies again (through B-DIST2) | Part C: C-DIST must have a link to `10.10.5.22` too |
| ☐ | F4 | C-DIST1 | `show ip ospf virtual-links` | The link to `10.10.5.22` is still `up` | Mission 15b |
| ☐ | F5 | Repair, B-DIST1 | `configure terminal` → `interface range gi1/1/1 - 4` → `no shutdown` → `end` → Fast Forward Time | C1 matches again. **Ctrl+C** the ping | — |

## Part G — Physical view and saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | G1 | **Physical** tab | Look | **Intercity › Haifa** with 4 buildings: `HFA Data Center`, `HFA Building A`, `HFA Building B`, `HFA Building C` | Mission 19b–c |
| ☐ | G2 | **Navigation** | Open the tree | Data Center: only CORE1, CORE2, SW-HFA-SRV and the 2 servers · each building's 4 switches in its closet · its PCs and printer in the building | Mission 19d — drag the device in the tree |
| ☐ | G3 | **Logical** tab | PC-HFA-HR-01 → `ping 10.10.1.10` | ✅ still replies | Cable Length Effects must be **off** (Mission 19a) |
| ☐ | G4 | Every changed switch | `copy running-config startup-config` → **Enter** | `[OK]` | — |
| ☐ | G5 | **File → Save** | — | Title shows `step09.pkt` | M1 |

---

## 🛠 Your own project
Run the same test in your own project. Replace:

| In the test | Use from your plan |
|---|---|
| Building blocks `10.10.32.0/20`, `10.10.48.0/20` | your building blocks |
| Router-IDs `10.10.5.21/.22`, `10.10.63.1/.2` | your loopbacks |
| VLANs 40/50/60/99 and 70/80/90/99 and their Active switch | your VLANs and your split |
| Domains `HFA-B`, `HFA-C` | your VTP domains |
| IT `10.10.49.0` / R&D `10.10.48.0` | your management-allowed department and one that isn't |

Your project passes when every row matches with **your** values. In your project the
building **without** a link to the core needs the virtual link. Find it in your plan before
you start.
