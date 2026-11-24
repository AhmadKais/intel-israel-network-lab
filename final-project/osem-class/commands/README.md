# 📘 All the commands of the project

In the order of the story: each command, what it does and why, and where we write it for the first time.

## Chapter 1 — one department
| Command | What it does, and why | First time |
|---|---|---|
| `enable` | From looking (`>`) to managing (`#`). This is where the enable password is asked for. | [lesson 1, Mission 4](../lesson-01/) |
| `configure terminal` | Opens configuration mode (`(config)#`). Changes are possible only from here. | [lesson 1, Mission 4](../lesson-01/) |
| `hostname SW-S-MAIN` | Gives the device its name, the one you see in the prompt. It prevents typing on the wrong device. | [lesson 1, Mission 4](../lesson-01/) |
| `no ip domain-lookup` | A typing mistake is not looked up as a name in DNS, so you don't wait after every typo. | [lesson 1, Mission 4](../lesson-01/) |
| `enable secret Osem#En1` | The password for `#`, stored as a hash (not readable). | [lesson 1, Mission 4](../lesson-01/) |
| `service password-encryption` | Hides the other passwords in `show run`. Weak, but better than plain text. | [lesson 1, Mission 4](../lesson-01/) |
| `banner motd # Osem - Authorized access only #` | A warning shown before login. The `#` signs mark where the text starts and ends. | [lesson 1, Mission 4](../lesson-01/) |
| `line console 0` | Enters the settings of the console port (the blue cable / the CLI tab). | [lesson 1, Mission 4](../lesson-01/) |
| `password Osem#Con` | The password of this line. | [lesson 1, Mission 4](../lesson-01/) |
| `login` | "Ask for the password". Without it the password exists and nobody is asked. | [lesson 1, Mission 4](../lesson-01/) |
| `logging synchronous` | System messages don't cut the line you are writing. | [lesson 1, Mission 4](../lesson-01/) |
| `exec-timeout 10 0` | Logs out after this many minutes and seconds without typing (10 minutes, 0 seconds). | [lesson 1, Mission 4](../lesson-01/) |
| `end` | Straight out to `#`, from any depth. | [lesson 1, Mission 4](../lesson-01/) |
| `copy running-config startup-config` | Saves what is in memory to the startup file. Without it a power failure erases the work. | [lesson 1, Mission 4](../lesson-01/) |
| `show run` | All the configuration that works now. | [lesson 1, Mission 4](../lesson-01/) |
| `show interfaces status` | Each port: connected or not, and its VLAN. | [lesson 1, Mission 4](../lesson-01/) |
| `arp -a` | The PC's list: which IP belongs to which MAC. | [lesson 1, Mission 6](../lesson-01/) |
| `show mac address-table` | What the switch learned: which MAC is on which port. | [lesson 1, Mission 6](../lesson-01/) |
| `ping 192.168.20.22` | Sends 4 test messages. Replies = the road works in **both** directions. | [lesson 1, Mission 6](../lesson-01/) |

