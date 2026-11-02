# Step 11 — Self-test

> 🧪 Do this **after** you finish step 11, on your saved file `my-work/step11.pkt`.
> It takes about 30 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Press **Fast Forward Time** several times first (M18). NTP needs a few minutes of simulated time.
2. Do the tests **in order**, top to bottom.
3. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
4. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
5. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
6. Every box ticked → step 11 works. Go to step 12.

On the ASA: CLI → `enable` → `Intel#En4ble`. Everywhere else: `admin` / `Intel#Adm1n`.
"M…" numbers are the basic moves in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — The firewall

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | ASA-HFA-FW1 | `show nameif` | `Gi1/1 outside 0` · `Gi1/2 inside 100` · `Gi1/3 dmz 50` | Mission 4b. Cables in the wrong ports → hover over the ASA ends: 1 = outside, 2 = inside, 3 = DMZ |
| ☐ | A2 | ASA | `show route` | `S* 0.0.0.0 … via 10.10.7.1, outside` and `S 10.10.0.0 … via 10.10.1.1, inside` | Mission 5a |
| ☐ | A3 | ASA | `show access-list OUTSIDE_IN` | 3 lines: `udp host 10.10.7.1 host 10.10.1.10 eq syslog` · `tcp host 10.10.7.1 host 10.10.1.11 eq tacacs` · `udp host 10.10.7.1 host 10.10.200.14 eq ntp` | Mission 6a |
| ☐ | A4 | ASA | `show running-config` → look for `access-group` | `access-group OUTSIDE_IN in interface outside` | Mission 6a, last line |
| ☐ | A5 | ASA | `show running-config` → look for `inspect` | `inspect icmp`, `inspect dns`, `inspect http`, `inspect ftp` | Mission 5b |
| ☐ | A6 | CORE1 | `ping 10.10.1.254` | `!!!!!` | Cable 1 not in Gi1/2, or SW-HFA-SRV Fa0/24 not in VLAN 100 |

## Part B — Default route and the edge router

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | CORE1 and CORE2 | `show ip route static` | `S* 0.0.0.0/0 [1/0] via 10.10.1.254` | Mission 7 |
| ☐ | B2 | A-DIST1 | `show ip route` | `O*E2 0.0.0.0/0 [110/1] via 10.10.4.2 …` near the top (also via 10.10.4.6, through CORE2) | `default-information originate` missing on the cores (Mission 7) |
| ☐ | B3 | R-HFA-EDGE | `ping 10.10.7.2` | `!!!!!` | Mission 8a. Router ports start shut → `no shutdown` |
| ☐ | B4 | PC-HFA-IT-01 | `ssh -l admin 10.10.7.1` → `Intel#Adm1n` | `R-HFA-EDGE#` (login checked by AAA, through the `tacacs` hole) | Mission 8b–c. AAA client `R-HFA-EDGE` / `10.10.7.1`? A3 line 2? |

## Part C — The DMZ

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | Each DMZ server → IP Configuration | Look | `10.10.200.11` DNS · `.12` MAIL · `.13` WEB · `.14` NTP · `.15` FTP · `.16` AAA · mask `255.255.255.0` · gateway `10.10.200.1` · DNS `10.10.200.11` | Mission 10 |
| ☐ | C2 | SRV-HFA-DMZ-DNS → Command Prompt | `ping 10.10.200.1`, then `ping 10.10.200.16` | ✅ both reply | Mission 9 (ports), cable 2 in Gi1/3 |
| ☐ | C3 | SRV-HFA-DMZ-DNS → Services → DNS | Look | **On**, 6 A records: `www.intel.lab` .13 · `mail.intel.lab` .12 · `ftp.intel.lab` .15 · `ntp.intel.lab` .14 · `www.isp-web1.com` 200.1.100.11 · `www.isp-web2.com` 200.1.100.12 | Mission 11c |
| ☐ | C4 | Each DMZ server → Services | Look at every service | Only its **own** service is On (DNS server: only DNS, …) | Mission 11a |
| ☐ | C5 | SRV-HFA-DMZ-FTP → Services → FTP | Look | User `ftpadmin`. **No** user `cisco` | Mission 11f |

## Part D — Through the firewall

| ☐ | # | From | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | D1 | An ENG PC | `ping 10.10.200.13` | ✅ replies (inside → DMZ) | `inspect icmp` (A5); default route (Part B) |
| ☐ | D2 | An ENG PC | `ipconfig /renew`, then `nslookup www.intel.lab` | `10.10.200.13` | C3; the PC's DNS must be `10.10.200.11` |
| ☐ | D3 | An ENG PC → Desktop → **Web Browser** | `http://www.intel.lab` → **Go** | The page **Intel Israel - Intranet** | "Host Name Unresolved" → D2. Other page → Mission 11d |
| ☐ | D4 | An ENG PC | `ping 10.10.7.1` | ✅ replies (inside → outside) | A2, B3 |
| ☐ | D5 | SRV-HFA-DMZ-WEB | `ping 10.10.16.1` | ❌ fails — **correct!** DMZ (50) → inside (100) is blocked | If it replies, a rule allows DMZ → inside. Remove it |
| ☐ | D6 | R-HFA-EDGE | `ping 10.10.200.13` | ❌ fails — **correct!** outside → DMZ, not on the ACL | If it replies, OUTSIDE_IN permits too much (A3) |
| ☐ | D7 | ASA, **while** a PC browses (D3) | `show conn` | Live connections are listed | — |

## Part E — NTP

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | E1 | Any 3 Haifa switches (e.g. A-ACC1, B-DIST2, CORE1) | `show ntp status` | `Clock is synchronized … reference is 10.10.200.14` | Fast Forward Time more. NTP service On (C4)? `ping 10.10.200.14` from the switch |
| ☐ | E2 | The same switches | `show clock` | The same time as SRV-HFA-DMZ-NTP | E1 |
| ☐ | E3 | All 15 Haifa switches | `show running-config \| include ntp` | `ntp server 10.10.200.14` | Mission 12 on the missing switch |

## Part F — Physical view and saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | F1 | **Physical** → Navigation | Look under **Haifa → HFA Data Center** | ASA-HFA-FW1, R-HFA-EDGE, SW-HFA-DMZ and the 6 DMZ servers | Drag them there (M26) |
| ☐ | F2 | ASA | `write memory` | `[OK]` | — |
| ☐ | F3 | R-HFA-EDGE, SW-HFA-DMZ, every switch you changed | `copy running-config startup-config` | `[OK]` | — |
| ☐ | F4 | **File → Save** | — | Title shows `step11.pkt` | M1 |

---

## 🛠 Your own project
Run the same test in your own project. Replace:

| In the test | Use from your plan |
|---|---|
| ASA addresses `10.10.7.2`, `10.10.1.254`, `10.10.200.1` | your outside, inside and DMZ addresses |
| DMZ servers `10.10.200.11`–`.16`, DNS names `*.intel.lab` | your DMZ servers and your domain |
| The 3 holes in OUTSIDE_IN | your edge router's address and your Syslog / AAA / NTP servers |
| NTP `10.10.200.14` | your NTP server |

Your project passes when every row matches with **your** values. D5 and D6 are the most
important tests: the DMZ must **not** reach the inside, and the outside must **not** reach the DMZ.
