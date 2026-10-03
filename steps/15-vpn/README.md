# Step 15 — VPN: site-to-site (GRE over IPsec) + remote access

## Goal
1. **Haifa ↔ Jerusalem** talk with their **private** addresses through an encrypted tunnel
   across the internet. Jerusalem users reach Haifa's DMZ servers (DNS, mail, web, NTP,
   FTP, AAA) privately.
2. A **teleworker** at home (on ISP-C) connects with the PC's VPN client to Haifa and works
   as if they were in the office.

Ministry requirements: **2, 21 (VPN access to DMZ servers), 23.4, 20.8**

## Concepts
- **IPsec** encrypts and authenticates IP packets. Two phases:
  - **Phase 1 (ISAKMP/IKE)** — the two routers authenticate each other (pre-shared key) and
    build a secure management channel. `crypto isakmp policy`.
  - **Phase 2 (IPsec SA)** — the actual data encryption (`transform-set`: ESP + AES + SHA).
  - A **crypto map** on the WAN interface says *which* traffic to encrypt (ACL) and *to
    whom* (peer).
- **GRE** — a plain tunnel: a virtual point-to-point interface (`Tunnel0`) between two public
  IPs. No encryption on its own.
- **GRE over IPsec** — we route branch traffic **into Tunnel0**, and IPsec encrypts the GRE
  packets. Why not plain IPsec? Because of **NAT**:
  > Haifa's web server has a static NAT (`10.10.200.13 ↔ 200.1.10.3`). With plain IPsec, its
  > reply to a Jerusalem PC would be NAT-ed *before* encryption → wrong source → broken.
  > With GRE, the reply leaves through `Tunnel0`, which is **not** an `ip nat outside`
  > interface → no NAT → private addresses preserved. Problem solved without route-map NAT.
- **Remote-access VPN (Easy VPN)** — the router is a VPN *server*. The client logs in with a
  **group** name + key and a **username** + password (XAUTH), receives an address from a pool
  (`172.16.250.0/24`) and a tunnel to HQ.

## Prerequisite — security licence on both edge routers
```
R-HFA-EDGE(config)# license boot module c2900 technology-package securityk9
   (accept: yes)
R-HFA-EDGE(config)# end
R-HFA-EDGE# wr
R-HFA-EDGE# reload
```
Same on **R-JLM-EDGE**. After the reload `show version` lists `securityk9`.

## Configure

### Part A — site-to-site on R-HFA-EDGE
```
configure terminal
! ---------- GRE tunnel ----------
interface tunnel 0
 description GRE to R-JLM-EDGE (encrypted by IPsec)
 ip address 172.16.12.1 255.255.255.252
 tunnel source gi0/1
 tunnel destination 200.2.1.2
 exit
! Jerusalem's private block goes into the tunnel (static - req. 22.17)
ip route 10.20.0.0 255.255.0.0 172.16.12.2

! ---------- IPsec phase 1 ----------
crypto isakmp policy 10
 encryption aes 256
 hash sha
 authentication pre-share
 group 5
 lifetime 86400
 exit
! no-xauth: this peer is a router, don't ask it for a username (the RA clients will be)
crypto isakmp key Intel#S2S-Key address 200.2.1.2 no-xauth

! ---------- IPsec phase 2 ----------
crypto ipsec transform-set S2S-SET esp-aes 256 esp-sha-hmac
ip access-list extended S2S-GRE
 permit gre host 200.1.1.2 host 200.2.1.2
 exit
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 200.2.1.2
 set transform-set S2S-SET
 match address S2S-GRE
 exit
interface gi0/1
 crypto map VPN-MAP
end
wr
```

### Part B — site-to-site on R-JLM-EDGE (the mirror)
```
configure terminal
interface tunnel 0
 description GRE to R-HFA-EDGE (encrypted by IPsec)
 ip address 172.16.12.2 255.255.255.252
 tunnel source gi0/1
 tunnel destination 200.1.1.2
 exit
ip route 10.10.0.0 255.255.0.0 172.16.12.1
! (EIGRP "redistribute static" from step 13 now also gives this route to CC1/CC2)

crypto isakmp policy 10
 encryption aes 256
 hash sha
 authentication pre-share
 group 5
 lifetime 86400
 exit
crypto isakmp key Intel#S2S-Key address 200.1.1.2 no-xauth
crypto ipsec transform-set S2S-SET esp-aes 256 esp-sha-hmac
ip access-list extended S2S-GRE
 permit gre host 200.2.1.2 host 200.1.1.2
 exit
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 200.1.1.2
 set transform-set S2S-SET
 match address S2S-GRE
 exit
interface gi0/1
 crypto map VPN-MAP
end
wr
```
(The `WAN-IN` ACL from step 13 already permits ISAKMP, ESP and GRE from Haifa.)

### Part C — let Jerusalem through the ASA
```
ASA-HFA-FW1(config)# access-list OUTSIDE_IN extended permit ip 10.20.0.0 255.255.0.0 10.10.0.0 255.255.0.0
ASA-HFA-FW1(config)# access-list OUTSIDE_IN extended permit ip 10.20.0.0 255.255.0.0 10.10.200.0 255.255.255.0
ASA-HFA-FW1(config)# write memory
```
(`10.10.200.0` is inside `10.10.0.0/16`, so the first line already covers the DMZ — the
second line is there to make the intent obvious in the book.)

