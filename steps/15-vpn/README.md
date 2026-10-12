# Step 15 — VPN: site-to-site (GRE over IPsec) + remote access

> ⏱ 2–3 lessons · 🎯 ministry requirements 2, 21, 23.4, 20.8 · 📁 start from: `my-work/step14.pkt` · 💾 save as: `my-work/step15.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 14 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## What you will have at the end
1. **Haifa ↔ Jerusalem** talk with their **private** addresses through an **encrypted tunnel**
   across the internet. Jerusalem users reach Haifa's DMZ servers (DNS, email, web, NTP, FTP, AAA).
2. A **teleworker** at home connects with the PC's VPN client to Haifa and works as if in the office.

```
   Haifa LAN 10.10.0.0/16                                           Jerusalem LAN 10.20.0.0/16
        │                                                                   │
   R-HFA-EDGE ══════ Tunnel0 172.16.12.1 ⇄ 172.16.12.2 (GRE, encrypted by IPsec) ══════ R-JLM-EDGE
   200.1.1.2            ... across ISP-A ⇄ ISP-B (the internet sees only ESP) ...        200.2.1.2
        ▲
        ╚═══════ remote-access VPN ═══════ PC-HOME-01 (200.3.50.10, gets 172.16.250.x)
```

---

## Part 1 — Understand it first (simple English)

### 1.1 Why a VPN?
Haifa and Jerusalem are connected through the **internet**:
- private addresses (10.x) **can't** travel on the internet (step 12);
- anything sent there can be **read** by others on the way.

A **VPN (Virtual Private Network)** solves both: it builds a **tunnel** — the private packet is put
**inside** a public packet and **encrypted**. Comparison: putting a letter in a locked armoured box
and sending the box by normal post. The post office sees only the box's outside address.

### 1.2 IPsec — the lock
**IPsec** encrypts and authenticates the packets. It starts in two phases:

| Phase | What happens | Commands |
|---|---|---|
| **Phase 1 (ISAKMP / IKE)** | The two routers check each other's identity (pre-shared key) and agree how to talk safely | `crypto isakmp policy`, `crypto isakmp key` |
| **Phase 2 (IPsec SA)** | They agree how to encrypt the **data** (ESP + AES + SHA) | `crypto ipsec transform-set` |

A **crypto map** on the internet interface says **which** traffic to encrypt and **to whom**.

### 1.3 GRE — the tunnel
**GRE** creates a virtual interface (`Tunnel0`) between two public addresses, like a private cable
through the internet. GRE alone doesn't encrypt — so we **encrypt the GRE packets with IPsec**:
**GRE over IPsec**.

### 1.4 Why GRE + IPsec and not IPsec alone? (an excellent exam question)
Because of **static NAT** in Haifa (step 12). The web server is always translated
10.10.200.13 ↔ 200.1.10.3. With plain IPsec, the web server's reply to a Jerusalem PC would be
**translated before encryption** — wrong address — broken.
With GRE, the reply leaves through `Tunnel0`, which is **not** a NAT interface → it isn't translated →
private addresses stay private end to end. Problem solved.

### 1.5 Remote-access VPN
A teleworker's PC is not a router. The edge router acts as a **VPN server** ("Easy VPN"):
1. the PC's VPN client logs in with a **group** name + group key, and a **username** + password;
2. the router gives the PC an address from a pool (`172.16.250.10 – .100`) and the DNS server;
3. the PC's traffic to HQ goes through the encrypted tunnel (req. 23.4).

### 1.6 The security licence
Encryption commands on a Cisco 2911 need the **securityk9** licence. In PT you activate it with one
command and a reboot.

---

## Part 2 — Prepare
**File → Open** → `my-work/step14.pkt` → **File → Save As** → `my-work/step15.pkt`.

## Part 3 — Build
Nothing new to place — the edge routers, the ASA and PC-HOME-01 already exist.

---

## Part 4 — Configure

### 4.1 Activate the security licence — on R-HFA-EDGE **and** R-JLM-EDGE
```
enable
configure terminal
license boot module c2900 technology-package securityk9
```
The router shows a licence text and asks `ACCEPT? [yes/no]:` → type **`yes`**.
```
end
copy running-config startup-config
reload
```
Press **Enter** to confirm the reload. Wait for the router to boot (Fast Forward Time), log in again.
Check: `show version` → in the licence table `securityk9` is **Permanent** / **Active**.

### 4.2 R-HFA-EDGE — site-to-site (Haifa side)
```
enable
configure terminal
! ===== 1. the GRE tunnel =====
interface tunnel 0
 description GRE to R-JLM-EDGE (encrypted by IPsec)
 ip address 172.16.12.1 255.255.255.252
 tunnel source gi0/1
 tunnel destination 200.2.1.2
 exit
