# Step 11 — ASA firewall, DMZ & the main NTP server

> ⏱ 3 lessons · 🎯 ministry requirements 11, 12, 21, 20.8, 22.13 · 📁 start from: `my-work/step10.pkt` · 💾 save as: `my-work/step11.pkt`

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
is hacked, the attacker lands in the **DMZ** — and from the DMZ he **cannot open connections
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

---

## Part 2 — Prepare
**File → Open** → `my-work/step10.pkt` → **File → Save As** → `my-work/step11.pkt`.

---

## Part 3 — Build (click by click)

### 3.1 Devices
| Where | Model | Display Name |
|---|---|---|
| Network Devices → **Security** | **5506-X** | `ASA-HFA-FW1` |
| Network Devices → **Routers** | **2911** | `R-HFA-EDGE` |
| Network Devices → Switches | **2960** | `SW-HFA-DMZ` |
| End Devices → End Devices | **Server** ×6 | the six names in 1.4 |

Draw a rectangle labelled `HAIFA - DMZ` around the DMZ switch and its servers.

### 3.2 Cables
| Click first | Port | Click second | Port | Cable |
|---|---|---|---|---|
| SW-HFA-SRV | **Fa0/24** | ASA-HFA-FW1 | **GigabitEthernet1/2** | Straight-Through |
| ASA-HFA-FW1 | **GigabitEthernet1/3** | SW-HFA-DMZ | **GigabitEthernet0/1** | Straight-Through |
| ASA-HFA-FW1 | **GigabitEthernet1/1** | R-HFA-EDGE | **GigabitEthernet0/0** | Straight-Through |
| SRV-HFA-DMZ-DNS | Fa0 | SW-HFA-DMZ | Fa0/1 | Straight-Through |
| SRV-HFA-DMZ-MAIL | Fa0 | SW-HFA-DMZ | Fa0/2 | Straight-Through |
| SRV-HFA-DMZ-WEB | Fa0 | SW-HFA-DMZ | Fa0/3 | Straight-Through |
| SRV-HFA-DMZ-NTP | Fa0 | SW-HFA-DMZ | Fa0/4 | Straight-Through |
| SRV-HFA-DMZ-FTP | Fa0 | SW-HFA-DMZ | Fa0/5 | Straight-Through |
| SRV-HFA-DMZ-AAA | Fa0 | SW-HFA-DMZ | Fa0/6 | Straight-Through |

(The ASA behaves like a router-type device: Straight-Through to switches and routers works in PT.
If a link stays red, try Cross-Over.)

---

## Part 4 — Configure

### 4.1 ASA-HFA-FW1
Click it → **CLI**. The ASA prompt is `ciscoasa>`.
```
enable
```
`Password:` → just press **Enter** (empty on a new ASA).
```
configure terminal
hostname ASA-HFA-FW1
enable password Intel#En4ble
domain-name intel.lab
! the factory config runs a DHCP server on "inside" — turn it off
no dhcpd enable inside

! ---------- the three zones ----------
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

! ---------- routing ----------
! everything unknown -> the edge router
route outside 0.0.0.0 0.0.0.0 10.10.7.1
! all of HQ is behind the cores (VLAN 100 gateway)
route inside 10.10.0.0 255.255.0.0 10.10.1.1

! ---------- inspection: let replies (ping, DNS, web, FTP) come back ----------
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

! ---------- the holes from outside (more in steps 12 and 15) ----------
! the edge router sends its logs, logins and time requests inward
access-list OUTSIDE_IN extended permit udp host 10.10.7.1 host 10.10.1.10 eq syslog
access-list OUTSIDE_IN extended permit tcp host 10.10.7.1 host 10.10.1.11 eq tacacs
access-list OUTSIDE_IN extended permit udp host 10.10.7.1 host 10.10.200.14 eq ntp
access-group OUTSIDE_IN in interface outside

! ---------- SSH to the ASA, only from the admin networks ----------
username admin password Intel#Adm1n
aaa authentication ssh console LOCAL
crypto key generate rsa modulus 1024
ssh 10.10.19.0 255.255.255.0 inside
ssh 10.10.35.0 255.255.255.0 inside
ssh 10.10.51.0 255.255.255.0 inside
ssh 10.10.49.0 255.255.255.0 inside
ssh timeout 5
end
write memory
```
> If `service-policy global_policy global` says it already exists — fine, PT's ASA has it by
> default. If `crypto key generate` asks to replace keys, answer `yes`.

**Optional (if your PT accepts it) — ASA logs to Syslog:**
```
configure terminal
logging enable
logging trap informational
logging host inside 10.10.1.10
end
write memory
```