## Chapter 2 — walls inside the switches
| Command | What it does, and why | First time |
|---|---|---|
| `show vlan brief` | The list of VLANs and the ports in each one. Trunks do not appear. | [lesson 2, Mission 1](../lesson-02/) |
| `vtp domain SAKHNIN` | The name of the VTP group. Only switches with **exactly** the same name share the VLAN list (capital letters matter). | [lesson 2, Mission 1](../lesson-02/) |
| `vtp password Osem#Vtp` | A shared secret, so a strange switch can't change our VLANs. It must be the same on all. | [lesson 2, Mission 1](../lesson-02/) |
| `vtp mode server` | This switch may create VLANs and teach the others. | [lesson 2, Mission 1](../lesson-02/) |
| `vlan 10` | Creates the VLAN with this number. The number is what travels in the tag. | [lesson 2, Mission 1](../lesson-02/) |
| `name MANAGEMENT` | A name for the VLAN, for people only. The switch works with the number. | [lesson 2, Mission 1](../lesson-02/) |
| `vtp mode client` | This switch only receives the list. `vlan 10` on it is refused. | [lesson 2, Mission 2](../lesson-02/) |
| `show interfaces trunk` | Which ports are trunks, and which VLANs pass. | [lesson 2, Mission 3](../lesson-02/) |
| `shutdown` | Turns the port off. | [lesson 2, Mission 3](../lesson-02/) |
| `interface range fa0/1 , fa0/3 - 8` | Configures several ports together. Note the spaces around the comma and the dash. | [lesson 2, Mission 3](../lesson-02/) |
| `switchport mode trunk` | The port carries **all** the VLANs, with a tag. For switch ↔ switch and switch ↔ router. | [lesson 2, Mission 3](../lesson-02/) |
| `exit` | One level back, for example from a port to general configuration. The next line belongs to the upper level. | [lesson 2, Mission 3](../lesson-02/) |
| `interface fa0/2` | Chooses the port. Everything below applies to it, until `exit`. | [lesson 2, Mission 3](../lesson-02/) |
| `switchport mode access` | The port belongs to one VLAN, for one end device. | [lesson 2, Mission 3](../lesson-02/) |
| `switchport access vlan 50` | Which VLAN the port belongs to. | [lesson 2, Mission 3](../lesson-02/) |
| `switchport port-security` | Turns port security **on**. Without this line the three others do nothing. | [lesson 2, Mission 3](../lesson-02/) |
| `switchport port-security maximum 2` | How many MAC addresses may be on this port. | [lesson 2, Mission 3](../lesson-02/) |
| `switchport port-security mac-address sticky` | Learns the MAC that is connected now and remembers it. | [lesson 2, Mission 3](../lesson-02/) |
| `switchport port-security violation shutdown` | What to do with a stranger: shut the port. | [lesson 2, Mission 3](../lesson-02/) |
| `spanning-tree portfast` | The port becomes green at once. Only for end devices: a PC can't create a loop. | [lesson 2, Mission 4](../lesson-02/) |
| `spanning-tree bpduguard enable` | If a switch is connected to this port, shut it. Protects PortFast. | [lesson 2, Mission 4](../lesson-02/) |
| `switchport trunk encapsulation dot1q` | A 3560 knows two kinds of tags and must be told which. A 2960 knows only dot1q. | [lesson 2, Mission 4](../lesson-02/) |
| `vtp mode transparent` | This switch keeps its own list and doesn't learn from VTP. Good for a switch that needs one VLAN only. | [lesson 2, Mission 4](../lesson-02/) |
| `show vtp status` | Domain, mode and revision number. All must have the same domain. | [lesson 2, Mission 4](../lesson-02/) |
| `show spanning-tree vlan 20` | Who the root is, and the state of each port: FWD works, BLK waits. | [lesson 2, Mission 5](../lesson-02/) |
| `spanning-tree mode pvst` | PVST: a separate tree for each VLAN. | [lesson 2, Mission 5](../lesson-02/) |
| `spanning-tree vlan 1,10,20,30,40,50 root primary` | Makes this switch the root of these VLANs (priority 24576; lower wins). | [lesson 2, Mission 5](../lesson-02/) |
| `spanning-tree guard root` | Root Guard: a switch connected here is **not** allowed to become root. | [lesson 2, Mission 5](../lesson-02/) |
| `spanning-tree vlan 1,10,20,30,40,50 root secondary` | The deputy (priority 28672): root only if the first falls. | [lesson 2, Mission 5](../lesson-02/) |
| `show port-security interface fa0/2` | State of the port, how many violations, and the last MAC. | [lesson 2, Mission 7](../lesson-02/) |
| `no shutdown` | Turns the port on. Router ports are off by default. | [lesson 2, Mission 7](../lesson-02/) |