! Jerusalem's private block goes into the tunnel (a static route - req. 22.17)
ip route 10.20.0.0 255.255.0.0 172.16.12.2

! ===== 2. IPsec phase 1 =====
crypto isakmp policy 10
 encryption aes 256
 hash sha
 authentication pre-share
 group 5
 lifetime 86400
 exit
! the shared secret with Jerusalem; no-xauth = it's a router, don't ask it for a username
crypto isakmp key Intel#S2S-Key address 200.2.1.2 no-xauth

! ===== 3. IPsec phase 2 =====
crypto ipsec transform-set S2S-SET esp-aes 256 esp-sha-hmac
ip access-list extended S2S-GRE
 remark --- encrypt the GRE packets between the two edges ---
 permit gre host 200.1.1.2 host 200.2.1.2
 exit
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 200.2.1.2
 set transform-set S2S-SET
 match address S2S-GRE
 exit
! ===== 4. apply on the internet interface =====
interface gi0/1
 crypto map VPN-MAP
 end
copy running-config startup-config
```
You see `%CRYPTO-6-ISAKMP_ON_OFF: ISAKMP is ON` — good.

### 4.3 R-JLM-EDGE — site-to-site (the mirror image)
```
enable
configure terminal
interface tunnel 0
 description GRE to R-HFA-EDGE (encrypted by IPsec)
 ip address 172.16.12.2 255.255.255.252
 tunnel source gi0/1
 tunnel destination 200.1.1.2
 exit
ip route 10.10.0.0 255.255.0.0 172.16.12.1

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
copy running-config startup-config
```
> The `redistribute static` from step 13 now also gives the route `10.10.0.0/16` to CC1/CC2 — so all
> of Jerusalem knows "Haifa is through the edge router". The CBAC ACL `WAN-IN` already lets the VPN in.

### 4.4 ASA — let Jerusalem in
```
enable
configure terminal
access-list OUTSIDE_IN extended permit ip 10.20.0.0 255.255.0.0 10.10.0.0 255.255.0.0
end
write memory
```
(10.10.0.0/16 includes the DMZ 10.10.200.0/24 — so Jerusalem reaches the inside **and** the DMZ servers.)

### 4.5 R-HFA-EDGE — remote-access VPN server
```
enable
configure terminal
! ===== who may log in =====
aaa new-model
aaa authentication login default group tacacs+ local
aaa authentication login VPN-AUTH local
aaa authorization network VPN-GROUP local
username remote1 secret Intel#Remote1
! ===== addresses for remote users =====
ip local pool RA-POOL 172.16.250.10 172.16.250.100
! ===== phase 1 for clients =====
crypto isakmp policy 20
 encryption aes 256
 hash sha
 authentication pre-share
 group 2
 exit
! ===== the group the PC logs into =====
crypto isakmp client configuration group HQ-RA
 key Intel#RA-Group
 dns 10.10.200.11
 domain intel.lab
 pool RA-POOL
 exit
! ===== phase 2 for clients =====
crypto ipsec transform-set RA-SET esp-aes 256 esp-sha-hmac
crypto dynamic-map RA-DYN 10
 set transform-set RA-SET
 reverse-route
 exit
! ===== add the clients to the same crypto map =====
crypto map VPN-MAP client authentication list VPN-AUTH
crypto map VPN-MAP isakmp authorization list VPN-GROUP
crypto map VPN-MAP client configuration address respond
crypto map VPN-MAP 65535 ipsec-isakmp dynamic RA-DYN
 end
copy running-config startup-config
```
And on the **ASA** — remote users may reach HQ:
```
configure terminal
access-list OUTSIDE_IN extended permit ip 172.16.250.0 255.255.255.0 10.10.0.0 255.255.0.0
end
write memory
```
(The PAT list `NAT-HFA` from step 12 already skips 172.16.250.0/24 — we prepared for this.)

### 4.6 The teleworker's VPN client (click by click)
1. Click **PC-HOME-01** → **Desktop** → **VPN**.
2. Fill in:

| Field | Value |
|---|---|
| GroupName | `HQ-RA` |
| Group Key | `Intel#RA-Group` |
| Host IP (Server IP) | `200.1.1.2` |
| Username | `remote1` |
| Password | `Intel#Remote1` |

3. Click **Connect** → after a moment: **"VPN is connected"**.
4. Close the VPN window → Command Prompt → `ipconfig` → you see an extra address `172.16.250.x`.

