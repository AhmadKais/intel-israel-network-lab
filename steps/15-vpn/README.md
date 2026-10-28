# Step 15 — VPN: site-to-site (GRE over IPsec) + remote access

> ⏱ 2–3 lessons · 🎯 ministry requirements 2, 21, 23.4, 20.8 · 📁 start from: `my-work/step14.pkt` · 💾 save as: `my-work/step15.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 14 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## 🗺 Your missions in this step
**Lesson 1 — the licence and the tunnel**
- [ ] **Mission 1** — Open step 14 and save as step 15 (2 min)
- [ ] **Mission 2** — See the problem first: Jerusalem can't reach Haifa (5 min)
- [ ] **Mission 3** — Activate the security licence on both edge routers (15 min)
- [ ] **Mission 4** — The GRE tunnel on both edges (15 min)

**Lesson 2 — encryption**
- [ ] **Mission 5** — IPsec on R-HFA-EDGE (15 min)
- [ ] **Mission 6** — IPsec on R-JLM-EDGE (10 min)
- [ ] **Mission 7** — Let Jerusalem through the ASA + test the site-to-site VPN (20 min)
- [ ] **Mission 8** — See the encryption in Simulation mode (10 min)

**Lesson 3 — the teleworker**
- [ ] **Mission 9** — R-HFA-EDGE as a remote-access VPN server (15 min)
- [ ] **Mission 10** — The teleworker's VPN client (10 min)
- [ ] **Mission 11** — Final tests and save (10 min)
- [ ] **Exercises** A and B
- [ ] **🛠 Your own project**

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

### 1.7 The rule of mirrors
Site-to-site VPN = **two mirror images**. Everything on Haifa's side points to Jerusalem's
address, and the opposite:

| | R-HFA-EDGE | R-JLM-EDGE |
|---|---|---|
| Tunnel0 address | 172.16.12.**1** | 172.16.12.**2** |
| `tunnel destination` / `set peer` / `isakmp key ... address` | **200.2.1.2** | **200.1.1.2** |
| ACL `permit gre host A host B` | A = 200.1.1.2, B = 200.2.1.2 | A = 200.2.1.2, B = 200.1.1.2 |
| Route into the tunnel | `10.20.0.0/16 → 172.16.12.2` | `10.10.0.0/16 → 172.16.12.1` |
| Policy, key, transform set | **identical** | **identical** |

> Nothing new to place in this step — the edge routers, the ASA and PC-HOME-01 already exist.

---

## Mission 1 — Open step 14 and save as step 15 (how? → M1)
**File → Open** → `my-work/step14.pkt` → **File → Save As** → `my-work/step15.pkt`.

---

## Mission 2 — See the problem first
👉 **Where:** **PC-JLM-SAL-01** → Desktop → Command Prompt.

⌨️ Type:
```
ping 10.10.16.1
```
✅ **You should see:** `Request timed out` (or `Destination host unreachable`).

🧠 **Why?** 10.10.16.1 is a **private** Haifa address. The packet reaches R-JLM-EDGE, is NAT-ed
and sent to the internet — and the internet has no route to 10.x. After this step, the same ping
will work.

---

## Mission 3 — Activate the security licence on both edge routers
Do this on **R-HFA-EDGE**, then repeat on **R-JLM-EDGE**.

👉 **Where:** the router → CLI → log in (`admin` / `Intel#Adm1n`) → `R-...#`.

⌨️ Type:
```
configure terminal
license boot module c2900 technology-package securityk9
```
The router shows a licence text and asks `ACCEPT? [yes/no]:` → type **`yes`** → Enter.

⌨️ Type:
```
end
copy running-config startup-config
reload
```
Press **Enter** to confirm the reload. Press **Fast Forward Time** until the router has booted,
then log in again.

🔍 **Check now:** `show version` → in the licence table at the bottom `securityk9` is
**Permanent** / **Active** (or `RightToUse`).

❌ **`crypto` commands still say `% Invalid input`:** you skipped the reload.

