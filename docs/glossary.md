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

## Packet Tracer views (steps 00, 09, 12–16)
| Term | Simple meaning |
|---|---|
| **Logical view** | The network map in Packet Tracer: who is connected to whom. You build and configure here |
| **Physical view** | Where the devices really are: Intercity → city → building → wiring closet → rack |
| **Intercity** | The top level of the Physical view — the country map with all the cities |
| **Wiring closet** | The room in a building where the switches and routers stand in racks (real names: MDF = main closet, IDF = closet on a floor) |
| **Working closet** | The closet where every new device you add in the Logical view starts. In this project: HFA Data Center → Main Wiring Closet |
| **Cluster** | Many devices shown as **one** icon in the Logical view — only the drawing changes |
| **Site** | One location of the company — in this course, one city (Haifa, Jerusalem, Kiryat Gat) |

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
| **LACP / PAgP** | The two protocols that build an EtherChannel — LACP is the open standard, PAgP is Cisco's own |
| **err-disabled** | A port the switch shut by itself because of a violation (e.g. BPDU Guard). Fix: `shutdown` → `no shutdown` |

## Coming later
| Term | Simple meaning | Step |
|---|---|---|
| **Inter-VLAN routing / SVI** | Routing between VLANs on a Layer-3 switch / its virtual interface per VLAN | 05 |
| **HSRP** | Two routers share one virtual gateway address | 05 |
| **DHCP** | Gives devices their IP settings automatically | 05 |
| **Split scope / helper-address** | Two DHCP servers each hand out half of the range / a gateway forwards a PC's DHCP request to a server in another network | 05, 13 |
| **Port-security / DHCP snooping** | Limit devices per port / block fake DHCP servers | 06 |
| **Sticky MAC** | Port-security remembers the MAC address of the first device on the port and blocks others | 06 |
| **SSH** | Encrypted remote login | 07 |
| **Wildcard mask** | The "reverse mask" in ACLs and OSPF: 0 = must match, 255 = anything (`0.0.0.255` = a /24) | 07, 08 |
| **OSPF / area / ABR** | Routing protocol / group of routers / router between areas | 08 |
| **Router-ID** | The name of a router inside OSPF/EIGRP/BGP — written like an IP address (we use Loopback0) | 08 |
| **AAA / TACACS+ / Syslog / NTP** | Central login check / its protocol / central logs / time sync | 08, 11 |
| **Virtual link** | A logical connection that joins a far area to area 0 | 09 |
| **SSID / WPA2** | WiFi network name / WiFi encryption | 10 |
| **RADIUS** | A login-check protocol (like TACACS+), used for WPA2-Enterprise WiFi | 10 |
| **Firewall / DMZ / security level** | Traffic filter / server zone / trust level of an ASA interface | 11 |
| **BGP / AS** | The internet's routing protocol / one organisation's network | 12 |
| **NAT / PAT** | Translating private addresses to public / many users sharing one public IP | 12 |
| **eBGP** | BGP between two different ASes (here: between the ISPs) | 12 |
| **Floating static route** | A backup static route with a higher distance — used only when the main route is gone | 12 |
| **EIGRP / collapsed core** | Cisco routing protocol / core and distribution in one layer | 13 |
| **CBAC** | A router firewall: lets replies to our own connections back in, blocks everything else from outside | 13 |
| **IPv6 / SLAAC / OSPFv3** | 128-bit addresses / automatic IPv6 addressing / OSPF for IPv6 | 14 |
| **Link-local address** | An IPv6 address (`FE80::…`) that works only on its own link; PCs use it as the gateway | 14 |
| **SDM template** | A switch memory setting — it must allow IPv6 before `ipv6` commands work | 14 |
| **VPN / IPsec / GRE** | Private tunnel over the internet / encryption / plain tunnel | 15 |
| **ISAKMP / transform-set / crypto map** | IPsec phase 1 (agreeing on keys) / how to encrypt / which traffic to encrypt, to which peer | 15 |
