# Step 14 — Kiryat Gat: the IPv6 branch

## Goal
The third branch runs **IPv6** for all users, servers and management: SLAAC + stateless
DHCPv6, **OSPFv3**, **HSRP for IPv6**, IPv6 SSH. A single legacy IPv4 VLAN reaches the
internet through **PAT (overload)** — the third NAT method.

Ministry requirements: **7, 10 (overload), 17, 20.x, 20.7 (student's choice)**

## Concepts
- **IPv6 address** = 128 bits, written in 8 hextets. `2001:DB8:30:31::2/64` =
  `2001:0DB8:0030:0031:0000:0000:0000:0002`. Leading zeros drop; one run of zeros → `::`.
- **Link-local (FE80::/10)** — every IPv6 interface has one automatically. Routing protocols
  and **default gateways** use link-local addresses.
- **SLAAC** — the router sends **Router Advertisements (RA)** with the /64 prefix; hosts
  build their own address from it. No DHCP needed for addressing.
- **Stateless DHCPv6** — SLAAC for the address, DHCPv6 only for extras (DNS, domain).
  The RA's **O-flag** (`ipv6 nd other-config-flag`) tells hosts "ask DHCPv6 for other info".
  → Kiryat Gat's DHCP choice (req. 22.11 "third branch: student's choice") = **SLAAC +
  stateless DHCPv6 on the L3 switches**.
- **OSPFv3** — OSPF for IPv6. Enabled **per interface** (`ipv6 ospf 1 area 0`), not with
  `network` statements. Needs a 32-bit router-ID set by hand (there is no IPv4 to borrow).
- **HSRP for IPv6** — version 2 only; the virtual gateway is a **link-local** address. We set
  it to the readable `FE80::1` in every VLAN.
- **VTP transparent** — this branch manages VLANs per switch (a third approach, so no
  branch copies another).
- **No NAT for IPv6** — every IPv6 address is globally unique. The legacy IPv4 VLAN 199 is
  the only thing that needs NAT → **PAT** on the edge.

## Build

| Device | Name |
|---|---|
| 2911 | `R-KG-EDGE` |
| 3650-24PS ×2 (PSU!) | `SWL3-KG-DIST1`, `SWL3-KG-DIST2` |
| 2960 ×4 | `SW-KG-ACC1`, `SW-KG-ACC2`, `SW-KG-ACC3`, `SW-KG-SRV` |
| Server-PT ×4 | `SRV-KG-DNS` ::10, `SRV-KG-MAIL` ::11, `SRV-KG-SYSLOG` ::12, `SRV-KG-AAA` ::13 (all `2001:DB8:30:100::x/64`, gateway `FE80::1`) |
| PCs | 2 per department (`PC-KG-FAB-01/02`, `-QUA-`, `-MNT-`, `-SAF-`, `-PLN-`, `-TRN-`), `PC-KG-ADMIN-01` (VLAN 99, `2001:DB8:30:99::100/64`, gw `FE80::1`) |
| Legacy | `PC-KG-LEGACY-01/02` (badge terminals) 10.30.199.10/.11, gw 10.30.199.1, DNS 200.1.100.10 |
| Printers | `PRN-KG-ACC2-01`, `PRN-KG-ACC3-01` (IPv6 auto) |

| From | To |
|---|---|
| DIST1 Gi1/0/1-4 | ACC1, ACC2, ACC3, SRV — Gi0/1 |
| DIST2 Gi1/0/1-4 | ACC1, ACC2, ACC3, SRV — Gi0/2 |
| DIST1 Gi1/0/24 | DIST2 Gi1/0/24 (trunk) |
| DIST1 Gi1/1/1 | R-KG-EDGE Gi0/0 |
| DIST2 Gi1/1/1 | R-KG-EDGE Gi0/2 |
| R-KG-EDGE Gi0/1 | R-ISPC-PE Gi0/1 |

| Access switch | Fa0/1-10 | Fa0/11-20 | Fa0/21 | Fa0/22 | Fa0/23 |
|---|---|---|---|---|---|
| ACC1 | 31 FAB-OPS | 32 QUALITY | 199 legacy | 199 legacy | 99 admin PC |
| ACC2 | 33 MAINTENANCE | 34 SAFETY | printer (VLAN 33) | parking | parking |
| ACC3 | 35 PLANNING | 36 TRAINING | printer (VLAN 35) | parking | parking |

## Configure

> If a 3650 or 2960 rejects `ipv6 ...` commands: `sdm prefer dual-ipv4-and-ipv6 default`
> → `wr` → `reload`. See [pt-gotchas](../../docs/pt-gotchas.md).

### SWL3-KG-DIST1
```
configure terminal
hostname SWL3-KG-DIST1
! (+ hardening)
ip routing
ipv6 unicast-routing
spanning-tree mode rapid-pvst
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
spanning-tree vlan 31-36,99,100,199 root primary

interface range gi1/0/1 - 4
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 31-36,99,100,199,998
 switchport nonegotiate
 spanning-tree guard root
 no shutdown
interface gi1/0/24
 description trunk to SWL3-KG-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 31-36,99,100,199,998
 switchport nonegotiate
 no shutdown
interface range gi1/0/5 - 23
 switchport mode access
 switchport access vlan 999
 shutdown

! ---------- stateless DHCPv6: DNS + domain only ----------
ipv6 dhcp pool KG-STATELESS
 dns-server 2001:DB8:30:100::10
 domain-name intel.lab
 exit

! ---------- uplink: dual-stack (IPv4 only for the legacy VLAN) ----------
interface gi1/1/1
 description to R-KG-EDGE Gi0/0
 no switchport
 ip address 10.30.4.2 255.255.255.252
 ipv6 address 2001:DB8:30:FF01::2/64
 ipv6 ospf 1 area 0
 no shutdown
```
SVI **pattern** — repeat for 31, 32, 33, 34, 35, 36, 99, 100 (use the VLAN number in the
prefix and the HSRP group):
```
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
```
(VLAN 99 and 100: skip the two DHCP lines — servers and admin PCs are static.)

Legacy IPv4 + OSPFv3 + NTP:
```
interface vlan 199
 description GW LEGACY-IPV4 (no IPv6 on purpose)
 ip address 10.30.199.1 255.255.255.0
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 10.30.4.1

ipv6 router ospf 1
 router-id 3.3.3.2
 passive-interface default
 no passive-interface gi1/1/1
 ! DIST1 <-> DIST2 adjacency over the server VLAN
 no passive-interface vlan 100
 exit

! branch NTP server: syncs to Haifa's public NTP (static NAT 200.1.10.4, step 12)
ntp server 200.1.10.4
end
wr
```

### SWL3-KG-DIST2
Same VLANs, trunks, DHCPv6 pool, `spanning-tree vlan 31-36,99,100,199 root secondary`.
SVIs with `::3` and `standby <vlan> priority 90`. Uplink Gi1/1/1:
`ipv6 address 2001:DB8:30:FF02::2/64`, `ipv6 ospf 1 area 0` — **no IPv4**.
`router-id 3.3.3.3`. No VLAN 199 SVI.

### R-KG-EDGE
```
configure terminal
hostname R-KG-EDGE
! (+ hardening)
ipv6 unicast-routing
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

! ---------- static toward the WAN, both families ----------
ipv6 route ::/0 2001:DB8:C:1::1
ip route 0.0.0.0 0.0.0.0 200.3.1.1
ip route 10.30.199.0 255.255.255.0 10.30.4.2

ipv6 router ospf 1
 router-id 3.3.3.1
 default-information originate
 exit

! ---------- PAT (overload) for the legacy VLAN + DIST1's NTP ----------
access-list 1 permit 10.30.199.0 0.0.0.255
access-list 1 permit 10.30.4.0 0.0.0.3
ip nat inside source list 1 interface gi0/1 overload

! ---------- inbound IPv6 filter ----------
ipv6 access-list WAN6-IN
 remark --- ICMPv6 is required (neighbor discovery, PMTU) ---
 permit icmp any any
 remark --- replies from web and DNS servers ---
 permit tcp any eq www any
 permit udp any eq domain any
 deny ipv6 any any log
 exit
interface gi0/1
 ipv6 traffic-filter WAN6-IN in
end
wr
```

### R-ISPC-PE — route the KG prefix + an IPv6 web server
```
ipv6 route 2001:DB8:30::/48 2001:DB8:C:1::2
interface gi0/2
 ipv6 address FE80::1 link-local
```
Add `SRV-ISPC-WEB6` on SW-ISPC-HOME Fa0/2: IPv6 `2001:DB8:C:50::80/64`, gw `FE80::1`,
HTTP on, page "IPv6 works!".

### Access switches (ACC1–3, SRV)
Your usual pattern with these differences:
- `vtp mode transparent` and create the VLANs **locally** on each switch.
- Trunk allowed list `31-36,99,100,199,998`.
- Management in IPv6:
  ```
  interface vlan 99
   ipv6 address 2001:DB8:30:99::11/64     ! ACC2 ::12, ACC3 ::13, SRV ::14
   no shutdown
  ipv6 access-list MGMT6
   permit ipv6 2001:DB8:30:99::/64 any
   deny ipv6 any any log
  line vty 0 15
   transport input ssh
   login local
   ipv6 access-class MGMT6 in
  ```
  (An L2 switch learns its IPv6 default router from the RA — no `default-gateway` needed.)
- DHCP snooping is IPv4-only — protect the IPv6 access ports with **port-security**,
  **BPDU Guard**, parking VLAN. Mention in the book that real switches add **RA Guard** and
  **DHCPv6 Guard**, which the simulator doesn't support.

### Servers
- **SRV-KG-DNS**: AAAA records — `mail.kg.intel.lab → 2001:DB8:30:100::11`,
  `www6.isp-web.com → 2001:DB8:C:50::80`.
- **SRV-KG-MAIL**: EMAIL on, domain `kg.intel.lab` (users in step 16).
- **SRV-KG-SYSLOG**: SYSLOG on. Devices: `logging host ipv6 2001:DB8:30:100::12`.
- **SRV-KG-AAA**: AAA on, register devices by their IPv6 management address.
  Devices: `aaa new-model` + `aaa authentication login default local` — if your PT build
  accepts TACACS+ over IPv6, add the server with
  `tacacs server KG-AAA` / `address ipv6 2001:DB8:30:100::13` / `key Intel#Tac` and put
  `group tacacs+ local` in the method list. If not, document that the branch uses the local
  database (simulator limitation).

### PCs
*Desktop → IP Configuration → IPv6 → **Automatic*** → they get `2001:DB8:30:3x::…` by SLAAC
and the gateway `FE80::1`. If your PT version doesn't fill in the DNS from DHCPv6, enter
`2001:DB8:30:100::10` manually.

## Verify

```
SWL3-KG-DIST1# show ipv6 interface brief
SWL3-KG-DIST1# show standby brief
Vl31  31 110 P Active  local  FE80::...  FE80::1
SWL3-KG-DIST1# show ipv6 ospf neighbor      <- 3.3.3.1 (edge) and 3.3.3.3 (DIST2)
SWL3-KG-DIST2# show ipv6 route
OE2 ::/0 [110/1] via FE80::..., Gi1/1/1     <- default from the edge
R-KG-EDGE#     show ip nat translations     <- 200.3.1.2:port  10.30.199.10:port
```

| From | Test | Expected |
|---|---|---|
| PC-KG-FAB-01 | `ipv6config` | `2001:DB8:30:31:…`, gw `FE80::1` |
| PC-KG-FAB-01 | `ping 2001:DB8:30:36::2` | ✅ inter-VLAN over IPv6 |
| PC-KG-FAB-01 | browser `http://www6.isp-web.com` | ✅ "IPv6 works!" |
| PC-KG-LEGACY-01 | `ping 200.1.100.11` | ✅ via PAT |
| PC-KG-ADMIN-01 | `ssh -l admin 2001:DB8:30:99::11` | ✅ |
| PC-KG-FAB-01 | `ssh -l admin 2001:DB8:30:99::11` | ❌ (MGMT6 ACL) |
| SWL3-KG-DIST1 | `show ntp status` | synchronized to 200.1.10.4 |

**HSRPv6 failover**: `ping -t` from a PC to the IT server, power off DIST1 → DIST2 becomes
Active for `FE80::1`, pings resume.

## Checkpoint
`my-work/step14.pkt` + commit. All three branches are built! 🎉

## Common mistakes
- OSPFv3 doesn't start → no `router-id` (there's no IPv4 address to take one from).
- PCs get only a link-local address → missing `ipv6 unicast-routing` (the switch sends no RAs).
- Inbound IPv6 ACL without `permit icmp` → neighbour discovery breaks and *everything* dies.

## Challenge
Why can a PC in Kiryat Gat **not** browse `http://www.intel.lab` (Haifa's web server), even
though Kiryat Gat has internet? Suggest two real-world solutions.

<details><summary>Answer</summary>

`www.intel.lab` lives at an IPv4 address (200.1.10.3 public / 10.10.200.13 private) and the
Kiryat Gat PCs have **no IPv4 stack**. An IPv6-only host can't talk to an IPv4-only server
without a translator. Real solutions: (1) **NAT64 + DNS64** on the Kiryat Gat edge;
(2) make Haifa's DMZ **dual-stack** (give the web server an IPv6 address and an AAAA
record); (3) a reverse proxy. Packet Tracer doesn't support NAT64, so document this as a
design trade-off — the ministry requirement "one branch entirely IPv6" creates it.
</details>