Tick: - [ ] R-HFA-EDGE  - [ ] R-JLM-EDGE

---

## Mission 4 — The GRE tunnel on both edges

### 4a — R-HFA-EDGE
👉 **Where:** R-HFA-EDGE → `R-HFA-EDGE#`.

⌨️ Type:
```
configure terminal
interface tunnel 0
 description GRE to R-JLM-EDGE (encrypted by IPsec)
 ip address 172.16.12.1 255.255.255.252
 tunnel source gi0/1
 tunnel destination 200.2.1.2
 exit
ip route 10.20.0.0 255.255.0.0 172.16.12.2
 end
copy running-config startup-config
```
| Command | In simple English |
|---|---|
| `interface tunnel 0` + `tunnel source` / `tunnel destination` | A virtual cable from my public address to theirs |
| `ip route 10.20.0.0 255.255.0.0 172.16.12.2` | "Jerusalem's private block goes into the tunnel" (a static route — req. 22.17) |

### 4b — R-JLM-EDGE (the mirror)
👉 **Where:** R-JLM-EDGE → `R-JLM-EDGE#`.

⌨️ Type:
```
configure terminal
interface tunnel 0
 description GRE to R-HFA-EDGE (encrypted by IPsec)
 ip address 172.16.12.2 255.255.255.252
 tunnel source gi0/1
 tunnel destination 200.1.1.2
 exit
ip route 10.10.0.0 255.255.0.0 172.16.12.1
 end
copy running-config startup-config
```
🔍 **Check now:** on R-JLM-EDGE `ping 172.16.12.1` → `!!!!!` — the tunnel works (still **not**
encrypted). `show interfaces tunnel 0` → `Tunnel0 is up, line protocol is up`.

> The `redistribute static` from step 13 now also gives the route `10.10.0.0/16` to CC1/CC2 — so all
> of Jerusalem knows "Haifa is through the edge router". The CBAC ACL `WAN-IN` already lets GRE in.

❌ **Tunnel0 up but ping fails:** on R-JLM-EDGE, check `WAN-IN` has the `permit gre` line (step 13,
Mission 15a).

---

## Mission 5 — IPsec on R-HFA-EDGE
👉 **Where:** R-HFA-EDGE → `configure terminal`.

### 5a — Phase 1: how the two routers trust each other
⌨️ Type:
```
crypto isakmp policy 10
 encryption aes 256
 hash sha
 authentication pre-share
 group 5
 lifetime 86400
 exit
crypto isakmp key Intel#S2S-Key address 200.2.1.2 no-xauth
```
| Command | In simple English |
|---|---|
| `crypto isakmp policy 10` (encryption/hash/authentication/group) | How we check each other in phase 1 (must match on both sides) |
| `crypto isakmp key KEY address PEER no-xauth` | The shared secret with that peer; `no-xauth` = it's a router, don't ask it for a username |

✅ `%CRYPTO-6-ISAKMP_ON_OFF: ISAKMP is ON` may appear — good.

### 5b — Phase 2: what to encrypt and how
⌨️ Type:
```
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
```
| Command | In simple English |
|---|---|
| `crypto ipsec transform-set ... esp-aes 256 esp-sha-hmac` | How the data is encrypted (AES-256) and checked (SHA) |
| `crypto map VPN-MAP 10 ipsec-isakmp` / `set peer` / `match address` | "Encrypt traffic matching this ACL, toward this peer" |

> PT may print `% NOTE: This new crypto map will remain disabled until a peer and a valid access
> list have been configured.` — normal, you add them on the next lines.

### 5c — Switch it on at the internet interface
⌨️ Type:
```
interface gi0/1
 crypto map VPN-MAP
 end
copy running-config startup-config
```

---

## Mission 6 — IPsec on R-JLM-EDGE
👉 **Where:** R-JLM-EDGE → `configure terminal`.

Same as Mission 5, mirrored (see table 1.7): the peer is **200.1.1.2** and the ACL is reversed.

