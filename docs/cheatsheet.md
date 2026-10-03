# Verification cheat-sheet

## CLI survival

| Keys / command | Does |
|---|---|
| `?` | list what you can type here |
| `Tab` | complete the word |
| `Ctrl+Shift+6` | stop a ping/traceroute or a stuck DNS lookup |
| `Ctrl+Z` / `end` | back to privileged EXEC |
| `do <show cmd>` | run a show command from config mode |
| `show run \| section ospf` | show only part of the config |
| `show run \| include hostname` | grep the config |
| `copy run start` / `wr` | **save** (do it after every step!) |

## Layer 2

| Command | Look for |
|---|---|
| `show vlan brief` | VLANs exist, ports in the right VLAN |
| `show interfaces trunk` | trunk up, native VLAN 998, allowed VLAN list |
| `show vtp status` | domain, mode, revision number |
| `show etherchannel summary` | `Po1(SU)` and member ports `(P)` |
| `show spanning-tree vlan 10` | "This bridge is the root" on the right switch |
| `show spanning-tree summary` | mode rapid-pvst, PortFast/BPDU guard defaults |
| `show port-security interface fa0/1` | Secure-up, violation count |
| `show ip dhcp snooping` / `... binding` | trusted ports, learned bindings |
| `show mac address-table` | which MAC is on which port |
| `show cdp neighbors` | who is cabled where |

## Layer 3

| Command | Look for |
|---|---|
| `show ip interface brief` | up/up and correct IPs |
| `show standby brief` | Active/Standby, VIP, priority, `P` (preempt) |
| `show ip route` | O / O IA / O E2 / D / D EX / S / B / C codes |
| `show ip ospf neighbor` | FULL state |
| `show ip ospf interface brief` | area per interface |
| `show ip ospf virtual-links` | `Virtual Link ... is up` |
| `show ip eigrp neighbors` | neighbours listed |
| `show ip bgp summary` | neighbour state shows a number (prefixes), not Idle/Active |
| `show ip bgp` | routes with `*>` (best) |
| `show ipv6 route` / `show ipv6 ospf neighbor` | IPv6 equivalents |
| `show ip nat translations` | inside local ↔ inside global |
| `show ip dhcp binding` | leases handed out |
| `show crypto isakmp sa` | `QM_IDLE` = phase 1 up |
| `show crypto ipsec sa` | `#pkts encaps/decaps` increasing |
| `show access-lists` | hit counters `(N match(es))` |

## ASA

| Command | Look for |
|---|---|
| `show nameif` | inside/outside/dmz with security levels |
| `show route` | default route + inside routes |
| `show access-list` | hit counts |
| `show conn` | active connections through the firewall |

## End-host tests (PC → Desktop → Command Prompt)

```
ipconfig /all          ipv6config
ping 10.10.200.13      ping -t 10.10.16.1   (continuous — stop with Ctrl+C)
tracert 200.1.100.11   nslookup www.intel.lab
ssh -l admin 10.10.19.11
```