## Chapter 3 — a full branch
| Command | What it does, and why | First time |
|---|---|---|
| `interface g0/0.10` | A **sub-interface**: a virtual door on the same physical port, one for each VLAN. | [lesson 3, Mission 1](../lesson-03/) |
| `encapsulation dot1Q 10` | "This sub-interface is VLAN number…". Must come **before** the IP address. | [lesson 3, Mission 1](../lesson-03/) |
| `ip address 192.168.10.254 255.255.255.0` | The address and the mask of this interface. In a LAN it is the address of the gateway. | [lesson 3, Mission 1](../lesson-03/) |
| `standby 10 ip 192.168.10.252` | HSRP: the shared virtual address (VIP) that both routers answer for. This is the gateway of the PCs. | [lesson 3, Mission 1](../lesson-03/) |
| `standby 10 priority 110` | The higher number is Active. Main 110, backup 105. | [lesson 3, Mission 1](../lesson-03/) |
| `standby 10 preempt` | A router with a higher priority takes the role back when it returns. | [lesson 3, Mission 1](../lesson-03/) |
| `ip helper-address 192.168.50.1` | The DHCP relay: turns the PC's broadcast into a direct message to this server. | [lesson 3, Mission 1](../lesson-03/) |
| `show standby brief` | Who is Active and who is Standby in each group, and the priority. | [lesson 3, Mission 2](../lesson-03/) |
| `ipconfig /renew` | The PC asks DHCP for an address again. | [lesson 3, Mission 4](../lesson-03/) |
| `tracert 192.168.40.21` | Shows every router on the way. | [lesson 3, Mission 5](../lesson-03/) |
| `ipconfig` | Shows the address, mask and gateway of the PC. | [lesson 3, Mission 5](../lesson-03/) |
| `show ip dhcp snooping` | Is snooping on, in which VLANs, and which ports are trusted. | [lesson 3, Mission 6](../lesson-03/) |
| `ip dhcp snooping` | Turns DHCP snooping on in the whole switch. | [lesson 3, Mission 6](../lesson-03/) |
| `ip dhcp snooping vlan 10,20,30,40` | In which VLANs to check DHCP. | [lesson 3, Mission 6](../lesson-03/) |
| `no ip dhcp snooping information option` | Doesn't add "option 82" to requests; without this line the relay drops them. | [lesson 3, Mission 6](../lesson-03/) |
| `ip dhcp snooping trust` | DHCP **answers** are accepted from this port: it leads to our server. All the rest are not trusted. | [lesson 3, Mission 6](../lesson-03/) |
| `show ip dhcp snooping binding` | The table of who received which address, on which port. | [lesson 3, Mission 6](../lesson-03/) |
| `ipconfig /release` | The PC gives its address back. | [lesson 3, Mission 6](../lesson-03/) |
| `ip dhcp excluded-address 192.168.20.1 192.168.20.199` | Addresses that this server will not give. | [lesson 3, Mission 6](../lesson-03/) |
| `ip dhcp pool ROGUE` | Creates a list of addresses to give. Here: the **attacker's** list. | [lesson 3, Mission 6](../lesson-03/) |
| `network 192.168.20.0 255.255.255.0` | The network from which this DHCP pool gives addresses (with its mask). | [lesson 3, Mission 6](../lesson-03/) |
| `default-router 192.168.20.99` | The gateway that the clients receive (here a false one). | [lesson 3, Mission 6](../lesson-03/) |
| `dns-server 6.6.6.6` | The DNS that the clients receive (here a false one). | [lesson 3, Mission 6](../lesson-03/) |
| `no ip dhcp snooping trust` | Takes the trust back: DHCP answers from here are dropped again. | [lesson 3, Mission 6](../lesson-03/) |
| `no ip dhcp pool ROGUE` | Deletes the attacker's pool (cleaning up). | [lesson 3, Mission 6](../lesson-03/) |
| `no ip dhcp excluded-address 192.168.20.1 192.168.20.199` | Removes the excluded addresses (cleaning up after the attack). | [lesson 3, Mission 6](../lesson-03/) |

