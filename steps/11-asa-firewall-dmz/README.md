# Step 11 — ASA firewall, DMZ & the main NTP server

> ⏱ 3 lessons · 🎯 ministry requirements 11, 12, 21, 20.8, 22.13 · 📁 start from: `my-work/step10.pkt` · 💾 save as: `my-work/step11.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 10 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## 🗺 Your missions in this step
**Lesson 1 — the firewall**
- [ ] **Mission 1** — Open step 10 and save as step 11 (2 min)
- [ ] **Mission 2** — Place the ASA, the edge router, the DMZ switch and 6 servers (10 min)
- [ ] **Mission 3** — Cable them (10 min)
- [ ] **Mission 4** — ASA, part 1: log in, name, the three zones (10 min)
- [ ] **Mission 5** — ASA, part 2: routes and inspection (10 min)
- [ ] **Mission 6** — ASA, part 3: the outside ACL and SSH (10 min)

**Lesson 2 — routing and the edge router**
- [ ] **Mission 7** — Default route on CORE1 and CORE2 (10 min)
- [ ] **Mission 8** — R-HFA-EDGE, the inside side (20 min)
- [ ] **Mission 9** — SW-HFA-DMZ (10 min)

**Lesson 3 — the six servers and the clock**
- [ ] **Mission 10** — IP addresses on the six DMZ servers (10 min)
- [ ] **Mission 11** — One job per server (25 min)
- [ ] **Mission 12** — NTP on every HQ switch (15 min)
- [ ] **Mission 13** — Test through the firewall (15 min)
- [ ] **Exercises** A and B
- [ ] **🛠 Your own project**

## What you will have at the end
An **ASA 5506-X firewall** splits Haifa HQ into three security zones, and a **DMZ** holds the
six servers the ministry requires (DNS, Email, Web, NTP, FTP, AAA). An **edge router** waits
at the outside, ready for the internet (step 12). Every HQ device takes its time from the
**main NTP server**.

```
                         R-HFA-EDGE  (Gi0/0 10.10.7.1)          -> internet in step 12
                              │ Gi1/1  OUTSIDE (level 0)  10.10.7.2
                       ┌──────┴──────┐
     inside (100)  Gi1/2│ ASA-HFA-FW1 │Gi1/3  DMZ (level 50)
     10.10.1.254   ────┘             └────  10.10.200.1 ── SW-HFA-DMZ ── 6 servers
           │                                                   (10.10.200.11 – .16)
     SW-HFA-SRV Fa0/24  (VLAN 100 — the cores route everything unknown here)
