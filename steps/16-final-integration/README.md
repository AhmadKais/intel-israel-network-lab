# Step 16 — Final integration, scaling & the project book

## Goal
Turn the working lab into a **submission**: 8+ PCs per department, email accounts, a full
end-to-end test run, and a project book that maps every ministry requirement to proof.

Ministry requirements: **20.2, 22.19**, and the whole **ספר הפרויקט** section

## 1. Scale every department to ≥ 8 PCs (req. 20.2)
You have 2 PCs per department. Add 6 more to each:
1. Select the two existing PCs of a department → **Ctrl+C / Ctrl+V** → drag the copies next
   to the switch.
2. Cable them to the next free ports **of that department's range** (e.g. Engineering =
   Fa0/1-10 on ACC1/ACC2).
3. Rename them (`PC-HFA-ENG-03` … `-08`) — names must stay logical (req. 22.1).
4. DHCP is already on → they get addresses automatically. Port-security sticky learns them.

| Branch | Departments × 8 | Total PCs |
|---|---|---|
| Haifa | ENG, FIN, HR, MKT, RND, IT | 48 |
| Jerusalem | SAL, LOG, PRD, QA, SEC, ADM | 48 |
| Kiryat Gat | FAB, QUA, MNT, SAF, PLN, TRN | 48 |

> PT gets slow with ~200 devices. Save often, close other programs, and use the
> *Logical → Cluster* feature to group each building into one icon if the canvas gets messy.

## 2. Email accounts — 2 PCs per department, per branch (req. 22.19)

**SRV-HFA-DMZ-MAIL** (domain `intel.lab`) — *Services → EMAIL → User Setup*, password
`Mail#2026` for all:

| Users | Department |
|---|---|
| `eng1`, `eng2` | Haifa Engineering |
| `fin1`, `fin2` | Haifa Finance |
| `hr1`, `hr2` | Haifa HR |
| `mkt1`, `mkt2` | Haifa Marketing |
| `rnd1`, `rnd2` | Haifa R&D |
| `it1`, `it2` | Haifa IT |
| `sal1`, `sal2`, `log1`, `log2`, `prd1`, `prd2`, `qa1`, `qa2`, `sec1`, `sec2`, `adm1`, `adm2` | Jerusalem (via VPN) |

**SRV-KG-MAIL** (domain `kg.intel.lab`): `fab1/2`, `qua1/2`, `mnt1/2`, `saf1/2`, `pln1/2`, `trn1/2`.

On each chosen PC → *Desktop → Email → Configure Mail*:

| Field | Haifa / Jerusalem | Kiryat Gat |
|---|---|---|
| Your Name | e.g. `Engineering 1` | `Fab Ops 1` |
| Email Address | `eng1@intel.lab` | `fab1@kg.intel.lab` |
| Incoming / Outgoing server | `mail.intel.lab` | `mail.kg.intel.lab` |
| User name / Password | `eng1` / `Mail#2026` | `fab1` / `Mail#2026` |

Test: `eng1` sends to `sal1` (Haifa → Jerusalem, across the VPN). `sal1` → *Receive*. ✅
Write the full list in the book's **email table** (טבלת אימיילים).

## 3. Final hardening sweep — on every router and switch
```
show run | include hostname|banner|enable secret|service password|ip ssh|logging|ntp server
```
Every device must show: logical hostname, banner, `enable secret`,
`service password-encryption`, SSH v2, `transport input ssh`, VTY ACL, logging host, NTP.
Fix whatever is missing. Then **`wr` on every device**.

## 4. End-to-end test plan (screenshot each ✅ for the book)

| # | From | To / command | Proves |
|---|---|---|---|
| 1 | PC-HFA-ENG-01 | `ping` PC-HFA-IT-01 | inter-VLAN + OSPF areas 1→0→2→3 (virtual link) |
| 2 | PC-HFA-ENG-01 | `http://www.isp-web1.com` | ASA → edge PAT → BGP |
| 3 | PC-HOME-01 | `http://www.intel.lab` | static NAT + ASA ACL |
| 4 | LT-HFA-GUEST-01 | `ping 10.10.1.10` ❌ / `http://www.isp-web2.com` ✅ | guest isolation |
| 5 | PC-JLM-SAL-01 | `ping 10.10.16.1` | GRE/IPsec VPN, private addressing |
| 6 | PC-JLM-SAL-01 | `http://www.intel.lab`, email to HQ | DMZ via VPN |
| 7 | PC-JLM-SAL-01 | `ping 200.1.100.11` + `show ip nat translations` on R-JLM-EDGE | dynamic NAT |
| 8 | PC-KG-FAB-01 | `http://www6.isp-web.com` | IPv6 end-to-end |
| 9 | PC-KG-LEGACY-01 | `ping 200.1.100.11` | PAT overload |
| 10 | PC-HOME-01 | VPN client → `ping 10.10.1.10` | remote-access VPN |
| 11 | PC-HFA-IT-01 | `ssh` to devices in all 3 branches | SSH + AAA + mgmt ACL |
| 12 | PC-HFA-RND-01 | `ssh` to a switch ❌ | mgmt ACL blocks non-admins |
| 13 | power off A-DIST1 | `ping -t` keeps going | HSRP + STP + OSPF redundancy |
| 14 | shut R-HFA-EDGE Gi0/1 | edge still reaches ISP-B | DSL floating static |
| 15 | shut ISPA-BR ↔ ISPC-BR link | home → www.intel.lab still works | BGP redundancy |
| 16 | rogue DHCP server on an access port | PC still gets the real lease | DHCP snooping |
| 17 | rogue switch on a PortFast port | port err-disabled | BPDU Guard |
| 18 | `show ntp status` in each branch | synchronized | NTP hierarchy |
| 19 | SRV-HFA-SYSLOG / SRV-JLM-SYSLOG | messages with timestamps | Syslog |