### 4.7 What the new commands mean
| Command | In simple English |
|---|---|
| `license boot module ... securityk9` | Unlock the encryption features (reboot needed) |
| `interface tunnel 0` + `tunnel source` / `tunnel destination` | A virtual cable from my public address to theirs |
| `crypto isakmp policy 10` (encryption/hash/authentication/group) | How we check each other in phase 1 (must match on both sides) |
| `crypto isakmp key KEY address PEER no-xauth` | The shared secret with that peer, no username asked |
| `crypto ipsec transform-set ... esp-aes 256 esp-sha-hmac` | How the data is encrypted (AES-256) and checked (SHA) |
| `crypto map VPN-MAP 10 ipsec-isakmp` / `set peer` / `match address` | "Encrypt traffic matching this ACL, toward this peer" |
| `interface gi0/1` → `crypto map VPN-MAP` | Switch the encryption on at the internet interface |
| `ip local pool RA-POOL ...` | Addresses to give remote users |
| `crypto isakmp client configuration group HQ-RA` | The group the VPN client logs into (key, DNS, pool) |
| `crypto dynamic-map` + `... 65535 ipsec-isakmp dynamic` | A template for clients whose address we don't know in advance |
| `reverse-route` | Add a route to each connected client automatically |

---

## Part 5 — Check your work

### 5.1 Make the tunnel come up
The tunnel starts only when traffic needs it. From **PC-JLM-SAL-01**: `ping 10.10.16.1` — the first
one or two pings may time out while the tunnel builds, then replies come.

### 5.2 On the routers
```
R-HFA-EDGE# show crypto isakmp sa
dst             src             state          conn-id status
200.2.1.2       200.1.1.2       QM_IDLE           1001 ACTIVE    <- phase 1 up with Jerusalem
200.1.1.2       200.3.50.10     QM_IDLE           1002 ACTIVE    <- the teleworker (when connected)
R-HFA-EDGE# show crypto ipsec sa
    #pkts encaps: 57, #pkts encrypt: 57, ...        <- growing while you ping = encrypted
    #pkts decaps: 57, #pkts decrypt: 57, ...
R-HFA-EDGE# show interfaces tunnel 0              <- Tunnel0 is up, line protocol is up
R-HFA-EDGE# show ip route static                  <- S 10.20.0.0/16 via 172.16.12.2
SWL3-JLM-CC1# show ip route | include 10.10.0.0  <- D EX 10.10.0.0/16 via 10.20.4.1
```

### 5.3 Tests
| From | Do | Expected |
|---|---|---|
| PC-JLM-SAL-01 | `ping 10.10.16.1` | ✅ a **private** Haifa address — through the VPN |
| PC-JLM-SAL-01 | `tracert 10.10.200.13` | 10.20.10.2 → 10.20.4.1 → **172.16.12.1** (the tunnel!) → … |
| PC-JLM-SAL-01 | Web Browser → `http://www.intel.lab` | ✅ DNS 10.10.200.11 is reachable now too |
| PC-JLM-SAL-01 | `nslookup ftp.intel.lab` | 10.10.200.15 (DMZ services via VPN — req. 21) |
| SWL3-JLM-CC1 | `show ntp status` | synchronized to 10.10.200.14 (req. 20.8) |
| PC-HFA-IT-01 | `ssh -l admin 10.20.0.11` | ✅ HQ's IT manages Jerusalem over the VPN |
| PC-HOME-01 (VPN connected) | `ping 10.10.1.10`, browse `www.intel.lab` | ✅ |
| PC-HOME-01 (VPN disconnected) | `ping 10.10.1.10` | ❌ |

### 5.4 See the encryption (Simulation mode)
1. **Simulation** → **Edit Filters** → **ICMP** (and ISAKMP/ESP if offered).
2. From PC-JLM-SAL-01 ping a Haifa PC. Follow the packet.
3. Click it on a cable **between the ISPs** → the outer header is `200.2.1.2 → 200.1.1.2`,
   protocol **ESP**. The private addresses inside are invisible to the providers. 🔒
4. Back to **Realtime**.

---

## Part 6 — Save
`copy running-config startup-config` on both edges, `write memory` on the ASA → **File → Save**.

## Checkpoint
- [ ] `QM_IDLE` with Jerusalem; encaps/decaps counters growing
- [ ] Jerusalem PCs reach Haifa's private addresses and DMZ servers by name
- [ ] Jerusalem's NTP synchronized to Haifa
- [ ] The teleworker connects and reaches HQ servers

## If something doesn't work
| Problem | Fix |
|---|---|
| `crypto` commands don't exist | Licence not active — 4.1 and **reload** |
| Phase 1 never comes up (no `QM_IDLE`) | Policy mismatch (encryption/hash/group must be equal on both sides) or different keys |
| Tunnel up, pings fail | Static route into the tunnel missing; ASA ACL line for 10.20.0.0/16 missing |
| Jerusalem's ACL blocks the VPN | WAN-IN must permit isakmp, esp **and** gre from 200.1.1.2 |
| S2S peer asks for a username / breaks after adding remote access | `no-xauth` missing on the `crypto isakmp key` line |
| VPN client: "authentication failed" | Group name/key or user/password typo; `aaa authorization network VPN-GROUP local` missing |
| Client connects but can't reach HQ | ASA ACL for 172.16.250.0/24 missing |