```

---

## Part 1 — Understand it first (simple English)

### 1.1 What a firewall does
A firewall sits between networks and decides, packet by packet, what may pass. Unlike a
router (which just finds the way), a firewall's job is to **say no**.

### 1.2 Zones and security levels
The ASA gives each interface a **name** and a **trust level** from 0 to 100:

| Zone | Name on the ASA | Level | What's there |
|---|---|---|---|
| Inside | `inside` | **100** | the whole LAN — most trusted |
| DMZ | `dmz` | **50** | servers the internet may reach |
| Outside | `outside` | **0** | toward the internet — untrusted |

**The rule:** traffic from a **higher** level to a **lower** level is **allowed**;
from **lower to higher** is **blocked** — unless an ACL allows it.

| From → To | Default |
|---|---|
| inside → DMZ, inside → outside | ✅ allowed |
| DMZ → inside | ❌ blocked |
| outside → DMZ / inside | ❌ blocked (we open exact holes with an ACL) |

Comparison: an office building. Employees (inside) may walk to the reception (DMZ) and out to
the street (outside). Visitors from the street may only enter the reception — and only if
they're on the list. Nobody from the reception walks into the offices.

### 1.3 Stateful — replies come back automatically
The ASA **remembers** every conversation started from inside. When the answer comes back from
the outside, the ASA recognises it and lets it in — no ACL needed. Ping is the exception (it
isn't a real "conversation"), so we tell the ASA to **inspect** ICMP, then ping replies return too.

### 1.4 What is the DMZ for?
Servers that the internet must reach (like the website) are the most attacked machines. If one
is hacked, the attacker lands in the **DMZ** — and from the DMZ they **cannot open connections
into the inside** (level 50 → 100 is blocked). The damage stays in the "reception".

The ministry requires six DMZ servers in the main branch (req. 21):

| Server | IP | Service |
|---|---|---|
| SRV-HFA-DMZ-DNS | 10.10.200.11 | Translates names (`www.intel.lab`) into IP addresses |
| SRV-HFA-DMZ-MAIL | 10.10.200.12 | Email for the company (`@intel.lab`) |
| SRV-HFA-DMZ-WEB | 10.10.200.13 | The company website |
| SRV-HFA-DMZ-NTP | 10.10.200.14 | **The main clock** of the company |
| SRV-HFA-DMZ-FTP | 10.10.200.15 | File transfer |
| SRV-HFA-DMZ-AAA | 10.10.200.16 | Central authentication (for remote users) |

### 1.5 NTP — everybody agrees on the time
Logs from 50 devices are useless if every device has a different clock ("the link went down at
… which time?"). **NTP** synchronises all clocks to one **NTP server**. SRV-HFA-DMZ-NTP is the
main NTP server; the other branches will sync to it later (req. 20.8).

### 1.6 The edge router and "who does NAT"
The **edge router** (R-HFA-EDGE) will connect HQ to the internet in step 12. All address
translation (NAT) happens there — **not** on the ASA. The ASA just filters and routes. That keeps
private addresses unchanged through the firewall, which the VPN needs in step 15.

### 1.7 How traffic finds the ASA
The cores don't know the internet. We give them a **default route** ("everything I don't know →
the ASA") and OSPF spreads it to every switch in HQ (`default-information originate`).

### 1.8 The ASA is **not** a switch — three differences
| | Switch / router (IOS) | ASA |
|---|---|---|
| Save | `copy running-config startup-config` | **`write memory`** |
| Mask in routes and ACLs | wildcard (`0.0.0.255`) | **normal mask** (`255.255.255.0`) |
| An interface works when… | it has an IP and `no shutdown` | it **also** has a `nameif` |

---

## Mission 1 — Open step 10 and save as step 11 (how? → M1)
**File → Open** → `my-work/step10.pkt` → **File → Save As** → `my-work/step11.pkt`.

---

## Mission 2 — Place the devices (how? → M2, M4)
Place them on the right of building A / the server farm. Then draw a rectangle labelled
`HAIFA - DMZ` around the DMZ switch and its servers (M19).

| Where in the device bar | Model | Display Name |
|---|---|---|
| Network Devices → **Security** | **5506-X** | `ASA-HFA-FW1` |
| Network Devices → **Routers** | **2911** | `R-HFA-EDGE` |
| Network Devices → Switches | **2960** | `SW-HFA-DMZ` |
| End Devices → End Devices | **Server** ×6 | `SRV-HFA-DMZ-DNS`, `SRV-HFA-DMZ-MAIL`, `SRV-HFA-DMZ-WEB`, `SRV-HFA-DMZ-NTP`, `SRV-HFA-DMZ-FTP`, `SRV-HFA-DMZ-AAA` |

✅ 9 new devices, all named.

---

## Mission 3 — Cable them (how? → M5)
| # | Cable | Click first | Port | Click second | Port |
|---|---|---|---|---|---|
| 1 | Straight | SW-HFA-SRV | **Fa0/24** | ASA-HFA-FW1 | **GigabitEthernet1/2** |
| 2 | Straight | ASA-HFA-FW1 | **GigabitEthernet1/3** | SW-HFA-DMZ | **GigabitEthernet0/1** |
| 3 | Straight | ASA-HFA-FW1 | **GigabitEthernet1/1** | R-HFA-EDGE | **GigabitEthernet0/0** |
| 4 | Straight | SRV-HFA-DMZ-DNS | Fa0 | SW-HFA-DMZ | Fa0/1 |
| 5 | Straight | SRV-HFA-DMZ-MAIL | Fa0 | SW-HFA-DMZ | Fa0/2 |
| 6 | Straight | SRV-HFA-DMZ-WEB | Fa0 | SW-HFA-DMZ | Fa0/3 |
| 7 | Straight | SRV-HFA-DMZ-NTP | Fa0 | SW-HFA-DMZ | Fa0/4 |
| 8 | Straight | SRV-HFA-DMZ-FTP | Fa0 | SW-HFA-DMZ | Fa0/5 |
| 9 | Straight | SRV-HFA-DMZ-AAA | Fa0 | SW-HFA-DMZ | Fa0/6 |

> ⚠️ The ASA's ports are **1/1, 1/2, 1/3** — remember: **1 = outside, 2 = inside, 3 = DMZ**.
> Swapping them is the #1 error in this step. Hover over each ASA cable end to check (M6).

(The ASA behaves like a router-type device: Straight-Through to switches and routers works in PT.
If a link stays red **after** Mission 4, try Cross-Over.)

✅ 9 cables. The ASA ends stay red until Mission 4 — that's normal.

---

## Mission 4 — ASA, part 1: log in, name, the three zones
👉 **Where:** click **ASA-HFA-FW1** → **CLI** → press **Enter** → `ciscoasa>`.

### 4a — Get into config mode
⌨️ Type:
```
enable
```
`Password:` → just press **Enter** (empty on a new ASA) → `ciscoasa#`.