⌨️ Type:
```
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

---

## Mission 7 — Let Jerusalem through the ASA + test the site-to-site VPN

### 7a — The ASA hole
🧠 Jerusalem's packets come out of the tunnel at R-HFA-EDGE — on the ASA's **outside**. Level 0 →
100 is blocked unless we allow it.

👉 **Where:** ASA → `enable` → `Intel#En4ble`.

⌨️ Type:
```
configure terminal
access-list OUTSIDE_IN extended permit ip 10.20.0.0 255.255.0.0 10.10.0.0 255.255.0.0
end
write memory
```
(10.10.0.0/16 includes the DMZ 10.10.200.0/24 — so Jerusalem reaches the inside **and** the DMZ servers.)

### 7b — Make the tunnel come up
The encryption starts only when traffic needs it. From **PC-JLM-SAL-01**: `ping 10.10.16.1` — the
first one or two may time out while the tunnel builds, then replies come. ✅
(This is the ping that failed in Mission 2!)

### 7c — On the routers
👉 **Where:** R-HFA-EDGE.

```
R-HFA-EDGE# show crypto isakmp sa
dst             src             state          conn-id status
200.2.1.2       200.1.1.2       QM_IDLE           1001 ACTIVE    <- phase 1 up with Jerusalem
R-HFA-EDGE# show crypto ipsec sa
    #pkts encaps: 57, #pkts encrypt: 57, ...        <- growing while you ping = encrypted
    #pkts decaps: 57, #pkts decrypt: 57, ...
R-HFA-EDGE# show ip route static                  <- S 10.20.0.0/16 via 172.16.12.2
```
And on **CC1**: `show ip route | include 10.10.0.0` → `D EX 10.10.0.0/16 via 10.20.4.1`.

❌ **No `QM_IDLE`:** compare the two `crypto isakmp policy 10` blocks and keys letter by letter
(`show running-config | section crypto`).
❌ **`QM_IDLE` but the ping fails:** the ASA line (7a) or the static routes (Mission 4).

### 7d — Tests
| # | From | Do | Expected |
|---|---|---|---|
| 1 | PC-JLM-SAL-01 | `tracert 10.10.200.13` | 10.20.10.2 → 10.20.4.1 → **172.16.12.1** (the tunnel!) → … |
| 2 | PC-JLM-SAL-01 | Web Browser → `http://www.intel.lab` | ✅ DNS 10.10.200.11 is reachable now too |
| 3 | PC-JLM-SAL-01 | `nslookup ftp.intel.lab` | 10.10.200.15 (DMZ services via VPN — req. 21) |
| 4 | SWL3-JLM-CC1 | `show ntp status` (after Fast Forward) | synchronized to 10.10.200.14 (req. 20.8) |
| 5 | PC-HFA-IT-01 | `ssh -l admin 10.20.0.11` | ✅ HQ's IT manages Jerusalem over the VPN |

---

## Mission 8 — See the encryption in Simulation mode (how? → M21)
1. Click **Simulation** (bottom right) → **Edit Filters** → tick **ICMP** (and ISAKMP/ESP if offered).
2. From PC-JLM-SAL-01 ping a Haifa PC.
3. Press **Capture/Forward** step by step. Click the packet envelope when it is on a cable
   **between the ISPs**.
4. Look at the outer header: `200.2.1.2 → 200.1.1.2`, protocol **ESP**. The private addresses
   inside are invisible to the providers. 🔒
5. Back to **Realtime**.

📝 Screenshot it for the book (M22).

---

## Mission 9 — R-HFA-EDGE as a remote-access VPN server
👉 **Where:** R-HFA-EDGE → `configure terminal`.

### 9a — Who may log in, and which addresses they get
⌨️ Type:
```
aaa new-model
aaa authentication login default group tacacs+ local
aaa authentication login VPN-AUTH local
aaa authorization network VPN-GROUP local
username remote1 secret Intel#Remote1
ip local pool RA-POOL 172.16.250.10 172.16.250.100
```
| Command | In simple English |
|---|---|
| `aaa authentication login VPN-AUTH local` | VPN users are checked against the local users |
| `ip local pool RA-POOL ...` | Addresses to give remote users |

