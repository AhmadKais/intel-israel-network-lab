# Step 12 — Self-test

> 🧪 Do this **after** you finish step 12, on your saved file `my-work/step12.pkt`.
> It takes about 30 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Press **Fast Forward Time** 3–4 times first (M18). BGP needs about a minute of simulated time.
2. Do the tests **in order**, top to bottom.
3. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
4. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
5. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
6. Every box ticked → step 12 works. Go to step 13.

ISP routers: CLI → `enable` → `Intel#En4ble`. R-HFA-EDGE: `admin` / `Intel#Adm1n`.
Part E **breaks things on purpose** and repairs them. "M…" numbers are the basic moves in
[docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — BGP between the three providers

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | R-ISPA-BR | `show ip bgp summary` | 2 neighbours: `200.0.12.2` (AS 200) and `200.0.13.2` (AS 300). The **last column is a number** | `Active` / `Idle` → ping the neighbour. Ping fails → cable/interface. Ping works → wrong `neighbor` IP or AS |
| ☐ | A2 | R-ISPB-BR | `show ip bgp summary` | `200.0.12.1` (AS 100) and `200.0.23.2` (AS 300), both end with a number | Mission 5a |
| ☐ | A3 | R-ISPC-BR | `show ip bgp summary` | `200.0.13.1` (AS 100) and `200.0.23.1` (AS 200), both end with a number | Mission 6a |
| ☐ | A4 | R-ISPA-BR | `show ip bgp` | `*> 200.1.0.0/16 0.0.0.0` · `200.2.0.0/16` with 2 paths (`*>` path `200 i`, `*` path `300 200 i`) · `200.3.0.0/16` with 2 paths (`*>` path `300 i`, `*` path `200 300 i`) | A block missing → its `network … mask` doesn't exactly match a route (check the `null0` route) |
| ☐ | A5 | R-ISPA-PE | `show ip route` | `O*E2 0.0.0.0/0` (default route from its BR) | Mission 4c |

## Part B — The internet servers

| ☐ | # | From | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | SRV-ISP-DNS → Services → DNS | Look | **On**, 3 records: `www.isp-web1.com` 200.1.100.11 · `www.isp-web2.com` 200.1.100.12 · `www.intel.lab` **200.1.10.3** | Mission 8b |
| ☐ | B2 | PC-HOME-01 → Web Browser | `http://www.isp-web1.com` | **ISP Web Server 1** (AS 300 → AS 100) | `ping 200.1.100.11` from PC-HOME-01. Fails → Part A. Works → DNS record / the PC's DNS `200.1.100.10` |

## Part C — The edge router and NAT

| ☐ | # | On / From | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | R-HFA-EDGE | `show ip interface brief` | Gi0/0 `10.10.7.1` · Gi0/1 `200.1.1.2` · Gi0/2 `200.2.9.2`, all `up up` | Missions 9, 12c |
| ☐ | C2 | R-HFA-EDGE | `show ip route static` | `S* 0.0.0.0/0 [1/0] via 200.1.1.1` and `S 10.10.0.0/16 … via 10.10.7.2`. **No** route via `200.2.9.1` (the backup waits) | Missions 9, 12c |
| ☐ | C3 | R-HFA-EDGE | `ping 200.2.9.1` | `!!!!!` (the DSL line works) | Cloud mapping `Modem4 ↔ Ethernet6` (Mission 12b) / phone cable on modem Port 0 |
| ☐ | C4 | An ENG PC | `ping 200.1.100.11` | ✅ replies | `ip nat inside` on Gi0/0 / `ip nat outside` on Gi0/1 / NAT-HFA ACL (Mission 10) |
| ☐ | C5 | An ENG PC → Web Browser | `http://www.isp-web1.com`, then `http://www.isp-web2.com` | Both pages open | DNS record on SRV-HFA-DMZ-DNS (step 11) |
| ☐ | C6 | R-HFA-EDGE | `show ip nat translations` | PAT lines (`200.1.1.2:… 10.10.x.x:…`) + 2 static lines: `200.1.10.3` ↔ `10.10.200.13` and `udp 200.1.10.4:123` ↔ `10.10.200.14:123` | Mission 11a |
| ☐ | C7 | ASA | `show access-list OUTSIDE_IN` | 5 lines: the 3 from step 11 + `tcp any host 10.10.200.13 eq www` + `udp any host 10.10.200.14 eq ntp` | Mission 11b |

## Part D — The internet sees us (and not more)

| ☐ | # | From | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | D1 | PC-HOME-01 → Web Browser | `http://www.intel.lab` | **Intel Israel - Intranet** (through 200.1.10.3) | In this order: B1 record; `ip route 200.1.10.0 …` + `redistribute static subnets` on R-ISPA-PE; C7 `www` line |
| ☐ | D2 | PC-HOME-01 | `ping 10.10.16.1` | ❌ fails — **correct!** Private addresses don't exist on the internet | — |
| ☐ | D3 | LT-HFA-GUEST-01 → Web Browser | `http://www.isp-web2.com` | ✅ **ISP Web Server 2**. Guests have internet now | GUEST-IN must end with `permit ip any any` (step 10) |
| ☐ | D4 | LT-HFA-GUEST-01 | `ping 10.10.1.10` | ❌ still fails — **correct!** | Step 10 self-test, Part B |

## Part E — Prove it (break it on purpose, then repair)

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | E1 | R-ISPA-BR | `configure terminal` → `interface gi0/2` → `shutdown` → `end`. Fast Forward Time (about a minute) | — | — |
| ☐ | E2 | R-ISPA-BR | `show ip bgp` | `200.3.0.0/16` only via path `200 300` | Wait longer |
| ☐ | E3 | PC-HOME-01 → Web Browser | `http://www.intel.lab` | ✅ still opens (the long way round) | Part A |
| ☐ | E4 | Repair, R-ISPA-BR | `configure terminal` → `interface gi0/2` → `no shutdown` → `end` | A1 matches again after a minute | — |
| ☐ | E5 | R-HFA-EDGE | `configure terminal` → `interface gi0/1` → `shutdown` → `end` → `show ip route static` | `S* 0.0.0.0/0 [10/0] via 200.2.9.1` (the backup took over) | `ip route 0.0.0.0 0.0.0.0 200.2.9.1 10` (Mission 12c) |
| ☐ | E6 | R-HFA-EDGE | `ping 200.2.0.1` | `!!!!!` through the DSL line | C3 |
| ☐ | E7 | Repair, R-HFA-EDGE | `configure terminal` → `interface gi0/1` → `no shutdown` → `end` → `show ip route static` | `S* 0.0.0.0/0 [1/0] via 200.1.1.1` again | — |

> PAT is tied to Gi0/1, so **users** don't browse over DSL in E5. That's expected (step 12,
> Exercise B).

## Part F — Physical view and saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | F1 | **Physical** → Intercity | Look | 2 cities: **Haifa** and **Internet**, joined by a line | Mission 14a |
| ☐ | F2 | **Navigation** | Look under **Internet** | ISP-A: R-ISPA-BR, R-ISPA-PE, SW-ISPA-NET, SRV-ISP-DNS, SRV-ISP-WEB1, SRV-ISP-WEB2 · ISP-B: R-ISPB-BR, R-ISPB-PE, CLOUD-DSL · ISP-C: R-ISPC-BR, R-ISPC-PE · Home: SW-ISPC-HOME, PC-HOME-01. **DSL-HFA** stays in HFA Data Center | Mission 14b |
| ☐ | F3 | **Logical** tab | PC-HOME-01 → `http://www.intel.lab` | ✅ still opens | Cable Length Effects must be **off** |
| ☐ | F4 | All routers / ASA | `copy running-config startup-config` / `write memory` | `[OK]` | — |
| ☐ | F5 | **File → Save** | — | Title shows `step12.pkt` | M1 |

---

## 🛠 Your own project
Run the same test in your own project. Replace:

| In the test | Use from your plan |
|---|---|
| AS 100 / 200 / 300, `200.x.x.x` addresses | your providers' AS numbers and public blocks |
| `200.1.10.3` (website), `200.1.10.4` (NTP) | your public addresses for static NAT |
| `www.intel.lab`, `www.isp-web1.com` … | your domain and your internet test sites |
| DSL `200.2.9.x`, distance `10` | your backup link and its distance |

Your project passes when every row matches with **your** values. D1 and D2 are the most
important tests: the internet reaches your **website** and nothing else.
