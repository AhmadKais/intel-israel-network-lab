# Step 05 — Self-test

> 🧪 Do this **after** you finish step 05, on your saved file `my-work/step05.pkt`.
> It takes about 25 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Press **Fast Forward Time** 2–3 times first (M18).
2. Do the tests **in order**, top to bottom.
3. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
4. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
5. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
6. Every box ticked → step 05 works. Go to step 06.

Part D **breaks things on purpose** and repairs them. Do every repair line before you go on.
"M…" numbers are the basic moves in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — Gateways (HSRP)

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | DIST1 | `show standby brief` | 6 lines. **Active** for Vl10, Vl30, Vl99, Vl110 · **Standby** for Vl20, Vl120 · every line has `P` · Virtual IPs `10.10.16.1`, `.17.1`, `.18.1`, `.19.1`, `.20.1`, `.21.1` | Mission 2–3. Missing line → that `interface vlan` has no `standby` lines |
| ☐ | A2 | DIST2 | `show standby brief` | The **opposite** of A1: Active for Vl20, Vl120 · Standby for the other 4 | Both say Active for one group → `standby version 2` missing on one side, or the VLAN isn't on Po1 (step 04) |
| ☐ | A3 | DIST1 | `show ip interface brief` | Vlan10, 20, 30, 99, 110, 120 are `up` `up`, with `.2` addresses (`10.10.16.2` …) | Mission 3. `down` → the VLAN has no active port/trunk |
| ☐ | A4 | DIST1 | `show ip route` | `C 10.10.16.0/24 … Vlan10`, `C 10.10.17.0/24 … Vlan20` … one `C` line per VLAN | `ip routing` missing (Mission 2) |

## Part B — DHCP

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | Each of the 6 user PCs | Desktop → IP Configuration | **DHCP** is selected. Address `.11`–`.254` of its VLAN, mask `255.255.255.0`, gateway `.1` (`10.10.16.1` ENG / `10.10.17.1` FIN), DNS `10.10.200.11` | APIPA `169.254…` → Fast Forward Time, then **Static** → **DHCP**. Still failing → Missions 6–7 |
| ☐ | B2 | A user PC → Command Prompt | `ipconfig /all` | `Default Gateway: 10.10.16.1` (or `.17.1`) · `DHCP Servers: 10.10.16.2` or `.16.3` (`.17.2` / `.17.3` for FIN) · `DNS Servers: 10.10.200.11` | B1 |
| ☐ | B3 | DIST1 | `show ip dhcp binding` | Addresses only between `.11` and `.130` | Mission 6a (excluded ranges) |
| ☐ | B4 | DIST2 | `show ip dhcp binding` | Only `.131`+ addresses, or an empty list (whoever answered first) | Mission 7 (DIST2 must exclude `.1`–`.130`) |
| ☐ | B5 | Printer and PC-HFA-ADMIN-01 | Hover over them | Still **static**: printer `10.10.18.10`, admin `10.10.19.100` | Put them back to static (step 02 Mission 8) |

## Part C — Routing between VLANs

Write down the address of one FIN PC first (hover over it).

| ☐ | # | From | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | An ENG PC | `ping 10.10.16.1` | ✅ replies (its gateway) | Part A |
| ☐ | C2 | An ENG PC | `ping <the FIN PC's address>` | ✅ replies. **Inter-VLAN routing works** | C1 works but C2 fails → `ip routing` (Mission 2), or the FIN PC has no gateway (B1) |
| ☐ | C3 | An ENG PC | `ping 10.10.18.10` (printer) | ✅ replies | Printer gateway must be `10.10.18.1` (step 02 Mission 8b) |
| ☐ | C4 | An ENG PC | `tracert <the FIN PC's address>` | 1st line `10.10.16.2` (DIST1, Active for VLAN 10). Some PT versions show `10.10.16.1`, which is also correct. Last line = the FIN PC | — |
| ☐ | C5 | PC-HFA-ADMIN-01 | `ping 10.10.19.11`, then `ping 10.10.19.12` | ✅ both reply (ACC1 and ACC2 management) | Mission 8 (`interface vlan 99`, `ip default-gateway 10.10.19.1`) |

## Part D — Prove it: kill a gateway (then repair)

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | D1 | An ENG PC | `ping -t <the FIN PC's address>` (leave it running) | Replies | Part C |
| ☐ | D2 | DIST1 | `configure terminal` → `interface vlan 10` → `shutdown` → `end` | The ping: a few timeouts (up to 10 s), then replies again | No replies again → DIST2's Vl10 standby lines (Mission 4) |
| ☐ | D3 | DIST2 | `show standby brief` | DIST2 is now **Active** for Vl10 too | — |
| ☐ | D4 | **Another** ENG PC | `ipconfig /release`, then `ipconfig /renew` | A **`.131`+** address, from DIST2 | Mission 7 |
| ☐ | D5 | Repair, DIST1 | `configure terminal` → `interface vlan 10` → `no shutdown` → `end` → Fast Forward Time → `show standby brief` | DIST1 is **Active** for Vl10 again (that's preempt) | `standby 10 preempt` missing on DIST1 (Mission 2) |
| ☐ | D6 | The ping window | **Ctrl+C** | — | — |

## Part E — Saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | E1 | Each of the 4 switches | `copy running-config startup-config` → **Enter** | `[OK]` | — |
| ☐ | E2 | **File → Save** | — | Title shows `step05.pkt` | M1 |

---

## 🛠 Your own project
Run the same test in your own project. Replace:

| In the test | Use from your plan |
|---|---|
| VLAN numbers and `10.10.16.x` … `10.10.21.x` | your VLANs and subnets (gateway always `.1`, real IPs `.2` / `.3`) |
| Which switch is Active for which VLAN | the same split as your STP bosses in step 04 |
| DHCP split `.11`–`.130` / `.131`–`.254` | your split |
| DNS `10.10.200.11` | your future internal DNS server address |
| Management `10.10.19.11` / `.12`, admin PC `10.10.19.100` | your management addresses |

Your project passes when every row matches with **your** values. A1 and A2 must agree with
step 04: the STP boss of a VLAN must be its HSRP Active too.
