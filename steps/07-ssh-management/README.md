# Step 07 — Secure management (SSH + management VLAN + ACL)

> ⏱ 1–2 lessons · 🎯 ministry requirements 13, 22.5 · 📁 start from: `my-work/step06.pkt` · 💾 save as: `my-work/step07.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 06 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## 🗺 Your missions in this step
- [ ] **Mission 1** — Open step 06, save as step 07, check the admin PC (5 min)
- [ ] **Mission 2** — ACC1: domain, user, encryption keys (10 min)
- [ ] **Mission 3** — ACC1: SSH settings, the guest list, the remote doors (10 min)
- [ ] **Mission 4** — Test SSH to ACC1 before you go on (5 min)
- [ ] **Mission 5** — The same on ACC2, DIST1, DIST2 (25 min)
- [ ] **Mission 6** — Test SSH to all four switches (10 min)
- [ ] **Mission 7** — Test the refusals (10 min)
- [ ] **Exercise** — see why Telnet is dangerous
- [ ] **🛠 Your own project**

## What you will have at the end
Every switch in building A can be managed **remotely**, but:
- only with **SSH** (encrypted) — never Telnet;
- only with a **personal username** and password;
- only **from the management network** (VLAN 99) — a PC in Engineering is refused even if it
  knows the password.

```
 PC-HFA-ADMIN-01 (10.10.19.100, VLAN 99)  ──SSH──►  any switch   ✅
 PC-HFA-ENG-01   (10.10.16.x,  VLAN 10)   ──SSH──►  any switch   ❌ refused by the ACL
```

---

## Part 1 — Understand it first (simple English)

### 1.1 Console vs remote management
- **Console** = standing next to the switch with a cable. Fine for a lab, impossible for 50
  switches in 3 cities.
- **Remote** = connecting over the network from the admin's desk. The switch has 16 virtual
  "remote doors" called **VTY lines** (`line vty 0 15`) — one per simultaneous remote session.

### 1.2 Telnet vs SSH
| | Telnet | SSH |
|---|---|---|
| Encrypted? | ❌ No — everything, **including the password**, travels as readable text | ✅ Yes |
| Comparison | A postcard: anyone handling it can read it | A sealed, locked envelope |
| Use it? | Never | Always |

You will **see** the difference yourself in Simulation mode (the exercise).

### 1.3 What SSH needs on a Cisco device
1. A **hostname** (not the default `Switch`) and a **domain name** (`ip domain-name intel.lab`)
   — together they name the encryption key.
2. **RSA keys** — the encryption keys (`crypto key generate rsa`, size **1024**).
3. A **username** with a password (`username admin ... secret ...`) — SSH logs in with a
   *user*, not just a password.
4. The VTY lines set to **accept SSH only** and to **check the local usernames**.

### 1.4 The management VLAN
All network devices have their management address in **VLAN 99** (`10.10.19.0/24` in
building A), separate from users. The admins' PCs sit in VLAN 99 too (later also the IT
department). Users never share a VLAN with the "control panel" of the network.

### 1.5 ACL — the guest list for the remote doors
An **ACL** (Access Control List) is a list of rules: *permit* or *deny*. The switch reads it
**top to bottom** and stops at the **first match**. At the end of every ACL there's an
invisible **"deny everything else"**.

We make a **standard** ACL (it checks only the **source** address — "who is knocking") and
attach it to the VTY lines with `access-class`.

**Wildcard masks — the ACL's way of writing a range.** It's the **opposite** of a subnet mask:
`0` = "this part must match", `255` = "anything goes".

| Wildcard | Means | Example |
|---|---|---|
| `0.0.0.0` | exactly this one address | `10.10.19.100 0.0.0.0` = only the admin PC |
| `0.0.0.255` | the whole /24 | `10.10.19.0 0.0.0.255` = 10.10.19.0 – 10.10.19.255 |
| `0.0.255.255` | the whole /16 | `10.10.0.0 0.0.255.255` = all of Haifa |

Easy rule for a /24: **255 minus each number of the subnet mask** → mask 255.255.255.0 →
wildcard 0.0.0.255.

### 1.6 Extra protections
| Setting | What it does |
|---|---|
| `login block-for 120 attempts 3 within 60` | 3 wrong passwords in 60 seconds → **all** logins refused for 120 seconds (stops password-guessing robots) |
| `exec-timeout 5 0` | Remote session closes after 5 idle minutes |
| `ip ssh version 2` | Only the newer, safer SSH version |
| `ip ssh authentication-retries 3` | 3 password tries per connection |
| `deny any log` | Every refused attempt is logged — you can see who tried |

### 1.7 The four switches and their management addresses
| Switch | Management IP (VLAN 99) |
|---|---|
| SW-HFA-A-ACC1 | 10.10.19.11 |
| SW-HFA-A-ACC2 | 10.10.19.12 |
| SWL3-HFA-A-DIST1 | 10.10.19.2 |
| SWL3-HFA-A-DIST2 | 10.10.19.3 |

---

## Mission 1 — Open step 06, save as step 07, check the admin PC
1. **File → Open** → `my-work/step06.pkt` → **File → Save As** → `my-work/step07.pkt` (M1).
2. Nothing new to build — the admin PC (`PC-HFA-ADMIN-01`, ACC1 Fa0/22, VLAN 99) has been
   waiting since step 02. Click it → **Desktop** → **IP Configuration** and check:
   **Static**, IP `10.10.19.100`, mask `255.255.255.0`, gateway `10.10.19.1`.
3. Its **Command Prompt** → `ping 10.10.19.11`.

✅ Replies. (No reply → fix this first: see step 05 Mission 8.)

---

## Mission 2 — ACC1: domain, user, encryption keys
👉 **Where:** **SW-HFA-A-ACC1** → CLI → log in (`Intel#Con`, `enable`, `Intel#En4ble`) →
`SW-HFA-A-ACC1#`.

