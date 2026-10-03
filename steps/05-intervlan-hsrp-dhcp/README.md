# Step 05 — Inter-VLAN routing, HSRP & DHCP

## Goal
The distribution switches become **routers** for building A. Every VLAN gets a gateway
that **survives the loss of either distribution switch** (HSRP), and PCs get their
addresses from **DHCP pools running on the L3 switches** — split between the two so DHCP
is redundant too.

Ministry requirements: **22.10, 17, 20.7, 22.11 (DHCP on routers in one branch), 6**

## Concepts
- **SVI** (`interface vlan 10`) — a virtual routed interface inside an L3 switch, one per
  VLAN. With `ip routing` the switch routes between its SVIs = inter-VLAN routing.
- **HSRP** — two switches share a **virtual IP** (`.1`) and a virtual MAC. One is *Active*
  (answers for `.1`), the other *Standby*. Highest **priority** wins; **preempt** lets the
  preferred switch take back the role when it returns. We use `standby version 2` and the
  group number = the VLAN number (easy to read).
- HSRP active must equal STP root for the same VLAN (step 04) — otherwise traffic crosses
  the EtherChannel for nothing.
- **DHCP split scope (80/20 or 50/50)** — both switches serve the same subnet but hand out
  *different ranges*, so they never give the same address, and if one dies the other
  still serves.
  - DIST1 leases `.11 – .130`
  - DIST2 leases `.131 – .254`

## Configure

### SWL3-HFA-A-DIST1
```
configure terminal
ip routing

interface vlan 10
 description GW ENGINEERING
 ip address 10.10.16.2 255.255.255.0
 standby version 2
 standby 10 ip 10.10.16.1
 standby 10 priority 110
 standby 10 preempt
 no shutdown
interface vlan 20
 description GW FINANCE
 ip address 10.10.17.2 255.255.255.0
 standby version 2
 standby 20 ip 10.10.17.1
 standby 20 priority 90
 standby 20 preempt
 no shutdown
interface vlan 30
 description GW PRINTERS-A
 ip address 10.10.18.2 255.255.255.0
 standby version 2
 standby 30 ip 10.10.18.1
 standby 30 priority 110
 standby 30 preempt
 no shutdown
interface vlan 99
 description GW MGMT-A
 ip address 10.10.19.2 255.255.255.0
 standby version 2
 standby 99 ip 10.10.19.1
 standby 99 priority 110
 standby 99 preempt
 no shutdown
interface vlan 110
 description GW WIFI-EMP
 ip address 10.10.20.2 255.255.255.0
 standby version 2
 standby 110 ip 10.10.20.1
 standby 110 priority 110
 standby 110 preempt
 no shutdown
interface vlan 120
 description GW WIFI-GUEST
 ip address 10.10.21.2 255.255.255.0
 standby version 2
 standby 120 ip 10.10.21.1
 standby 120 priority 90
 standby 120 preempt
 no shutdown

! ---------- DHCP: this switch leases .11 - .130 ----------
ip dhcp excluded-address 10.10.16.1 10.10.16.10
ip dhcp excluded-address 10.10.16.131 10.10.16.254
ip dhcp excluded-address 10.10.17.1 10.10.17.10
ip dhcp excluded-address 10.10.17.131 10.10.17.254
ip dhcp excluded-address 10.10.20.1 10.10.20.10
ip dhcp excluded-address 10.10.20.131 10.10.20.254
ip dhcp excluded-address 10.10.21.1 10.10.21.10
ip dhcp excluded-address 10.10.21.131 10.10.21.254

ip dhcp pool HFA-ENGINEERING
 network 10.10.16.0 255.255.255.0
 default-router 10.10.16.1
 dns-server 10.10.200.11
 domain-name intel.lab
ip dhcp pool HFA-FINANCE
 network 10.10.17.0 255.255.255.0
 default-router 10.10.17.1
 dns-server 10.10.200.11
 domain-name intel.lab
ip dhcp pool HFA-WIFI-EMP
 network 10.10.20.0 255.255.255.0
 default-router 10.10.20.1
 dns-server 10.10.200.11
 domain-name intel.lab
ip dhcp pool HFA-WIFI-GUEST
 network 10.10.21.0 255.255.255.0
 default-router 10.10.21.1
 ! guests get the ISP's public DNS, not our internal one
 dns-server 200.1.100.10
end
wr
```

> Why does the pool point to `.1` (the HSRP VIP) and not `.2`? Because if DIST1 dies, `.2`
> dies with it — `.1` moves to DIST2.

