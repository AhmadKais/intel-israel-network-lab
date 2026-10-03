# Step 04 — EtherChannel & Spanning Tree

## Goal
DIST1 and DIST2 are joined by a 2-cable **LACP EtherChannel**. **Rapid-PVST+** runs
everywhere, **you** decide who is root per VLAN, user ports come up instantly (**PortFast**)
and are protected by **BPDU Guard**; distribution downlinks are protected by **Root Guard**.

Ministry requirements: **14, 15, 22.14, 6, 13**

## Concepts
- **EtherChannel** bundles 2–8 physical links into one logical link (`Port-channel1`).
  STP sees one link → nothing blocked → double bandwidth + survives one cable failure.
  - **LACP** (IEEE, `active`/`passive`) — we use it in Haifa.
  - **PAgP** (Cisco, `desirable`/`auto`) — we use it in Jerusalem (step 13).
  - All member ports must match: speed, duplex, mode, native VLAN, allowed VLANs.
- **Rapid-PVST+** — one STP instance per VLAN, converges in ~1 s instead of 30–50 s.
- **Root bridge** — the centre of the STP tree. It should be the distribution switch that is
  also the **HSRP active gateway** for that VLAN (step 05), so traffic takes the shortest path.
  - DIST1: root for 10, 30, 99, 110 / secondary for 20, 120
  - DIST2: root for 20, 120 / secondary for 10, 30, 99, 110
  → both distribution switches carry traffic (load sharing).
- **PortFast** — port goes straight to forwarding (PCs get DHCP before they time out).
  Only on ports to end devices!
- **BPDU Guard** — if a BPDU arrives on a PortFast port (someone plugged in a switch),
  the port is **err-disabled**. Stops rogue switches and loops.
- **Root Guard** — on the distribution downlinks: if an access switch suddenly claims to be
  a better root, the port is blocked (`root-inconsistent`). Stops root-bridge hijacking.

## Build
Two cables: **DIST1 Gi1/0/23 ↔ DIST2 Gi1/0/23** and **DIST1 Gi1/0/24 ↔ DIST2 Gi1/0/24**.

## Configure

### SWL3-HFA-A-DIST1
```
configure terminal
spanning-tree mode rapid-pvst
spanning-tree vlan 10,30,99,110 root primary
spanning-tree vlan 20,120 root secondary

! ---- EtherChannel to DIST2 ----
interface range gi1/0/23 - 24
 description Po1 member to SWL3-HFA-A-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,99,110,120,998
 switchport nonegotiate
 channel-group 1 mode active
 no shutdown
interface port-channel 1
 description EtherChannel to SWL3-HFA-A-DIST2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,99,110,120,998
 switchport nonegotiate

! ---- Root Guard on the downlinks to access switches ----
interface range gi1/0/1 - 2
 spanning-tree guard root
end
wr
```

### SWL3-HFA-A-DIST2
Same as DIST1, but swap the root lines:
```
spanning-tree mode rapid-pvst
spanning-tree vlan 20,120 root primary
spanning-tree vlan 10,30,99,110 root secondary
```
…and `channel-group 1 mode active` (or `passive`) on Gi1/0/23-24, same Port-channel 1
config, and `spanning-tree guard root` on Gi1/0/1-2.

### SW-HFA-A-ACC1 and ACC2
```
configure terminal
spanning-tree mode rapid-pvst

! user/printer/AP ports: PortFast + BPDU Guard
interface range fa0/1 - 23
 spanning-tree portfast
 spanning-tree bpduguard enable
end
wr
```
(PortFast on an AP port is fine — an AP is an end device to STP.)

## Verify

```
SWL3-HFA-A-DIST1# show etherchannel summary
```
Expected: `1  Po1(SU)  LACP  Gi1/0/23(P) Gi1/0/24(P)` — **S**=Layer2, **U**=in use,
**P**=bundled. `(SD)` / `(I)` / `(s)` means something doesn't match.

```
SWL3-HFA-A-DIST1# show spanning-tree vlan 10      -> "This bridge is the root"
SWL3-HFA-A-DIST2# show spanning-tree vlan 20      -> "This bridge is the root"
SW-HFA-A-ACC1#    show spanning-tree vlan 10      -> Root port = Gi0/1 (toward DIST1)
SW-HFA-A-ACC1#    show spanning-tree vlan 20      -> Root port = Gi0/2 (toward DIST2)
SW-HFA-A-ACC1#    show spanning-tree summary      -> mode rapid-pvst
```

### Break things on purpose (the best way to learn)
1. **Redundancy**: `ping -t 10.10.16.103` from ENG-01, then delete the ACC1–DIST1 cable.
   You lose 1–2 pings, then it continues via DIST2. Put the cable back.
2. **EtherChannel**: delete one of the two DIST1–DIST2 cables. `show etherchannel summary`
   → Po1 still up with one member.
3. **BPDU Guard**: drag a new 2960 and connect it to ACC1 Fa0/9. The port goes
   **err-disabled** (`show interfaces status err-disabled`). Remove the rogue switch and
   recover the port: `interface fa0/9` → `shutdown` → `no shutdown`.

## Checkpoint
`my-work/step04.pkt` + commit.

## Common mistakes
- Configuring `channel-group` on ports that already differ (e.g. one port has a different
  allowed list) → `Po1(SD)`. Fix: `default interface range gi1/0/23 - 24` and redo both
  members identically.
- `root primary` on both switches for the same VLAN — the last one wins, unpredictably.
- PortFast on a trunk/uplink → temporary loops. Never.

## Challenge
`spanning-tree vlan 10 root primary` is a macro. Run `show run | include priority` on
DIST1: what priority did it actually set? Why is that number a multiple of 4096?

<details><summary>Answer</summary>

`spanning-tree vlan 10 priority 24576` (secondary = 28672). The bridge priority field is
4 bits on the wire — the remaining 12 bits of the old 16-bit field carry the VLAN ID
(extended system ID), so the priority can only change in steps of 2^12 = 4096.
</details>
