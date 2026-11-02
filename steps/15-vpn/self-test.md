# Step 15 — Self-test

> 🧪 Do this **after** you finish step 15, on your saved file `my-work/step15.pkt`.
> It takes about 25 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Press **Fast Forward Time** 2–3 times first (M18).
2. Do the tests **in order**, top to bottom.
3. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
4. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
5. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
6. Every box ticked → step 15 works. Go to step 16.

Routers: `admin` / `Intel#Adm1n`. ASA: `enable` → `Intel#En4ble`. "M…" numbers are the basic
moves in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — Licence and the GRE tunnel

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | R-HFA-EDGE and R-JLM-EDGE | `show version` | In the licence table: `securityk9` **Permanent** / **Active** (or `RightToUse`) | Mission 3. `crypto` commands refused → you skipped the **reload** |
| ☐ | A2 | R-HFA-EDGE | `show interfaces tunnel 0` | `Tunnel0 is up, line protocol is up` · address `172.16.12.1` | Mission 4a |
| ☐ | A3 | R-JLM-EDGE | `ping 172.16.12.1` | `!!!!!` | WAN-IN must have `permit gre host 200.1.1.2 host 200.2.1.2` (step 13) |
| ☐ | A4 | R-HFA-EDGE | `show ip route static` | `S 10.20.0.0/16 [1/0] via 172.16.12.2` | Mission 4a |
| ☐ | A5 | R-JLM-EDGE | `show ip route static` | `S 10.10.0.0/16 [1/0] via 172.16.12.1` | Mission 4b |
| ☐ | A6 | CC1 (Jerusalem) | `show ip route \| include 10.10.0.0` | `D EX 10.10.0.0/16 … via 10.20.4.1` | `redistribute static` on R-JLM-EDGE (step 13) |

## Part B — Site-to-site VPN: Jerusalem ↔ Haifa

| ☐ | # | From / On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | PC-JLM-SAL-01 | `ping 10.10.16.1` | ✅ replies (the first one or two may time out while the tunnel builds) | ASA line `permit ip 10.20.0.0 255.255.0.0 10.10.0.0 255.255.0.0` (Mission 7a); A4/A5 |
| ☐ | B2 | R-HFA-EDGE | `show crypto isakmp sa` | `200.2.1.2  200.1.1.2  QM_IDLE … ACTIVE` | No line → the two `crypto isakmp policy 10` blocks or keys differ. Compare letter by letter |
| ☐ | B3 | R-HFA-EDGE, **while** the Jerusalem PC pings | `show crypto ipsec sa` | `#pkts encaps` and `#pkts decaps` are **not 0** and grow each time you run it | The crypto ACL doesn't match GRE between the two public addresses (Mission 5b) |
| ☐ | B4 | PC-JLM-SAL-01 | `tracert 10.10.200.13` | `10.20.10.2` → `10.20.4.1` → **`172.16.12.1`** (the tunnel) → … | A4 / A5 |
| ☐ | B5 | PC-JLM-SAL-01 → Web Browser | `http://www.intel.lab` | **Intel Israel - Intranet** | DNS 10.10.200.11 must be reachable (B1) |
| ☐ | B6 | PC-JLM-SAL-01 | `nslookup ftp.intel.lab` | `10.10.200.15` | Step 11, DNS record |
| ☐ | B7 | CC1 (after Fast Forward Time) | `show ntp status` | Synchronized to `10.10.200.14` | B1 works but NTP doesn't → wait more |
| ☐ | B8 | PC-HFA-IT-01 | `ssh -l admin 10.20.0.11` | `SW-JLM-F1-ACC1#` (HQ's IT manages Jerusalem) | The Jerusalem ACL `MGMT-ONLY` must permit `10.10.49.0` |

## Part C — Remote-access VPN: the teleworker

| ☐ | # | From / On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | PC-HOME-01 (VPN **not** connected) | `ping 10.10.1.10` | ❌ fails — **correct!** Without the VPN, HQ is private | — |
| ☐ | C2 | PC-HOME-01 → Desktop → **VPN** | GroupName `HQ-RA`, Group Key `Intel#RA-Group`, Host IP `200.1.1.2`, Username `remote1`, Password `Intel#Remote1` → **Connect** | **VPN is connected** | "authentication failed" → typo, or `aaa authorization network VPN-GROUP local` missing (Mission 9a) |
| ☐ | C3 | PC-HOME-01 → Command Prompt | `ipconfig` | An extra address `172.16.250.x` | `ip local pool RA-POOL …` + `pool RA-POOL` in group HQ-RA |
| ☐ | C4 | PC-HOME-01 | `ping 10.10.1.10` | ✅ replies now | ASA line for `172.16.250.0` (Mission 9d) |
| ☐ | C5 | PC-HOME-01 → Web Browser | `http://www.intel.lab` | **Intel Israel - Intranet** | C4 |
| ☐ | C6 | R-HFA-EDGE | `show crypto isakmp sa` | **Two** `QM_IDLE` lines: Jerusalem (`200.2.1.2`) **and** the teleworker (`200.3.50.10`) | Jerusalem gone → `no-xauth` missing on the `crypto isakmp key` line |
| ☐ | C7 | PC-JLM-SAL-01 | `ping 10.10.16.1` | ✅ still replies. Remote access didn't break site-to-site | C6 |
| ☐ | C8 | PC-HOME-01 → VPN | **Disconnect**, then `ping 10.10.1.10` | ❌ fails again — **correct!** | — |

## Part D — See the encryption (optional, but good for the book)

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | D1 | **Simulation** mode, filter ICMP (+ ISAKMP / ESP if offered) | Ping a Haifa PC from PC-JLM-SAL-01 → Capture/Forward → click the envelope **between the ISPs** | Outer header `200.2.1.2 → 200.1.1.2`, protocol **ESP**. No private address visible | B2 / B3 |
| ☐ | D2 | Back to **Realtime** | — | — | — |

## Part E — Saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | E1 | R-HFA-EDGE, R-JLM-EDGE | `copy running-config startup-config` | `[OK]` | — |
| ☐ | E2 | ASA | `write memory` | `[OK]` | — |
| ☐ | E3 | **File → Save** | — | Title shows `step15.pkt` | M1 |

---

## 🛠 Your own project
Run the same test in your own project. Replace:

| In the test | Use from your plan |
|---|---|
| Tunnel `172.16.12.0/30`, public `200.1.1.2` / `200.2.1.2` | your tunnel subnet and your edge routers' public addresses |
| `10.10.0.0/16` / `10.20.0.0/16` | your two sites' blocks |
| Group `HQ-RA`, keys, user `remote1`, pool `172.16.250.0/24` | your remote-access values |

Your project passes when every row matches with **your** values. B3 is the most important
test: the counters must **grow**. That proves the traffic is really encrypted.