### 2a — Domain name and admin user
⌨️ Type:
```
configure terminal
! names for the encryption key (the hostname is already set since step 02)
ip domain-name intel.lab
! a personal admin user with full rights
username admin privilege 15 secret Intel#Adm1n
```

### 2b — The encryption keys
⌨️ Type: `crypto key generate rsa` → **Enter**.

The switch asks: `How many bits in the modulus [512]:` → type **`1024`** → **Enter**.

✅ You see `[OK]` and `%SSH-5-ENABLED: SSH 1.99 has been enabled`.
❌ `% Please define a domain-name first` → you skipped 2a. Type `ip domain-name intel.lab`, then
try again.

---

## Mission 3 — ACC1: SSH settings, the guest list, the remote doors
👉 **Where:** ACC1, prompt `SW-HFA-A-ACC1(config)#`.

### 3a — SSH settings
⌨️ Type:
```
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
login block-for 120 attempts 3 within 60
```
> If `login block-for ...` gives `% Invalid input`, skip that one line on this switch —
> everything else still works.

### 3b — The guest list: only the management network
⌨️ Type:
```
ip access-list standard MGMT-ONLY
 remark --- who may manage network devices ---
 permit 10.10.19.0 0.0.0.255
 deny any log
 exit
```
✅ After the first line the prompt is `(config-std-nacl)#`; after `exit`, back to `(config)#`.

### 3c — The remote doors: SSH only, usernames, guest list
⌨️ Type:
```
line vty 0 15
 transport input ssh
 login local
 access-class MGMT-ONLY in
 exec-timeout 5 0
 exit
```
> **PT note:** if `line vty 0 15` is refused, type the same four lines under `line vty 0 4`,
> then again under `line vty 5 15`.

### 3d — The console also asks for the username
⌨️ Type:
```
line console 0
 login local
 end
copy running-config startup-config
```
✅ `[OK]`.

> From now on the console asks for **Username: admin** and **Password: Intel#Adm1n**. The old
> console password `Intel#Con` is no longer used.

---

