# Step 10 — Wireless: employees + guests

> ⏱ 1–2 lessons · 🎯 ministry requirement 18 · 📁 start from: `my-work/step09.pkt` · 💾 save as: `my-work/step10.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 09 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## 🗺 Your missions in this step
- [ ] **Mission 1** — Open step 09 and save as step 10 (2 min)
- [ ] **Mission 2** — Place and cable the two access points (5 min)
- [ ] **Mission 3** — Place 3 laptops and give them a WiFi card (15 min)
- [ ] **Mission 4** — Configure the employee AP: INTEL-CORP (5 min)
- [ ] **Mission 5** — Configure the guest AP: INTEL-GUEST (5 min)
- [ ] **Mission 6** — Connect the laptops (10 min)
- [ ] **Mission 7** — Test **before** the ACL: see the danger (5 min)
- [ ] **Mission 8** — The guest-isolation ACL on A-DIST1 and A-DIST2 (15 min)
- [ ] **Mission 9** — Test again and save (10 min)
- [ ] **Exercises** A and B
- [ ] **🛠 Your own project**

## What you will have at the end
Building A has two WiFi networks:
- **INTEL-CORP** for employees → full access, like a wired PC;
- **INTEL-GUEST** for visitors → internet only, **blocked** from every internal network.

```
   SW-HFA-A-ACC2  Fa0/22 (VLAN 110) ── AP-HFA-A-EMP   ))) "INTEL-CORP"  ── LT-HFA-EMP-01/02
                  Fa0/23 (VLAN 120) ── AP-HFA-A-GUEST ))) "INTEL-GUEST" ── LT-HFA-GUEST-01