## Chapter 4 — three branches and the internet
| Command | What it does, and why | First time |
|---|---|---|
| `show ip interface brief` | All the ports, the addresses, and whether they are up. | [lesson 4, Mission 1](../lesson-04/) |
| `clock rate 2000000` | The DCE side sets the speed of the serial line (bits per second), and the other side follows. | [lesson 4, Mission 1](../lesson-04/) |
| `standby 60 track s0/0/0` | Watch this line: if it falls, lower my priority by 10 (110 → 100 < 105), and the backup takes over. | [lesson 4, Mission 1](../lesson-04/) |
| `router eigrp 54` | Starts EIGRP. The number (AS) must be **the same** on all the routers of the branch. | [lesson 4, Mission 1](../lesson-04/) |
| `network 192.168.60.0` | EIGRP works on the interface of this network, and announces it. | [lesson 4, Mission 1](../lesson-04/) |
| `no auto-summary` | Announces the exact networks, and not one rounded big network. | [lesson 4, Mission 1](../lesson-04/) |
| `passive-interface g0/0.60` | The network is announced, but no routing messages are sent to the PCs: nobody there can pretend to be a router. | [lesson 4, Mission 1](../lesson-04/) |
| `ip route 0.0.0.0 0.0.0.0 200.110.0.1` | The default route: "everything I don't know, send there". | [lesson 4, Mission 1](../lesson-04/) |
| `router bgp 411` | Starts BGP with **our** AS number. | [lesson 4, Mission 1](../lesson-04/) |
| `neighbor 200.200.0.2 remote-as 311` | The address of the BGP neighbor and **its** AS number. Both sides must write it. | [lesson 4, Mission 1](../lesson-04/) |
| `network 200.200.0.0 mask 255.255.255.0` | Announces our public network to the provider. The mask must be exactly as in the routing table. | [lesson 4, Mission 1](../lesson-04/) |
| `ip nat inside source static 192.168.63.1 200.200.0.10` | Static NAT: one inside address ↔ one public address, forever. That's how the world reaches the server. | [lesson 4, Mission 1](../lesson-04/) |
| `access-list 1 permit 192.168.60.0 0.0.3.255` | A standard list: the addresses that will be translated (`0.0.3.255` = four networks together). | [lesson 4, Mission 1](../lesson-04/) |
| `ip nat inside source list 1 interface s0/0/0 overload` | PAT: everyone in the list goes out with the **one** address of this port; `overload` tells them apart by port numbers. | [lesson 4, Mission 1](../lesson-04/) |
| `ip nat inside` | This port is on the **home** side. | [lesson 4, Mission 1](../lesson-04/) |
| `ip nat outside` | This port is on the **internet** side. | [lesson 4, Mission 1](../lesson-04/) |
| `ip nat pool EILAT-POOL 200.150.0.20 200.150.0.30 netmask 255.255.255.0` | The pool: from the first to the last public address. | [lesson 4, Mission 1](../lesson-04/) |
| `ip access-list extended NAT-E` | Opens a list by name or number, so lines can be added or inserted. | [lesson 4, Mission 1](../lesson-04/) |
| `permit ip 192.168.80.0 0.0.3.255 any` | Who is translated: from these networks to anywhere. | [lesson 4, Mission 1](../lesson-04/) |
| `ip nat inside source list NAT-E pool EILAT-POOL` | Dynamic NAT: everyone in the list gets a free address from the pool. | [lesson 4, Mission 1](../lesson-04/) |
| `show ip route` | The routing table. The letter shows who taught it: `C` connected, `O` OSPF, `D` EIGRP, `B` BGP, `S` static. | [lesson 4, Mission 3](../lesson-04/) |
| `router ospf 52` | Starts OSPF. The number is local to this router. | [lesson 4, Mission 3](../lesson-04/) |
| `router-id 1.1.1.1` | The name of the router in OSPF, fixed by us. | [lesson 4, Mission 3](../lesson-04/) |
| `network 192.168.10.0 0.0.0.255 area 0` | Announces this network in OSPF. The mask is **wildcard** (0.0.0.255 = /24), and area 0 is the backbone. | [lesson 4, Mission 3](../lesson-04/) |
| `show ip ospf neighbor` | The OSPF neighbors. FULL = they exchanged everything. | [lesson 4, Mission 3](../lesson-04/) |
| `show ip eigrp neighbors` | The EIGRP neighbors. An empty list = a wrong AS number or network. | [lesson 4, Mission 3](../lesson-04/) |
| `show ip bgp summary` | The list of BGP neighbors and the messages received (MsgRcvd). In Packet Tracer a neighbor can look fine even with a wrong AS, so check the routes too. | [lesson 4, Mission 4](../lesson-04/) |
| `show ip nat translations` | Who was translated to which address. | [lesson 4, Mission 5](../lesson-04/) |

