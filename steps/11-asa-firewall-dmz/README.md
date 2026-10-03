# Step 11 — ASA firewall, DMZ & the main NTP server

## Goal
An **ASA 5506-X** splits HQ into three security zones: **inside** (the LAN),
**DMZ** (public-facing servers) and **outside** (toward the internet edge router). The DMZ
holds the six required servers, including the **main NTP server** every device in the
company will sync to.

Ministry requirements: **11, 12, 21 (DNS, EMAIL, WEB, NTP, FTP, AAA), 20.8, 22.13**

## Concepts
- **Security levels** (0–100). Traffic from a **higher** level to a **lower** level is
  allowed by default; lower → higher is **denied** unless an ACL permits it.

  | Zone | nameif | Level | Means |
  |---|---|---|---|
  | LAN | inside | 100 | most trusted |
  | DMZ | dmz | 50 | servers the internet may reach |
  | Edge side | outside | 0 | untrusted |

  → inside → DMZ ✅ · inside → outside ✅ · DMZ → inside ❌ · outside → anything ❌ (until ACL)
- **Stateful** — the ASA remembers connections, so *replies* to allowed traffic come back
  automatically. ICMP is not stateful by default → we add `inspect icmp` so ping replies
  return.
- **Why a DMZ?** If the web server is hacked, the attacker lands in the DMZ — and the DMZ
  can't open connections into the inside network.
- **Who does NAT?** Not the ASA. The **edge router** (step 12) does all NAT. The ASA only
  filters and routes. This keeps private IPs visible end-to-end for the VPN (step 15).

## Build

| Device | Name |
|---|---|
| ASA 5506-X | `ASA-HFA-FW1` |
| 2911 router | `R-HFA-EDGE` |
| 2960-24TT | `SW-HFA-DMZ` |
| Server-PT ×6 | see table |

| From | To |
|---|---|
| SW-HFA-SRV Fa0/24 | ASA Gi1/2 (**inside**) |
| ASA Gi1/3 (**dmz**) | SW-HFA-DMZ Gi0/1 |
| ASA Gi1/1 (**outside**) | R-HFA-EDGE Gi0/0 |
| DMZ servers | SW-HFA-DMZ Fa0/1 – Fa0/6 |

| Server | IP (gw 10.10.200.1, DNS 10.10.200.11) | Service to turn on |
|---|---|---|
| SRV-HFA-DMZ-DNS | 10.10.200.11 | DNS |
| SRV-HFA-DMZ-MAIL | 10.10.200.12 | EMAIL (domain `intel.lab`) |
| SRV-HFA-DMZ-WEB | 10.10.200.13 | HTTP / HTTPS |
| SRV-HFA-DMZ-NTP | 10.10.200.14 | NTP |
| SRV-HFA-DMZ-FTP | 10.10.200.15 | FTP (user `ftpadmin` / `Intel#Ftp`) |
| SRV-HFA-DMZ-AAA | 10.10.200.16 | AAA |

**Turn off every other service** on each server (each server does one job — smaller attack
surface). In PT, HTTP/DNS/etc. are ON by default on every Server-PT.

## Configure

### 1. ASA-HFA-FW1
```
enable
! (first time the enable password is empty — just press Enter)
configure terminal
hostname ASA-HFA-FW1
enable password Intel#En4ble
domain-name intel.lab
! the factory config runs a DHCP server on inside — kill it
no dhcpd enable inside

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
route outside 0.0.0.0 0.0.0.0 10.10.7.1
route inside 10.10.0.0 255.255.0.0 10.10.1.1

! ---------- stateful inspection (ping replies, DNS, HTTP, FTP) ----------
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

! ---------- outside -> in: only what we need (more lines in steps 12 & 15) ----------
! the edge router sends syslog, TACACS+ and NTP inward
access-list OUTSIDE_IN extended permit udp host 10.10.7.1 host 10.10.1.10 eq syslog
access-list OUTSIDE_IN extended permit tcp host 10.10.7.1 host 10.10.1.11 eq tacacs
access-list OUTSIDE_IN extended permit udp host 10.10.7.1 host 10.10.200.14 eq ntp
access-group OUTSIDE_IN in interface outside

! ---------- SSH management of the ASA itself, from the MGMT/IT networks only ----------
username admin password Intel#Adm1n
aaa authentication ssh console LOCAL
crypto key generate rsa modulus 1024
ssh 10.10.19.0 255.255.255.0 inside
ssh 10.10.35.0 255.255.255.0 inside
ssh 10.10.51.0 255.255.255.0 inside
ssh 10.10.49.0 255.255.255.0 inside
ssh timeout 5

! ---------- logging ----------
logging enable
logging trap informational
logging host inside 10.10.1.10
end
write memory
```