```

---

## Part 1 — Understand it first (simple English)

### 1.1 What an access point does
An **access point (AP)** is a bridge between radio and cable: wireless devices connect to it,
and it puts their traffic onto the wired network — into **one VLAN** (the VLAN of the switch
port it's plugged into). Each AP broadcasts a network name, the **SSID**.

| SSID | AP | Switch port | VLAN | Subnet |
|---|---|---|---|---|
| INTEL-CORP | AP-HFA-A-EMP | ACC2 Fa0/22 | 110 WIFI-EMP | 10.10.20.0/24 |
| INTEL-GUEST | AP-HFA-A-GUEST | ACC2 Fa0/23 | 120 WIFI-GUEST | 10.10.21.0/24 |

The VLANs, gateways (HSRP) and DHCP pools for 110 and 120 already exist since steps 02–05 —
we prepared for this moment. **You don't need to configure anything on ACC2.**

### 1.2 WiFi security
Radio goes through walls — anyone in the car park can try to connect. So WiFi must be
**encrypted** and need a **key**.

| Security | How it works | We use it for |
|---|---|---|
| Open | No password, no encryption | Never |
| WEP | Old encryption, broken in minutes | Never |
| **WPA2-Personal (PSK)** + AES | One shared password for everyone | **Guests**, and employees in this lab |
| **WPA2-Enterprise** | Every user logs in with their **own** username via a RADIUS (AAA) server | Real company employees (see Exercise B) |

### 1.3 Guest isolation — the most important part
Visitors must get internet but **never** reach our PCs or servers. We do this with an
**ACL on the guest VLAN's gateway** (on both distribution switches): packets *from* guests are
checked the moment they reach the router.

| Rule (top to bottom) | Why |
|---|---|
| permit DHCP | so the guest can get an address |
| permit DNS to the ISP's DNS server only | so names work |
| **deny** anything to 10.x, 172.16–31.x, 192.168.x | no internal (private) address at all |
| permit everything else | = the internet |

Guests also get the **ISP's** DNS (200.1.100.10) from DHCP, never our internal one (step 05).

This is an **extended ACL**: unlike the standard ACL from step 07 (source only), it checks
source **and destination**, and even the protocol/port (`udp ... eq domain` = DNS).

> **The order matters.** An ACL is read top to bottom and stops at the **first** line that
> matches. Every ACL ends with an invisible `deny any`.

---

## Mission 1 — Open step 09 and save as step 10 (how? → M1)
**File → Open** → `my-work/step09.pkt` → **File → Save As** → `my-work/step10.pkt`.

---

## Mission 2 — Place and cable the two access points
🧠 **Why:** each AP = one SSID = one VLAN.

👣 **Steps:**
1. In the device bar at the bottom click **Network Devices** (the router icon) → **Wireless Devices**.
2. Drag an **AccessPoint-PT** next to SW-HFA-A-ACC2. Rename it `AP-HFA-A-EMP` (M4).
3. Drag a second **AccessPoint-PT** → rename `AP-HFA-A-GUEST`.
4. Cable with **Copper Straight-Through** (M5):

| # | Click first | Port | Click second | Port |
|---|---|---|---|---|
| 1 | AP-HFA-A-EMP | Port 0 | SW-HFA-A-ACC2 | **Fa0/22** |
| 2 | AP-HFA-A-GUEST | Port 0 | SW-HFA-A-ACC2 | **Fa0/23** |

✅ **You should see:** two cables; after a few seconds both ends are green (Fast Forward Time helps).

❌ **If a port is red on the switch side:** Fa0/22–23 may still be shut. Log in to ACC2 →
`configure terminal` → `interface range fa0/22 - 23` → `no shutdown` → `end`.

🔍 **Check now:** on ACC2 `show vlan brief` → Fa0/22 in VLAN **110**, Fa0/23 in VLAN **120**.

---

## Mission 3 — Place 3 laptops and give them a WiFi card
🧠 **Why:** a new laptop in Packet Tracer has a **wired** card only. It can't see any WiFi until
you swap the card.

### 3a — Place the laptops
1. **End Devices** → **End Devices** → drag 3 **Laptop** icons near the APs.
2. Rename: `LT-HFA-EMP-01`, `LT-HFA-EMP-02`, `LT-HFA-GUEST-01`.

### 3b — Swap the card (do this on **each** of the 3 laptops — full pictures in M8)
1. Click the laptop → **Physical** tab.
2. Click the laptop's **power button** (on the picture) → the green light goes off.
3. Drag the existing module (**PT-LAPTOP-NM-1CFE**, in the slot at the side of the laptop)
   **out** — drop it on the **MODULES** list on the left.
4. From MODULES, drag **WPC300N** into the empty slot.
5. Click the **power button** again → on.
6. Close the window.

Tick: - [ ] EMP-01  - [ ] EMP-02  - [ ] GUEST-01

✅ **You should see:** in the laptop's **Config** tab the left list now shows **Wireless0**
(and no FastEthernet0).

❌ **If "Cannot add a module when the device is powered on":** you skipped step 2.

---

## Mission 4 — Configure the employee AP: INTEL-CORP
👉 **Where:** click **AP-HFA-A-EMP** → **Config** tab → in the left list click **Port 1**
(Port 1 = the radio side; Port 0 = the cable).

⌨️ Fill in:

| Field | Value |
|---|---|
| SSID | `INTEL-CORP` |
| 2.4 GHz Channel | `1` |
| Authentication | click **WPA2-PSK** |
| PSK Pass Phrase | `IntelCorp#2026` |
| Encryption Type | **AES** |

Close the window (PT saves fields as you type).

🔍 **Check now:** open the AP again → Port 1 → the values are still there.

---

## Mission 5 — Configure the guest AP: INTEL-GUEST
👉 **Where:** **AP-HFA-A-GUEST** → **Config** → **Port 1**.

| Field | Value |
|---|---|
| SSID | `INTEL-GUEST` |
| 2.4 GHz Channel | `6` (a different channel = no interference with the other AP) |
| Authentication | **WPA2-PSK** |
| PSK Pass Phrase | `WelcomeGuest2026` |
| Encryption Type | **AES** |

> ⚠️ SSIDs and keys are **case-sensitive**. `intel-guest` ≠ `INTEL-GUEST`.

---

## Mission 6 — Connect the laptops
👉 **Where:** click the laptop → **Config** tab → **Wireless0** (left list).

| Field | LT-HFA-EMP-01 and LT-HFA-EMP-02 | LT-HFA-GUEST-01 |
|---|---|---|
| SSID | `INTEL-CORP` | `INTEL-GUEST` |
| Authentication | **WPA2-PSK** | **WPA2-PSK** |
| PSK Pass Phrase | `IntelCorp#2026` | `WelcomeGuest2026` |
| Encryption Type | **AES** | **AES** |
| IP Configuration | **DHCP** | **DHCP** |