⌨️ Type:
```
configure terminal
hostname ASA-HFA-FW1
enable password Intel#En4ble
domain-name intel.lab
no dhcpd enable inside
```
✅ The prompt is now `ASA-HFA-FW1(config)#`.

> `no dhcpd enable inside` — the factory config runs a DHCP server on "inside". If you leave it,
> your PCs may get `192.168.1.x` addresses from the firewall.

> If PT asks "Would you like to enable anonymous error reporting…?" → type `N`, Enter.

### 4b — The three zones
⌨️ Type:
```
interface gigabitEthernet 1/1
 nameif outside
 security-level 0
 ip address 10.10.7.2 255.255.255.252
 no shutdown
interface gigabitEthernet 1/2
 nameif inside
 security-level 100
 ip address 10.10.1.254 255.255.255.0
 no shutdown
interface gigabitEthernet 1/3
 nameif dmz
 security-level 50
 ip address 10.10.200.1 255.255.255.0
 no shutdown
 exit
```
🔍 **Check now:** `show nameif`
```
Interface                Name                     Security
GigabitEthernet1/1       outside                    0
GigabitEthernet1/2       inside                   100
GigabitEthernet1/3       dmz                       50
```
🔍 And from the inside: log in to **CORE1** → `ping 10.10.1.254` → `!!!!!`.

❌ **If the ping fails:** cable 1 is not in **Gi1/2**, or Fa0/24 on SW-HFA-SRV is not in VLAN 100
(`show vlan brief` on SW-HFA-SRV — step 08 put it there).

| Command | In simple English |
|---|---|
| `nameif outside` | Give this ASA interface a zone name (without it, the port does **nothing**) |
| `security-level 0` | Its trust level |

---

## Mission 5 — ASA, part 2: routes and inspection
👉 **Where:** ASA, prompt `ASA-HFA-FW1(config)#`.

### 5a — Routes
⌨️ Type:
```
route outside 0.0.0.0 0.0.0.0 10.10.7.1
route inside 10.10.0.0 255.255.0.0 10.10.1.1
```
| Command | In simple English |
|---|---|
| `route outside 0.0.0.0 0.0.0.0 10.10.7.1` | "Unknown destinations: out of `outside` to the edge router" |
| `route inside 10.10.0.0 255.255.0.0 10.10.1.1` | "All of HQ is behind the cores (the VLAN 100 gateway)" |

🔍 **Check now:** `show route` → you see `S* 0.0.0.0 ... via 10.10.7.1, outside` and
`S 10.10.0.0 ... via 10.10.1.1, inside`.

### 5b — Inspection: let the replies come back
⌨️ Type:
```
class-map inspection_default
 match default-inspection-traffic
 exit
policy-map global_policy
 class inspection_default
  inspect icmp
  inspect dns
  inspect http
  inspect ftp
  exit
 exit
service-policy global_policy global
```
> If `service-policy global_policy global` says it already exists — fine, PT's ASA has it by
> default.

| Command | In simple English |
|---|---|
| `inspect icmp` | "Remember pings so their replies may come back" |

---

## Mission 6 — ASA, part 3: the outside ACL and SSH