### 4.2 CORE1 and CORE2 — the default route
On **both** cores:
```
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 10.10.1.254
router ospf 1
 default-information originate
 end
copy running-config startup-config
```

### 4.3 R-HFA-EDGE — first part (the inside side)
Click it → **CLI**. A new router asks:
`Would you like to enter the initial configuration dialog? [yes/no]:` → type **`no`**, Enter.
Press Enter again → `Router>`.
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
! all of HQ is behind the ASA
ip route 10.10.0.0 255.255.0.0 10.10.7.2

! ---------- management ----------
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
(`1024`)
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
Then on **SRV-HFA-AAA → Services → AAA → Network Configuration** add: `R-HFA-EDGE`, `10.10.7.1`,
`Intel#Tac`, **Tacacs**.

### 4.4 SW-HFA-DMZ
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

### 4.5 The six DMZ servers (click by click)
**IP settings** — each server → **Desktop → IP Configuration → Static**:

| Server | IPv4 | Mask | Gateway | DNS |
|---|---|---|---|---|
| SRV-HFA-DMZ-DNS | 10.10.200.11 | 255.255.255.0 | 10.10.200.1 | 10.10.200.11 |
| SRV-HFA-DMZ-MAIL | 10.10.200.12 | 255.255.255.0 | 10.10.200.1 | 10.10.200.11 |
| SRV-HFA-DMZ-WEB | 10.10.200.13 | 255.255.255.0 | 10.10.200.1 | 10.10.200.11 |
| SRV-HFA-DMZ-NTP | 10.10.200.14 | 255.255.255.0 | 10.10.200.1 | 10.10.200.11 |
| SRV-HFA-DMZ-FTP | 10.10.200.15 | 255.255.255.0 | 10.10.200.1 | 10.10.200.11 |
| SRV-HFA-DMZ-AAA | 10.10.200.16 | 255.255.255.0 | 10.10.200.1 | 10.10.200.11 |

**One job per server** — on each server → **Services** tab: switch **Off** every service except
its own (in PT many services are On by default).

**SRV-HFA-DMZ-NTP** → Services → **NTP** → **On**. Check the date and time shown.

**SRV-HFA-DMZ-DNS** → Services → **DNS** → **On**. Add each record: type the *Name*, choose
*Type* **A Record**, type the *Address* → **Add**:

| Name | Address |
|---|---|
| `www.intel.lab` | 10.10.200.13 |
| `mail.intel.lab` | 10.10.200.12 |
| `ftp.intel.lab` | 10.10.200.15 |
| `ntp.intel.lab` | 10.10.200.14 |
| `www.isp-web1.com` | 200.1.100.11 |
| `www.isp-web2.com` | 200.1.100.12 |

(PT's DNS server can't ask other DNS servers, so we copy the internet names of step 12 in too.)

**SRV-HFA-DMZ-WEB** → Services → **HTTP** (On) → in the file list click **index.html** → **(edit)**
→ change the text to `<h1>Intel Israel - Intranet</h1>` → **Save** → **Yes**.

**SRV-HFA-DMZ-MAIL** → Services → **EMAIL** → *Domain Name*: `intel.lab` → **Set**. (Users in step 16.)

**SRV-HFA-DMZ-FTP** → Services → **FTP** (On) → *Username* `ftpadmin`, *Password* `Intel#Ftp`,
tick all permissions (Write, Read, Delete, Rename, List) → **Add**. Select the default user
`cisco` → **Remove**.

**SRV-HFA-DMZ-AAA** → Services → **AAA** → **On** (it will serve remote-access users; nothing to add yet).

### 4.6 NTP on every HQ switch
On **all 15 Haifa switches** (A/B/C ACC + DIST, CORE1/2, SW-HFA-SRV):
```
enable
configure terminal
ntp server 10.10.200.14
end
copy running-config startup-config
```

### 4.7 What the new commands mean
| Command | In simple English |
|---|---|
| `nameif outside` | Give this ASA interface a zone name |
| `security-level 0` | Its trust level |
| `route outside 0.0.0.0 0.0.0.0 10.10.7.1` | "Unknown destinations: out of `outside` to 10.10.7.1" |
| `inspect icmp` | "Remember pings so their replies may come back" |
| `access-list OUTSIDE_IN extended permit ...` | A hole in the firewall for exactly this traffic |
| `access-group OUTSIDE_IN in interface outside` | "Check traffic entering from outside against this list" |
| `ssh 10.10.19.0 255.255.255.0 inside` | "Allow SSH to the ASA from this network, on the inside" |
| `ip route 0.0.0.0 0.0.0.0 10.10.1.254` (core) | "Everything unknown goes to the ASA" — a **default route** |
| `default-information originate` | "Tell all OSPF neighbours about my default route" |
| `ntp server 10.10.200.14` | "Take your time from this clock" |

