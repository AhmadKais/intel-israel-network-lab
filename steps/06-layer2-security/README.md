# Step 06 — Layer-2 security

> ⏱ 1–2 lessons · 🎯 ministry requirements 13, 16, 22.9 · 📁 start from: `my-work/step05.pkt` · 💾 save as: `my-work/step06.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 05 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## What you will have at the end
The access switches defend themselves against the classic attacks a person can do just by
**plugging something into a wall socket**: flooding the switch with fake MACs, running a fake
DHCP server, plugging in an unknown device. You will **try the attacks yourself** and watch
the protections stop them.

---

## Part 1 — Understand it first (simple English)

### 1.1 Why the access layer needs security
Firewalls protect the company from the **internet**. But many attacks start **inside**: a
visitor, an unhappy employee, an infected laptop plugged into a free socket. The access switch
is the first device they touch — so it must defend itself.

### 1.2 The attacks and our defences

| Attack | What the attacker does (simple) | Our defence |
|---|---|---|
| **MAC flooding** | Sends thousands of frames with fake MAC addresses. The switch's MAC table fills up, and a full switch sends every frame to **every** port, like a hub → the attacker can read other people's traffic | **Port-security**: only N MAC addresses allowed per port |
| **Rogue DHCP server** | Runs his own DHCP server that gives PCs **his** address as the gateway → all their traffic passes through him ("man in the middle") | **DHCP snooping**: DHCP answers are only accepted from trusted ports (the uplinks to the real servers) |
| **DHCP starvation** | Asks for every free address so real PCs get none | DHCP snooping **rate limit** + port-security |
| **Unknown device** | Unplugs the printer, plugs in his laptop | Port-security with **one** allowed MAC (sticky) |
| **Rogue switch** | Plugs in a switch to become STP root | BPDU Guard + Root Guard (done in step 04) |
| **VLAN hopping** | Negotiates a trunk, or uses the native VLAN | `mode access`, `nonegotiate`, native VLAN 998 (done in steps 02–03) |
| **Free socket** | Plugs into an unused port | Parking VLAN + shutdown (done in step 02) |
| **Reconnaissance** | Uses CDP messages to learn the switch's model, software and IP | `no cdp enable` on user ports |

### 1.3 Port-security — the door bouncer
Port-security is a bouncer at each port with a list of allowed guests (MAC addresses).

| Setting | Meaning |
|---|---|
| `maximum 2` | At most 2 different MACs on this port (a PC + an IP phone / docking station) |
| `mac-address sticky` | Remember the first MACs that appear, and write them into the config — no typing MACs by hand |
| `violation protect` | Unknown MAC → its frames are silently dropped |
| `violation restrict` | Dropped **+ logged + counted** ← we use this for users (one mistake doesn't kill the port) |
| `violation shutdown` | The port shuts down completely (err-disabled) ← we use this for printers and the admin PC (only one device ever belongs there) |

### 1.4 DHCP snooping — only trusted ports may give addresses
The switch watches all DHCP messages:
- **Untrusted** ports (PCs): may *ask* for an address, may **not** *offer* one. Fake offers are dropped.
- **Trusted** ports (the uplinks toward DIST1/DIST2, our real DHCP servers): offers allowed.

While watching, the switch also builds a **binding table**: which MAC got which IP on which
port. (Used by other protections, see the challenge.)

### 1.5 The "option 82" trap (why PCs suddenly get 169.254.x.x)
With snooping on, the switch adds extra information called **option 82** to DHCP requests.
Our DHCP servers (the IOS L3 switches) see option 82 coming from an unexpected place and
**throw the request away** → PCs get no address. The fix on the access switches:
`no ip dhcp snooping information option`. This is the #1 mistake in this step.

---

## Part 2 — Prepare
**File → Open** → `my-work/step05.pkt` → **File → Save As** → `my-work/step06.pkt`.

## Part 3 — Build
Nothing to build now (the attack lab in Part 5 adds devices temporarily).

---

## Part 4 — Configure (type every line)

### 4.1 SW-HFA-A-ACC1
```
enable
configure terminal

! ---------- DHCP snooping ----------
ip dhcp snooping
ip dhcp snooping vlan 10,20,30,99,110,120
! the option-82 trap (section 1.5)
no ip dhcp snooping information option

! uplinks lead to the real DHCP servers (DIST1/DIST2) -> trusted
interface range gi0/1 - 2
 ip dhcp snooping trust
 exit

! ---------- user ports: Engineering + Finance ----------
interface range fa0/1 - 20
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 ip dhcp snooping limit rate 10
 no cdp enable
 exit

! ---------- printer port: exactly one device ----------
interface fa0/21
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 no cdp enable
 exit

! ---------- admin PC port: exactly one device ----------
interface fa0/22
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 no cdp enable
 end
copy running-config startup-config
```

### 4.2 SW-HFA-A-ACC2
```
enable
configure terminal

ip dhcp snooping
ip dhcp snooping vlan 10,20,30,99,110,120
no ip dhcp snooping information option

interface range gi0/1 - 2
 ip dhcp snooping trust
 exit