Press **Fast Forward Time** (M18) twice.

✅ **You should see:** a dashed "radio" line between each laptop and its AP.

❌ **If there is no dashed line:** compare SSID, key and encryption letter by letter with the AP
— they must match **exactly**.
(Alternative way to connect: laptop → **Desktop → PC Wireless → Connect** tab → choose the
network → **Connect** → type the key.)

🔍 **Check now:** each laptop → **Desktop → Command Prompt** → `ipconfig`:
- EMP laptops → `10.10.20.x`, gateway `10.10.20.1`
- GUEST laptop → `10.10.21.x`, gateway `10.10.21.1`

❌ **If `169.254.x.x`:** the laptop didn't get DHCP. Wait, then `ipconfig /renew`. Still not? See
"If something doesn't work" at the bottom.

---

## Mission 7 — Test **before** the ACL: see the danger
👉 **Where:** **LT-HFA-GUEST-01** → Desktop → Command Prompt.

⌨️ Type:
```
ping 10.10.1.10
```
✅ **You should see:** **Reply from 10.10.1.10** — a visitor in the lobby can reach our Syslog
server. Also try pinging an Engineering PC → replies too.

🧠 That's the danger. Write in your notebook: "Before the ACL, a guest could reach ______".
Now close the door.

---

## Mission 8 — The guest-isolation ACL on A-DIST1 and A-DIST2
🧠 **Why both?** A-DIST2 is the HSRP **Active** for VLAN 120 — guest traffic enters there today.
A-DIST1 needs it too, for the day A-DIST2 fails.

### 8a — On SWL3-HFA-A-DIST2
👉 **Where:** A-DIST2 → CLI → log in `admin` / `Intel#Adm1n` → `SWL3-HFA-A-DIST2#`.

⌨️ Type — first the list:
```
configure terminal
ip access-list extended GUEST-IN
 remark --- DHCP requests to the gateway ---
 permit udp any any eq bootps
 remark --- DNS only to the ISP resolver ---
 permit udp any host 200.1.100.10 eq domain
 remark --- nothing internal (all private ranges) ---
 deny ip any 10.0.0.0 0.255.255.255
 deny ip any 172.16.0.0 0.15.255.255
 deny ip any 192.168.0.0 0.0.255.255
 remark --- everything else = the internet ---
 permit ip any any
 exit
```
🔍 **Check now:** `do show access-lists GUEST-IN` → 6 numbered lines (10, 20, 30…) in this order:
bootps, domain, three denies, permit ip any any.

❌ **If a line is in the wrong place:** `no ip access-list extended GUEST-IN` and type the whole
list again.

⌨️ Then apply it to the guest VLAN's gateway, **in**:
```
interface vlan 120
 ip access-group GUEST-IN in
 end
copy running-config startup-config
```

| Command | In simple English |
|---|---|
| `ip access-list extended GUEST-IN` | A detailed rule list (source + destination + protocol) named GUEST-IN |
| `permit udp any any eq bootps` | "Allow DHCP requests" (bootps = the DHCP server port 67) |
| `permit udp any host 200.1.100.10 eq domain` | "Allow DNS (port 53) only to that one server" |
| `deny ip any 10.0.0.0 0.255.255.255` | "Block anything going to any 10.x address" |
| `permit ip any any` | "Allow the rest" (without it the invisible deny-all would block the internet too) |
| `ip access-group GUEST-IN in` | "Check packets **coming in** from the guest VLAN with this list" |

### 8b — On SWL3-HFA-A-DIST1
👉 **Where:** A-DIST1 → CLI → log in → `SWL3-HFA-A-DIST1#`.

⌨️ Type the **same two blocks** (exactly as in 8a):
```
configure terminal
ip access-list extended GUEST-IN
 remark --- DHCP requests to the gateway ---
 permit udp any any eq bootps
 remark --- DNS only to the ISP resolver ---
 permit udp any host 200.1.100.10 eq domain
 remark --- nothing internal (all private ranges) ---
 deny ip any 10.0.0.0 0.255.255.255
 deny ip any 172.16.0.0 0.15.255.255
 deny ip any 192.168.0.0 0.0.255.255
 remark --- everything else = the internet ---
 permit ip any any
 exit
interface vlan 120
 ip access-group GUEST-IN in
 end
copy running-config startup-config
```
🔍 **Check now:** `show running-config interface vlan 120` → the line
`ip access-group GUEST-IN in` is there.