### Part D — remote-access VPN server on R-HFA-EDGE
```
configure terminal
aaa new-model
aaa authentication login default group tacacs+ local
aaa authentication login VPN-AUTH local
aaa authorization network VPN-GROUP local
username remote1 secret Intel#Remote1

ip local pool RA-POOL 172.16.250.10 172.16.250.100

crypto isakmp policy 20
 encryption aes 256
 hash sha
 authentication pre-share
 group 2
 exit
crypto isakmp client configuration group HQ-RA
 key Intel#RA-Group
 dns 10.10.200.11
 domain intel.lab
 pool RA-POOL
 exit
crypto ipsec transform-set RA-SET esp-aes 256 esp-sha-hmac
crypto dynamic-map RA-DYN 10
 set transform-set RA-SET
 reverse-route
 exit
crypto map VPN-MAP client authentication list VPN-AUTH
crypto map VPN-MAP isakmp authorization list VPN-GROUP
crypto map VPN-MAP client configuration address respond
! dynamic entries always get the highest sequence number
crypto map VPN-MAP 65535 ipsec-isakmp dynamic RA-DYN
end
wr
```
And on the ASA — remote users may reach HQ:
```
access-list OUTSIDE_IN extended permit ip 172.16.250.0 255.255.255.0 10.10.0.0 255.255.0.0
```
(The PAT ACL `NAT-HFA` from step 12 already excludes `172.16.250.0/24`.)

### Part E — the teleworker
PC-HOME-01 → *Desktop → VPN*:

| Field | Value |
|---|---|
| GroupName | `HQ-RA` |
| Group Key | `Intel#RA-Group` |
| Host IP (server) | `200.1.1.2` |
| Username | `remote1` |
| Password | `Intel#Remote1` |

→ *Connect* → "VPN is connected". `ipconfig` shows a tunnel address `172.16.250.x`.

## Verify

```
R-HFA-EDGE# show crypto isakmp sa
dst          src          state      ...
200.2.1.2    200.1.1.2    QM_IDLE        <- phase 1 up with Jerusalem
200.1.1.2    200.3.50.10  QM_IDLE        <- phase 1 up with the teleworker
R-HFA-EDGE# show crypto ipsec sa
   #pkts encaps: 57, #pkts encrypt: 57    <- grows while you ping = encrypted
   #pkts decaps: 57, #pkts decrypt: 57
R-HFA-EDGE# show interfaces tunnel 0      <- up/up
R-HFA-EDGE# show ip route static          <- 10.20.0.0/16 via 172.16.12.2
```
(No ISAKMP SA yet? Send some traffic first — the tunnel only comes up when "interesting"
traffic arrives.)

| From | Test | Expected |
|---|---|---|
| PC-JLM-SAL-01 | `ping 10.10.16.1` | ✅ private Haifa address — through the VPN |
| PC-JLM-SAL-01 | `tracert 10.10.200.13` | 10.20.10.2 → 10.20.4.1 → **172.16.12.1** → … (the tunnel!) |
| PC-JLM-SAL-01 | browser `http://www.intel.lab` | ✅ — DNS 10.10.200.11 now reachable too |
| PC-JLM-SAL-01 | email / FTP to `mail.intel.lab` / `ftp.intel.lab` | ✅ (req. 21) |
| SWL3-JLM-CC1 | `show ntp status` | synchronized to 10.10.200.14 (req. 20.8) |
| PC-HFA-IT-01 | `ssh -l admin 10.20.0.11` | ✅ HQ IT manages Jerusalem over the VPN |
| PC-HOME-01 (VPN up) | `ping 10.10.1.10`, browse `www.intel.lab` | ✅ |
| PC-HOME-01 (VPN down) | `ping 10.10.1.10` | ❌ |

**See the encryption**: Simulation mode, ping from a JLM PC to a Haifa PC, click the packet
on the ISP link → outer header `200.2.1.2 → 200.1.1.2`, protocol **ESP** — the private
addresses inside are invisible to the ISPs.

## Checkpoint
`my-work/step15.pkt` + commit.

## Common mistakes
- Phase 1 never comes up → ISAKMP policy mismatch (encryption/hash/group must match on both
  peers) or different pre-shared keys.
- Tunnel up, pings fail → missing static route into the tunnel, or the ASA ACL line for
  10.20.0.0/16.
- The S2S peer suddenly asks for XAUTH → `no-xauth` missing on `crypto isakmp key`.
- `crypto` commands don't exist → securityk9 licence not active (reload!).

## Challenge
Kiryat Gat has no VPN. Without adding one, explain which of the DMZ services KG **can**
still use and how (look at what we published with static NAT and what KG has).

<details><summary>Answer</summary>

Only what is published to the internet **and** reachable over IPv4 from KG:
- **NTP** — DIST1 has an IPv4 path (legacy transit + PAT) and syncs to the public static
  NAT `200.1.10.4` → req. 20.8 satisfied.
- **Web** — only the legacy IPv4 VLAN 199 devices can browse `200.1.10.3`; IPv6 users can't
  (no IPv4 stack — step 14 challenge).
- Mail/DNS/FTP/AAA — not published, so no. That's why KG has its **own** DNS, mail, syslog
  and AAA servers. Write this trade-off in the book; adding a third tunnel (needs dual-stack
  or an IPv6-capable VPN) would be the real fix.
</details>