interface range fa0/1 - 20
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 ip dhcp snooping limit rate 10
 no cdp enable
 exit

interface fa0/21
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 no cdp enable
 exit

! ---------- the two access-point ports ----------
! every wireless laptop's MAC appears on the AP's port -> allow many
interface range fa0/22 - 23
 switchport port-security
 switchport port-security maximum 30
 switchport port-security violation restrict
 ip dhcp snooping limit rate 20
 no cdp enable
 end
copy running-config startup-config
```

### 4.3 Let the switches learn the MACs
Sticky learning needs traffic. On **every PC**: Desktop → Command Prompt → `ipconfig /renew`
(and ping its gateway). The printer: ping it from a PC (`ping 10.10.18.10`).

### 4.4 What the new commands mean
| Command | In simple English |
|---|---|
| `ip dhcp snooping` | "Start watching DHCP" |
| `ip dhcp snooping vlan ...` | "…in these VLANs" |
| `no ip dhcp snooping information option` | "Don't add option 82" (avoids the trap) |
| `ip dhcp snooping trust` | "Real DHCP servers are behind this port" |
| `ip dhcp snooping limit rate 10` | "Max 10 DHCP messages per second from this port" (stops starvation) |
| `switchport port-security` | "Turn the bouncer on" |
| `... maximum 2` / `mac-address sticky` / `violation restrict` | See the table in 1.3 |
| `no cdp enable` | "Don't tell this port who I am" |

---

## Part 5 — Check your work

### 5.1 DHCP snooping
On **ACC1**: `show ip dhcp snooping`
```
Switch DHCP snooping is enabled
DHCP snooping is configured on following VLANs:
10,20,30,99,110,120
Insertion of option 82 is disabled
Interface                  Trusted    Rate limit (pps)
-----------------------    -------    ----------------
GigabitEthernet0/1         yes        unlimited
GigabitEthernet0/2         yes        unlimited
FastEthernet0/1            no         10
...
```
`show ip dhcp snooping binding` → one line per PC that got an address (MAC, IP, VLAN, port).

### 5.2 Port-security
```
show port-security
```
```
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
     Fa0/1         2             1               0               Restrict
     Fa0/21        1             1               0               Shutdown
...
```
`show port-security interface fa0/1` → `Port Status : Secure-up`.
`show running-config | include sticky` → the learned MACs are now written in the config.

### 5.3 PCs still work
On each PC: `ipconfig /release`, then `ipconfig /renew` → still gets an address ✅.
If not → section 1.5 / troubleshooting table.

---

## Part 6 — Attack lab (try the attacks!)

### Attack 1 — the rogue DHCP server
1. **End Devices** → drag a **Server** next to ACC1. Rename it `SRV-ROGUE`.
2. Cable it with **Copper Straight-Through**: SRV-ROGUE **FastEthernet0** → ACC1 **Fa0/9**
   (a free Engineering port — VLAN 10).
3. SRV-ROGUE → **Desktop → IP Configuration**: IP `10.10.16.250`, mask `255.255.255.0`.
4. SRV-ROGUE → **Services** tab → **DHCP** (left list):
   - Service: **On**
   - Default Gateway: `10.10.16.250` (the attacker sends himself as gateway!)
   - Start IP Address: `10.10.16.200`, Subnet Mask `255.255.255.0`
   - Click **Save**.
5. On an Engineering PC: Desktop → IP Configuration → click **Static**, then **DHCP** again.
6. Result: the PC gets an address between **.11 and .254** with gateway **10.10.16.1** — from
   DIST1/DIST2. The rogue's offers were **dropped** at Fa0/9 (an untrusted port). ✅
7. *Bonus:* temporarily make Fa0/9 trusted (`interface fa0/9` → `ip dhcp snooping trust`),
   renew the PC again — now it may get `.200` with gateway `.250`: the attack works.
   **Remove the trust** (`no ip dhcp snooping trust`) right away.
8. Delete SRV-ROGUE (Delete tool).

### Attack 2 — an unknown device on the printer port
1. Delete the cable between **PRN-HFA-A-01** and ACC1.
2. Drag a **Laptop**, cable it (Straight-Through) to ACC1 **Fa0/21**.
3. The laptop sends a frame (set any IP on it, e.g. `10.10.18.50`, and ping `10.10.18.1`).
4. ACC1 shows `%PORT_SECURITY-2-PSECURE_VIOLATION` and Fa0/21 goes **down** (red light).
   `show port-security interface fa0/21` → `Port Status : Secure-shutdown`.
5. Recover: delete the laptop, reconnect the printer to Fa0/21, then on ACC1:
```
configure terminal
interface fa0/21
 shutdown
 no shutdown
 end