### 6a — The holes from outside (more in steps 12 and 15)
🧠 The edge router is **outside** (level 0). It must send logs, logins and time requests
**inward** — so we open exactly three holes, no more.

⌨️ Type:
```
access-list OUTSIDE_IN extended permit udp host 10.10.7.1 host 10.10.1.10 eq syslog
access-list OUTSIDE_IN extended permit tcp host 10.10.7.1 host 10.10.1.11 eq tacacs
access-list OUTSIDE_IN extended permit udp host 10.10.7.1 host 10.10.200.14 eq ntp
access-group OUTSIDE_IN in interface outside
```
| Command | In simple English |
|---|---|
| `access-list OUTSIDE_IN extended permit ...` | A hole in the firewall for exactly this traffic |
| `access-group OUTSIDE_IN in interface outside` | "Check traffic entering from outside against this list" |

🔍 **Check now:** `show access-list OUTSIDE_IN` → 3 lines.

### 6b — SSH to the ASA, only from the admin networks
⌨️ Type:
```
username admin password Intel#Adm1n
aaa authentication ssh console LOCAL
crypto key generate rsa modulus 1024
```
If asked `Do you really want to replace them? [yes/no]:` → `yes`.

⌨️ Type:
```
ssh 10.10.19.0 255.255.255.0 inside
ssh 10.10.35.0 255.255.255.0 inside
ssh 10.10.51.0 255.255.255.0 inside
ssh 10.10.49.0 255.255.255.0 inside
ssh timeout 5
end
write memory
```
✅ `[OK]` after `write memory`.

| Command | In simple English |
|---|---|
| `ssh 10.10.19.0 255.255.255.0 inside` | "Allow SSH to the ASA from this network, on the inside" |
| `write memory` | The ASA's version of `copy run start` |

**Optional (if your PT accepts it) — ASA logs to Syslog:**
```
configure terminal
logging enable
logging trap informational
logging host inside 10.10.1.10
end
write memory
```

---

## Mission 7 — Default route on CORE1 and CORE2
🧠 **Why:** without it, a PC's packet to the internet reaches the core, and the core says "I
don't know that network" and drops it.

👉 **Where:** **CORE1**, then **CORE2** (log in `admin` / `Intel#Adm1n`).

⌨️ Type on **each**:
```
configure terminal
ip route 0.0.0.0 0.0.0.0 10.10.1.254
router ospf 1
 default-information originate
 end
copy running-config startup-config
```
| Command | In simple English |
|---|---|
| `ip route 0.0.0.0 0.0.0.0 10.10.1.254` | "Everything unknown goes to the ASA" — a **default route** |
| `default-information originate` | "Tell all OSPF neighbours about my default route" |

Tick: - [ ] CORE1  - [ ] CORE2

🔍 **Check now:** on **A-DIST1** `show ip route` → near the top:
`O*E2 0.0.0.0/0 [110/1] via 10.10.4.2 ...` ✅ (the `*` = default route).

---

## Mission 8 — R-HFA-EDGE, the inside side
👉 **Where:** click **R-HFA-EDGE** → **CLI**. A new router asks:
`Would you like to enter the initial configuration dialog? [yes/no]:` → type **`no`**, Enter.
Press **Enter** again → `Router>`.

### 8a — Hardening and the link to the ASA
⌨️ Type:
```
enable
configure terminal
hostname R-HFA-EDGE
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
interface gi0/0
 description to ASA-HFA-FW1 outside
 ip address 10.10.7.1 255.255.255.252
 no shutdown
 exit
ip route 10.10.0.0 255.255.0.0 10.10.7.2
```
> Router ports start **shut** (unlike switch ports) — `no shutdown` is a must.

