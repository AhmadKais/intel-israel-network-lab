# Step 14 — Self-test

> 🧪 Do this **after** you finish step 14, on your saved file `my-work/step14.pkt`.
> It takes about 30 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Press **Fast Forward Time** 3–4 times first (M18).
2. Do the tests **in order**, top to bottom.
3. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
4. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
5. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
6. Every box ticked → step 14 works. Go to step 15.

Kiryat Gat devices: `admin` / `Intel#Adm1n` (local users). IPv6: capital or small letters both
work. Part E **deletes a switch on purpose**, so save first. "M…" numbers are the basic moves
in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — The distribution switches

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | DIST1 | `show ipv6 interface brief` | Vlan31–36, Vlan99, Vlan100 `up`, each with `2001:DB8:30:<vlan>::2` | `ipv6` refused → SDM template (Mission 4b). SVI down → VLAN missing **on that switch** (VTP transparent) |
| ☐ | A2 | DIST2 | `show ipv6 interface brief` | The same VLANs with `…::3` | Mission 8 |
| ☐ | A3 | DIST1 | `show standby brief` | One line per VLAN (31–36, 99, 100), **Active**, virtual IP `FE80::1` | `standby … ipv6 FE80::1` refused → troubleshooting table (`autoconfig`) |
| ☐ | A4 | DIST2 | `show standby brief` | The same VLANs, **Standby** | Mission 8c |
| ☐ | A5 | DIST1 | `show ipv6 ospf neighbor` | `3.3.3.1` (edge) and `3.3.3.3` (DIST2), both `FULL` | No `router-id` under `ipv6 router ospf 1` |
| ☐ | A6 | DIST2 | `show ipv6 route` | `OE2 ::/0 [110/1] via FE80::…` (the default from the edge) | `default-information originate` on R-KG-EDGE (Mission 11b) |
| ☐ | A7 | Each of ACC1, ACC2, ACC3 | `show vlan brief` | The ports in the VLANs of the Mission 9 table | VTP transparent: create the VLAN **on that switch** |

## Part B — The PCs get their addresses by themselves

| ☐ | # | From | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | PC-KG-FAB-01 | `ipconfig /all` | IPv6 `2001:DB8:30:31:…` · gateway `FE80::1` · DNS `2001:DB8:30:100::10` | Only `FE80::` → click **Automatic** again; `ipv6 unicast-routing`; SVI down. No DNS → `ipv6 nd other-config-flag` / `ipv6 dhcp server` |
| ☐ | B2 | One PC of each other department (QUA, MNT, SAF, PLN, TRN) | `ipconfig` | `2001:DB8:30:32:…` … `2001:DB8:30:36:…` | The same as B1, for that VLAN |
| ☐ | B3 | Each server + PC-KG-ADMIN-01 → IP Configuration | Look | Static: DNS `…:100::10`, MAIL `::11`, SYSLOG `::12`, AAA `::13`, admin `2001:DB8:30:99::100`, all /64, gateway `FE80::1`. IPv4 fields **empty** | Mission 12a |
| ☐ | B4 | SRV-KG-DNS → Services → DNS | Look | **On**, 2 **AAAA** records: `mail.kg.intel.lab` → `2001:DB8:30:100::11`, `www6.isp-web.com` → `2001:DB8:C:50::80` | Mission 12c. **A** record instead of **AAAA** is wrong |

## Part C — IPv6 traffic

| ☐ | # | From | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | PC-KG-FAB-01 | `ping 2001:DB8:30:36::2` | ✅ replies (inter-VLAN over IPv6) | Part A |
| ☐ | C2 | PC-KG-FAB-01 | `ping 2001:DB8:30:100::10` | ✅ replies (the DNS server) | B3 |
| ☐ | C3 | PC-KG-FAB-01 → Web Browser | `http://www6.isp-web.com` | **IPv6 works!** | `ipv6 route 2001:DB8:30::/48 …` on R-ISPC-PE (Mission 11e); WAN6-IN must start with `permit icmp any any` |
| ☐ | C4 | PC-KG-ADMIN-01 | `ssh -l admin 2001:DB8:30:99::11` | `SW-KG-ACC1#` | ACC1's MGMT6 / `ipv6 access-class` (Mission 9a) |
| ☐ | C5 | PC-KG-FAB-01 | `ssh -l admin 2001:DB8:30:99::11` | ❌ refused — **correct!** | MGMT6 permits too much |

