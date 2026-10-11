# Glossary — every term in simple English

Grouped by the step where you first meet it.

## Basics (step 00)
| Term | Simple meaning |
|---|---|
| **Network** | Devices connected so they can exchange data |
| **End device** | A device people use: PC, laptop, printer, phone |
| **Switch** | Connects devices in the same local network; delivers by MAC address |
| **Router** | Connects different networks; chooses the path by IP address |
| **Layer-3 switch** | A switch that can also route |
| **Server** | A computer that provides a service (web, email, DNS, DHCP…) |
| **MAC address** | The hardware ID of a network card. Never changes. Like an ID number |
| **IP address** | The logical address of a device in a network. Like a home address |
| **Subnet mask / prefix** | Says which part of the IP is the network ("street") and which is the host ("house") |
| **Default gateway** | The router a device sends traffic to when the destination is on another network |
| **Ping** | A test message: "are you there?" — a reply proves the path works both ways |
| **ARP** | "Who has IP X? Tell me your MAC" — how a device finds the MAC for an IP |
| **MAC address table** | The switch's list of which MAC is on which port |
| **Broadcast** | A message sent to everyone in the local network |
| **IOS / CLI** | Cisco's operating system / the text interface where you type commands |
| **Running-config** | The settings active right now (in memory — lost on reboot) |
| **Startup-config** | The saved settings loaded at boot (`copy run start` saves to it) |

## Addressing (step 01)
| Term | Simple meaning |
|---|---|
| **Private addresses** | 10.x, 172.16–31.x, 192.168.x — for use inside organisations, not on the internet |
| **Subnet** | One network ("street") — e.g. 10.10.16.0/24 |
| **/24, /30, /32…** | How many bits are the network part. Bigger number = smaller network |
| **Network address** | The first address of a subnet (its name) — can't be given to a device |
| **Broadcast address** | The last address of a subnet ("everyone") — can't be given to a device |
| **Hierarchical addressing** | Big block → smaller blocks → smaller blocks, like country → city → street |
| **Route summarization** | Advertising one big route instead of many small ones |
| **Loopback** | A virtual interface that never goes down — used as the device's ID |

## Switching (steps 02–04)
| Term | Simple meaning |
|---|---|
| **VLAN** | A separate virtual network ("room") inside a switch |
| **Access port** | Port for one end device, in one VLAN |
| **Trunk** | One link carrying many VLANs, with tags |
| **802.1Q tag** | The "sticker" on a frame that says which VLAN it belongs to |
| **Native VLAN** | The one VLAN that crosses a trunk without a tag (we use unused VLAN 998) |
| **DTP** | Automatic trunk negotiation — we turn it off |
| **VTP** | Shares the VLAN list from a server switch to client switches |
| **VTP revision number** | Version number of the VLAN list — the highest one wins |
| **Hardening** | Basic security settings: passwords, banner, timeouts, unused ports off |
| **Banner (MOTD)** | Legal warning shown before login |
| **Parking VLAN** | An unused VLAN where we put (and shut) all unused ports |
| **Loop** | Two or more paths between switches — broadcasts circle forever unless STP blocks one |
| **STP (Spanning Tree)** | Blocks redundant links to prevent loops; unblocks them if the active link fails |
| **Root bridge** | The "centre" switch of the spanning tree |
| **EtherChannel** | Several cables bundled into one logical link |
| **PortFast / BPDU Guard / Root Guard** | STP features for fast PC ports and protection against rogue switches |

## Coming later
| Term | Simple meaning | Step |
|---|---|---|
| **Inter-VLAN routing / SVI** | Routing between VLANs on a Layer-3 switch / its virtual interface per VLAN | 05 |
| **HSRP** | Two routers share one virtual gateway address | 05 |
| **DHCP** | Gives devices their IP settings automatically | 05 |
| **Port-security / DHCP snooping** | Limit devices per port / block fake DHCP servers | 06 |
| **SSH** | Encrypted remote login | 07 |
| **OSPF / area / ABR** | Routing protocol / group of routers / router between areas | 08 |
| **AAA / TACACS+ / Syslog / NTP** | Central login check / its protocol / central logs / time sync | 08, 11 |
| **Virtual link** | A logical connection that joins a far area to area 0 | 09 |
| **SSID / WPA2** | WiFi network name / WiFi encryption | 10 |
| **Firewall / DMZ / security level** | Traffic filter / server zone / trust level of an ASA interface | 11 |
| **BGP / AS** | The internet's routing protocol / one organisation's network | 12 |
| **NAT / PAT** | Translating private addresses to public / many users sharing one public IP | 12 |
| **EIGRP / collapsed core** | Cisco routing protocol / core and distribution in one layer | 13 |
| **IPv6 / SLAAC / OSPFv3** | 128-bit addresses / automatic IPv6 addressing / OSPF for IPv6 | 14 |
| **VPN / IPsec / GRE** | Private tunnel over the internet / encryption / plain tunnel | 15 |