### 9b — Phase 1 for clients, and the group the PC logs into
⌨️ Type:
```
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
```
| Command | In simple English |
|---|---|
| `crypto isakmp client configuration group HQ-RA` | The group the VPN client logs into (key, DNS, pool) |

### 9c — Phase 2 for clients, added to the same crypto map
⌨️ Type:
```
crypto ipsec transform-set RA-SET esp-aes 256 esp-sha-hmac
crypto dynamic-map RA-DYN 10
 set transform-set RA-SET
 reverse-route
 exit
crypto map VPN-MAP client authentication list VPN-AUTH
crypto map VPN-MAP isakmp authorization list VPN-GROUP
crypto map VPN-MAP client configuration address respond
crypto map VPN-MAP 65535 ipsec-isakmp dynamic RA-DYN
 end
copy running-config startup-config
```
| Command | In simple English |
|---|---|
| `crypto dynamic-map` + `... 65535 ipsec-isakmp dynamic` | A template for clients whose address we don't know in advance |
| `reverse-route` | Add a route to each connected client automatically |

> Sequence **65535** = checked **last**. Jerusalem (seq 10) must be matched first.

### 9d — The ASA: remote users may reach HQ
👉 **Where:** ASA → `configure terminal`.

⌨️ Type:
```
access-list OUTSIDE_IN extended permit ip 172.16.250.0 255.255.255.0 10.10.0.0 255.255.0.0
end
write memory
```
(The PAT list `NAT-HFA` from step 12 already skips 172.16.250.0/24 — we prepared for this.)

🔍 **Check now:** on R-JLM-EDGE ping a Haifa PC again → still works. Adding remote access must
not break the site-to-site VPN.

---

## Mission 10 — The teleworker's VPN client
👉 **Where:** **PC-HOME-01** → **Desktop** → **VPN**.

1. Fill in:

| Field | Value |
|---|---|
| GroupName | `HQ-RA` |
| Group Key | `Intel#RA-Group` |
| Host IP (Server IP) | `200.1.1.2` |
| Username | `remote1` |
| Password | `Intel#Remote1` |

2. Click **Connect**.

✅ **You should see:** after a moment, **"VPN is connected"**.

3. Close the VPN window → **Command Prompt** → `ipconfig` → you see an extra address
   `172.16.250.x` (the tunnel address).

❌ **"authentication failed":** group name/key or user/password typo; or
`aaa authorization network VPN-GROUP local` missing.

---

## Mission 11 — Final tests and save

| # | From | Do | Expected |
|---|---|---|---|
| 1 | PC-HOME-01 (VPN connected) | `ping 10.10.1.10` | ✅ |
| 2 | PC-HOME-01 (VPN connected) | Web Browser → `http://www.intel.lab` | ✅ |
| 3 | R-HFA-EDGE | `show crypto isakmp sa` | two `QM_IDLE` lines: Jerusalem **and** `200.3.50.10` (the teleworker) |
| 4 | PC-HOME-01 | VPN → **Disconnect**, then `ping 10.10.1.10` | ❌ |

💾 `copy running-config startup-config` on both edges, `write memory` on the ASA → **File → Save**.

---

## Checkpoint
- [ ] `QM_IDLE` with Jerusalem; encaps/decaps counters growing
- [ ] Jerusalem PCs reach Haifa's private addresses and DMZ servers by name
- [ ] Jerusalem's NTP synchronized to Haifa
- [ ] The teleworker connects and reaches HQ servers

## If something doesn't work
| Problem | Fix |
|---|---|
| `crypto` commands don't exist | Licence not active — Mission 3 and **reload** |
| Phase 1 never comes up (no `QM_IDLE`) | Policy mismatch (encryption/hash/group must be equal on both sides) or different keys |
| Tunnel up, pings fail | Static route into the tunnel missing; ASA ACL line for 10.20.0.0/16 missing |
| Jerusalem's ACL blocks the VPN | WAN-IN must permit isakmp, esp **and** gre from 200.1.1.2 |
| S2S peer asks for a username / breaks after adding remote access | `no-xauth` missing on the `crypto isakmp key` line |
| VPN client: "authentication failed" | Group name/key or user/password typo; `aaa authorization network VPN-GROUP local` missing |
| Client connects but can't reach HQ | ASA ACL for 172.16.250.0/24 missing |