## Chapter 5 — a protected company
| Command | What it does, and why | First time |
|---|---|---|
| `access-list 110 permit udp any any eq bootps` | First of all allow DHCP, otherwise the PCs of this VLAN lose their address. | [lesson 5, Mission 2](../lesson-05/) |
| `access-list 110 deny ip 192.168.62.0 0.0.0.255 192.168.60.0 0.0.0.255` | Blocks everything from the first network to the second one. | [lesson 5, Mission 2](../lesson-05/) |
| `access-list 110 permit tcp 192.168.62.0 0.0.0.255 host 192.168.63.1 eq www` | Allow the website of the server (TCP 80), and only from this network. | [lesson 5, Mission 2](../lesson-05/) |
| `access-list 110 permit udp 192.168.62.0 0.0.0.255 host 192.168.63.1 eq domain` | Allow DNS questions to the server (UDP 53). | [lesson 5, Mission 2](../lesson-05/) |
| `access-list 110 permit tcp 192.168.62.0 0.0.0.255 host 192.168.63.1 eq ftp` | Allow the FTP **control** connection (TCP 21). The data connection needs another line. | [lesson 5, Mission 2](../lesson-05/) |
| `access-list 110 permit ip 192.168.62.0 0.0.0.255 any` | Allows the rest. Without it the invisible `deny` at the end blocks everything. | [lesson 5, Mission 2](../lesson-05/) |
| `ip access-group 110 in` | Puts the list on the port; `in` = what enters the router from this network. A list that isn't on a port does nothing. | [lesson 5, Mission 2](../lesson-05/) |
| `show access-lists` | The lines of the list, and how many packets met each one. | [lesson 5, Mission 2](../lesson-05/) |
| `52 permit tcp 192.168.62.0 0.0.0.255 host 192.168.63.1 gt 1023` | Inserts a line **between** existing ones (52 is after 50): the FTP data ports, above 1023. | [lesson 5, Mission 2](../lesson-05/) |
| `license boot module c1900 technology-package securityk9` | Turns on the security package of the 1941. Without it there are no `crypto` commands. | [lesson 5, Mission 3](../lesson-05/) |
| `reload` | Restarts the router. The licence starts to work only then. | [lesson 5, Mission 3](../lesson-05/) |
| `show version \| include security` | Device details. The part after the bar shows only the lines with "security". | [lesson 5, Mission 3](../lesson-05/) |
| `crypto isakmp policy 10` | Phase 1: the rules of how the two routers identify themselves. Must match on both sides. | [lesson 5, Mission 3](../lesson-05/) |
| `encryption aes 256` | The encryption of phase 1: AES with a 256-bit key. | [lesson 5, Mission 3](../lesson-05/) |
| `hash sha` | SHA checks that nobody changed the message on the way. | [lesson 5, Mission 3](../lesson-05/) |
| `authentication pre-share` | The routers identify themselves with a shared key (written below). | [lesson 5, Mission 3](../lesson-05/) |
| `group 5` | The Diffie-Hellman group: how the routers agree on a secret over an open line. | [lesson 5, Mission 3](../lesson-05/) |
| `lifetime 86400` | After how many seconds the keys are replaced (86400 = one day). | [lesson 5, Mission 3](../lesson-05/) |
| `crypto isakmp key OsemVpn2026 address 200.150.0.1` | The shared key, and the public address **of the other side**. | [lesson 5, Mission 3](../lesson-05/) |
| `crypto ipsec transform-set VPN-SET esp-aes 256 esp-sha-hmac` | Phase 2: how the data itself is encrypted (AES 256) and checked (SHA). | [lesson 5, Mission 3](../lesson-05/) |
| `permit ip 192.168.0.0 0.0.63.255 192.168.80.0 0.0.3.255` | The traffic that enters the tunnel: from our networks to theirs. The other side writes the mirror image. | [lesson 5, Mission 3](../lesson-05/) |
| `crypto map VPN-MAP 10 ipsec-isakmp` | Creates the map that joins everything: who, how and what. | [lesson 5, Mission 3](../lesson-05/) |
| `set peer 200.150.0.1` | Who: the public address of the other side. | [lesson 5, Mission 3](../lesson-05/) |
| `set transform-set VPN-SET` | How: with which encryption. | [lesson 5, Mission 3](../lesson-05/) |
| `match address VPN-S-E` | What: the list that says which traffic enters the tunnel. | [lesson 5, Mission 3](../lesson-05/) |
| `crypto map VPN-MAP` | Puts the map on the port that **goes out** to the internet. Only now does the VPN work. | [lesson 5, Mission 3](../lesson-05/) |
| `show crypto isakmp sa` | Phase 1. `QM_IDLE` = good. | [lesson 5, Mission 3](../lesson-05/) |
| `5 deny ip 192.168.0.0 0.0.63.255 192.168.80.0 0.0.3.255` | Inserts a line at the **beginning** (5 is before 10): traffic for the tunnel is not translated by NAT. | [lesson 5, Mission 3](../lesson-05/) |
| `clear ip nat translation *` | Deletes old translations, so the new rule works immediately. | [lesson 5, Mission 3](../lesson-05/) |
| `show crypto ipsec sa` | Phase 2. The `encaps` and `decaps` counters must grow. | [lesson 5, Mission 3](../lesson-05/) |
| `no crypto map VPN-MAP` | Takes the map off the port: the tunnel is turned off. | [lesson 5, Mission 3](../lesson-05/) |
| `clear crypto isakmp` | Deletes the old tunnel, so it will be built again. | [lesson 5, Mission 3](../lesson-05/) |
| `telnet 192.168.10.254` | Remote management **without** encryption. Here it must fail. | [lesson 5, Mission 4](../lesson-05/) |
| `show clock` | The time of the device. A `*` at the start = not synchronized. | [lesson 5, Mission 4](../lesson-05/) |
| `ip domain-name osem.com` | A domain name. SSH needs it to build the name of the key. | [lesson 5, Mission 4](../lesson-05/) |
| `username admin privilege 15 secret Osem#Adm1` | A local user with full rights (15). The backup if the AAA server doesn't answer. | [lesson 5, Mission 4](../lesson-05/) |
| `crypto key generate rsa general-keys modulus 1024` | Creates the keys of SSH, 1024 bits long. SSH version 2 needs at least 768. | [lesson 5, Mission 4](../lesson-05/) |
| `ip ssh version 2` | Allows only version 2, which is the secure one. | [lesson 5, Mission 4](../lesson-05/) |
| `aaa new-model` | Turns AAA on. From now on logins work by lists. That's why a local user is created **before**. | [lesson 5, Mission 4](../lesson-05/) |
| `tacacs-server host 192.168.50.1 key Osem#Aaa` | The address of the TACACS+ server, and the key we share with it. | [lesson 5, Mission 4](../lesson-05/) |
| `aaa authentication login default group tacacs+ local` | **Who are you?** First ask the server; if it doesn't answer, use the local user. | [lesson 5, Mission 4](../lesson-05/) |
| `aaa authorization exec default group tacacs+ local` | **What may you do?** The server decides the rights after login. | [lesson 5, Mission 4](../lesson-05/) |
| `aaa authentication login CONSOLE local` | Another list, only for the console: a local user, so you're never locked out. | [lesson 5, Mission 4](../lesson-05/) |
| `login authentication CONSOLE` | This line uses the list called CONSOLE. | [lesson 5, Mission 4](../lesson-05/) |
| `access-list 10 permit 192.168.10.0 0.0.0.255` | A standard list (source only): VLAN 10, the managers. | [lesson 5, Mission 4](../lesson-05/) |
| `line vty 0 4` | Enters the settings of the **remote** lines. `0 4` = five sessions together, `0 15` = sixteen. | [lesson 5, Mission 4](../lesson-05/) |
| `transport input ssh` | Only SSH enters from remote. Telnet (readable) is closed. | [lesson 5, Mission 4](../lesson-05/) |
| `login authentication default` | This line uses the default list: first the server, then the local user. | [lesson 5, Mission 4](../lesson-05/) |
| `access-class 10 in` | Puts the list on the remote lines: only VLAN 10 may manage. | [lesson 5, Mission 4](../lesson-05/) |
| `ntp authentication-key 1 md5 Osem#Ntp` | The key with which the time server signs (number and text). | [lesson 5, Mission 4](../lesson-05/) |
| `ntp authenticate` | Demand a signature: don't believe just anybody about the time. | [lesson 5, Mission 4](../lesson-05/) |
| `ntp trusted-key 1` | Which key number we trust. | [lesson 5, Mission 4](../lesson-05/) |
| `ntp server 192.168.50.1 key 1` | From whom to take the time, and with which key. | [lesson 5, Mission 4](../lesson-05/) |
| `ntp update-calendar` | Also updates the hardware clock, so the time survives a restart. | [lesson 5, Mission 4](../lesson-05/) |
| `logging host 192.168.50.1` | Sends the messages of the device to this Syslog server. | [lesson 5, Mission 4](../lesson-05/) |
| `service timestamps log datetime msec` | Adds date, time and milliseconds to each message. | [lesson 5, Mission 4](../lesson-05/) |
| `ssh -l osemuser 192.168.10.254` | Encrypted remote management: `-l` and the user name, then the address of the device. | [lesson 5, Mission 4](../lesson-05/) |
| `show users` | Who is logged in now, and from where. | [lesson 5, Mission 4](../lesson-05/) |
| `show ntp status` | Is the clock synchronized with the server. | [lesson 5, Mission 4](../lesson-05/) |
| `radius-server host 192.168.63.1 key Osem#Aaa` | The address of the RADIUS server, and the key we share with it. | [lesson 5, Mission 4](../lesson-05/) |
| `login local` | The line asks for a **user and password** from the list inside the device. | [lesson 5, Mission 4](../lesson-05/) |
| `interface vlan 10` | A virtual interface of the VLAN: the address **of the device itself** in that VLAN (for management, or on the ASA for a side). | [lesson 5, Mission 4](../lesson-05/) |
| `ip default-gateway 192.168.10.252` | The gateway **of the switch itself**, so its answers can leave the VLAN. | [lesson 5, Mission 4](../lesson-05/) |
| `nameif inside` | Gives the side a name. The ASA works with names. | [lesson 5, Mission 5](../lesson-05/) |
| `security-level 100` | How much we trust the side: 100 = fully, 0 = not at all. From high to low is allowed, the opposite is blocked. | [lesson 5, Mission 5](../lesson-05/) |
| `no ip address` | Deletes the old address, so we can give a new one. | [lesson 5, Mission 5](../lesson-05/) |
| `route outside 0.0.0.0 0.0.0.0 192.168.10.252` | The default route of the ASA: out, through this gateway. | [lesson 5, Mission 5](../lesson-05/) |
| `object network INSIDE-NET` | Gives a network a name, so we can write rules about it. | [lesson 5, Mission 5](../lesson-05/) |
| `subnet 192.168.1.0 255.255.255.0` | Which network the object is. | [lesson 5, Mission 5](../lesson-05/) |
| `nat (inside,outside) dynamic interface` | PAT: the lab goes out with the outside address of the ASA. | [lesson 5, Mission 5](../lesson-05/) |
| `no dhcpd auto_config outside` | Cancels an automatic setting that comes from the factory and disturbs us. | [lesson 5, Mission 5](../lesson-05/) |
| `dhcpd address 192.168.1.10-192.168.1.30 inside` | The addresses that the ASA gives to the lab. | [lesson 5, Mission 5](../lesson-05/) |
| `dhcpd dns 192.168.50.1` | The DNS that the lab receives. | [lesson 5, Mission 5](../lesson-05/) |
| `dhcpd enable inside` | Turns on the ASA's DHCP server, on the inside. | [lesson 5, Mission 5](../lesson-05/) |
| `class-map inspection_default` | Chooses traffic: "what do we look at?" | [lesson 5, Mission 5](../lesson-05/) |
| `match default-inspection-traffic` | The common protocols, by their default ports. | [lesson 5, Mission 5](../lesson-05/) |
| `policy-map global_policy` | The policy: "what do we do with it?" | [lesson 5, Mission 5](../lesson-05/) |
| `class inspection_default` | For the traffic chosen above: | [lesson 5, Mission 5](../lesson-05/) |
| `inspect icmp` | Remember this conversation (icmp / http / dns), so that the **answers** can return. | [lesson 5, Mission 5](../lesson-05/) |
| `service-policy global_policy global` | Turns the policy on, for all the ports. | [lesson 5, Mission 5](../lesson-05/) |
| `write memory` | The same save, as it is written on the ASA. | [lesson 5, Mission 5](../lesson-05/) |
| `show switch vlan` | On the ASA: which ports are in which VLAN. | [lesson 5, Mission 5](../lesson-05/) |
| `show interface ip brief` | On the ASA: the ports, addresses and state. | [lesson 5, Mission 5](../lesson-05/) |
| `show xlate` | On the ASA: the active translations. | [lesson 5, Mission 5](../lesson-05/) |

⬅ [Back to the project](../README.md)