---

## Mission 9 — Test again and save

| # | From | Type | Expected |
|---|---|---|---|
| 1 | LT-HFA-EMP-01 | `ping 10.10.1.10` | ✅ employees reach the servers |
| 2 | LT-HFA-EMP-01 | `ping` an Engineering PC | ✅ |
| 3 | LT-HFA-GUEST-01 | `ipconfig /renew` then `ipconfig /all` | `10.10.21.x`, DNS `200.1.100.10` — DHCP still works |
| 4 | LT-HFA-GUEST-01 | `ping 10.10.1.10` | ❌ `Destination host unreachable` / timed out — **blocked** |
| 5 | LT-HFA-GUEST-01 | `ping` an Engineering PC | ❌ blocked |
| 6 | LT-HFA-GUEST-01 | `ping 200.1.100.11` | ✅ **only after step 12** (the internet doesn't exist yet) |

🔍 On **A-DIST2**: `show access-lists GUEST-IN` → the `deny ip any 10.0.0.0` line now shows
`(N match(es))` — those are your blocked pings.
🔍 On **ACC2**: `show port-security interface fa0/23` → the laptop's MAC counted on the AP port.

💾 **File → Save** (the DIST switches were saved with `copy run start`).

---

## Checkpoint
- [ ] Both SSIDs work; laptops get addresses from the right VLAN
- [ ] Employees reach internal servers; guests don't
- [ ] `GUEST-IN` applied `in` on VLAN 120 on **both** A-DIST switches

## If something doesn't work
| Problem | Fix |
|---|---|
| Laptop has no Wireless0 in Config | The WPC300N card isn't installed (Mission 3b) |
| No dashed radio line | SSID / key / encryption typo — they must match the AP **exactly** |
| Laptop connects but gets `169.254.x.x` | AP port in the wrong VLAN (`show vlan brief` on ACC2), or port-security on Fa0/22-23 too strict (step 06 set 30) |
| Guest reaches internal PCs | ACL applied only on DIST1, or applied `out` instead of `in` |
| Guest gets no address after the ACL | The `bootps` line is missing or below the deny lines |

---

## 🏋 Exercises

### Exercise A — The wrong order ⭐⭐
- 🎯 **Task:** prove that line order in an ACL matters.
- 📁 **Start from:** `step10.pkt` → **Save As** `step10-exA.pkt`.
- 👣 **Steps:**
  1. On **both** A-DIST switches: `configure terminal` → `no ip access-list extended GUEST-IN`.
  2. Type the list again, but put `permit ip any any` as the **first** line, then the rest.
  3. From LT-HFA-GUEST-01: `ping 10.10.1.10`.
- ✅ **Done when:** you can explain the result in one sentence.

<details><summary>💡 Hint</summary>

An ACL stops at the first line that matches. Which line matches **every** packet?
</details>
<details><summary>✔ Answer</summary>

The ping **works** — guests are not blocked. `permit ip any any` matches every packet, so the
deny lines below it are never read. Specific lines first, general lines last.
</details>

### Exercise B — WPA2-Enterprise for employees ⭐⭐⭐
- 🎯 **Task:** change INTEL-CORP to **WPA2-Enterprise** so each employee logs in with a personal
  account checked by SRV-HFA-AAA (RADIUS).
- 📁 **Start from:** `step10.pkt` → **Save As** `step10-exB.pkt`.
- 👣 **Steps:** think about **three** places: the AP, the AAA server, the laptop. What does each need?
- ✅ **Done when:** LT-HFA-EMP-01 connects with user `emp1` — or you can explain in the book why
  PT refused and what you tried.

<details><summary>💡 Hint</summary>

The AP must know **where** the RADIUS server is and the shared secret. The server must know the
AP as a client **and** have users. The laptop types a username instead of a passphrase.
</details>
<details><summary>✔ Answer</summary>

1. **AP-HFA-A-EMP → Port 1:** Authentication **WPA2** (Enterprise), RADIUS server `10.10.1.11`,
   shared secret `Intel#Rad`, AES.
2. **SRV-HFA-AAA → Services → AAA → Network Configuration:** client `AP-HFA-A-EMP`, the IP its
   requests come from, secret `Intel#Rad`, type **Radius** → Add. **User Setup:** `emp1` /
   `Emp#2026`, `emp2` / … → Add.
3. **Laptop → Config → Wireless0:** WPA2 (Enterprise), user `emp1`, password `Emp#2026`.

Packet Tracer's AccessPoint-PT has no IP address of its own, so this can be difficult. If it
won't authenticate, use a **WLC-2504 + LAP** design or a **WRT300N**. Write in the book which one
worked — it shows understanding.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

- [ ] Open `my-project/step09.pkt` → **Save As** `my-project/step10.pkt`
- [ ] Place 2 APs on the access ports of your WiFi VLANs; check `show vlan brief`
- [ ] Place laptops, swap their card to WPC300N
- [ ] Configure your SSIDs and keys; laptops get the right addresses
- [ ] Ping from the guest laptop **before** the ACL (see the danger)
- [ ] Write your guest ACL for your subnets; apply `in` on **both** gateways of the guest VLAN
- [ ] Test: employee reaches servers, guest doesn't

| In the example | In your project — take it from |
|---|---|
| SSIDs `INTEL-CORP`, `INTEL-GUEST` | your SSIDs (company name) |
| WiFi keys | plan part F |
| VLANs 110 / 120 and their subnets | plan part D (your WiFi VLANs) |
| Guest-isolation ACL | written for **your** guest subnet and **your** internal blocks |
| Wireless at HQ | plan part E (the ministry needs employee + guest WiFi in at least one site — req. 18) |

**Check:** an employee laptop reaches internal servers; a guest laptop reaches only the internet.

💾 Save as `my-project/step10.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★3 How can you use an ACL to let some computers connect but block access to others?**

<details><summary>Answer</summary>

An extended ACL on the gateway of the guest VLAN: deny from the guest subnet to the internal blocks, then permit everything else (the internet). Applied `in` on the guest SVI (`ip access-group GUEST-IN in`). This is the guest-isolation ACL of this step.

**Show it:** `show access-lists GUEST-IN`, `show running-config interface vlan <guest>`
</details>

**2. ◆ What is an SSID? Why two SSIDs?**

<details><summary>Answer</summary>

The name of a wireless network. Each SSID is mapped to its own VLAN — employees to the employee VLAN, guests to the guest VLAN — so guests are separated from the company (req. 18).

**Show it:** AP → Config → Wireless
</details>

**3. ◆ What is the difference between WPA2-Personal (PSK) and WPA2-Enterprise?**

<details><summary>Answer</summary>

Personal: everyone shares one password. Enterprise: every user logs in with their own username/password, checked by a RADIUS server — a leaving employee is just disabled.

**Show it:** AP → Config → Wireless → Authentication
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

In a copy of your file, delete the guest ACL and reset one AP. Without this page, configure the
SSIDs and keys from your plan and rewrite the guest-isolation ACL. Target: **10 minutes**.
Then show that a guest laptop reaches the internet but not an internal server.

---

## 👩‍🏫 Teacher notes
- **Lesson plan (~90 min):** 15 min AP/SSID/VLAN mapping + WiFi security table → 15 min guest
  isolation story ("a visitor in the lobby scanning your network") → 30 min Missions 1–6 (the
  laptop card swap takes time the first time) → Mission 7 together on the projector (the danger)
  → 15 min Mission 8 → 15 min Mission 9 + Exercise A.
- **Common student questions**
  - *"Why not one AP with two SSIDs?"* — Real APs do that (one SSID per VLAN on a trunk port). PT's
    simple AccessPoint-PT supports one SSID per AP, so we use two.
  - *"Why `in` and not `out`?"* — `in` on VLAN 120 = packets from guests, filtered at the first
    router they touch.
- **Typical errors:** forgetting the WPC300N swap; ACL only on one DIST; extended ACL line order
  (Exercise A shows it).