---

## Part 5 — Check your work

### 5.1 The ASA
```
ASA-HFA-FW1# show nameif
Interface                Name                     Security
GigabitEthernet1/1       outside                    0
GigabitEthernet1/2       inside                   100
GigabitEthernet1/3       dmz                       50
ASA-HFA-FW1# show route
S*   0.0.0.0 0.0.0.0 [1/0] via 10.10.7.1, outside
S    10.10.0.0 255.255.0.0 [1/0] via 10.10.1.1, inside
C    10.10.1.0 255.255.255.0 is directly connected, inside
...
```

### 5.2 The default route reached every switch
On **A-DIST1**: `show ip route | include 0.0.0.0` → `O*E2 0.0.0.0/0 ... via 10.10.4.2 ...` ✅

### 5.3 Tests through the firewall
| From | Type | Expected | Why |
|---|---|---|---|
| an ENG PC | `ping 10.10.200.13` | ✅ | inside (100) → DMZ (50) |
| an ENG PC | Desktop → **Web Browser** → `http://www.intel.lab` → Go | ✅ the intranet page | DNS + HTTP through the ASA |
| an ENG PC | `nslookup www.intel.lab` | `10.10.200.13` | DNS works |
| an ENG PC | `ping 10.10.7.1` | ✅ | inside → outside |
| SRV-HFA-DMZ-WEB | `ping 10.10.16.1` | ❌ | DMZ (50) → inside (100) |
| R-HFA-EDGE | `ping 10.10.200.13` | ❌ | outside → DMZ, not on the ACL |
| any switch | `show ntp status` | `Clock is synchronized ... reference is 10.10.200.14` | NTP |
| any switch | `show clock` | the same time as the NTP server | |

On the ASA, while you browse from a PC: `show conn` lists the live connections.

> NTP can take a few minutes of simulated time — press **Fast Forward Time** several times.

---

## Part 6 — Save
`write memory` on the ASA, `copy running-config startup-config` on the rest → **File → Save**.

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

## Challenge
A hacker takes over SRV-HFA-DMZ-WEB. List three things he **can** reach and three he **cannot**.
Prove two of them with pings from the web server's Command Prompt.

<details><summary>Answer</summary>

**Can:** the other DMZ servers (same subnet — no firewall between them), the ASA's DMZ interface,
the outside (edge router, and the internet after step 12).
**Cannot:** any inside network — users (10.10.16.x…), the server farm (10.10.1.x), the management
VLANs — because DMZ (50) → inside (100) is blocked and no ACL opens it.
For the book: real networks also separate the DMZ servers from each other ("private VLANs") —
PT doesn't support it well, so mention it as a known limit.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

| In the example | In your project — take it from |
|---|---|
| DMZ `10.10.200.0/24` and the 6 servers | plan part C/D — your DMZ subnet and server addresses (the server **list** is set by the ministry, req. 21) |
| inside / outside / dmz, levels 100 / 0 / 50 | keep (this is the standard ASA model) |
| ASA link `10.10.7.0/30` | your own /30 |
| ACL rules | rewritten for **your** addresses |
| Main NTP server in the DMZ | your NTP server address; all your sites sync from it (req. 20.8) |

**Check:** run the table of 5.3 with **your** addresses — inside reaches the DMZ, the DMZ
cannot reach inside, the outside cannot reach the DMZ; `show nameif` shows your three
interfaces; `show ntp status` on a switch shows your NTP server.

💾 Save as `my-project/step11.pkt`.

---

## 👩‍🏫 Teacher notes
- **Lesson 1:** firewall vs router; zones and the office-building comparison; stateful; DMZ purpose
  → build and cable → configure the ASA.
- **Lesson 2:** default route + `default-information originate` → edge router (first router in the
  course: the "initial configuration dialog" → `no`) → DMZ switch.
- **Lesson 3:** the six servers (lots of clicking — split the class: each student sets up two servers
  and explains them) → NTP → tests → challenge.
- **Common student questions**
  - *"Why is the DMZ switch without an IP?"* — Managing it from the inside would need a hole from
    DMZ → inside, which defeats the DMZ. Console-only is the safe choice.
  - *"Why `write memory` on the ASA?"* — It's the ASA's version of `copy run start`.
- **Typical errors:** forgetting `nameif`; mixing up the ASA's Gi1/1/1/2/1/3 cabling; leaving DHCP
  on the ASA; DNS records typed with a typo (`www.intel.lab` vs `www.intel.lab.`).