> `route inside 10.10.0.0 255.255.0.0 10.10.1.1` — "the whole HQ block is behind the core
> (HSRP VIP of VLAN 100)". The connected subnets (10.10.1.0/24, 10.10.200.0/24) are more
> specific, so they still win.

### 2. CORE1 and CORE2 — send unknown traffic to the ASA, tell OSPF about it
```
configure terminal
ip route 0.0.0.0 0.0.0.0 10.10.1.254
router ospf 1
 default-information originate
end
wr
```
Every OSPF router in HQ now has `O*E2 0.0.0.0/0` → the "gateway of last resort".

### 3. R-HFA-EDGE — first part (inside only; WAN comes in step 12)
Hardening + SSH (steps 02/07), then:
```
configure terminal
interface gi0/0
 description to ASA-HFA-FW1 outside
 ip address 10.10.7.1 255.255.255.252
 no shutdown
 exit
ip route 10.10.0.0 255.255.0.0 10.10.7.2

service timestamps log datetime msec
logging host 10.10.1.10
ntp server 10.10.200.14
end
wr
```
Add `R-HFA-EDGE` / `10.10.7.1` to SRV-HFA-AAA and configure AAA on it as in step 08.

### 4. DMZ servers
- **NTP** (SRV-HFA-DMZ-NTP): *Services → NTP → On*, set the date/time. This is the
  company's **main NTP server**.
- **DNS** (SRV-HFA-DMZ-DNS): *Services → DNS → On*, add A records:

  | Name | Address |
  |---|---|
  | `www.intel.lab` | 10.10.200.13 |
  | `mail.intel.lab` | 10.10.200.12 |
  | `ftp.intel.lab` | 10.10.200.15 |
  | `ntp.intel.lab` | 10.10.200.14 |
  | `www.isp-web1.com` | 200.1.100.11 |
  | `www.isp-web2.com` | 200.1.100.12 |

  (PT's DNS server can't forward to the internet, so we copy the internet names in.)
- **WEB**: edit `index.html` → "Intel Israel — Intranet".
- **MAIL**: domain `intel.lab` (users in step 16).
- **FTP**: add user `ftpadmin` / `Intel#Ftp` with all permissions; delete the default `cisco` user.
- **SW-HFA-DMZ**: hardening; all ports VLAN 1 is OK here (it's a single-zone switch), but put
  unused ports in shutdown. No IP — it's managed from the console only (DMZ must not reach
  the inside).

### 5. NTP on every HQ switch
```
ntp server 10.10.200.14
```
Then `show clock` — after a short while all devices show the same time, and Syslog
messages carry real timestamps.

## Verify

```
ASA-HFA-FW1# show nameif
Interface              Name      Security
GigabitEthernet1/1     outside     0
GigabitEthernet1/2     inside    100
GigabitEthernet1/3     dmz        50
ASA-HFA-FW1# show route
ASA-HFA-FW1# show conn          <- live connections while you test
```

| From | Test | Expected | Why |
|---|---|---|---|
| ENG PC | `ping 10.10.200.13` | ✅ | inside(100) → dmz(50) |
| ENG PC | browser `http://www.intel.lab` | ✅ intranet page | DNS + HTTP through the ASA |
| ENG PC | `ping 10.10.7.1` | ✅ | inside → outside |
| SRV-HFA-DMZ-WEB | `ping 10.10.16.1` | ❌ | dmz(50) → inside(100) |
| R-HFA-EDGE | `ping 10.10.200.13` | ❌ | outside → dmz, not in ACL |
| any switch | `show ntp status` | `Clock is synchronized` | |

## Checkpoint
`my-work/step11.pkt` + commit.

## Common mistakes
- Pings to the DMZ fail but browsing works → you forgot `inspect icmp`.
- Inside PCs reach the ASA but nothing beyond → the cores have no default route, or you
  forgot `default-information originate`.
- `ip address` on an ASA interface without `nameif` first → the ASA ignores the interface.

## Challenge
A hacker owns SRV-HFA-DMZ-WEB. List three things he **can** reach and three things he
**cannot**, and prove two of them with pings.

<details><summary>Answer</summary>

**Can**: the other DMZ servers (same subnet, no firewall between them), the ASA's dmz
interface, the outside (edge router / later the internet).
**Cannot**: any inside subnet (10.10.16.x users, 10.10.1.x servers, management VLANs) —
dmz(50) → inside(100) is denied and no ACL permits it.
Lesson for the book: in real life you'd also isolate DMZ servers from each other
(private VLANs) — the simulator doesn't support that well, so mention it as a known limit.
</details>