---

## 🏋 Exercises

### Exercise A — A wrong key ⭐⭐
- 🎯 **Task:** see what a pre-shared-key mismatch looks like.
- 📁 **Start from:** `step15.pkt` → **Save As** `step15-exA.pkt`.
- 👣 **Steps:**
  1. On R-JLM-EDGE: `configure terminal` →
     `no crypto isakmp key Intel#S2S-Key address 200.1.1.2` →
     `crypto isakmp key WrongKey address 200.1.1.2 no-xauth` → `end` → `clear crypto isakmp`.
  2. From PC-JLM-SAL-01: `ping 10.10.16.1`.
  3. `show crypto isakmp sa` on both edges.
- ✅ **Done when:** you can say which phase failed and how you know.

<details><summary>💡 Hint</summary>

Phase 1 checks identity with the key. Look at the `state` column.
</details>
<details><summary>✔ Answer</summary>

The pings fail. There is no `QM_IDLE` — the state stays at something like `MM_NO_STATE` /
`MM_KEY_EXCH`, or the line disappears. Phase 1 (identity check) fails because the keys differ, so
phase 2 never starts and nothing is encrypted. Both sides must have the **same** key.
</details>

### Exercise B — What can Kiryat Gat still use? ⭐⭐⭐
- 🎯 **Task:** Kiryat Gat has no VPN. Without adding one, which DMZ services can Kiryat Gat still
  use, and how?
- 📁 **Start from:** your `step15.pkt` (nothing to change).
- 👣 **Steps:** for each DMZ service (DNS, mail, web, NTP, FTP, AAA) ask: is it published on the
  internet (step 12)? Can Kiryat Gat reach it with the IP version it has (step 14)? Test what you can.
- ✅ **Done when:** you have a 6-row table: service / yes or no / why.

<details><summary>✔ Answer</summary>

Only what is published on the internet **and** reachable over IPv4 from Kiryat Gat:
- **NTP** — DIST1 has an IPv4 path (legacy transit + PAT) and syncs to the public 200.1.10.4 → req. 20.8 OK.
- **Web** — only the legacy IPv4 devices can open 200.1.10.3; IPv6 users can't (step 14 exercise A).
- Mail/DNS/FTP/AAA are not published → no. That's why Kiryat Gat has its **own** DNS, mail, syslog and
  AAA servers. A real fix would be a third tunnel over IPv6 or a dual-stack DMZ — write the trade-off in the book.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

- [ ] Open `my-project/step14.pkt` → **Save As** `my-project/step15.pkt`
- [ ] Fill in your own "mirror" table like 1.7
- [ ] Licence + reload on both edges
- [ ] GRE tunnel + routes; ping across the tunnel
- [ ] IPsec on both sides; firewall holes; `QM_IDLE` and counters growing
- [ ] Remote-access server + the home PC's VPN client

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
  story) → Missions 1–4 (the failed ping, licence + reload, tunnel).
- **Lesson 2:** Missions 5–8 — IPsec, ASA hole, tests → encryption in Simulation mode on the projector.
- **Lesson 3:** Missions 9–11 — remote access → VPN client → tests → exercises.
- **Common student questions**
  - *"Why is the first ping lost?"* — The tunnel is built on demand (phase 1 + 2 take a moment).
  - *"Why sequence 65535 for the dynamic map?"* — Specific peers (Jerusalem, seq 10) must be checked
    first; the "anyone" template must be last.
- **Typical errors:** forgetting the reload after the licence; mismatched ISAKMP policy; forgetting the
  ASA ACL lines; mixing up the two sides of the mirror (table 1.7).
