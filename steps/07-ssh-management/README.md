# Step 07 — Secure management (SSH + management VLAN + ACL)

> ⏱ 1–2 lessons · 🎯 ministry requirements 13, 22.5 · 📁 start from: `my-work/step06.pkt` · 💾 save as: `my-work/step07.pkt`

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

You will **see** the difference yourself in Simulation mode (challenge).

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

---

## Part 2 — Prepare
**File → Open** → `my-work/step06.pkt` → **File → Save As** → `my-work/step07.pkt`.

## Part 3 — Build
Nothing new — the admin PC (`PC-HFA-ADMIN-01`, ACC1 Fa0/22, VLAN 99, 10.10.19.100, gw
10.10.19.1) has been waiting since step 02. Check its IP settings now.

---

## Part 4 — Configure

The configuration is **the same on all 4 switches** of building A:
`SW-HFA-A-ACC1`, `SW-HFA-A-ACC2`, `SWL3-HFA-A-DIST1`, `SWL3-HFA-A-DIST2`.
Type it on each one (4 times = you'll know it by heart).

### 4.1 Type this on each switch
```
enable
configure terminal

! ---------- 1. names for the encryption key ----------
! (the hostname is already set since step 02/03)
ip domain-name intel.lab

! ---------- 2. a personal admin user ----------
username admin privilege 15 secret Intel#Adm1n

! ---------- 3. the encryption keys ----------
crypto key generate rsa
```
The switch asks: `How many bits in the modulus [512]:` → type **`1024`** → Enter.
You see `[OK]` and `%SSH-5-ENABLED: SSH 1.99 has been enabled`. Continue:
```
! ---------- 4. SSH settings ----------
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
login block-for 120 attempts 3 within 60

! ---------- 5. the guest list: only the management network ----------
ip access-list standard MGMT-ONLY
 remark --- who may manage network devices ---
 permit 10.10.19.0 0.0.0.255
 deny any log
 exit

! ---------- 6. the remote doors: SSH only, usernames, guest list ----------
line vty 0 15
 transport input ssh
 login local
 access-class MGMT-ONLY in
 exec-timeout 5 0
 exit

! ---------- 7. the console also asks for the username ----------
line console 0
 login local
 end
copy running-config startup-config
```

> **PT note:** if `line vty 0 15` is refused, configure `line vty 0 4`, then repeat the same
> lines under `line vty 5 15`.

> After `line console 0` → `login local`, the console asks for **Username: admin** and
> **Password: Intel#Adm1n**. The old console password `Intel#Con` is no longer used.

### 4.2 What the new commands mean
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

## Part 5 — Check your work

### 5.1 SSH from the admin PC
1. Click **PC-HFA-ADMIN-01** → **Desktop** → **Command Prompt**.
2. Type: `ssh -l admin 10.10.19.11` (the `-l` is a small **L** = login name).
3. `Password:` → `Intel#Adm1n` (nothing shows while typing).
4. You're in:
```
*  Intel Israel - AUTHORIZED ACCESS ONLY ...
SW-HFA-A-ACC1#
```
5. Type `exit` to leave. Repeat for the other switches:

| Switch | Management IP |
|---|---|
| SW-HFA-A-ACC1 | 10.10.19.11 |
| SW-HFA-A-ACC2 | 10.10.19.12 |
| SWL3-HFA-A-DIST1 | 10.10.19.2 |
| SWL3-HFA-A-DIST2 | 10.10.19.3 |

### 5.2 The refusals (they prove the security works)
| From | Type | Expected | Why |
|---|---|---|---|
| PC-HFA-ADMIN-01 | `telnet 10.10.19.11` | ❌ `Connection refused` | `transport input ssh` |
| an ENG PC | `ssh -l admin 10.10.19.11` | ❌ `Connection refused` | ACL — source 10.10.16.x isn't on the list |
| an ENG PC | `ssh -l admin 10.10.16.2` (DIST1's address *in the user VLAN*!) | ❌ refused | The ACL checks **who is knocking**, so every address of the switch is protected |
| PC-HFA-ADMIN-01 | `ssh -l admin 10.10.19.11` with a wrong password 3 times quickly | blocked for 120 s | `login block-for` |

### 5.3 On the switch
```
show ip ssh                     -> SSH Enabled - version 2.0
show access-lists MGMT-ONLY     -> the "deny any log" line shows (N match(es)) after the ENG PC tried
show users                      -> who is connected right now (vty 0 admin ...)
show login                      -> the block-for settings and whether the switch is in "quiet mode"
```

---

## Part 6 — Save
`copy running-config startup-config` on all 4 → **File → Save**.

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
| SSH refused from the admin PC too | ACL name typo: `access-class` points to a name that doesn't exist = **deny all**. Compare `show run \| section vty` with `show access-lists` |
| `% Connection timed out` from the admin PC | No network path: can it `ping 10.10.19.11`? Check its IP/gateway, and that the switch has `interface vlan 99` up and `ip default-gateway` (step 05) |
| Locked out of everything | The console still works with `admin` / `Intel#Adm1n` |

## Challenge — see why Telnet is dangerous
1. On ACC1 allow Telnet **temporarily**: `line vty 0 15` → `transport input ssh telnet`.
2. Click **Simulation** → **Edit Filters** → only **Telnet** and **SSH**.
3. From PC-HFA-ADMIN-01: `telnet 10.10.19.11`, type the username and password.
4. Click the Telnet envelopes → look at the application layer. What do you see?
5. Do the same with `ssh -l admin 10.10.19.11`. Compare.
6. **Remove Telnet again:** `transport input ssh`. Back to **Realtime**.

<details><summary>Answer</summary>

In the Telnet packets you can read the characters you typed — including the username and
password, one letter per packet. In the SSH packets you only see encrypted data. That's why
requirement 13 asks for SSH, and why Telnet is never allowed in our network.
</details>

---

## 👩‍🏫 Teacher notes
- **Lesson plan (~90 min):** 15 min console vs remote, Telnet vs SSH (postcard vs sealed
  envelope) → 20 min ACLs and wildcard masks on the board (do 5 examples together) → 35 min
  configure the 4 switches → 20 min tests + the Telnet-sniffing challenge on the projector.
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