## 5. The project book (ספר הפרויקט — 30% of the grade)
Follow the ministry's order. Where each part comes from:

| # | Chapter | Source in this repo |
|---|---|---|
| 1 | דף שער | school, project name "Intel Israel — Enterprise Network", your name, ID, year, mentor |
| 2 | תוכן עניינים | auto-generated in Word/Docs |
| 3 | מבוא | why a company needs a secure multi-site network |
| 4 | אודות החברה | Intel Israel: Haifa (HQ, R&D), Jerusalem, Kiryat Gat (fab) |
| 5 | דרישות הארגון מהרשת | [requirements-matrix.md](../../docs/requirements-matrix.md), in business language |
| 6 | בחירות וחלופות | the "Concepts" sections: LACP vs PAgP, OSPF vs EIGRP, GRE/IPsec vs plain IPsec, collapsed core vs 3-tier, dynamic NAT vs PAT, the IPv6/NAT trade-off |
| 7 | רשימת סניפים ומחלקות | [design.md](../../docs/design.md) §3–5 |
| 8 | טופולוגיה פיזית | PT **Physical** view screenshots (buildings / floors) |
| 9 | טופולוגיה לוגית | PT **Logical** view screenshots, one per branch + WAN |
| 10–11 | חלוקת ציוד / מפרט ציוד | device count per branch + datasheet facts (2911, 3650, 2960, ASA 5506-X) |
| 12 | טבלת שמות | [design.md §1](../../docs/design.md) naming convention + the list of all devices |
| 13 | טבלאות VLAN / IP | design.md tables |
| 14 | טבלת סיסמאות | [passwords.md](../../docs/passwords.md) |
| 15 | טבלת אימיילים | section 2 above |
| 16 | פקודות | `show run` of each device type, explained (reuse the commented configs from the steps) + the test screenshots from section 4 |
| 17 | ביבליוגרפיה | Cisco CCNA/CCNP docs, RFC 4271 (BGP), RFC 2328 (OSPF), RFC 7868 (EIGRP), RFC 4861 (IPv6 ND), RFC 4301 (IPsec), Cisco ASA config guide |

**Known simulator limitations** — put them in chapter 6, it shows understanding:
- IPv6-only branch can't reach IPv4-only services (no NAT64 in PT).
- RA Guard / DHCPv6 Guard not available.
- DSL backup carries router traffic only unless route-map NAT is supported.
- Private VLANs for DMZ server isolation not available.

## Checkpoint — final
- [ ] `my-work/final.pkt` saved and committed
- [ ] every row of the test plan has a screenshot
- [ ] every row of the requirements matrix is ✅
- [ ] project book draft written

## Final challenge — defend it
Your examiner will ask "why?" about every choice. Answer these out loud without notes:
1. Why is building C's area not connected to area 0, and what does the virtual link do?
2. Why does the ASA not do NAT in this design?
3. Why GRE over IPsec instead of plain IPsec?
4. Why is HSRP active always the STP root for the same VLAN?
5. What happens to Jerusalem if R-JLM-EDGE dies? What would you add to fix it?

<details><summary>Short answers</summary>

1. To demonstrate (and satisfy req. 8/22.16) the rule that every area must touch area 0.
   The virtual link is a logical area-0 adjacency through transit area 2 between
   B-DIST (ABR 0/2) and C-DIST (ABR 2/3).
2. So that private addresses pass the ASA unchanged — the edge router does NAT and the VPN
   sees real 10.x addresses; one NAT point is simpler to troubleshoot.
3. Static NAT on the web server would translate its VPN replies before encryption; traffic
   routed through a GRE tunnel interface (not `nat outside`) bypasses NAT. GRE also allows
   routing protocols/multicast through the tunnel if needed later.
4. To keep the Layer-2 path to the gateway optimal — otherwise frames cross the
   inter-distribution link unnecessarily.
5. Jerusalem loses internet and the VPN (single edge router = single point of failure).
   Fix: a second edge router + second ISP link, HSRP/EIGRP between them, and a second
   tunnel to Haifa.
</details>
