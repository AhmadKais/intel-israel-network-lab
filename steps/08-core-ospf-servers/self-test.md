# Step 08 — Self-test

> 🧪 Do this **after** you finish step 08, on your saved file `my-work/step08.pkt`.
> It takes about 30 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Press **Fast Forward Time** 3–4 times first (M18). OSPF needs a little time.
2. Do the tests **in order**, top to bottom.
3. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
4. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
5. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
6. Every box ticked → step 08 works. Go to step 09.

Log in everywhere with `admin` / `Intel#Adm1n`. Part E **breaks things on purpose** and repairs
them. "M…" numbers are the basic moves in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — Addresses and cables

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | CORE1 | `show ip interface brief` | `Gi1/0/1 10.10.4.2` · `Gi1/0/2 10.10.4.10` · `Gi1/0/23 10.10.4.49` · `Vlan100 10.10.1.2` · `Loopback0 10.10.5.1`, all `up up` | Mission 5. `% Invalid input` on `ip address` → `no switchport` first. `down` → cable (Mission 3) |
| ☐ | A2 | CORE2 | `show ip interface brief` | `Gi1/0/1 10.10.4.6` · `Gi1/0/2 10.10.4.14` · `Gi1/0/23 10.10.4.50` · `Vlan100 10.10.1.3` · `Loopback0 10.10.5.2`, all `up up` | Mission 7 |
| ☐ | A3 | DIST1 | `show ip interface brief` | `Gi1/1/1 10.10.4.1` · `Gi1/1/2 10.10.4.5` · `Loopback0 10.10.5.11` | Mission 8a |
| ☐ | A4 | DIST2 | `show ip interface brief` | `Gi1/1/1 10.10.4.9` · `Gi1/1/2 10.10.4.13` · `Loopback0 10.10.5.12` | Mission 9 |

## Part B — OSPF

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | CORE1 | `show ip ospf neighbor` | **3** lines, all `FULL`: `10.10.5.11` (Gi1/0/1), `10.10.5.12` (Gi1/0/2), `10.10.5.2` (Gi1/0/23). DR/BDR labels may differ | Stuck INIT/EXSTART or missing → MD5 key typo, port still passive, or wrong side of the /30 |
| ☐ | B2 | CORE2 | `show ip ospf neighbor` | 3 lines, all `FULL`: `10.10.5.11`, `10.10.5.12`, `10.10.5.1` | Mission 7c |
| ☐ | B3 | DIST1 | `show ip ospf neighbor` | 3 lines, all `FULL`: `10.10.5.1`, `10.10.5.2` **and** `10.10.5.12` (DIST2, over Vlan99) | DIST2 missing → Mission 8b (MD5 on Vlan99) / `no passive-interface vlan 99` |
| ☐ | B4 | CORE1 | `show ip route ospf` | `O IA 10.10.16.0/20` with **two** lines: `via 10.10.4.1` and `via 10.10.4.9`. **No** separate `10.10.16.0/24`, `10.10.17.0/24` … | No `/20` → `area 1 range 10.10.16.0 255.255.240.0` (Mission 8c / 9). Only one path → B1 |
| ☐ | B5 | DIST1 | `show ip ospf interface` | Gi1/1/1, Gi1/1/2, Loopback0 → `Area 0` · the VLANs → `Area 1` | Mission 8c (`network` lines) |
| ☐ | B6 | DIST1 | `show ip ospf interface gi1/1/1` | `Message digest authentication enabled` | Mission 8a — `ip ospf authentication message-digest` |

## Part C — Servers and gateways

| ☐ | # | On / From | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | CORE1 | `show standby brief` | `Vl100  100  110 P Active … 10.10.1.1` | Mission 5b |
| ☐ | C2 | CORE2 | `show standby brief` | `Vl100` … `Standby` … `10.10.1.1` | Mission 7b |
| ☐ | C3 | SRV-HFA-SYSLOG and SRV-HFA-AAA → IP Configuration | Look | `10.10.1.10` / `10.10.1.11`, mask `255.255.255.0`, gateway `10.10.1.1`, DNS `10.10.200.11` | Mission 12a |
| ☐ | C4 | An ENG PC | `ping 10.10.1.1` | ✅ replies (through the core) | Part B |
| ☐ | C5 | An ENG PC | `ping 10.10.1.10`, then `ping 10.10.1.11` | ✅ both reply | C3 (the servers' gateway) |
| ☐ | C6 | An ENG PC | `tracert 10.10.1.1` | First line `10.10.16.2` (DIST1), last line `10.10.1.1`. If it went through CORE2 you also see `10.10.4.6` in the middle. Both are correct | — |
| ☐ | C7 | PC-HFA-ADMIN-01 | `ssh -l admin 10.10.1.5` | `SW-HFA-SRV#` | Mission 11 |

## Part D — AAA and Syslog

| ☐ | # | On / From | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | D1 | SRV-HFA-AAA → Services → AAA | Look | Service **On** · **7** clients, secret `Intel#Tac`, type Tacacs · user `admin` | Mission 13 |
| ☐ | D2 | PC-HFA-ADMIN-01 | `ssh -l admin 10.10.5.1` → `Intel#Adm1n` | `SWL3-HFA-CORE1#` | Mission 14. Always the local user → client IP doesn't match (Mission 14b note) |
| ☐ | D3 | PC-HFA-ADMIN-01 | The same SSH to `10.10.5.2`, `10.10.5.11`, `10.10.5.12`, `10.10.19.11`, `10.10.19.12`, `10.10.1.5` | The right prompt on each of the 7 switches | Mission 14 on that switch |
| ☐ | D4 | ACC1 | `configure terminal` → `interface fa0/24` → `no shutdown` → `shutdown` → `end` | — | — |
| ☐ | D5 | SRV-HFA-SYSLOG → Services → SYSLOG | Look | New messages from ACC1 (`10.10.19.11`) with time | SYSLOG service off, wrong `logging host`, or ACC1 can't reach 10.10.1.10 |

## Part E — Prove it (break it on purpose, then repair)

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | E1 | SRV-HFA-AAA → Physical | Click the **power button** (light off) | — | — |
| ☐ | E2 | PC-HFA-ADMIN-01 | `ssh -l admin 10.10.5.1` → `Intel#Adm1n` | **Still works** (local user = backup). It may take a few seconds longer | `aaa authentication login default group tacacs+ local`: the word `local` is missing |
| ☐ | E3 | Repair | Power SRV-HFA-AAA back **on**. Fast Forward Time | D2 works | — |

## Part F — Saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | F1 | All 7 switches | `copy running-config startup-config` → **Enter** | `[OK]` | — |
| ☐ | F2 | **File → Save** | — | Title shows `step08.pkt` | M1 |

---

## 🛠 Your own project
Run the same test in your own project. Replace:

| In the test | Use from your plan |
|---|---|
| Links `10.10.4.x/30`, loopbacks `10.10.5.x` | your link and loopback addresses |
| Area 1 summary `10.10.16.0/20` | your building's block |
| VLAN 100 `10.10.1.0/24`, servers `.10` / `.11` | your server VLAN and server addresses |
| `Intel#Ospf`, `Intel#Tac` | your OSPF key and TACACS key |

Your project passes when every row matches with **your** values. B4 is the most important
test: the core must see **one** summary route for the building, through **two** paths.