## Mission 4 — Test SSH to ACC1 before you go on
🧠 **Why:** if ACC1 works, you know the recipe is right. Then copy it to the other three.

1. Click **PC-HFA-ADMIN-01** → **Desktop** → **Command Prompt** (M17).
2. Type: `ssh -l admin 10.10.19.11` (the `-l` is a small **L** = login name).
3. `Password:` → `Intel#Adm1n` (nothing shows while typing) → **Enter**.

✅ You're in:
```
*  Intel Israel - AUTHORIZED ACCESS ONLY ...
SW-HFA-A-ACC1#
```
4. Type `exit` to leave.

❌ `% Connection refused by remote host` → the ACL name in `access-class` doesn't match the ACL
(a typo = deny all). On ACC1: `show running-config` → compare the `line vty` part with
`show access-lists`.
❌ `% Connection timed out` → no network path. Does `ping 10.10.19.11` work?

---

## Mission 5 — The same on ACC2, DIST1, DIST2
The configuration is **identical** on every switch of building A. Do Missions 2 and 3 again on
each of the three switches below. (After 4 times you'll know it by heart.)

For each switch: 👉 click it → CLI → log in (`Intel#Con` → `enable` → `Intel#En4ble`) → then type:

⌨️ The full block (Missions 2 + 3 together):
```
configure terminal
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n
crypto key generate rsa
```
→ answer **`1024`** → then:
```
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
login block-for 120 attempts 3 within 60
ip access-list standard MGMT-ONLY
 remark --- who may manage network devices ---
 permit 10.10.19.0 0.0.0.255
 deny any log
 exit
line vty 0 15
 transport input ssh
 login local
 access-class MGMT-ONLY in
 exec-timeout 5 0
 exit
line console 0
 login local
 end
copy running-config startup-config
```
Tick each switch when it shows `[OK]`:
- [ ] SW-HFA-A-ACC2
- [ ] SWL3-HFA-A-DIST1
- [ ] SWL3-HFA-A-DIST2

### What the new commands mean
| Command | In simple English |
|---|---|
| `ip domain-name intel.lab` | The company's domain — part of the key's name |
| `username admin privilege 15 secret Intel#Adm1n` | Create user `admin` with full rights (15 = highest), password hashed |
| `crypto key generate rsa` | Create the encryption keys for SSH |
| `ip ssh version 2` | Only allow SSH version 2 |
| `ip access-list standard MGMT-ONLY` | Start a guest list called MGMT-ONLY (checks the source address) |
| `permit 10.10.19.0 0.0.0.255` | "Anyone from 10.10.19.x may knock" |
| `deny any log` | "Everyone else: refused, and write it down" |
| `line vty 0 15` | The 16 remote doors |
| `transport input ssh` | "Only SSH comes through these doors" |
| `login local` | "Check the username/password in my local user list" |
| `access-class MGMT-ONLY in` | "Check the guest list before opening the door" |

---

## Mission 6 — Test SSH to all four switches
From **PC-HFA-ADMIN-01** → Command Prompt, for each row: `ssh -l admin <IP>` → `Intel#Adm1n`
→ check the prompt → `exit`.

| # | Type | ✅ You should land on |
|---|---|---|
| 1 | `ssh -l admin 10.10.19.11` | `SW-HFA-A-ACC1#` |
| 2 | `ssh -l admin 10.10.19.12` | `SW-HFA-A-ACC2#` |
| 3 | `ssh -l admin 10.10.19.2` | `SWL3-HFA-A-DIST1#` |
| 4 | `ssh -l admin 10.10.19.3` | `SWL3-HFA-A-DIST2#` |

📸 Screenshot one successful SSH login for your book (M22).

---

## Mission 7 — Test the refusals (they prove the security works)
| # | From | Type | Expected | Why |
|---|---|---|---|---|
| 1 | PC-HFA-ADMIN-01 | `telnet 10.10.19.11` | ❌ `Connection refused` | `transport input ssh` |
| 2 | an ENG PC | `ssh -l admin 10.10.19.11` | ❌ `Connection refused` | ACL — source 10.10.16.x isn't on the list |
| 3 | an ENG PC | `ssh -l admin 10.10.16.2` (DIST1's address *in the user VLAN*!) | ❌ refused | The ACL checks **who is knocking**, so every address of the switch is protected |
| 4 | PC-HFA-ADMIN-01 | `ssh -l admin 10.10.19.11` with a wrong password 3 times quickly | blocked for 120 s | `login block-for` |

✅ All four refused as expected.

👉 Then on **ACC1** (log in at the console with `admin` / `Intel#Adm1n`):
```
show ip ssh                     -> SSH Enabled - version 2.0
show access-lists MGMT-ONLY     -> the "deny any log" line shows (N match(es)) after the ENG PC tried
show users                      -> who is connected right now (vty 0 admin ...)
```
Finally: **File → Save**.

---

## Checkpoint
- [ ] SSH works from the admin PC to all 4 switches with `admin` / `Intel#Adm1n`
- [ ] Telnet is refused
- [ ] SSH from a user PC is refused
- [ ] The console now asks for a username

## If something doesn't work
| Problem | Fix |
|---|---|
| `crypto key generate rsa` → `% Please define a domain-name first` | Type `ip domain-name intel.lab` first |
| `% Please define a hostname other than Switch` | Set the hostname first |
| SSH refused from the admin PC too | ACL name typo: `access-class` points to a name that doesn't exist = **deny all**. Compare the `line vty` part of `show running-config` with `show access-lists` |
| `% Connection timed out` from the admin PC | No network path: can it `ping 10.10.19.11`? Check its IP/gateway, and that the switch has `interface vlan 99` up and `ip default-gateway` (step 05) |
| Locked out of everything | The console still works with `admin` / `Intel#Adm1n` |

---

## 🏋 Exercise — See why Telnet is dangerous ⭐⭐
- 🎯 **Task:** capture a Telnet login and an SSH login in Simulation mode, and find the password.
- 📁 **Start from:** your `step07.pkt` — **Save As** `step07-telnet.pkt` first.
- 👣 **Steps:**
  1. On ACC1 allow Telnet **temporarily**: `configure terminal` → `line vty 0 15` →
     `transport input ssh telnet` → `end`.
  2. Click **Simulation** → **Show All/None** to clear the filters → **Edit Filters** → tick only
     **Telnet** and **SSH**. Close the filter window (M21).
  3. From PC-HFA-ADMIN-01: `telnet 10.10.19.11`, type the username and password. Press
     **Capture / Forward** to move the packets.
  4. Click the Telnet envelopes → look at the application layer (the top layer). What do you see?
  5. Do the same with `ssh -l admin 10.10.19.11`. Compare.
  6. **Remove Telnet again:** `line vty 0 15` → `transport input ssh`. Back to **Realtime**.
- ✅ **Done when:** you found (or didn't find) the password in each, and can say in one sentence
  why the project forbids Telnet.

<details><summary>💡 Hint</summary>

Telnet sends **one letter per packet**. Open several envelopes one after the other.
</details>
<details><summary>✔ Answer</summary>

In the Telnet packets you can read the characters you typed — including the username and
password, one letter per packet. In the SSH packets you only see encrypted data. That's why
requirement 13 asks for SSH, and why Telnet is never allowed in our network.

Go back to `step07.pkt` for step 08.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

- [ ] Open `my-project/step06.pkt` → **Save As** `my-project/step07.pkt`
- [ ] Check your admin PC: static, in your MGMT VLAN, pings a switch's management IP
- [ ] On the first switch: domain, user, keys, SSH settings, ACL, VTY, console → test SSH
- [ ] The same on every other switch of the building
- [ ] Mission 6 and Mission 7 tests pass in your project

| In the example | In your project — take it from |
|---|---|
| Management VLAN 99 and its subnet | plan part D (your MGMT VLAN) |
| Switch management IPs `.11`, `.12`, admin PC `.100` | plan part D (fixed addresses) |
| `ip domain-name intel.lab` | plan part A (your domain name) |
| User `admin` / `Intel#Adm1n` | plan part F (your local admin user) |
| VTY ACL permitting the admin PC / management subnet | your admin PC address / your MGMT subnet |

**Check:** SSH from your admin PC works; SSH from a user PC is refused; Telnet is refused.

💾 Save as `my-project/step07.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ★1 What is SSH? Which protocols does it use for encryption? Show the configuration on a switch or router.**

<details><summary>Answer</summary>

Secure Shell — remote management over an **encrypted** session, TCP port 22 (Telnet, port 23, is clear text). An RSA key pair (`crypto key generate rsa`) is used to authenticate the device and exchange keys; the session itself is encrypted with a symmetric cipher (e.g. AES). Needed lines: `hostname`, `ip domain-name`, `crypto key generate rsa`, `ip ssh version 2`, a local user, `line vty` → `login local` + `transport input ssh`.

**Show it:** `show ip ssh`, `show running-config` → `line vty`
</details>

**2. ★1 What is the difference between a standard and an extended access list?**

<details><summary>Answer</summary>

Standard (1–99) checks only the **source** IP; place it close to the destination. Extended (100–199) checks source **and** destination, protocol and port; place it close to the source.

**Show it:** `show access-lists`
</details>

**3. ★1 Show me where you configured a standard access list.**

<details><summary>Answer</summary>

The VTY ACL: it permits only the management subnet / admin PC to open SSH sessions, applied with `access-class <name> in` under `line vty`.

**Show it:** `show access-lists`, `show running-config` → `line vty`
</details>

**4. ★1 In what order is an access list processed? How do you check that it works?**

<details><summary>Answer</summary>

Top to bottom; the first line that matches decides, and the rest is ignored. At the end there is an invisible **deny any**. Check with `show access-lists` — the match counters grow when you test.

**Show it:** `show access-lists` before and after an SSH attempt
</details>

**5. ◆ Why a separate management VLAN?**

<details><summary>Answer</summary>

Management traffic (SSH to network devices) is kept apart from users. Users can't even reach the devices' management addresses, and the ACL can allow just this one subnet (req. 22.5).

**Show it:** `show ip interface brief` → `Vlan99`
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

On a fresh switch in a copy of your file: without this page, set the management SVI and default
gateway, the domain name, RSA keys, SSH v2, the local admin user, the VTY lines and the VTY ACL
from your plan. Target: **10 minutes**. Then SSH from the admin PC (works) and from a user PC
(refused), and explain why.

---

## 👩‍🏫 Teacher notes
- **Lesson plan (~90 min):** 15 min console vs remote, Telnet vs SSH (postcard vs sealed
  envelope) → 20 min ACLs and wildcard masks on the board (do 5 examples together) → 35 min
  Missions 2–5 → 20 min Missions 6–7 + the Telnet-sniffing exercise on the projector.
- **Insist on Mission 4.** Students who configure all four switches before testing one end up
  with four copies of the same typo.
- **Wildcard practice for the board:** "write the wildcard for 10.10.35.0/24" (0.0.0.255),
  "for only 10.10.19.100" (0.0.0.0), "for all of 10.10.0.0/16" (0.0.255.255).
- **Common student questions**
  - *"Why does the user PC get refused even with the right password?"* — The ACL is checked
    **before** the password prompt. Wrong source = the door doesn't even open.
  - *"Why a username and not just a password?"* — Accountability: logs show **who** did what.
    In step 08 the usernames move to a central AAA server.
  - *"What does privilege 15 mean?"* — Levels 0–15; 15 = full control (lands directly in `#`).
- **Typical errors:** forgetting the domain name before generating keys; ACL name typo;
  testing from a PC that isn't in VLAN 99.
- **Later update:** in step 09, when buildings B and C and the IT department exist, the ACL is
  extended to `10.10.35.0`, `10.10.51.0` and `10.10.49.0` (IT).
