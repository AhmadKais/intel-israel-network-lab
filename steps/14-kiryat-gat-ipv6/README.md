# Step 14 — Kiryat Gat: the IPv6 branch

> ⏱ 4 lessons · 🎯 ministry requirements 7, 10 (overload), 17, 20.x, 22.11 (student's choice) · 📁 start from: `my-work/step13.pkt` · 💾 save as: `my-work/step14.pkt`

## What you will have at the end
The third branch runs **IPv6** for all users, servers and management: automatic addresses
(SLAAC + stateless DHCPv6), **OSPFv3** routing, **HSRP for IPv6**. One small legacy IPv4 VLAN
reaches the internet through **PAT (overload)** — the third NAT type.

```
                          R-ISPC-PE (ISP-C, dual-stack)
                                │ 2001:DB8:C:1::/64 + 200.3.1.0/30
                           R-KG-EDGE    (OSPFv3, static defaults, PAT for VLAN 199)
                    Gi0/0 ╱         ╲ Gi0/2
          SWL3-KG-DIST1 ════ trunk ════ SWL3-KG-DIST2      OSPFv3 area 0, HSRPv6
           ╱    │    │   ╲            ╱    │    │   ╲
        ACC1  ACC2  ACC3  SW-KG-SRV   (each with a cable to BOTH DIST switches)
```

---

## Part 1 — Understand it first (simple English)

### 1.1 Why IPv6?
IPv4 has about 4 billion addresses — the world ran out. That's why we need private addresses
and NAT. **IPv6** has 340 undecillion addresses (a 39-digit number): every device can have a
**public, unique** address. No NAT needed. The ministry requires one branch to be fully IPv6 (req. 7).

### 1.2 How to read an IPv6 address
128 bits, written as **8 groups** of 4 hexadecimal digits (0–9, A–F), separated by `:`.
```
2001:0DB8:0030:0031:0000:0000:0000:0002
```
Two shortening rules:
1. Drop **leading zeros** in each group: `0DB8 → DB8`, `0030 → 30`, `0002 → 2`.
2. Replace **one** run of all-zero groups with `::`.
```
2001:DB8:30:31::2          (the same address)
```
- `/64` — like `/24` in IPv4, but standard for **every** LAN: the first 64 bits are the network,
  the last 64 are the device.
- `2001:DB8::/32` is the range reserved for documentation and labs (like 192.0.2.x).

### 1.3 Our plan: the VLAN number is in the address
Kiryat Gat owns `2001:DB8:30::/48`. Each VLAN gets a /64 whose **4th group = the VLAN number**:
VLAN 31 → `2001:DB8:30:31::/64`, VLAN 99 → `2001:DB8:30:99::/64`. Easy to read.

| VLAN | Name | Prefix | DIST1 | DIST2 | Gateway |
|---|---|---|---|---|---|
| 31 | FAB-OPS | 2001:DB8:30:31::/64 | ::2 | ::3 | FE80::1 |
| 32 | QUALITY | 2001:DB8:30:32::/64 | ::2 | ::3 | FE80::1 |
| 33 | MAINTENANCE | 2001:DB8:30:33::/64 | ::2 | ::3 | FE80::1 |
| 34 | SAFETY | 2001:DB8:30:34::/64 | ::2 | ::3 | FE80::1 |
| 35 | PLANNING | 2001:DB8:30:35::/64 | ::2 | ::3 | FE80::1 |
| 36 | TRAINING | 2001:DB8:30:36::/64 | ::2 | ::3 | FE80::1 |
| 99 | MGMT | 2001:DB8:30:99::/64 | ::2 | ::3 | FE80::1 |
| 100 | SERVERS | 2001:DB8:30:100::/64 | ::2 | ::3 | FE80::1 |
| 199 | LEGACY-IPV4 | **10.30.199.0/24** | .1 | — | 10.30.199.1 |

### 1.4 Link-local addresses — `FE80::`
Every IPv6 interface automatically has a **link-local** address starting with `FE80`. It works
only on its own link (never routed). IPv6 uses them for neighbour discovery and as the **default
gateway** address. We set the virtual gateway to the easy `FE80::1` in every VLAN.

### 1.5 How a PC gets an IPv6 address without typing — SLAAC
1. The router sends **Router Advertisements (RA)**: "the network here is 2001:DB8:30:31::/64, and I'm
   your gateway".
2. The PC builds its own address: the prefix + a host part it makes itself.
This is **SLAAC** (Stateless Address Auto-Configuration). No DHCP server needed for the address.

But SLAAC doesn't tell the PC the **DNS** server. So we add **stateless DHCPv6**: the RA carries a
flag ("O-flag" = *other information*) telling PCs: "ask DHCPv6 for the extras (DNS, domain)".
→ Kiryat Gat's DHCP choice (req. 22.11, "third branch: student's choice") = **SLAAC + stateless
DHCPv6 on the distribution switches**.

### 1.6 OSPFv3 and HSRP for IPv6
- **OSPFv3** = OSPF for IPv6. Same ideas (neighbours, map, areas); different config: it's switched on
  **per interface** (`ipv6 ospf 1 area 0`), not with `network` lines. It needs a router-ID written
  like an IPv4 address (3.3.3.1) — there is no IPv4 address to borrow it from.
- **HSRP for IPv6** — version 2 only; the shared gateway is a **link-local** address (`FE80::1`).

### 1.7 Different on purpose
| | Haifa | Jerusalem | **Kiryat Gat** |
|---|---|---|---|
| VTP | server/client | server/client | **transparent** (each switch keeps its own VLANs) |
| EtherChannel | LACP | PAgP | **none** (single trunk between DISTs) |
| NAT | static (+PAT) | dynamic | **PAT (overload)** |

### 1.8 Why there is still a little IPv4
NAT exists only in IPv4. The ministry asks for one IPv6 branch **and** NAT in every branch. Our
answer: one **legacy VLAN 199** for old devices that only speak IPv4 (badge readers, cameras). Only
it uses PAT to reach the internet. Everything else is IPv6. **Confirm this interpretation with your
mentor.**

---

## Part 2 — Prepare
**File → Open** → `my-work/step13.pkt` → **File → Save As** → `my-work/step14.pkt`.
Draw a rectangle labelled `KIRYAT GAT BRANCH (IPv6)`.

---

## Part 3 — Build (click by click)

### 3.1 Devices
| Model | Display Name |
|---|---|
| 2911 | `R-KG-EDGE` |
| 3650-24PS (+ **AC-POWER-SUPPLY**!) | `SWL3-KG-DIST1`, `SWL3-KG-DIST2` |
| 2960 | `SW-KG-ACC1`, `SW-KG-ACC2`, `SW-KG-ACC3`, `SW-KG-SRV` |
| Server ×4 | `SRV-KG-DNS`, `SRV-KG-MAIL`, `SRV-KG-SYSLOG`, `SRV-KG-AAA` |
| PC ×12 | `PC-KG-FAB-01/02`, `PC-KG-QUA-01/02`, `PC-KG-MNT-01/02`, `PC-KG-SAF-01/02`, `PC-KG-PLN-01/02`, `PC-KG-TRN-01/02` |
| PC ×3 | `PC-KG-ADMIN-01`, `PC-KG-LEGACY-01`, `PC-KG-LEGACY-02` |
| Printer ×2 | `PRN-KG-ACC2-01`, `PRN-KG-ACC3-01` |
| Server ×1 (at ISP-C's home LAN) | `SRV-ISPC-WEB6` |

### 3.2 Cables
| Click first | Port | Click second | Port | Cable |
|---|---|---|---|---|
| SWL3-KG-DIST1 | Gi1/0/1 | SW-KG-ACC1 | Gi0/1 | Cross-Over |
| SWL3-KG-DIST2 | Gi1/0/1 | SW-KG-ACC1 | Gi0/2 | Cross-Over |
| SWL3-KG-DIST1 | Gi1/0/2 | SW-KG-ACC2 | Gi0/1 | Cross-Over |
| SWL3-KG-DIST2 | Gi1/0/2 | SW-KG-ACC2 | Gi0/2 | Cross-Over |
| SWL3-KG-DIST1 | Gi1/0/3 | SW-KG-ACC3 | Gi0/1 | Cross-Over |
| SWL3-KG-DIST2 | Gi1/0/3 | SW-KG-ACC3 | Gi0/2 | Cross-Over |
| SWL3-KG-DIST1 | Gi1/0/4 | SW-KG-SRV | Gi0/1 | Cross-Over |
| SWL3-KG-DIST2 | Gi1/0/4 | SW-KG-SRV | Gi0/2 | Cross-Over |
| SWL3-KG-DIST1 | Gi1/0/24 | SWL3-KG-DIST2 | Gi1/0/24 | Cross-Over |
| SWL3-KG-DIST1 | Gi1/1/1 | R-KG-EDGE | Gi0/0 | Straight-Through |
| SWL3-KG-DIST2 | Gi1/1/1 | R-KG-EDGE | Gi0/2 | Straight-Through |
| R-KG-EDGE | Gi0/1 | R-ISPC-PE | Gi0/1 | Cross-Over |
| SRV-ISPC-WEB6 | Fa0 | SW-ISPC-HOME | Fa0/2 | Straight-Through |

End devices (Straight-Through):

| Device | Switch | Port |
|---|---|---|
| PC-KG-FAB-01 / 02 | SW-KG-ACC1 | Fa0/1 / Fa0/2 |
| PC-KG-QUA-01 / 02 | SW-KG-ACC1 | Fa0/11 / Fa0/12 |
| PC-KG-LEGACY-01 / 02 | SW-KG-ACC1 | Fa0/21 / Fa0/22 |
| PC-KG-ADMIN-01 | SW-KG-ACC1 | Fa0/23 |
| PC-KG-MNT-01 / 02 | SW-KG-ACC2 | Fa0/1 / Fa0/2 |
| PC-KG-SAF-01 / 02 | SW-KG-ACC2 | Fa0/11 / Fa0/12 |
| PRN-KG-ACC2-01 | SW-KG-ACC2 | Fa0/21 |
| PC-KG-PLN-01 / 02 | SW-KG-ACC3 | Fa0/1 / Fa0/2 |
| PC-KG-TRN-01 / 02 | SW-KG-ACC3 | Fa0/11 / Fa0/12 |
| PRN-KG-ACC3-01 | SW-KG-ACC3 | Fa0/21 |
| SRV-KG-DNS / MAIL / SYSLOG / AAA | SW-KG-SRV | Fa0/1 / 2 / 3 / 4 |

### 3.3 Link addresses
| Link | IPv6 | IPv4 |
|---|---|---|
| R-KG-EDGE Gi0/0 ↔ DIST1 Gi1/1/1 | 2001:DB8:30:FF01::1 / ::2 (/64) | 10.30.4.1 / 10.30.4.2 (/30) |
| R-KG-EDGE Gi0/2 ↔ DIST2 Gi1/1/1 | 2001:DB8:30:FF02::1 / ::2 (/64) | — |
| R-KG-EDGE Gi0/1 ↔ R-ISPC-PE Gi0/1 | 2001:DB8:C:1::2 / ::1 (/64) | 200.3.1.2 / 200.3.1.1 (/30) |

| Device | OSPFv3 router-ID | Mgmt IPv6 |
|---|---|---|
| R-KG-EDGE | 3.3.3.1 | 2001:DB8:30:FF01::1 |
| SWL3-KG-DIST1 / DIST2 | 3.3.3.2 / 3.3.3.3 | 2001:DB8:30:99::2 / ::3 |
| SW-KG-ACC1 / ACC2 / ACC3 / SRV | — | 2001:DB8:30:99::11 / ::12 / ::13 / ::14 |
| PC-KG-ADMIN-01 | — | 2001:DB8:30:99::100 |

---

## Part 4 — Configure

### 4.0 Before you start — IPv6 on the switches
Try `configure terminal` → `ipv6 unicast-routing` on a 3650, and `interface vlan 1` →
`ipv6 address 2001:DB8::1/64` on a 2960. If a command is **refused**, enable IPv6 in the switch's
memory template (once per switch), then reboot it:
```
configure terminal
sdm prefer dual-ipv4-and-ipv6 default
end
copy running-config startup-config
reload
```
(Press Enter to confirm the reload. If you tested with `interface vlan 1`, remove the test address
afterwards: `no ipv6 address 2001:DB8::1/64`.)

### 4.1 SWL3-KG-DIST1 — complete
```
enable
configure terminal
! ===== hardening =====
hostname SWL3-KG-DIST1
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
! ===== routing for both IP versions =====
ip routing
ipv6 unicast-routing
! ===== VTP transparent: VLANs are created here, locally =====
vtp mode transparent
vlan 31
 name FAB-OPS
vlan 32
 name QUALITY
vlan 33
 name MAINTENANCE
vlan 34
 name SAFETY
vlan 35
 name PLANNING
vlan 36
 name TRAINING
vlan 99
 name MGMT
vlan 100
 name SERVERS
vlan 199
 name LEGACY-IPV4
vlan 998
 name NATIVE
vlan 999
 name PARKING
 exit
spanning-tree mode rapid-pvst
spanning-tree vlan 31-36,99,100,199 root primary
! ===== trunks =====
interface range gi1/0/1 - 4
 description TRUNK to access switch
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 31-36,99,100,199,998
 switchport nonegotiate
 spanning-tree guard root
 no shutdown
interface gi1/0/24
 description TRUNK to SWL3-KG-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 31-36,99,100,199,998
 switchport nonegotiate
 no shutdown
interface range gi1/0/5 - 23
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
! ===== stateless DHCPv6: only DNS + domain =====
ipv6 dhcp pool KG-STATELESS
 dns-server 2001:DB8:30:100::10
 domain-name intel.lab
 exit
! ===== OSPFv3 process =====
ipv6 router ospf 1
 router-id 3.3.3.2
 exit
! ===== uplink to the edge (dual-stack) =====
interface gi1/1/1
 description to R-KG-EDGE Gi0/0
 no switchport
 ip address 10.30.4.2 255.255.255.252
 ipv6 address 2001:DB8:30:FF01::2/64
 ipv6 ospf 1 area 0
 no shutdown
! ===== user VLAN gateways: IPv6 + RA flags + DHCPv6 + HSRPv6 =====
interface vlan 31
 description GW FAB-OPS
 ipv6 address 2001:DB8:30:31::2/64
 ipv6 ospf 1 area 0
 ipv6 nd other-config-flag
 ipv6 dhcp server KG-STATELESS
 standby version 2
 standby 31 ipv6 FE80::1
 standby 31 priority 110
 standby 31 preempt
 no shutdown
interface vlan 32
 description GW QUALITY
 ipv6 address 2001:DB8:30:32::2/64
 ipv6 ospf 1 area 0
 ipv6 nd other-config-flag
 ipv6 dhcp server KG-STATELESS
 standby version 2
 standby 32 ipv6 FE80::1
 standby 32 priority 110
 standby 32 preempt
 no shutdown
interface vlan 33
 description GW MAINTENANCE
 ipv6 address 2001:DB8:30:33::2/64
 ipv6 ospf 1 area 0
 ipv6 nd other-config-flag
 ipv6 dhcp server KG-STATELESS
 standby version 2
 standby 33 ipv6 FE80::1
 standby 33 priority 110
 standby 33 preempt
 no shutdown
interface vlan 34
 description GW SAFETY
 ipv6 address 2001:DB8:30:34::2/64
 ipv6 ospf 1 area 0
 ipv6 nd other-config-flag
 ipv6 dhcp server KG-STATELESS
 standby version 2
 standby 34 ipv6 FE80::1
 standby 34 priority 110
 standby 34 preempt
 no shutdown
interface vlan 35
 description GW PLANNING
 ipv6 address 2001:DB8:30:35::2/64
 ipv6 ospf 1 area 0
 ipv6 nd other-config-flag
 ipv6 dhcp server KG-STATELESS
 standby version 2
 standby 35 ipv6 FE80::1
 standby 35 priority 110
 standby 35 preempt
 no shutdown
interface vlan 36
 description GW TRAINING
 ipv6 address 2001:DB8:30:36::2/64
 ipv6 ospf 1 area 0
 ipv6 nd other-config-flag
 ipv6 dhcp server KG-STATELESS
 standby version 2
 standby 36 ipv6 FE80::1
 standby 36 priority 110
 standby 36 preempt
 no shutdown
! ===== management and servers: static addresses, no DHCPv6 =====
interface vlan 99
 description GW MGMT
 ipv6 address 2001:DB8:30:99::2/64
 ipv6 ospf 1 area 0
 standby version 2
 standby 99 ipv6 FE80::1
 standby 99 priority 110
 standby 99 preempt
 no shutdown
interface vlan 100
 description GW SERVERS (+ OSPFv3 peering DIST1-DIST2)
 ipv6 address 2001:DB8:30:100::2/64
 ipv6 ospf 1 area 0
 standby version 2
 standby 100 ipv6 FE80::1
 standby 100 priority 110
 standby 100 preempt
 no shutdown
! ===== the legacy IPv4 VLAN (only on DIST1) =====
interface vlan 199
 description GW LEGACY-IPV4
 ip address 10.30.199.1 255.255.255.0
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 10.30.4.1
! ===== OSPFv3: talk only on the uplink and the server VLAN =====
ipv6 router ospf 1
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface vlan 100
 exit
! ===== management =====
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
(`1024`)
```
ip ssh version 2
login block-for 120 attempts 3 within 60
ipv6 access-list MGMT6
 permit ipv6 2001:DB8:30:99::/64 any
 deny ipv6 any any log
 exit
line vty 0 15
 transport input ssh
 login local
 ipv6 access-class MGMT6 in
 exec-timeout 5 0
line console 0
 login local
 logging synchronous
 exit
service timestamps log datetime msec
logging host ipv6 2001:DB8:30:100::12
! branch NTP: syncs to Haifa's public NTP (static NAT, step 12) over IPv4
ntp server 200.1.10.4
 end
copy running-config startup-config
```

### 4.2 SWL3-KG-DIST2 — complete
```
enable
configure terminal
hostname SWL3-KG-DIST2
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
ipv6 unicast-routing
vtp mode transparent
vlan 31
 name FAB-OPS
vlan 32
 name QUALITY
vlan 33
 name MAINTENANCE
vlan 34
 name SAFETY
vlan 35
 name PLANNING
vlan 36
 name TRAINING
vlan 99
 name MGMT
vlan 100
 name SERVERS
vlan 199
 name LEGACY-IPV4
vlan 998
 name NATIVE
vlan 999
 name PARKING
 exit
spanning-tree mode rapid-pvst
spanning-tree vlan 31-36,99,100,199 root secondary
interface range gi1/0/1 - 4
 description TRUNK to access switch
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 31-36,99,100,199,998
 switchport nonegotiate
 spanning-tree guard root
 no shutdown
interface gi1/0/24
 description TRUNK to SWL3-KG-DIST1
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 31-36,99,100,199,998
 switchport nonegotiate
 no shutdown
interface range gi1/0/5 - 23
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 exit
ipv6 dhcp pool KG-STATELESS
 dns-server 2001:DB8:30:100::10
 domain-name intel.lab
 exit
ipv6 router ospf 1
 router-id 3.3.3.3
 exit
interface gi1/1/1
 description to R-KG-EDGE Gi0/2
 no switchport
 ipv6 address 2001:DB8:30:FF02::2/64
 ipv6 ospf 1 area 0
 no shutdown
interface vlan 31
 description GW FAB-OPS
 ipv6 address 2001:DB8:30:31::3/64
 ipv6 ospf 1 area 0
 ipv6 nd other-config-flag
 ipv6 dhcp server KG-STATELESS
 standby version 2
 standby 31 ipv6 FE80::1
 standby 31 priority 90
 standby 31 preempt
 no shutdown
interface vlan 32
 description GW QUALITY
 ipv6 address 2001:DB8:30:32::3/64
 ipv6 ospf 1 area 0
 ipv6 nd other-config-flag
 ipv6 dhcp server KG-STATELESS
 standby version 2
 standby 32 ipv6 FE80::1
 standby 32 priority 90
 standby 32 preempt
 no shutdown
interface vlan 33
 description GW MAINTENANCE
 ipv6 address 2001:DB8:30:33::3/64
 ipv6 ospf 1 area 0
 ipv6 nd other-config-flag
 ipv6 dhcp server KG-STATELESS
 standby version 2
 standby 33 ipv6 FE80::1
 standby 33 priority 90
 standby 33 preempt
 no shutdown
interface vlan 34
 description GW SAFETY
 ipv6 address 2001:DB8:30:34::3/64
 ipv6 ospf 1 area 0
 ipv6 nd other-config-flag
 ipv6 dhcp server KG-STATELESS
 standby version 2
 standby 34 ipv6 FE80::1
 standby 34 priority 90
 standby 34 preempt
 no shutdown
interface vlan 35
 description GW PLANNING
 ipv6 address 2001:DB8:30:35::3/64
 ipv6 ospf 1 area 0
 ipv6 nd other-config-flag
 ipv6 dhcp server KG-STATELESS
 standby version 2
 standby 35 ipv6 FE80::1
 standby 35 priority 90
 standby 35 preempt
 no shutdown
interface vlan 36
 description GW TRAINING
 ipv6 address 2001:DB8:30:36::3/64
 ipv6 ospf 1 area 0
 ipv6 nd other-config-flag
 ipv6 dhcp server KG-STATELESS
 standby version 2
 standby 36 ipv6 FE80::1
 standby 36 priority 90
 standby 36 preempt
 no shutdown
interface vlan 99
 description GW MGMT
 ipv6 address 2001:DB8:30:99::3/64
 ipv6 ospf 1 area 0
 standby version 2
 standby 99 ipv6 FE80::1
 standby 99 priority 90
 standby 99 preempt
 no shutdown
interface vlan 100
 description GW SERVERS (+ OSPFv3 peering DIST1-DIST2)
 ipv6 address 2001:DB8:30:100::3/64
 ipv6 ospf 1 area 0
 standby version 2
 standby 100 ipv6 FE80::1
 standby 100 priority 90
 standby 100 preempt
 no shutdown
 exit
ipv6 router ospf 1
 passive-interface default
 no passive-interface gi1/1/1
 no passive-interface vlan 100
 exit
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
(`1024`)
```
ip ssh version 2
login block-for 120 attempts 3 within 60
ipv6 access-list MGMT6
 permit ipv6 2001:DB8:30:99::/64 any
 deny ipv6 any any log
 exit
line vty 0 15
 transport input ssh
 login local
 ipv6 access-class MGMT6 in
 exec-timeout 5 0
line console 0
 login local
 logging synchronous
 exit
service timestamps log datetime msec
logging host ipv6 2001:DB8:30:100::12
 end
copy running-config startup-config
```

### 4.3 SW-KG-ACC1 — complete (Fab-Ops, Quality, legacy, admin PC)
```
enable
configure terminal
hostname SW-KG-ACC1
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
vlan 31
 name FAB-OPS
vlan 32
 name QUALITY
vlan 99
 name MGMT
vlan 199
 name LEGACY-IPV4
vlan 998
 name NATIVE
vlan 999
 name PARKING
 exit
spanning-tree mode rapid-pvst
interface range gi0/1 - 2
 description TRUNK uplink to DIST1 / DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 31-36,99,100,199,998
 switchport nonegotiate
 no shutdown
interface range fa0/1 - 10
 description FAB-OPS users
 switchport mode access
 switchport access vlan 31
interface range fa0/11 - 20
 description QUALITY users
 switchport mode access
 switchport access vlan 32
interface range fa0/21 - 22
 description LEGACY IPv4 devices
 switchport mode access
 switchport access vlan 199
interface fa0/23
 description PC-KG-ADMIN-01 (management)
 switchport mode access
 switchport access vlan 99
interface range fa0/1 - 23
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no cdp enable
interface fa0/24
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
interface vlan 99
 description MANAGEMENT
 ipv6 address 2001:DB8:30:99::11/64
 no shutdown
 exit
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
(`1024`)
```
ip ssh version 2
login block-for 120 attempts 3 within 60
ipv6 access-list MGMT6
 permit ipv6 2001:DB8:30:99::/64 any
 deny ipv6 any any log
 exit
line vty 0 15
 transport input ssh
 login local
 ipv6 access-class MGMT6 in
 exec-timeout 5 0
line console 0
 login local
 logging synchronous
 exit
service timestamps log datetime msec
logging host ipv6 2001:DB8:30:100::12
 end
copy running-config startup-config
```
> An L2 switch learns its IPv6 gateway from the Router Advertisements — no `default-gateway` line needed.
> DHCP snooping is IPv4-only, so here the access ports are protected by port-security, BPDU Guard and
> the parking VLAN. (Real switches add **RA Guard** and **DHCPv6 Guard** — not in PT. Mention it in the book.)

### 4.4 SW-KG-ACC2 — complete (Maintenance, Safety, printer)
```
enable
configure terminal
hostname SW-KG-ACC2
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
vlan 33
 name MAINTENANCE
vlan 34
 name SAFETY
vlan 99
 name MGMT
vlan 998
 name NATIVE
vlan 999
 name PARKING
 exit
spanning-tree mode rapid-pvst
interface range gi0/1 - 2
 description TRUNK uplink to DIST1 / DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 31-36,99,100,199,998
 switchport nonegotiate
 no shutdown
interface range fa0/1 - 10
 description MAINTENANCE users
 switchport mode access
 switchport access vlan 33
interface range fa0/11 - 20
 description SAFETY users
 switchport mode access
 switchport access vlan 34
interface fa0/21
 description PRN-KG-ACC2-01
 switchport mode access
 switchport access vlan 33
interface range fa0/1 - 21
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 no cdp enable
interface range fa0/22 - 24
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
interface vlan 99
 description MANAGEMENT
 ipv6 address 2001:DB8:30:99::12/64
 no shutdown
 exit
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
(`1024`)
```
ip ssh version 2
login block-for 120 attempts 3 within 60
ipv6 access-list MGMT6
 permit ipv6 2001:DB8:30:99::/64 any
 deny ipv6 any any log
 exit
line vty 0 15
 transport input ssh
 login local
 ipv6 access-class MGMT6 in
 exec-timeout 5 0
line console 0
 login local
 logging synchronous
 exit
service timestamps log datetime msec
logging host ipv6 2001:DB8:30:100::12
 end
copy running-config startup-config
```

### 4.5 SW-KG-ACC3 — complete (Planning, Training, printer)
Type the whole block of 4.4 again with these lines different:

| Line in 4.4 | Becomes |
|---|---|
| `hostname SW-KG-ACC2` | `hostname SW-KG-ACC3` |
| `vlan 33` / `name MAINTENANCE` | `vlan 35` / `name PLANNING` |
| `vlan 34` / `name SAFETY` | `vlan 36` / `name TRAINING` |
| `description MAINTENANCE users` / `switchport access vlan 33` | `description PLANNING users` / `switchport access vlan 35` |
| `description SAFETY users` / `switchport access vlan 34` | `description TRAINING users` / `switchport access vlan 36` |
| `description PRN-KG-ACC2-01` / `switchport access vlan 33` | `description PRN-KG-ACC3-01` / `switchport access vlan 35` |
| `ipv6 address 2001:DB8:30:99::12/64` | `ipv6 address 2001:DB8:30:99::13/64` |

### 4.6 SW-KG-SRV — complete
```
enable
configure terminal
hostname SW-KG-SRV
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
vtp mode transparent
vlan 99
 name MGMT
vlan 100
 name SERVERS
vlan 998
 name NATIVE
vlan 999
 name PARKING
 exit
spanning-tree mode rapid-pvst
interface range gi0/1 - 2
 description TRUNK uplink to DIST1 / DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 31-36,99,100,199,998
 switchport nonegotiate
 no shutdown
interface range fa0/1 - 4
 description servers
 switchport mode access
 switchport access vlan 100
 spanning-tree portfast
 spanning-tree bpduguard enable
interface range fa0/5 - 24
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
interface vlan 99
 description MANAGEMENT
 ipv6 address 2001:DB8:30:99::14/64
 no shutdown
 exit
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
(`1024`)
```
ip ssh version 2
login block-for 120 attempts 3 within 60
ipv6 access-list MGMT6
 permit ipv6 2001:DB8:30:99::/64 any
 deny ipv6 any any log
 exit
line vty 0 15
 transport input ssh
 login local
 ipv6 access-class MGMT6 in
 exec-timeout 5 0
line console 0
 login local
 exit
logging host ipv6 2001:DB8:30:100::12
 end
copy running-config startup-config
```

### 4.7 R-KG-EDGE — complete
```
enable
configure terminal
hostname R-KG-EDGE
no ip domain-lookup
enable secret Intel#En4ble
service password-encryption
banner motd #
*  Intel Israel - AUTHORIZED ACCESS ONLY - all activity is logged  *
#
ipv6 unicast-routing
ipv6 router ospf 1
 router-id 3.3.3.1
 exit
interface gi0/0
 description to SWL3-KG-DIST1
 ip address 10.30.4.1 255.255.255.252
 ip nat inside
 ipv6 address 2001:DB8:30:FF01::1/64
 ipv6 ospf 1 area 0
 no shutdown
interface gi0/2
 description to SWL3-KG-DIST2
 ipv6 address 2001:DB8:30:FF02::1/64
 ipv6 ospf 1 area 0
 no shutdown
interface gi0/1
 description WAN to R-ISPC-PE (dual-stack)
 ip address 200.3.1.2 255.255.255.252
 ip nat outside
 ipv6 address 2001:DB8:C:1::2/64
 no shutdown
 exit
! ===== static toward the WAN, both IP versions =====
ipv6 route ::/0 2001:DB8:C:1::1
ip route 0.0.0.0 0.0.0.0 200.3.1.1
ip route 10.30.199.0 255.255.255.0 10.30.4.2
! ===== tell the DIST switches about the IPv6 default route =====
ipv6 router ospf 1
 default-information originate
 exit
! ===== PAT (overload) for the legacy VLAN + DIST1's NTP =====
access-list 1 permit 10.30.199.0 0.0.0.255
access-list 1 permit 10.30.4.0 0.0.0.3
ip nat inside source list 1 interface gi0/1 overload
! ===== inbound IPv6 filter =====
ipv6 access-list WAN6-IN
 remark --- ICMPv6 is required by IPv6 itself (neighbour discovery) ---
 permit icmp any any
 remark --- replies from web and DNS servers ---
 permit tcp any eq www any
 permit udp any eq domain any
 deny ipv6 any any log
 exit
interface gi0/1
 ipv6 traffic-filter WAN6-IN in
 exit
! ===== management =====
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
(`1024`)
```
ip ssh version 2
login block-for 120 attempts 3 within 60
ipv6 access-list MGMT6
 permit ipv6 2001:DB8:30:99::/64 any
 deny ipv6 any any log
 exit
line vty 0 15
 transport input ssh
 login local
 ipv6 access-class MGMT6 in
 exec-timeout 5 0
line console 0
 login local
 exit
service timestamps log datetime msec
logging host ipv6 2001:DB8:30:100::12
 end
copy running-config startup-config
```

### 4.8 R-ISPC-PE — route Kiryat Gat's IPv6 block
```
enable
configure terminal
ipv6 route 2001:DB8:30::/48 2001:DB8:C:1::2
end
copy running-config startup-config
```

### 4.9 Servers and end devices (click by click)
**IPv6 settings on a server or PC:** Desktop → **IP Configuration** → section **IPv6 Configuration**.

| Device | IPv6 setting | Address | Gateway | IPv6 DNS |
|---|---|---|---|---|
| SRV-KG-DNS | Static | 2001:DB8:30:100::10 / 64 | FE80::1 | 2001:DB8:30:100::10 |
| SRV-KG-MAIL | Static | 2001:DB8:30:100::11 / 64 | FE80::1 | 2001:DB8:30:100::10 |
| SRV-KG-SYSLOG | Static | 2001:DB8:30:100::12 / 64 | FE80::1 | 2001:DB8:30:100::10 |
| SRV-KG-AAA | Static | 2001:DB8:30:100::13 / 64 | FE80::1 | 2001:DB8:30:100::10 |
| PC-KG-ADMIN-01 | Static | 2001:DB8:30:99::100 / 64 | FE80::1 | 2001:DB8:30:100::10 |
| All 12 department PCs | **Automatic** | (SLAAC fills it in) | | |
| SRV-ISPC-WEB6 | Static | 2001:DB8:C:50::80 / 64 | FE80::1 | — |

For the IPv6-only devices, leave the **IPv4** fields empty.

**Legacy PCs (IPv4 only):** PC-KG-LEGACY-01 `10.30.199.10`, PC-KG-LEGACY-02 `10.30.199.11`,
mask `255.255.255.0`, gateway `10.30.199.1`, DNS `200.1.100.10`.

**Printers:** Config → FastEthernet0 → IPv6 → **Automatic** (SLAAC).

**Services:**
- **SRV-KG-DNS** → DNS **On** → records of type **AAAA Record**:
  `mail.kg.intel.lab → 2001:DB8:30:100::11`, `www6.isp-web.com → 2001:DB8:C:50::80`.
- **SRV-KG-MAIL** → EMAIL → domain `kg.intel.lab` → Set (users in step 16).
- **SRV-KG-SYSLOG** → SYSLOG **On**.
- **SRV-KG-AAA** → AAA **On** (see the note below).
- **SRV-ISPC-WEB6** → HTTP → edit `index.html` → `<h1>IPv6 works!</h1>` → Save.
- Turn off unneeded services on each server.

> **AAA over IPv6 in Packet Tracer.** PT's support for TACACS+/RADIUS over IPv6 is limited, so the
> Kiryat Gat devices above use the **local** user database (`login local`). The AAA server exists in
> the branch (req. 20.10). If your PT version accepts it, try on a switch:
> `aaa new-model` → `tacacs server KG-AAA` → `address ipv6 2001:DB8:30:100::13` → `key Intel#Tac`
> → `exit` → `aaa authentication login default group tacacs+ local`, and register the switch in
> SRV-KG-AAA by its IPv6 address. Write in the book which way worked.
>
> Same for `logging host ipv6 ...`: if refused, try `logging 2001:DB8:30:100::12`.

### 4.10 What the new commands mean
| Command | In simple English |
|---|---|
| `ipv6 unicast-routing` | "Route IPv6 and send Router Advertisements" |
| `ipv6 address 2001:DB8:30:31::2/64` | This interface's IPv6 address |
| `ipv6 ospf 1 area 0` | "This interface joins OSPFv3 process 1, area 0" |
| `ipv6 router ospf 1` / `router-id 3.3.3.2` | Start OSPFv3, with this ID |
| `ipv6 nd other-config-flag` | Set the **O-flag** in RAs: "ask DHCPv6 for DNS" |
| `ipv6 dhcp pool KG-STATELESS` + `dns-server` | The extras handed out by stateless DHCPv6 |
| `ipv6 dhcp server KG-STATELESS` | "Answer DHCPv6 questions on this VLAN" |
| `standby 31 ipv6 FE80::1` | HSRP for IPv6: shared gateway FE80::1 |
| `ipv6 route ::/0 2001:DB8:C:1::1` | IPv6 default route (`::/0` = everything) |
| `ip nat inside source list 1 interface gi0/1 overload` | **PAT** — all legacy devices share 200.3.1.2 |
| `ipv6 traffic-filter WAN6-IN in` | Apply an IPv6 ACL on an interface |
| `ipv6 access-class MGMT6 in` | The IPv6 guest list for the VTY lines |

---

## Part 5 — Check your work

```
SWL3-KG-DIST1# show ipv6 interface brief         <- every VLAN up with its 2001:DB8:30:x::2
SWL3-KG-DIST1# show standby brief
Vl31   31  110 P Active   local   FE80::...   FE80::1
SWL3-KG-DIST1# show ipv6 ospf neighbor           <- 3.3.3.1 (edge) and 3.3.3.3 (DIST2) FULL
SWL3-KG-DIST2# show ipv6 route
OE2 ::/0 [110/1] via FE80::..., GigabitEthernet1/1/1   <- default from the edge
R-KG-EDGE#     show ip nat translations          <- 200.3.1.2:port  10.30.199.10:port
```

| From | Do | Expected |
|---|---|---|
| PC-KG-FAB-01 | Command Prompt → `ipconfig /all` | IPv6 `2001:DB8:30:31:…`, gateway `FE80::1`, DNS `2001:DB8:30:100::10` |
| PC-KG-FAB-01 | `ping 2001:DB8:30:36::2` | ✅ inter-VLAN routing over IPv6 |
| PC-KG-FAB-01 | `ping 2001:DB8:30:100::10` | ✅ the DNS server |
| PC-KG-FAB-01 | Web Browser → `http://www6.isp-web.com` | ✅ "IPv6 works!" |
| PC-KG-LEGACY-01 | `ping 200.1.100.11` | ✅ through PAT |
| PC-KG-ADMIN-01 | `ssh -l admin 2001:DB8:30:99::11` | ✅ |
| PC-KG-FAB-01 | `ssh -l admin 2001:DB8:30:99::11` | ❌ refused (MGMT6) |
| SWL3-KG-DIST1 | `show ntp status` | synchronized to 200.1.10.4 (after Fast Forward) |

**HSRPv6 failover:** `ping -t 2001:DB8:30:100::10` from a PC, power off DIST1 → DIST2 becomes
Active for `FE80::1`, pings continue.

---

## Part 6 — Save
`copy running-config startup-config` everywhere → **File → Save**.

## Checkpoint
- [ ] PCs get IPv6 addresses automatically, with gateway FE80::1 and a DNS server
- [ ] OSPFv3 neighbours FULL; DIST switches have the `::/0` default route
- [ ] IPv6 browsing to www6.isp-web.com works
- [ ] Legacy PCs reach the internet via PAT
- [ ] HSRPv6 failover works

## If something doesn't work
| Problem | Fix |
|---|---|
| `ipv6 ...` commands refused on a switch | Part 4.0 — `sdm prefer dual-ipv4-and-ipv6 default` + reload |
| OSPFv3 doesn't start | No `router-id` under `ipv6 router ospf 1` |
| PCs get only an `FE80::` address | `ipv6 unicast-routing` missing → no RAs; or the VLAN SVI is down |
| PCs have no DNS | `ipv6 nd other-config-flag` / `ipv6 dhcp server` missing; or type the DNS by hand |
| `standby 31 ipv6 FE80::1` refused | Use `standby 31 ipv6 autoconfig`, read the virtual address with `show standby brief`, and use it as the servers' gateway |
| HSRP for IPv6 not supported at all | IPv6 has its own gateway redundancy: both DISTs send RAs and PCs learn two gateways. Write this in the book |
| Everything IPv6 dies after the edge ACL | The `permit icmp any any` line is missing (IPv6 needs ICMPv6) |
| Legacy PC has no internet | `ip route 10.30.199.0 ...` on the edge, `ip route 0.0.0.0 ... 10.30.4.1` on DIST1, NAT inside/outside |

## Challenge
Why can't a Kiryat Gat PC open `http://www.intel.lab` (Haifa's website), even though it has
internet? Suggest two real-world solutions.

<details><summary>Answer</summary>

`www.intel.lab` has only an **IPv4** address (200.1.10.3 public / 10.10.200.13 private) and the
Kiryat Gat PCs speak **only IPv6**. An IPv6-only host can't talk to an IPv4-only server without a
translator. Real solutions: (1) **NAT64 + DNS64** on the Kiryat Gat edge; (2) make Haifa's DMZ
**dual-stack** (an IPv6 address + AAAA record for the web server); (3) a reverse proxy. PT doesn't
support NAT64 — document it as a design trade-off caused by the "one branch entirely IPv6" requirement.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

This is the example's **IPv6 branch**. Build **your** third site from plan parts C–E:

| In the example | In your project — take it from |
|---|---|
| `2001:DB8:30::/48`, rule "VLAN number = 4th group" | plan part C (your /48 — keep a simple rule like this one) |
| VLANs 31–36, 99, 100 | plan part D (site 3) |
| OSPFv3 router-IDs 3.3.3.x | your own router-IDs |
| HSRP for IPv6, virtual gateway `FE80::1` | keep the method, your groups |
| SLAAC / stateless DHCPv6 | plan part E (DHCP row) |
| Legacy IPv4 VLAN 199 with PAT | plan part E (NAT row) — check this interpretation with your mentor, as the design note in design.md says |

The ministry requires one site that is **entirely IPv6** (req. 7).

**Check:** `show ipv6 ospf neighbor`; PCs get an IPv6 address from your prefix; ping from this
site to an IPv6 internet server works.

💾 Save as `my-project/step14.pkt`.

---

## 👩‍🏫 Teacher notes
- **Lesson 1:** IPv6 — why, how to read and shorten addresses (practise 10 on the board), /64, link-local,
  our VLAN-number plan → build and cable.
- **Lesson 2:** SLAAC + stateless DHCPv6 (show the RA and DHCPv6 in Simulation mode, filters **ICMPv6** and
  **DHCPv6**), OSPFv3, HSRPv6 → DIST1 and DIST2 (pairs: one types DIST1, the other DIST2).
- **Lesson 3:** access switches, server switch, edge router, servers.
- **Lesson 4:** tests, failover, the legacy VLAN and PAT, the challenge discussion (IPv4/IPv6 coexistence).
- **Common student questions**
  - *"Where is the subnet mask?"* — The `/64` is the mask. IPv6 always writes the prefix length.
  - *"Why FE80::1 as gateway and not 2001:…::1?"* — Routers advertise themselves with link-local
    addresses; it never changes even if the global prefix changes.
  - *"No NAT at all?"* — Not needed: every IPv6 address is unique. Security comes from firewalls/ACLs.
- **Typical errors:** forgetting the SDM template + reload; typos in long IPv6 addresses (zeros); OSPFv3
  without router-ID; forgetting `permit icmp` in IPv6 ACLs.