## Challenge
Kiryat Gat has no VPN. Without adding one, which DMZ services can Kiryat Gat still use, and how?

<details><summary>Answer</summary>

Only what is published on the internet **and** reachable over IPv4 from Kiryat Gat:
- **NTP** — DIST1 has an IPv4 path (legacy transit + PAT) and syncs to the public 200.1.10.4 → req. 20.8 OK.
- **Web** — only the legacy IPv4 devices can open 200.1.10.3; IPv6 users can't (step 14 challenge).
- Mail/DNS/FTP/AAA are not published → no. That's why Kiryat Gat has its **own** DNS, mail, syslog and
  AAA servers. A real fix would be a third tunnel over IPv6 or a dual-stack DMZ — write the trade-off in the book.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

| In the example | In your project — take it from |
|---|---|
| GRE tunnel `172.16.12.0/30` between HQ and branch 2 | your own tunnel /30, between **your** two IPv4 sites (req. 2) |
| Pre-shared key `Intel#S2S-Key` | plan part F |
| Remote-access group `HQ-RA`, pool `RA-POOL 172.16.250.10–100`, user `remote1` | your group name, your pool, your user and passwords (req. 23.4) |

**Check:** `show crypto isakmp sa` shows `QM_IDLE`; ping between the two sites goes through the
tunnel; the home PC connects with the VPN client and reaches HQ.

💾 Save as `my-project/step15.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★2 What is a site-to-site VPN? What is a remote-access VPN?**

<details><summary>Answer</summary>

Site-to-site: a permanent encrypted tunnel between two routers, joining two offices; users notice nothing. Remote access: one user (at home) runs a VPN client and connects to the company's VPN server over the internet.

**Show it:** `show crypto isakmp sa`; the home PC → Desktop → VPN
</details>

**2. ★2 What happens in phase 1 of setting up the VPN? Show it in the project.**

<details><summary>Answer</summary>

IKE phase 1 (ISAKMP): the two peers agree on a policy — encryption (AES), hash (SHA), authentication (pre-shared key), Diffie-Hellman group, lifetime — authenticate each other and build a secure channel. Phase 2 then negotiates the IPsec SAs (transform set) that encrypt the user data.

**Show it:** `show crypto isakmp sa` → `QM_IDLE`; `show crypto ipsec sa` → encrypted/decrypted counters
</details>

**3. ★3 What is Diffie-Hellman?**

<details><summary>Answer</summary>

A method for two sides to create the **same secret key** over an insecure network without ever sending the key itself. The DH group decides the key strength.

**Show it:** `show running-config` → `crypto isakmp policy` → `group`
</details>

**4. ★3 What is the difference between symmetric and asymmetric encryption? Name protocols of each.**

<details><summary>Answer</summary>

Symmetric: the same key encrypts and decrypts — fast (AES, 3DES, DES). Asymmetric: a public key and a private key — slow, used for key exchange and signatures (RSA, Diffie-Hellman, ECC).

**Show it:** —
</details>

**5. ★2 What is integrity, and which protocols do hashing?**

<details><summary>Answer</summary>

Integrity = proof that the data wasn't changed on the way. A hash (fingerprint) of the data is sent with it; any change gives a different hash. MD5, SHA-1, SHA-256 (in VPN as HMAC).

**Show it:** `crypto ipsec transform-set` → `esp-sha-hmac`
</details>

**6. ◆ Why GRE over IPsec and not only IPsec?**

<details><summary>Answer</summary>

IPsec alone carries only unicast IP; routing protocols use multicast. GRE wraps everything (including multicast) in a tunnel interface, and IPsec encrypts the GRE packets.

**Show it:** `show interfaces tunnel 0`
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

In a copy of your file, remove the crypto configuration from one end of the tunnel. Without this
page, write it again from your plan (policy, key, transform set, ACL or profile, apply). Target:
**15 minutes**. Then ping across the tunnel and show `show crypto isakmp sa` and the counters of
`show crypto ipsec sa`.

---

## 👩‍🏫 Teacher notes
- **Lesson 1:** why VPN (armoured box comparison), IPsec phases, GRE, *why GRE over IPsec* (the static-NAT
  story) → licence + reload on both edges.
- **Lesson 2:** site-to-site configuration → tests → encryption in Simulation mode on the projector.
- **Lesson 3:** remote access → VPN client → tests → challenge.
- **Common student questions**
  - *"Why is the first ping lost?"* — The tunnel is built on demand (phase 1 + 2 take a moment).
  - *"Why sequence 65535 for the dynamic map?"* — Specific peers (Jerusalem, seq 10) must be checked
    first; the "anyone" template must be last.
- **Typical errors:** forgetting the reload after the licence; mismatched ISAKMP policy; forgetting the
  ASA ACL lines.
