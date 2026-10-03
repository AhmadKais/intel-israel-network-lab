# Step 06 — Layer-2 security

## Goal
The access layer resists the classic LAN attacks: MAC flooding, rogue DHCP servers,
rogue switches, VLAN hopping.

Ministry requirements: **13, 16, 22.9**

## Concepts

| Attack | What the attacker does | Our defence |
|---|---|---|
| MAC flooding | Sends thousands of fake MACs → switch table full → switch floods every frame like a hub → attacker sniffs | **Port-security**: max N MACs per port |
| Rogue DHCP | Runs a DHCP server, gives *himself* as gateway → man-in-the-middle | **DHCP snooping**: DHCP *offers* only accepted from trusted ports (uplinks) |
| DHCP starvation | Requests every address in the pool | DHCP snooping **rate limit** + port-security |
| Rogue switch / STP attack | Plugs in a switch, becomes root | **BPDU Guard** + **Root Guard** (done in step 04) |
| VLAN hopping (switch spoofing) | Negotiates a trunk with DTP | `switchport mode access` + `nonegotiate` (done) |
| VLAN hopping (double tagging) | Uses the native VLAN | Native VLAN 998 with no users (done) |
| Plugging into a free port | — | Parking VLAN + shutdown (done) |
| CDP reconnaissance | Learns model/IOS/IP of the switch | `no cdp enable` on user ports |

**Port-security options**
- `maximum 2` — a PC + an IP phone, or a docking station.
- `mac-address sticky` — learned MACs are written into the running-config.
- `violation`: `protect` (drop silently) · `restrict` (drop + log + count) ·
  `shutdown` (err-disable the port — the default). We use **restrict** for users so one
  mistake doesn't kill a port, and log it to Syslog (step 08).

**DHCP snooping and option 82** — when snooping is on, the switch inserts *option 82*
into DHCP requests. Our DHCP server is an IOS L3 switch that, seeing option 82 without a
relay address, **drops the request**. So on access switches we disable inserting it:
`no ip dhcp snooping information option`. This is the #1 reason students' PCs get
169.254.x.x after this step.

## Configure — SW-HFA-A-ACC1 and SW-HFA-A-ACC2

```
configure terminal

! ---------- DHCP snooping ----------
ip dhcp snooping
ip dhcp snooping vlan 10,20,30,99,110,120
no ip dhcp snooping information option

! uplinks lead to the real DHCP servers (DIST1/DIST2) -> trusted
interface range gi0/1 - 2
 ip dhcp snooping trust

! ---------- user ports ----------
interface range fa0/1 - 20
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 ip dhcp snooping limit rate 10
 no cdp enable

! printer port: exactly one device
interface fa0/21
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 no cdp enable
end
wr
```

**ACC1 only** — the admin PC port:
```
interface fa0/22
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 no cdp enable
```

**ACC2 only** — the two access-point ports. Every wireless client's MAC appears on the AP
port, so allow many:
```
interface range fa0/22 - 23
 switchport port-security
 switchport port-security maximum 30
 switchport port-security violation restrict
 ip dhcp snooping limit rate 20
 no cdp enable
```

## Verify

```
SW-HFA-A-ACC1# show ip dhcp snooping
   Switch DHCP snooping is enabled
   DHCP snooping is configured on following VLANs: 10,20,30,99,110,120
   Interface   Trusted   Rate limit (pps)
   Gi0/1       yes       unlimited
   Gi0/2       yes       unlimited
   Fa0/1       no        10
SW-HFA-A-ACC1# show ip dhcp snooping binding      <- one line per DHCP PC
SW-HFA-A-ACC1# show port-security
SW-HFA-A-ACC1# show port-security interface fa0/1
SW-HFA-A-ACC1# show run | include sticky          <- the learned MACs
```
On each PC: `ipconfig /release` then `ipconfig /renew` → still gets an address.

### Attack lab
1. **Rogue DHCP**: add a *Server-PT* `SRV-ROGUE` on ACC1 Fa0/9, IP 10.10.16.250/24,
   *Services → DHCP → On*, pool gateway 10.10.16.250, start 10.10.16.200. Renew a PC in
   VLAN 10 → it still gets a **10.10.16.11-130** address with gateway **.1** from DIST1.
   The rogue's offers were dropped. Delete the rogue server afterwards.
2. **MAC violation**: unplug PRN-HFA-A-01 from Fa0/21 and plug a laptop in its place →
   the port goes err-disabled (`show port-security interface fa0/21` → `Secure-shutdown`).
   Recover: put the printer back, `shutdown` / `no shutdown` on Fa0/21.

## Checkpoint
`my-work/step06.pkt` + commit.

## Common mistakes
- PCs get 169.254.x.x → missing `trust` on uplinks, or missing
  `no ip dhcp snooping information option`.
- Port-security on a port still in `dynamic` mode is refused — the port must be
  `switchport mode access` first.

## Challenge (bonus security — careful!)
**Dynamic ARP Inspection (DAI)** uses the snooping binding table to drop forged ARP
replies (ARP poisoning). Enable it on ACC1 for VLAN 10 only and make sure printers/static
devices are not broken. What has to be trusted?

<details><summary>Answer</summary>

```
ip arp inspection vlan 10
interface range gi0/1 - 2
 ip arp inspection trust
```
Only VLAN 10 because its hosts all use DHCP (they're in the binding table). VLAN 30
(static printer) and 99 (static admin PC) would need ARP ACLs, otherwise DAI drops their
ARP and they go offline. Uplinks must be trusted, otherwise the gateway's ARP replies
(which arrive on the uplink and are not in the binding table) are dropped.
</details>