> The DNS servers (10.10.200.11, 200.1.100.10) don't exist yet — they arrive in steps 11
> and 12. That's fine; PCs just won't resolve names until then.

### SWL3-HFA-A-DIST2
Same SVIs, with **`.3`** addresses, and the priorities mirrored:

| VLAN | DIST2 IP | DIST2 priority |
|---|---|---|
| 10 | 10.10.16.3 | 90 |
| 20 | 10.10.17.3 | **110** |
| 30 | 10.10.18.3 | 90 |
| 99 | 10.10.19.3 | 90 |
| 110 | 10.10.20.3 | 90 |
| 120 | 10.10.21.3 | **110** |

Example for VLAN 20:
```
interface vlan 20
 description GW FINANCE
 ip address 10.10.17.3 255.255.255.0
 standby version 2
 standby 20 ip 10.10.17.1
 standby 20 priority 110
 standby 20 preempt
 no shutdown
```
DHCP on DIST2 — leases `.131 – .254`:
```
ip dhcp excluded-address 10.10.16.1 10.10.16.130
ip dhcp excluded-address 10.10.17.1 10.10.17.130
ip dhcp excluded-address 10.10.20.1 10.10.20.130
ip dhcp excluded-address 10.10.21.1 10.10.21.130
```
…plus the **same four `ip dhcp pool` blocks** as DIST1.

### Access switches — management IP (so you can reach them later)
```
! ACC1: 10.10.19.11   ACC2: 10.10.19.12
interface vlan 99
 ip address 10.10.19.11 255.255.255.0
 no shutdown
exit
ip default-gateway 10.10.19.1
```

### PCs
Change every PC from static to **DHCP** (*Desktop → IP Configuration → DHCP*).
Printer and admin PC stay static (printer: gateway 10.10.18.1; admin: 10.10.19.100, gw .1).

## Verify

```
SWL3-HFA-A-DIST1# show standby brief
Interface Grp Pri P State   Active      Standby     Virtual IP
Vl10      10  110 P Active  local       10.10.16.3  10.10.16.1
Vl20      20  90  P Standby 10.10.17.3  local       10.10.17.1
...
```
```
SWL3-HFA-A-DIST1# show ip dhcp binding         <- leases .11-.130
SWL3-HFA-A-DIST2# show ip dhcp binding         <- maybe some .131+ leases
SWL3-HFA-A-DIST1# show ip route connected      <- one C route per SVI
```

| Test | Expected |
|---|---|
| ENG-01 `ipconfig` | 10.10.16.x, gateway 10.10.16.1 |
| ENG-01 → `ping 10.10.17.x` (a FIN PC) | ✅ inter-VLAN routing works |
| ENG-01 → `ping 10.10.18.10` (printer) | ✅ |
| ENG-01 → `tracert 10.10.17.x` | first hop 10.10.16.2 (DIST1 — the active) |

### Failover test
1. ENG-01: `ping -t 10.10.17.x`
2. Power off DIST1 (Physical tab → power button) — or `shutdown` its uplink ports.
3. A few timeouts, then the pings continue — DIST2 became Active for VLAN 10.
4. Power DIST1 back on → after it boots, **preempt** makes it Active again.
5. With DIST1 off, release/renew DHCP on a PC (`ipconfig /release`, `ipconfig /renew`) →
   you get a `.131+` address from DIST2. Split scope works.

## Checkpoint
`my-work/step05.pkt` + commit.

## Common mistakes
- Forgot `ip routing` → SVIs are up but nothing routes.
- SVI is `down/down` → the VLAN doesn't exist on the switch, or no port in that VLAN is up
  (trunks count). `show vlan brief` + `show interfaces trunk`.
- Different `standby version` on DIST1 and DIST2 → both become Active.

## Challenge
Load-balancing question: VLAN 10 traffic goes through DIST1, VLAN 20 through DIST2. If you
configured DIST1 as HSRP active for VLAN 20 *but* DIST2 stays STP root for VLAN 20, draw
the path a FIN PC on ACC1 takes to reach its gateway. Why is it worse?

<details><summary>Answer</summary>

PC → ACC1 → (STP forwards toward root DIST2) → DIST2 → Po1 → DIST1 (HSRP active) →
routed. The frame crosses the EtherChannel for nothing: one extra hop and the inter-dist
link carries load it shouldn't. Rule: **HSRP active = STP root** per VLAN.
</details>
