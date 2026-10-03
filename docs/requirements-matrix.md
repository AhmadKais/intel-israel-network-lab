# Ministry requirements → where they are built (שאלון 735918)

| # | Requirement (short) | Where | Step |
|---|---|---|---|
| 1 | 3 branches joined by a WAN running BGP | 3 ISPs, eBGP AS 100/200/300 | 12 |
| 2 | VPN between ≥ 2 branches (private IPs) | GRE over IPsec Haifa ↔ Jerusalem | 15 |
| 3 | WAN routing = BGP | ISP border routers | 12 |
| 4 | OSPF/EIGRP inside the provider AS | OSPF 10 between BR and PE in each ISP | 12 |
| 5 | Collapsed core with L3 switches in ≥ 1 branch | Jerusalem | 13 |
| 6 | L2 and L3 redundancy | dual uplinks, STP, EtherChannel, HSRP, dual cores, split DHCP, DSL backup | 03–05, 08, 12 |
| 7 | One branch fully IPv6 | Kiryat Gat | 14 |
| 8 | Multi-area OSPF with a virtual link | Haifa areas 0/1/2/3, vlink through area 2 | 08–09 |
| 9 | Internet connection | ISP-A / ISP-B / ISP-C | 12 |
| 10 | NAT: dynamic / static / overload, one per branch | JLM dynamic · HFA static · KG overload | 12–14 |
| 11 | inside / outside / DMZ zones | ASA in Haifa | 11 |
| 12 | Firewall (ASA) on ≥ 1 branch | ASA 5506-X | 11 |
| 13 | Port security, SSH, L2/L3 attack protection | port-security, DHCP snooping, BPDU/root guard, DTP off, parking VLAN, VTY ACL, routing auth, CBAC | 06, 07, 04, 13 |
| 14 | STP, manual root bridge, PortFast | Rapid-PVST+, root primary/secondary per VLAN | 04 |
| 15 | BPDU Guard + Root Guard | access ports / distribution downlinks | 04 |
| 16 | DHCP snooping | all access switches | 06 |
| 17 | HSRP in every branch, on L3 switches in ≥ 1 | HFA dist L3, JLM CC L3, KG HSRPv6 | 05, 13, 14 |
| 18 | Wireless with guest + employee access, secured | Haifa building A | 10 |
| 19 | One main branch | Haifa | — |
| 20.1 | 3 buildings/floors, access/dist/core | HFA 3 buildings · JLM 3 floors · KG 3 access switches | 02–09, 13, 14 |
| 20.2 | 6 departments × ≥ 8 PCs | every branch | 16 |
| 20.3–20.6 | switches, routers, printers, APs | every branch | — |
| 20.7 | DHCP server (router or dedicated) | HFA L3 switches · JLM server · KG DHCPv6 | 05, 13, 14 |
| 20.8 | NTP synced to the main NTP | DMZ NTP is master; JLM via VPN, KG via public static NAT | 11, 13, 14 |
| 20.9 | Syslog server + logging on all devices | SRV-HFA-SYSLOG, SRV-JLM-SYSLOG, SRV-KG-SYSLOG | 08, 13, 14 |
| 20.10 | AAA server | SRV-HFA-AAA, SRV-JLM-AAA, SRV-KG-AAA | 08, 13, 14 |
| 21 | DMZ: DNS, Email, Web, NTP, FTP, AAA — reachable via VPN | Haifa DMZ | 11, 15 |
| 22.1 | Logical hostnames | [design.md §1](design.md) | all |
| 22.2 | Banner on all switches/routers | | 02 → all |
| 22.3 | Hashed passwords | `enable secret`, `username ... secret`, `service password-encryption` | 02 → all |
| 22.4 | Hierarchical IP plan from one master block | 10.0.0.0/8 → /16 per site → /20 per area | 01 |
| 22.5 | SSH/AAA management, dedicated mgmt VLAN, block other IPs | VLAN 99 + VTY access-class | 07, 08 |
| 22.6–22.8 | VLANs, trunks, VTP | | 02–03 |
| 22.9 | Port security | | 06 |
| 22.10 | Inter-VLAN routing | SVIs | 05 |
| 22.11 | DHCP by routers in one branch, server in another | HFA L3 switches / JLM server | 05, 13 |
| 22.12 | NAT for browsing + static NAT to internal web | PAT + `200.1.10.3 → 10.10.200.13` | 12 |
| 22.13 | ACLs, CBAC | VTY ACLs, guest ACL, ASA ACL, CBAC in JLM | 07, 10, 11, 13 |
| 22.14 | EtherChannel | LACP in HFA, PAgP in JLM | 04, 13 |
| 22.15 | EIGRP/OSPF + summarization + passive + password | HFA ABR `area range`, JLM `summary-address`, MD5 on both | 08, 09, 13 |
| 22.16 | Multi-area OSPF with virtual link | Haifa | 09 |
| 22.17 | Static routing toward the WAN | every edge router has a static default | 12 |
| 22.18 | Routing-protocol security | MD5 auth + passive interfaces | 08, 13 |
| 22.19 | Email account on 2 PCs per department, per branch | | 16 |
| 23.1 | Redundancy inside and between branches | BGP triangle, dual uplinks, DSL backup | 12 |
| 23.2 | ISP reached via DSL modem | Haifa backup line | 12 |
| 23.3 | ISP DNS + 2 web servers reachable from every PC | ISP-A server LAN | 12 |
| 23.4 | Remote-access VPN to main branch | teleworker on ISP-C | 15 |
