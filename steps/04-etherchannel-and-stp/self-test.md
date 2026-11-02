# Step 04 — Self-test

> 🧪 Do this **after** you finish step 04, on your saved file `my-work/step04.pkt`.
> It takes about 25 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Press **Fast Forward Time** 2–3 times first (M18).
2. Do the tests **in order**, top to bottom.
3. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
4. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
5. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
6. Every box ticked → step 04 works. Go to step 05.

Part C **breaks things on purpose** and repairs them. Do every repair line before you go on.
"M…" numbers are the basic moves in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — EtherChannel

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | Workspace | Look between DIST1 and DIST2 | 2 **dashed** cables, `Gi1/0/23` ↔ `Gi1/0/23` and `Gi1/0/24` ↔ `Gi1/0/24`, green | Mission 2 |
| ☐ | A2 | DIST1 | `show etherchannel summary` | `1  Po1(SU)  LACP  Gig1/0/23(P) Gig1/0/24(P)` | `(SD)` / `(I)` / `(s)` → the two members differ, or DIST2 is missing its half (Missions 4–5, troubleshooting table) |
| ☐ | A3 | DIST2 | `show etherchannel summary` | The same line as A2 | Mission 5b |
| ☐ | A4 | DIST1 | `show interfaces trunk` | `Po1` is listed, `trunking`, native `998`. `Gig1/0/1` and `Gig1/0/2` are still there | Mission 4b. Members (Gi1/0/23–24) are **not** listed on their own; only `Po1` is. That's correct |

## Part B — Spanning Tree: who is the boss

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | DIST1 | `show spanning-tree vlan 10` | `Spanning tree enabled protocol rstp` and `This bridge is the root` | Mission 3 |
| ☐ | B2 | DIST1 | `show spanning-tree vlan 30`, then `99`, then `110` | `This bridge is the root` each time | Mission 3: `spanning-tree vlan 10,30,99,110 root primary` |
| ☐ | B3 | DIST2 | `show spanning-tree vlan 20`, then `120` | `This bridge is the root` each time | Mission 5a |
| ☐ | B4 | DIST1 | `show spanning-tree vlan 20` | **No** `This bridge is the root` line. DIST2 is the boss here | Mission 3 — the `root secondary` line has the wrong VLANs |
| ☐ | B5 | ACC1 | `show spanning-tree vlan 10` | `Gi0/1` = `Root FWD` · `Gi0/2` = `Altn BLK` · `Fa0/1` … `P2p Edge` | Root on Gi0/2 → B1 failed. No `Edge` → Mission 6a (PortFast) |
| ☐ | B6 | ACC1 | `show spanning-tree vlan 20` | This time **`Gi0/2`** is the Root port, `Gi0/1` is `Altn BLK` | B3 |
| ☐ | B7 | ACC1 and ACC2 | `show spanning-tree summary` | `Switch is in rapid-pvst mode` | Mission 6 |

## Part C — Prove it (break it on purpose, then repair)

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | PC-HFA-ENG-01 | `ping -t 10.10.16.103` (leave it running) | `Reply from …` again and again | Step 03 self-test, Part D |
| ☐ | C2 | Workspace | Delete the cable ACC1 Gi0/1 ↔ DIST1 (M7). Watch the ping | 1–2 `Request timed out`, then `Reply from` again | B5: Gi0/2 must be a backup (`Altn`) |
| ☐ | C3 | Repair | **Ctrl+C** in the ping. New **Cross-Over**: ACC1 **Gi0/1** → DIST1 **Gi1/0/1**. Fast Forward Time | B5 matches again | — |
| ☐ | C4 | Workspace + DIST1 | Delete **one** DIST1–DIST2 cable → `show etherchannel summary` | `Po1(SU)` still up, now with **one** `(P)` member | A2 |
| ☐ | C5 | Repair | Cross-Over again on the same ports (Gi1/0/23 or 24 at both ends) → Fast Forward Time → `show etherchannel summary` | Both members `(P)` again | Same port number at both ends? |
| ☐ | C6 | Workspace | Drag a new **2960** next to ACC1. Cross-Over: new switch **Fa0/1** → ACC1 **Fa0/9** | ACC1 shows `%SPANTREE-2-BLOCK_BPDUGUARD: Received BPDU on port Fa0/9 …` | Mission 6a (`bpduguard enable`) |
| ☐ | C7 | ACC1 | `show interfaces status` | Fa0/9 = `err-disabled` | C6 |
| ☐ | C8 | Repair | Delete the rogue switch (M7). On ACC1: `configure terminal` → `interface fa0/9` → `shutdown` → `no shutdown` → `end` | `show interfaces status`: Fa0/9 is `notconnect` (nothing plugged in), not `err-disabled` | Do the `shutdown` / `no shutdown` again |

## Part D — Saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | D1 | Each of the 4 switches | `copy running-config startup-config` → **Enter** | `[OK]` | — |
| ☐ | D2 | **File → Save** | — | Title shows `step04.pkt` | M1 |

---

## 🛠 Your own project
Run the same test in your own project. Replace:

| In the test | Use from your plan |
|---|---|
| VLANs 10/30/99/110 (DIST1 boss) and 20/120 (DIST2 boss) | your split of VLANs between the two distribution switches |
| Ports Gi1/0/23–24, Po1 | your EtherChannel ports and group number |
| Native `998` | your native VLAN |
| `10.10.16.103`, ACC1/DIST1 names | your PC address and your switch names |

Your project passes when every row matches with **your** values. Make sure each
distribution switch is the boss of **its own** VLANs. If one switch is the boss of every
VLAN, the load isn't shared.