```
6. Ping the printer from a PC → works again ✅.

---

## Part 7 — Save
`copy running-config startup-config` on ACC1 and ACC2 (again, because sticky MACs were
learned) → **File → Save**.

## Checkpoint
- [ ] Snooping on, uplinks trusted, option 82 disabled — on both access switches
- [ ] All PCs still get DHCP addresses
- [ ] Rogue DHCP server failed to give addresses
- [ ] Printer port shut down when a laptop took its place, and you recovered it

## If something doesn't work
| Problem | Fix |
|---|---|
| PCs get `169.254.x.x` after this step | Missing `no ip dhcp snooping information option`, or uplinks not `trust`ed |
| `% Command rejected: Fa0/x is a dynamic port` | The port isn't `switchport mode access` — set it first |
| A real PC's port went into violation | Too many MACs (e.g. you moved PCs around). `clear port-security sticky interface fa0/x`, then `shutdown` / `no shutdown` |
| Printer port down after you only moved the printer cable | Sticky remembered the old port's MAC; recover as in Attack 2 step 5 |

## Challenge (bonus — careful!)
**Dynamic ARP Inspection (DAI)** uses the snooping binding table to drop fake ARP answers (ARP
poisoning — another man-in-the-middle trick). Turn it on for VLAN 10 only on ACC1, without
breaking anything. Which ports must be trusted, and why only VLAN 10?

<details><summary>Answer</summary>

```
configure terminal
ip arp inspection vlan 10
interface range gi0/1 - 2
 ip arp inspection trust
 end
```
Only VLAN 10, because all its devices use DHCP — so they're in the binding table. VLAN 30
(static printer) and VLAN 99 (static admin PC) are **not** in the table, so DAI would drop
their ARP and they'd go offline (unless you write ARP ACLs for them). The uplinks must be
trusted, otherwise the gateway's ARP answers (which come from the uplink and are not in the
table) would be dropped.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

| In the example | In your project — take it from |
|---|---|
| Port-security on the user ports | your user ports from the port plan |
| DHCP snooping VLAN list | plan part D (your user VLANs) |
| Trusted ports = the uplinks to DIST1/DIST2 | your uplink ports |

**Check:** repeat the attack lab of this step against **your** network — it must be blocked.

💾 Save as `my-project/step06.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★3 Which attack related to DHCP do you know, and what is the solution? Where did you implement it?**

<details><summary>Answer</summary>

**Rogue DHCP server:** an attacker's device answers DHCP requests and hands out itself as the gateway (man-in-the-middle). Solution: **DHCP snooping** — only ports marked `trust` (the uplinks toward the real server) may send DHCP offers; offers from any other port are dropped.

**Show it:** `show ip dhcp snooping`, `show running-config interface <uplink>` → `ip dhcp snooping trust`
</details>

**2. ★3 What is DHCP starvation? How do you prevent it, and where did you configure it?**

<details><summary>Answer</summary>

The attacker sends thousands of DHCP requests with fake MAC addresses until the pool is empty; real users get no address (denial of service), and then a rogue server can serve them. Prevention: **port-security** (limit MACs per port) and DHCP snooping rate-limiting.

**Show it:** `show port-security`, `show port-security interface fa0/1`
</details>

**3. ◆ What is port-security? What are the violation modes? What is `sticky`?**

<details><summary>Answer</summary>

It limits which and how many MAC addresses may use a port. Violation modes: `shutdown` (port goes err-disabled — default), `restrict` (drops and logs), `protect` (drops silently). `sticky` learns the first MAC(s) automatically and writes them into the running-config.

**Show it:** `show port-security interface fa0/1`
</details>

**4. ◆ How do you bring back a port that port-security shut down?**

<details><summary>Answer</summary>

Remove the cause (unplug the foreign device), then on the interface: `shutdown` and `no shutdown`.

**Show it:** `show interfaces status` → err-disabled
</details>

**5. ◆ What is Dynamic ARP Inspection?**

<details><summary>Answer</summary>

It checks ARP replies against the DHCP-snooping binding table and drops fake ones, which stops ARP spoofing (an attacker claiming to be the gateway).

**Show it:** `show running-config | include arp inspection`
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

In a copy of your finished step-06 file, remove port-security and DHCP snooping from one access switch.
Without this page, configure them again from your plan. Target: **10 minutes**. Then repeat the
attack lab of this step in front of the examiner and show that it is blocked.

---

## 👩‍🏫 Teacher notes
- **Lesson plan (~90–120 min):** 25 min the attacks table — tell it as stories ("a visitor in
  the meeting room…") → 10 min bouncer + trusted-port explanations → 30 min configuration →
  10 min checks → 30 min attack lab (students enjoy being the attacker).
- **Show DHCP in Simulation mode** with the filter set to **DHCP**: you see the rogue's OFFER
  arrive at ACC1 and disappear.
- **Common student questions**
  - *"Why restrict for users but shutdown for the printer?"* — A user may legitimately connect
    a second device by mistake; we log it without killing their work. A printer port should
    only ever see that printer — anything else is suspicious.
  - *"Why trust the uplinks?"* — The real DHCP servers (DIST1/2) are behind them.
  - *"Can the attacker just copy the printer's MAC?"* — Yes (MAC spoofing). Port-security is one
    layer; real networks add 802.1X authentication. Good discussion point for the book.
- **Typical errors:** the option-82 trap; forgetting to save after sticky learning.
