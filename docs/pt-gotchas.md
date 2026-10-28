# Packet Tracer gotchas

Packet Tracer is a *simulator*: it implements a subset of IOS, sometimes with older syntax.
When a command is rejected (`% Invalid input detected at '^' marker`), check this list
before assuming you made a mistake.

## Hardware

| Problem | Fix |
|---|---|
| **3650 switch has no power / all ports down** | Physical tab → drag the **AC-POWER-SUPPLY** module into the empty slot. Every 3650 needs this. |
| Router has too few ports | 2911 has Gi0/0-0/2. Power off → add an HWIC/EHWIC module → power on. |
| Laptop can't do WiFi | Physical tab → power off → remove the Ethernet module → drag **WPC300N** in → power on. |
| Which cable? | **Never** use the Automatic (lightning bolt) cable — it picks the ports itself. Straight-Through between different device types, Cross-Over between same types ([basic-moves.md](basic-moves.md) M5). Check with `show cdp neighbors` that the ports match the cabling table (M6). DSL modem ↔ cloud uses the **phone** cable. |

## Commands

| Command that fails | What to do |
|---|---|
| `switchport trunk encapsulation dot1q` on a 3650 | 3650 is dot1q-only — just skip it and use `switchport mode trunk`. On a 3560 it is **required** before `mode trunk`. |
| `ipv6 unicast-routing` / `ipv6 address` rejected on a switch | `sdm prefer dual-ipv4-and-ipv6 default` → `wr` → `reload`. On a 2960 use `sdm prefer lanbase-routing` if the first one is missing. |
| `logging host 10.10.1.10` rejected | Older syntax: `logging 10.10.1.10`. |
| `tacacs server NAME` block rejected | Older syntax: `tacacs-server host 10.10.1.11 key Intel#Tac`. |
| `ip tacacs source-interface Loopback0` rejected | Register the device in the AAA server with the IP of the interface it uses to *reach* the server (usually its uplink IP). |
| `crypto key generate rsa` asks for size | Answer `1024` (SSH v2 needs ≥ 768). |
| `crypto isakmp ...` / `crypto map` missing on a router | Enable the security license: `license boot module c2900 technology-package securityk9` → accept → `wr` → `reload`. |
| ASA rejects `enable password` prompt | The ASA's enable password is empty on first boot — just press Enter. |
| ASA keeps giving 192.168.1.x addresses | The default config has a DHCP server on `inside`: `no dhcpd enable inside`. |
| `standby 31 ipv6 FE80::1` rejected | Use `standby 31 ipv6 autoconfig`, then read the virtual link-local with `show standby brief` and use that as the static gateway on servers. |
| HSRP for IPv6 not supported on your PT build | IPv6 already has built-in gateway redundancy: both DIST switches send Router Advertisements, so hosts learn two default routers. Document this in the book. |
| OSPFv3 `passive-interface default` rejected | Leave out the `passive-interface` lines. OSPFv3 then also sends hellos into the user VLANs — nothing breaks, because no router listens there, and the VLAN prefixes are still advertised. Note in the book that passive interfaces were not available. |
| `ip summary-address eigrp` rejected on the switch | Put the command on the physical routed interface (`Gi1/1/1`), not on the SVI. |
| `spanning-tree guard root` not shown | It is there — check with `show running-config interface gi1/0/1`; when it acts, `show spanning-tree vlan 10` shows the port blocked. |

## Behaviour

| Symptom | Reason / fix |
|---|---|
| First ping shows 1–2 timeouts | ARP is resolving. Ping again. |
| Everything is green but PCs get 169.254.x.x | DHCP snooping dropped the offer: you forgot `ip dhcp snooping trust` on the uplink, or forgot `no ip dhcp snooping information option` (option 82). |
| OSPF neighbours stuck in INIT/EXSTART | MD5 key or area mismatch, or one side passive. |
| Changes don't seem to take effect | Click the **fast-forward** (Fast Forward Time) button a few times; STP/OSPF/BGP need simulated time to converge. |
| File gets slow with 150+ devices | Save often, close other programs, use **Realtime** mode only. |
| VTP wiped all your VLANs | A switch with a higher revision number joined the domain. Before joining any switch, set `vtp mode transparent` then back to `client` — this resets the revision to 0. |