🔍 **Check now:** `do ping 10.10.7.2` → `!!!!!` (the ASA's outside port).

### 8b — Management
⌨️ Type:
```
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
→ answer **`1024`** → then:
```
ip ssh version 2
login block-for 120 attempts 3 within 60
ip access-list standard MGMT-ONLY
 permit 10.10.19.0 0.0.0.255
 permit 10.10.35.0 0.0.0.255
 permit 10.10.51.0 0.0.0.255
 permit 10.10.49.0 0.0.0.255
 deny any log
 exit
service timestamps log datetime msec
logging trap informational
logging host 10.10.1.10
ntp server 10.10.200.14
aaa new-model
tacacs-server host 10.10.1.11 key Intel#Tac
aaa authentication login default group tacacs+ local
line vty 0 15
 transport input ssh
 login authentication default
 access-class MGMT-ONLY in
 exec-timeout 5 0
line console 0
 login authentication default
 logging synchronous
 end
copy running-config startup-config
```

### 8c — Register it on the AAA server
👉 **SRV-HFA-AAA → Services → AAA → Network Configuration** → Client Name `R-HFA-EDGE`,
Client IP `10.10.7.1`, Secret `Intel#Tac`, ServerType **Tacacs** → **Add**.

🔍 **Check now:** from an IT PC (10.10.49.x): `ssh -l admin 10.10.7.1` → password `Intel#Adm1n`
→ `R-HFA-EDGE#`. This works only because of the `tacacs` hole you opened in 6a. Type `exit`.

---

## Mission 9 — SW-HFA-DMZ
👉 **Where:** **SW-HFA-DMZ** → CLI → **Enter** → `Switch>`.

⌨️ Type:
```
enable
configure terminal
hostname SW-HFA-DMZ
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
username admin privilege 15 secret Intel#Adm1n
line console 0
 login local
 exit
interface range fa0/1 - 6
 description DMZ servers
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
interface range fa0/7 - 24
 description UNUSED
 shutdown
interface gi0/2
 description UNUSED
 shutdown
 end
copy running-config startup-config
```
> **No IP address on purpose.** The DMZ is not allowed to reach the inside, so this switch is
> managed only from the console. Write this design choice in the project book.

---

## Mission 10 — IP addresses on the six DMZ servers (how? → M14)
Each server → **Desktop → IP Configuration → Static**:

| Server | IPv4 | Mask | Gateway | DNS |
|---|---|---|---|---|
| SRV-HFA-DMZ-DNS | 10.10.200.11 | 255.255.255.0 | 10.10.200.1 | 10.10.200.11 |
| SRV-HFA-DMZ-MAIL | 10.10.200.12 | 255.255.255.0 | 10.10.200.1 | 10.10.200.11 |
| SRV-HFA-DMZ-WEB | 10.10.200.13 | 255.255.255.0 | 10.10.200.1 | 10.10.200.11 |
| SRV-HFA-DMZ-NTP | 10.10.200.14 | 255.255.255.0 | 10.10.200.1 | 10.10.200.11 |
| SRV-HFA-DMZ-FTP | 10.10.200.15 | 255.255.255.0 | 10.10.200.1 | 10.10.200.11 |
| SRV-HFA-DMZ-AAA | 10.10.200.16 | 255.255.255.0 | 10.10.200.1 | 10.10.200.11 |

🔍 **Check now:** SRV-HFA-DMZ-DNS → Command Prompt → `ping 10.10.200.1` (the ASA) → replies.
Then `ping 10.10.200.16` → replies.

---

## Mission 11 — One job per server (how? → M20)
🧠 **Why:** in PT many services are **On** by default. A real server runs only its own job —
every extra service is another door for an attacker.

### 11a — Switch off the extras (on all six)
Each server → **Services** tab → click each service in the left list (HTTP, HTTPS, DHCP, DNS,
EMAIL, FTP, NTP, AAA, SYSLOG…) → **Off**, **except** the server's own one.
Tick: - [ ] DNS  - [ ] MAIL  - [ ] WEB  - [ ] NTP  - [ ] FTP  - [ ] AAA

### 11b — SRV-HFA-DMZ-NTP
**Services → NTP** → **On**. Look at the date and time shown — every device will copy it.

### 11c — SRV-HFA-DMZ-DNS
**Services → DNS** → **On**. For each row: type the *Name*, choose *Type* **A Record**, type the
*Address* → **Add**:

| Name | Address |
|---|---|
| `www.intel.lab` | 10.10.200.13 |
| `mail.intel.lab` | 10.10.200.12 |
| `ftp.intel.lab` | 10.10.200.15 |
| `ntp.intel.lab` | 10.10.200.14 |
| `www.isp-web1.com` | 200.1.100.11 |
| `www.isp-web2.com` | 200.1.100.12 |

✅ 6 lines in the table below. (PT's DNS server can't ask other DNS servers, so we copy the
internet names of step 12 in too.)

❌ **A typo?** Click the line → **Remove** → add it again.

### 11d — SRV-HFA-DMZ-WEB
**Services → HTTP** (On) → in the file list click **index.html** → **(edit)** → replace the text
with `<h1>Intel Israel - Intranet</h1>` → **Save** → **Yes**.

### 11e — SRV-HFA-DMZ-MAIL
**Services → EMAIL** → *Domain Name*: `intel.lab` → **Set**. (Users come in step 16.)

### 11f — SRV-HFA-DMZ-FTP
**Services → FTP** (On) → *Username* `ftpadmin`, *Password* `Intel#Ftp`, tick all permissions
(Write, Read, Delete, Rename, List) → **Add**. Then select the default user `cisco` → **Remove**.

### 11g — SRV-HFA-DMZ-AAA
**Services → AAA** → **On** (it will serve remote-access users; nothing to add yet).

---

## Mission 12 — NTP on every HQ switch
👉 **Where:** **all 15 Haifa switches** — log in with `admin` / `Intel#Adm1n`.

⌨️ Type on each:
```
configure terminal
ntp server 10.10.200.14
end
copy running-config startup-config
```
Tick:
- Building A: - [ ] A-ACC1  - [ ] A-ACC2  - [ ] A-DIST1  - [ ] A-DIST2
- Building B: - [ ] B-ACC1  - [ ] B-ACC2  - [ ] B-DIST1  - [ ] B-DIST2
- Building C: - [ ] C-ACC1  - [ ] C-ACC2  - [ ] C-DIST1  - [ ] C-DIST2
- Core + servers: - [ ] CORE1  - [ ] CORE2  - [ ] SW-HFA-SRV

> 💡 Faster: SSH from one admin PC to each switch in turn (step 07) — that's how real admins do it.

Press **Fast Forward Time** several times (NTP takes a few minutes of simulated time).

🔍 **Check now:** on any switch `show ntp status` → `Clock is synchronized ... reference is
10.10.200.14`. `show clock` → the same time as the NTP server.

---

## Mission 13 — Test through the firewall

| # | From | Type | Expected | Why |
|---|---|---|---|---|
| 1 | an ENG PC | `ping 10.10.200.13` | ✅ | inside (100) → DMZ (50) |
| 2 | an ENG PC | `ipconfig /renew`, then `nslookup www.intel.lab` | `10.10.200.13` | DNS works |
| 3 | an ENG PC | Desktop → **Web Browser** → `http://www.intel.lab` → **Go** | ✅ the intranet page | DNS + HTTP through the ASA |
| 4 | an ENG PC | `ping 10.10.7.1` | ✅ | inside → outside |
| 5 | SRV-HFA-DMZ-WEB | `ping 10.10.16.1` | ❌ | DMZ (50) → inside (100) |
| 6 | R-HFA-EDGE | `ping 10.10.200.13` | ❌ | outside → DMZ, not on the ACL |
| 7 | any switch | `show ntp status` | synchronized to 10.10.200.14 | NTP |

🔍 On the ASA, **while** you browse from a PC: `show conn` lists the live connections.

💾 `write memory` on the ASA, `copy running-config startup-config` on the rest → **File → Save**.

---

## Checkpoint
- [ ] `show nameif` shows the three zones
- [ ] PCs browse `http://www.intel.lab`
- [ ] DMZ can't reach the inside
- [ ] All switches synchronized to 10.10.200.14

## If something doesn't work
| Problem | Fix |
|---|---|
| ASA interfaces ignore traffic | `nameif` missing — an ASA interface without a name does nothing |
| PCs get `192.168.1.x` addresses | The ASA's factory DHCP server: `no dhcpd enable inside` |
| Browsing works but ping to the DMZ fails | `inspect icmp` missing |
| PCs can't reach anything past the cores | Default route or `default-information originate` missing on the cores |
| Page shows "Host Name Unresolved" | DNS server's DNS service off, record typo, or PC's DNS isn't 10.10.200.11 (renew DHCP) |
| `show ntp status` = unsynchronized | Wait (Fast Forward); check the NTP server's service is On; ping 10.10.200.14 from the switch |
| Edge router can't log in through TACACS | Missing ACL line for tacacs on the ASA, or R-HFA-EDGE not registered on the AAA server |
| Everything on the ASA is red | Cables in the wrong ASA ports — 1/1 outside, 1/2 inside, 1/3 DMZ |

---

## 🏋 Exercises

### Exercise A — The hacked web server ⭐⭐
- 🎯 **Task:** a hacker takes over SRV-HFA-DMZ-WEB. List three things they **can** reach and three
  they **cannot**. Prove two of each with pings.
- 📁 **Start from:** your `step11.pkt` (nothing is changed).
- 👣 **Steps:**
  1. Open SRV-HFA-DMZ-WEB → **Desktop → Command Prompt**.
  2. Ping: another DMZ server, the ASA's DMZ port, the edge router, an Engineering PC, the
     Syslog server, a MGMT address.
  3. Write a table: target / result / why.
- ✅ **Done when:** your table has 6 rows and every "why" uses the words **security level**.

<details><summary>💡 Hint</summary>

Which pings pass through the ASA from a lower level to a higher level?
</details>
<details><summary>✔ Answer</summary>

**Can:** the other DMZ servers (same subnet — no firewall between them), the ASA's DMZ interface,
the outside (edge router, and the internet after step 12).
**Cannot:** any inside network — users (10.10.16.x…), the server farm (10.10.1.x), the management
VLANs — because DMZ (50) → inside (100) is blocked and no ACL opens it.
For the book: real networks also separate the DMZ servers from each other ("private VLANs") —
PT doesn't support it well, so mention it as a known limit.
</details>

### Exercise B — Forget a `nameif` ⭐
- 🎯 **Task:** see what an ASA interface without a name does.
- 📁 **Start from:** `step11.pkt` → **Save As** `step11-exB.pkt`.
- 👣 **Steps:**
  1. ASA → `configure terminal` → `interface gigabitEthernet 1/3` → `no nameif` → `end`.
  2. From an ENG PC: `ping 10.10.200.13`.
  3. `show nameif` and `show interface ip brief` on the ASA.
- ✅ **Done when:** you can explain the result, then fix it (`nameif dmz`, `security-level 50`,
  `ip address 10.10.200.1 255.255.255.0`).

<details><summary>✔ Answer</summary>

The ping fails. Removing `nameif` also removes the security level and the IP address — the port
is "up" but the ASA ignores it. You must type all three lines again.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

- [ ] Open `my-project/step10.pkt` → **Save As** `my-project/step11.pkt`
- [ ] Place and cable the ASA, edge router, DMZ switch, 6 servers (check the ASA ports!)
- [ ] ASA: zones (`show nameif`), routes, inspection, outside ACL, SSH
- [ ] Default route + `default-information originate` on your cores; check `O*E2` on a DIST
- [ ] Edge router inside side + management; register it on AAA
- [ ] DMZ switch, 6 server IPs, one service per server
- [ ] NTP on every HQ switch; the test table of Mission 13 with your addresses

| In the example | In your project — take it from |
|---|---|
| DMZ `10.10.200.0/24` and the 6 servers | plan part C/D — your DMZ subnet and server addresses (the server **list** is set by the ministry, req. 21) |
| inside / outside / dmz, levels 100 / 0 / 50 | keep (this is the standard ASA model) |
| ASA link `10.10.7.0/30` | your own /30 |
| ACL rules | rewritten for **your** addresses |
| Main NTP server in the DMZ | your NTP server address; all your sites sync from it (req. 20.8) |

**Check:** run the table of Mission 13 with **your** addresses — inside reaches the DMZ, the DMZ
cannot reach inside, the outside cannot reach the DMZ; `show nameif` shows your three
interfaces; `show ntp status` on a switch shows your NTP server.

💾 Save as `my-project/step11.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★1 What is an ASA?**

<details><summary>Answer</summary>

Adaptive Security Appliance — Cisco's firewall. It inspects traffic between zones (inside, outside, DMZ) and allows only what the policy permits. It is **stateful**: return traffic of a permitted connection is allowed automatically.

**Show it:** `show nameif`, `show conn`
</details>

**2. ★1 What is a DMZ?**

<details><summary>Answer</summary>

A separate zone for servers that the internet must reach (web, mail, DNS…). If one of them is hacked, the attacker is still outside the inside network.

**Show it:** `show nameif` → dmz
</details>

**3. ★1 A firewall has 3 interfaces: inside, DMZ, outside. Which security level do you give each?**

<details><summary>Answer</summary>

inside 100, DMZ 50, outside 0. Traffic from a higher level to a lower level is allowed by default; from lower to higher it needs an ACL.

**Show it:** `show nameif`
</details>

**4. ★1 What is the role of an HTTP server? Which port? What is HTTPS and what is the difference?**

<details><summary>Answer</summary>

Serves web pages. HTTP = TCP 80, clear text. HTTPS = TCP 443, HTTP encrypted with TLS (certificate) — passwords and data can't be read on the way.

**Show it:** Server → Services → HTTP / HTTPS
</details>

**5. ★1 What is the role of a DNS server? Which port? Show its configuration.**

<details><summary>Answer</summary>

Translates names to IP addresses (`www.intel.lab` → `10.10.200.13`). UDP 53 (TCP 53 for large transfers).

**Show it:** Server → Services → DNS (the records); PC → `nslookup <name>`
</details>

**6. ★1 What is the role of an FTP server? Which port? Show its configuration.**

<details><summary>Answer</summary>

File transfer. TCP 21 (control) and TCP 20 (data). Users and their permissions are defined on the server.

**Show it:** Server → Services → FTP; PC → `ftp <IP>`
</details>

**7. ★1 What is the role of a mail server? Show its configuration.**

<details><summary>Answer</summary>

Sends and stores email: SMTP (TCP 25) to send, POP3 (TCP 110) to download to the client. The server has the domain name and the user accounts.

**Show it:** Server → Services → EMAIL; PC → Desktop → Email
</details>

**8. ★3 How can an ACL allow internet browsing but block certain websites?**

<details><summary>Answer</summary>

An extended ACL: `deny tcp <users> host <site IP> eq 80` and `eq 443` for each blocked site, then `permit ip <users> any`. Applied as close to the users as possible.

**Show it:** —
</details>

**9. ★3 How can ACLs help protect a network against DDoS attacks?**

<details><summary>Answer</summary>

On the outside interface: permit only the services you publish (e.g. TCP 80/443 to the web server), deny everything else; drop packets with private or your own source addresses arriving from outside (spoofing). An ACL reduces exposure but cannot stop a large flood alone — the ISP has to filter that.

**Show it:** The ACL on the ASA outside interface
</details>

**10. ★3 In the routing table, what is the difference between E1 and E2? Which is the default?**

<details><summary>Answer</summary>

Both are external routes (type 5). E2 (default): the cost stays the external cost only — it doesn't grow inside OSPF. E1: the internal OSPF cost is added along the way. The default route this project sends into OSPF shows as `O*E2`.

**Show it:** `show ip route` on a distribution switch → `O*E2 0.0.0.0/0`
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

In a copy of your file, replace the ASA with a new one from the device list.
Without this page, configure the three interfaces with names, levels and addresses from your plan,
the routes and the ACL. Target: **20 minutes**. Then run the test table of Mission 13 with your
addresses and explain each ✅ and ❌.

---

## 👩‍🏫 Teacher notes
- **Lesson 1:** firewall vs router; zones and the office-building comparison; stateful; DMZ purpose
  → Missions 1–6 (build, cable, ASA). Check every student's `show nameif` before they leave.
- **Lesson 2:** Missions 7–9 — default route + `default-information originate` → edge router (first
  router in the course: the "initial configuration dialog" → `no`) → DMZ switch.
- **Lesson 3:** Missions 10–13 — the six servers (lots of clicking — split the class: each student
  sets up two servers and explains them) → NTP → tests → Exercise A.
- **Common student questions**
  - *"Why is the DMZ switch without an IP?"* — Managing it from the inside would need a hole from
    DMZ → inside, which defeats the DMZ. Console-only is the safe choice.
  - *"Why `write memory` on the ASA?"* — It's the ASA's version of `copy run start`.
- **Typical errors:** forgetting `nameif`; mixing up the ASA's 1/1 / 1/2 / 1/3 cabling; leaving DHCP
  on the ASA; DNS records typed with a typo (`www.intel.lab` vs `www.intel.lab.`).