## Part D — The legacy IPv4 VLAN and NTP

| ☐ | # | From / On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | D1 | PC-KG-LEGACY-01 → IP Configuration | Look | `10.30.199.10` / `255.255.255.0` / gateway `10.30.199.1` / DNS `200.1.100.10` | Mission 12b |
| ☐ | D2 | PC-KG-LEGACY-01 | `ping 200.1.100.11` | ✅ replies (through PAT) | `ip route 10.30.199.0 …` on the edge, `ip route 0.0.0.0 … 10.30.4.1` on DIST1, NAT inside/outside |
| ☐ | D3 | R-KG-EDGE | `show ip nat translations` | `200.3.1.2:<port>  10.30.199.10:<port>` (PAT: one public address, many ports) | Mission 11c |
| ☐ | D4 | DIST1 (after Fast Forward Time) | `show ntp status` | Synchronized to `200.1.10.4`. Only DIST1 has `ntp server` in Kiryat Gat; that's by design | Static NAT `200.1.10.4` in Haifa (step 12), ASA `ntp` hole |

## Part E — Prove it: HSRPv6 failover (save first!)

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | E1 | **File → Save** | — | — | — |
| ☐ | E2 | PC-KG-FAB-01 | `ping -t 2001:DB8:30:100::10` (leave it running) | Replies | C2 |
| ☐ | E3 | Workspace | Delete **SWL3-KG-DIST1** (M7) | A few lost replies, then the pings continue | DIST2's `standby … ipv6 FE80::1` lines (Mission 8c) |
| ☐ | E4 | DIST2 | `show standby brief` | **Active** for `FE80::1` | — |
| ☐ | E5 | Repair | **Ctrl+C**. **File → Open** your saved `step14.pkt` → answer **No** to saving | DIST1 is back. A3 matches again | — |

## Part F — Physical view and saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | F1 | **Physical** → Intercity | Look | 4 cities: Haifa, Jerusalem, Kiryat Gat, Internet | Mission 15a |
| ☐ | F2 | **Navigation** | Look | `Kiryat Gat → KG Fab → KG Main Closet`: R-KG-EDGE, DIST1, DIST2, ACC1–3, SW-KG-SRV · `KG Fab`: 4 servers, 15 PCs, 2 printers · **SRV-ISPC-WEB6** under `Internet → ISP-C`. No `-KG-` name left under Haifa | Mission 15b |
| ☐ | F3 | **Logical** tab | PC-KG-FAB-01 → `http://www6.isp-web.com` | ✅ still opens | Cable Length Effects must be **off** |
| ☐ | F4 | Every Kiryat Gat device + R-ISPC-PE | `copy running-config startup-config` | `[OK]` | — |
| ☐ | F5 | **File → Save** | — | Title shows `step14.pkt` | M1 |

---

## 🛠 Your own project
Run the same test in your own project. Replace:

| In the test | Use from your plan |
|---|---|
| `2001:DB8:30::/48`, `…:<vlan>::/64` | your IPv6 block and your VLAN prefixes |
| Router-IDs `3.3.3.x` | your OSPFv3 router-IDs |
| Legacy VLAN 199 `10.30.199.0/24`, PAT to `200.3.1.2` | your IPv4 VLAN and your public address |
| `www6.isp-web.com`, `2001:DB8:C:50::80` | your IPv6 test site |

Your project passes when every row matches with **your** values. B1 is the most important
test: a PC must get its IPv6 address, gateway **and** DNS without you typing anything.
