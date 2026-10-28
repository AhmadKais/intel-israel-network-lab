# Step 02 — First switch: hardening & VLANs

> ⏱ 1–2 lessons · 🎯 ministry requirements 22.1, 22.2, 22.3, 22.6 · 📁 start from: `my-work/step01.pkt` · 💾 save as: `my-work/step02.pkt`

> ✅ **Before you start:** every box in the checkpoint of step 01 is ticked, and you did its
> 🔁 drill in your own project. Don't skip steps — each step builds on the file of the one before.

## 🗺 Your missions in this step
- [ ] **Mission 1** — Open step 01 and save as step 02 (2 min)
- [ ] **Mission 2** — Place and name the switch (5 min)
- [ ] **Mission 3** — Place and name 5 PCs and a printer (10 min)
- [ ] **Mission 4** — Cable them to the exact ports (10 min)
- [ ] **Mission 5** — Harden the switch: name, passwords, banner (15 min)
- [ ] **Mission 6** — Create the VLANs (5 min)
- [ ] **Mission 7** — Put each port in its VLAN, park the unused ports (10 min)
- [ ] **Mission 8** — Give the PCs and the printer their addresses (10 min)
- [ ] **Mission 9** — Test: who can talk to whom? (10 min)
- [ ] **Mission 10** — Test the passwords and save (5 min)
- [ ] **Exercises** A and B
- [ ] **🛠 Your own project**

## What you will have at the end
One access switch in Haifa building A, protected with the basic security settings every
device in the project needs, with one VLAN per department. PCs in the **same** VLAN can talk;
PCs in **different** VLANs can't (yet — on purpose).

```
                        SW-HFA-A-ACC1  (2960)
   Fa0/1  Fa0/2     Fa0/11  Fa0/12     Fa0/21        Fa0/22       Fa0/23-24
     │      │          │       │          │             │             ✖ (shut)
  ENG-01 ENG-02     FIN-01  FIN-02    PRN-A-01     ADMIN-01
  └─ VLAN 10 ─┘     └─ VLAN 20 ─┘    VLAN 30       VLAN 99     VLAN 999 (parking)
```

---

## Part 1 — Understand it first (simple English)

### 1.1 What is a VLAN?
Without VLANs, a switch is **one big room**: every device hears every "shout" (broadcast)
of every other device, and anyone can talk to anyone.

A **VLAN (Virtual LAN)** cuts one physical switch into several **separate virtual
switches**. Imagine one office floor divided by walls into closed rooms: Engineering in
one room, Finance in another. They share the same building (switch), but:
- a broadcast in Engineering **never** reaches Finance;
- an Engineering PC **cannot** talk to a Finance PC directly — it needs a router (step 05),
  and that's where we can later control who may talk to whom.

| Why companies use VLANs | Example |
|---|---|
| **Security** | Guests can't see the Finance PCs |
| **Less noise** | Fewer devices per broadcast domain → less wasted traffic |
| **Organisation** | Move a person's desk anywhere — just put their port in their department's VLAN |

Each VLAN has a **number** (1–4094) and a **name**. In our design each VLAN = one IP subnet
(VLAN 10 = `10.10.16.0/24`, VLAN 20 = `10.10.17.0/24`…).

### 1.2 Port names and access ports
`Fa0/1` = **F**ast **E**thernet (100 Mb/s), slot **0**, port **1**. `Gi0/1` = **Gi**gabit (1000 Mb/s).
The 2960 has 24 Fa ports for devices and 2 Gi ports for uplinks.

An **access port** belongs to exactly **one** VLAN and connects **one end device** (PC,
printer). The PC doesn't know VLANs exist — the switch decides which "room" it's in.

### 1.3 Hardening — the basic protection every device gets
"Hardening" = closing the easy doors before an attacker finds them.

| Setting | What it does | Why |
|---|---|---|
| `hostname` | Gives the device a name | Logical names (req. 22.1); also needed later for SSH |
| `enable secret` | Password to enter `#` mode, stored as a **hash** (scrambled one way, can't be turned back) | Anyone reading the config can't see it (req. 22.3) |
| `service password-encryption` | Scrambles the **other** passwords (console etc.) | A printed config doesn't show them in clear text. Weak (type 7), but better than nothing |
| `banner motd` | Message shown before login | Legal warning: "authorised access only". Needed to prosecute an intruder (req. 22.2) |
| `line console 0` + `password` + `login` | Password on the physical console port | Someone standing next to the switch still needs a password |
| `no ip domain-lookup` | Stops the switch treating typos as hostnames | Saves 30 seconds of waiting after every typo |
| `exec-timeout 10 0` | Logs out after 10 minutes without typing | An admin who walks away doesn't leave an open session |
| `logging synchronous` | Messages don't cut into the line you are typing | Comfort |

> `secret` vs `password`: **always use `secret`** where it exists. `password` is stored in
> clear text (or weak type 7).

### 1.4 Unused ports — the "parking" VLAN
An empty port in a meeting room is an open door: anyone can plug in a laptop. We put every
unused port in VLAN 999 (which leads nowhere) **and** shut it down. To use it, an admin has
to deliberately turn it on.

### 1.5 The port plan (why these ports?)
Every access switch in building A follows the same plan, so anybody can guess where a
device is plugged in:

| Ports | VLAN | Use |
|---|---|---|
| Fa0/1–10 | 10 ENGINEERING | users |
| Fa0/11–20 | 20 FINANCE | users |
| Fa0/21 | 30 PRINTERS-A | printer |
| Fa0/22 | 99 MGMT-A | admin PC (on ACC1) |
| Fa0/23–24 | 999 PARKING | unused, shut down |
| Gi0/1, Gi0/2 | trunks to DIST1 / DIST2 | step 03 |

---

## Mission 1 — Open step 01 and save as step 02 (how? → M1)
1. Packet Tracer → **File → Open** → `my-work/step01.pkt` → **Open**.
2. **File → Save As** → `step02.pkt` → **Save**.

✅ The title bar shows `step02.pkt`. (Now step01 stays untouched — if you break something you
can always go back.)

---

## Mission 2 — Place and name the switch
1. Bottom left: **Network Devices** → **Switches** (how? → M2).
2. Drag a **2960** (`2960-24TT`) into the **Building A** rectangle.
3. Click it → **Config** tab → **Display Name**: type `SW-HFA-A-ACC1` → close the window
   (how? → M4).

✅ The label under the switch says `SW-HFA-A-ACC1`.

---

## Mission 3 — Place and name 5 PCs and a printer
1. Bottom left: **End Devices** → **End Devices**.
2. Hold **Ctrl**, click the **PC** icon once, let go of Ctrl, then click 5 times **under** the
   switch → 5 PCs appear. Press **Esc** (how? → M3).
3. Drag one **Printer** next to them.
4. Rename each one (click → **Config** → **Display Name**). Go from left to right:

| Rename | to |
|---|---|
| first PC | `PC-HFA-ENG-01` |
| second PC | `PC-HFA-ENG-02` |
| third PC | `PC-HFA-FIN-01` |
| fourth PC | `PC-HFA-FIN-02` |
| fifth PC | `PC-HFA-ADMIN-01` |
| printer | `PRN-HFA-A-01` |

✅ Six end devices under the switch, each with its new name.

---

## Mission 4 — Cable them to the exact ports
🧠 **Why:** the port decides the VLAN. A PC on the wrong port lands in the wrong department.

PC ↔ switch = different device types → **Copper Straight-Through** every time (how? → M5).
For each row: click **Connections** (⚡) → **Copper Straight-Through** → click the first device →
choose its port → click the switch → choose its port.

| # | Click first | Port | Click second | Port |
|---|---|---|---|---|
| 1 | PC-HFA-ENG-01 | FastEthernet0 | SW-HFA-A-ACC1 | **FastEthernet0/1** |
| 2 | PC-HFA-ENG-02 | FastEthernet0 | SW-HFA-A-ACC1 | **FastEthernet0/2** |
| 3 | PC-HFA-FIN-01 | FastEthernet0 | SW-HFA-A-ACC1 | **FastEthernet0/11** |
| 4 | PC-HFA-FIN-02 | FastEthernet0 | SW-HFA-A-ACC1 | **FastEthernet0/12** |
| 5 | PRN-HFA-A-01 | FastEthernet0 | SW-HFA-A-ACC1 | **FastEthernet0/21** |
| 6 | PC-HFA-ADMIN-01 | FastEthernet0 | SW-HFA-A-ACC1 | **FastEthernet0/22** |

Then press **Fast Forward Time** 2–3 times (how? → M18).

✅ Six cables, all dots green. The labels next to the switch read `Fa0/1`, `Fa0/2`, `Fa0/11`,
`Fa0/12`, `Fa0/21`, `Fa0/22`.
❌ A label shows another port → delete that cable (M7) and connect it again.
❌ No labels on the cables → Options → Preferences → Interface → tick **Always Show Port Labels**.

---

## Mission 5 — Harden the switch: name, passwords, banner
👉 **Where:** click **SW-HFA-A-ACC1** → **CLI** tab → click in the black area → **Enter**.
You must see `Switch>` (how? → M9).

> Type **each line** and press **Enter** (how? → M10). Lines starting with `!` are comments —
> don't type them. If you see `% Invalid input detected at '^' marker` — press **↑**, fix the
> line, press **Enter** (M12). Keyboard on **English**, Caps Lock off. Passwords are
> case-sensitive: `Intel#En4ble` ≠ `intel#en4ble`.

### 5a — Enter configuration mode
⌨️ Type:
```
enable
configure terminal
```
✅ The prompt is now `Switch(config)#`.

### 5b — Name and passwords
⌨️ Type:
```
! give the switch its logical name - the prompt changes immediately
hostname SW-HFA-A-ACC1
! stop the switch trying to resolve typos as hostnames
no ip domain-lookup
! the password for "enable" (# mode), stored as a hash
enable secret Intel#En4ble
! scramble every other password in the config
service password-encryption
```
✅ After the `hostname` line the prompt changed to `SW-HFA-A-ACC1(config)#`.

### 5c — The banner
1. Type `banner motd #` and press **Enter**.
2. The switch answers `Enter TEXT message. End with the character '#'.`
3. Type the text lines below, pressing **Enter** after each.
4. Finish with a line that has **only** `#` → **Enter**.
```
banner motd #
*********************************************************
*  Intel Israel - AUTHORIZED ACCESS ONLY                *
*  All activity is logged. Disconnect now if you are    *
*  not an authorized network administrator.             *
*********************************************************
#
```
✅ The prompt is back to `SW-HFA-A-ACC1(config)#`.
❌ The prompt doesn't come back → you didn't type the closing `#` line. Type `#` and Enter.

> Write the banner text in English (IOS can't show Hebrew or Arabic). The `#` character must
> not appear inside the text — it ends the message.

### 5d — The console password
⌨️ Type:
```
line console 0
 password Intel#Con
 login
 logging synchronous
 exec-timeout 10 0
 exit
```
✅ After `line console 0` the prompt was `(config-line)#`; after `exit` it is back to `(config)#`.

---

## Mission 6 — Create the VLANs
👉 **Where:** SW-HFA-A-ACC1, prompt `SW-HFA-A-ACC1(config)#`.

Each `vlan` command enters the VLAN's sub-mode `(config-vlan)#`; the `name` line labels it.
You can go from one `vlan` straight to the next without `exit`.

⌨️ Type:
```
vlan 10
 name ENGINEERING
vlan 20
 name FINANCE
vlan 30
 name PRINTERS-A
vlan 99
 name MGMT-A
vlan 110
 name WIFI-EMP
vlan 120
 name WIFI-GUEST
vlan 998
 name NATIVE
vlan 999
 name PARKING
 exit
```
🔍 **Check now:** type `do show vlan brief`.

✅ VLANs 10, 20, 30, 99, 110, 120, 998, 999 are listed with their **names** (no ports yet —
they come in Mission 7).
❌ A name shows as `VLAN0010` → type `vlan 10` → `name ENGINEERING` again.

---

## Mission 7 — Put each port in its VLAN, park the unused ports
👉 **Where:** SW-HFA-A-ACC1, prompt `SW-HFA-A-ACC1(config)#`.

`interface range` configures many ports at once — the prompt becomes `(config-if-range)#`.

### 7a — Users, printer, admin
⌨️ Type:
```
interface range fa0/1 - 10
 description ENGINEERING users
 switchport mode access
 switchport access vlan 10
interface range fa0/11 - 20
 description FINANCE users
 switchport mode access
 switchport access vlan 20
interface fa0/21
 description PRN-HFA-A-01
 switchport mode access
 switchport access vlan 30
interface fa0/22
 description PC-HFA-ADMIN-01 (management)
 switchport mode access
 switchport access vlan 99
```
> ⚠️ Type `fa0/1 - 10` **with a space before and after the dash**, exactly as shown. That
> form works in every version.

### 7b — Park and shut the unused ports
⌨️ Type:
```
interface range fa0/23 - 24
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 end
```
✅ You see messages like `%LINK-5-CHANGED: Interface FastEthernet0/23, changed state to
administratively down` — that's the `shutdown` working. The prompt is `SW-HFA-A-ACC1#`.

### 7c — Check
⌨️ Type: `show vlan brief`

✅ You should see:
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gig0/1, Gig0/2
10   ENGINEERING                      active    Fa0/1, Fa0/2, Fa0/3, Fa0/4 ... Fa0/10
20   FINANCE                          active    Fa0/11, Fa0/12 ... Fa0/20
30   PRINTERS-A                       active    Fa0/21
99   MGMT-A                           active    Fa0/22
110  WIFI-EMP                         active
120  WIFI-GUEST                       active
998  NATIVE                           active
999  PARKING                          active    Fa0/23, Fa0/24
```
(At the end of the list you will also see four built-in VLANs: 1002, 1003, 1004, 1005 —
`fddi-default` etc. Every Cisco switch has them, they can't be deleted, and you can ignore them.)

VLAN 1 has **only** the two Gi uplinks (they become trunks in step 03).

❌ A port is in the wrong VLAN → `configure terminal` → `interface fa0/X` →
`switchport access vlan <right number>` → `end`.

### 7d — Save the switch (how? → M13)
⌨️ Type: `copy running-config startup-config` → it asks `Destination filename [startup-config]?`
→ press **Enter**.

✅ `[OK]`.

### What the new commands mean
| Command | In simple English |
|---|---|
| `vlan 10` → `name ENGINEERING` | "Create room number 10 and put a sign on the door: ENGINEERING" |
| `interface range fa0/1 - 10` | "The next commands apply to ports 1 to 10 at once" |
| `description ...` | A label for humans — shows in `show` commands, does nothing technical |
| `switchport mode access` | "This port is for one end device, never a trunk." Also blocks an attack where a laptop pretends to be a switch (req. 13) |
| `switchport access vlan 10` | "Whatever is plugged in here is in room 10" |
| `shutdown` / `no shutdown` | Turn the port off / on |
| `end` | Jump straight back to `#` mode |
| `copy running-config startup-config` | Copy the whiteboard into the notebook (step 00) |

---

## Mission 8 — Give the PCs and the printer their addresses

### 8a — The five PCs (how? → M14)
For each PC: click it → **Desktop** → **IP Configuration** → **Static** → fill in → close.

| Device | IPv4 Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC-HFA-ENG-01 | 10.10.16.101 | 255.255.255.0 | (leave empty for now) |
| PC-HFA-ENG-02 | 10.10.16.102 | 255.255.255.0 | (empty) |
| PC-HFA-FIN-01 | 10.10.17.101 | 255.255.255.0 | (empty) |
| PC-HFA-FIN-02 | 10.10.17.102 | 255.255.255.0 | (empty) |
| PC-HFA-ADMIN-01 | 10.10.19.100 | 255.255.255.0 | 10.10.19.1 (permanent) |

The user PCs get temporary addresses now. In step 05 they switch to **DHCP** (automatic).

### 8b — The printer (how? → M15)
**The printer has no Desktop tab.**
1. Click it → **Config** tab → on the left **Settings** (under GLOBAL) → in the **Gateway**
   field type `10.10.18.1`.
2. On the left click **FastEthernet0** (under INTERFACE) → choose **Static** → **IPv4 Address**
   `10.10.18.10`, **Subnet Mask** `255.255.255.0`.
3. Close the window.

✅ Hover over each device — the box shows the address from the table.

---

## Mission 9 — Test: who can talk to whom?
For each row: click the "From" PC → **Desktop** → **Command Prompt** → type the command
(how? → M17).

| # | From | Type | Expected | Why |
|---|---|---|---|---|
| 1 | PC-HFA-ENG-01 | `ping 10.10.16.102` | ✅ replies | same VLAN |
| 2 | PC-HFA-FIN-01 | `ping 10.10.17.102` | ✅ replies | same VLAN |
| 3 | PC-HFA-ENG-01 | `ping 10.10.17.101` | ❌ fails | different VLAN, no router yet |

✅ Tests 1 and 2 reply; test 3 fails. **All three results are correct** — test 3 is supposed
to fail.
❌ Test 1 or 2 fails → on the switch `show vlan brief`: is each PC's port in the right VLAN?
Hover over the cable: is it on the right port? Check the PC's IP and mask.

### What just happened? Why does ENG → FIN fail?
Two reasons, both on purpose:
1. **Different VLANs** — the switch keeps rooms 10 and 20 completely apart. A frame from
   VLAN 10 is never delivered to a port in VLAN 20.
2. **Different subnets** — `10.10.16.x` and `10.10.17.x` are different "streets". The PC knows
   it must use a **gateway** (router) for another street — and there is none yet.

Departments are isolated by default. In step 05 we add routing and decide *deliberately*
which traffic may cross.

---

## Mission 10 — Test the passwords and save
1. In the switch CLI type `exit` until you see `Press RETURN to get started.` Press **Enter**.

✅ You see the **banner**, then `User Access Verification` → `Password:`.

2. Type `Intel#Con` → **Enter** (nothing appears while you type — that's normal).
3. Type `enable` → `Password:` → `Intel#En4ble` → **Enter**.

✅ The prompt is `SW-HFA-A-ACC1#`.

4. Type `show running-config` (press **Space** to page down). Find:
   - `enable secret 5 $1$...` → hashed ✅
   - under `line con 0`: `password 7 0822455D0A16...` → scrambled ✅
     (The digits after `7` may be different for you — what matters is the `7` and that the
     clear-text password is gone.)
5. Save the file: **File → Save** (`my-work/step02.pkt`).

---

## Checkpoint
- [ ] `show vlan brief` matches Mission 7c
- [ ] Same-VLAN pings work, cross-VLAN pings fail
- [ ] Banner, console password and enable secret all work
- [ ] Device names on the workspace follow the naming rule
- [ ] Switch saved (`copy run start`) **and** file saved (`step02.pkt`)

## If something doesn't work
| Problem | Fix |
|---|---|
| Ping in the same VLAN fails | `show vlan brief`: is the PC's port in the right VLAN? Check the cable goes to the right port (port labels!). Check the PC's IP/mask |
| `% Invalid input` on `interface range` | Type it exactly with spaces: `interface range fa0/1 - 10` |
| VLAN name shows `VLAN0010` | You assigned a port before creating/naming the VLAN — go back: `vlan 10` → `name ENGINEERING` |
| Banner never ends | You must type a line with just `#` |
| Password not accepted | Check the keyboard is on English and Caps Lock is off. Passwords are case-sensitive |
| Forgot the console password | Click **Physical** → power off/on won't help (it's saved). Re-open your saved `step01.pkt` and redo (M23), or practise "password recovery" with the teacher |

---

## 🏋 Exercises

### Exercise A — The locked door ⭐
- 🎯 **Task:** find out what happens when someone plugs a new PC into port **Fa0/24**, and why.
- 📁 **Start from:** your `step02.pkt`.
- 👣 **Steps:**
  1. Place a new PC and connect it to SW-HFA-A-ACC1 **Fa0/24** (Straight-Through).
  2. Look at the cable's dots. Press Fast Forward Time.
  3. In the switch CLI (`#` mode) run `show interfaces fa0/24 switchport`.
  4. Find the line that proves the port is in VLAN 999.
- ✅ **Done when:** you can answer in two sentences: does the new PC work? What must an admin
  do to let it work?

<details><summary>💡 Hint 1</summary>

Look at the colour of the dot on the switch side. What did you type on fa0/23 - 24 in Mission 7b?
</details>
<details><summary>💡 Hint 2</summary>

In the output, look for a line starting with `Access Mode VLAN`.
</details>
<details><summary>✔ Answer</summary>

1. No — the dot stays red: the port is `administratively down` (shut down) and in the parking
   VLAN. The admin must move it to the right VLAN and type `no shutdown`. Nobody can just walk
   in and plug a laptop in.
2. `Access Mode VLAN: 999 (PARKING)`

Delete the extra PC before you continue (M7).
</details>

### Exercise B — Move a desk ⭐⭐
- 🎯 **Task:** PC-HFA-ENG-02 moves to the Finance department. Without touching any cable, put
  it in Finance and prove it can now reach PC-HFA-FIN-01.
- 📁 **Start from:** your `step02.pkt` (save a copy first: **File → Save As** →
  `step02-exB.pkt`).
- 👣 **Steps:**
  1. Which port is PC-HFA-ENG-02 on? (Mission 4.)
  2. Change that port's VLAN on the switch.
  3. Change the PC's address to a free Finance address (e.g. `10.10.17.103`).
  4. Ping PC-HFA-FIN-01.
- ✅ **Done when:** `show vlan brief` shows that port under VLAN 20, and the ping replies.

<details><summary>💡 Hint 1</summary>

The VLAN is a setting of the **port**, not of the PC. `interface fa0/2` → one command.
</details>
<details><summary>✔ Answer</summary>

```
configure terminal
interface fa0/2
 switchport access vlan 20
 end
```
PC-HFA-ENG-02 → IP `10.10.17.103` / `255.255.255.0` → `ping 10.10.17.101` → replies. This is
"organisation" from 1.1: the desk moved, only the port's VLAN changed.
Go back to `step02.pkt` (not the exercise copy) for step 03.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

- [ ] Open `my-project/step01.pkt` → **Save As** `my-project/step02.pkt`
- [ ] Place your first access switch and its PCs, with **your** names (plan part B)
- [ ] Cable them following **your** port plan (plan part D)
- [ ] Harden it with **your** banner (plan part A — English, no `#`) and **your** passwords (plan part F)
- [ ] Create **your** VLANs with **your** names (plan part D)
- [ ] Assign the ports, park and shut the unused ones
- [ ] Give the PCs addresses from **your** subnets
- [ ] Save the switch and the file

| In the example | In your project — take it from |
|---|---|
| `SW-HFA-A-ACC1`, `PC-HFA-ENG-01`… | plan part B (your hostnames) |
| VLANs 10, 20, 30, 99, 110, 120, 998, 999 and their names | plan part D (your HQ VLAN table) |
| Ports Fa0/1–10, Fa0/11–20, Fa0/21… | plan part D (your port plan) |
| Banner text "Intel Israel …" | plan part A (your banner — English, no `#`) |
| `Intel#En4ble`, `Intel#Con` | plan part F (your passwords) |
| PC addresses `10.10.16.101`… | plan part D (your subnets) |

**Check:** `show vlan brief` shows **your** VLAN names on **your** ports; same-VLAN ping works,
cross-VLAN ping fails.

💾 Save as `my-project/step02.pkt`.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ◆ What is a VLAN and why do you use VLANs?**

<details><summary>Answer</summary>

A VLAN splits one physical switch into separate logical switches — each VLAN is its own broadcast domain. Why: security (departments are separated), less broadcast traffic, and organisation (a user's VLAN follows the port, not the location).

**Show it:** `show vlan brief`
</details>

**2. ◆ What is the difference between an access port and a trunk port?**

<details><summary>Answer</summary>

An access port belongs to one VLAN and connects one end device; frames are not tagged. A trunk carries many VLANs between switches (step 03) and tags each frame with its VLAN number (802.1Q).

**Show it:** `show interfaces fa0/1 switchport`
</details>

**3. ◆ Why do the unused ports go to VLAN 999 and get shut down?**

<details><summary>Answer</summary>

An unused port is an open door — anyone could plug in a laptop. Shut down = the port doesn't work at all; VLAN 999 = even if someone enables it by mistake, it leads nowhere.

**Show it:** `show interfaces status`, `show vlan brief`
</details>

**4. ◆ What is the difference between `enable secret` and `enable password`? What does `service password-encryption` do?**

<details><summary>Answer</summary>

`enable secret` is stored as an MD5 hash (type 5) — cannot be reversed. `enable password` is clear text. `service password-encryption` scrambles the other passwords (console, VTY) with type 7, which is weak but hides them from someone reading the config.

**Show it:** `show running-config` → `enable secret 5 …`, `password 7 …`
</details>

**5. ◆ Why the banner?**

<details><summary>Answer</summary>

Legal warning before login: only authorised users may enter. Without it, it is harder to take legal action against an intruder (req. 22.2).

**Show it:** Log out (`exit`) and log in again.
</details>

**6. ◆ Why `switchport mode access` and not leaving the default?**

<details><summary>Answer</summary>

It forces the port to stay an access port, so a device cannot negotiate a trunk (DTP) and reach every VLAN — a VLAN-hopping attack.

**Show it:** `show interfaces fa0/1 switchport` → `Administrative Mode: static access`
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

1. **File → New**, drag one 2960. Open only your plan — not this page.
2. Type from memory: hostname, `no ip domain-lookup`, `enable secret`, `service password-encryption`,
   banner, console line, all your VLANs, the port plan, and parking for unused ports.
3. Target: **15 minutes**. Then `show vlan brief` and `show running-config`, and explain every line.

---

## 👩‍🏫 Teacher notes
- **Lesson plan (~90–120 min):** 20 min VLAN concept (draw one switch with coloured rooms on
  the board) → 15 min hardening table → Missions 1–4 together on the projector (15 min) →
  Missions 5–8: students type the config (40 min) → Mission 9 + the "why does ENG → FIN fail"
  discussion (15 min).
- **Check at the 🔍 points.** Mission 6 and Mission 7c each end with a `show` command. Walk
  around and look at the output before students go on — that catches 90% of mistakes early.
- **Make them type, not paste.** The hardening block repeats on *every* device in the project;
  by the 3rd switch they should type it from memory. Quiz idea: "write the hardening block on paper".
- **Common student questions**
  - *"Is a VLAN the same as a subnet?"* — Not the same thing, but we always pair them 1:1.
    The VLAN is the Layer-2 room; the subnet is that room's Layer-3 street address.
  - *"Why don't we use VLAN 1?"* — It's the default on every Cisco port, so attackers know it.
  - *"Console password vs enable secret?"* — Console password = getting into the lobby through
    the physical cable; enable secret = the key to the manager's office (`#` mode).
- **Typical errors:** typing a VLAN command while still inside an interface (prompt shows
  `(config-if)#` — that's fine actually, `vlan 10` works from there too, but explain the
  prompt); forgetting `switchport mode access`; not saving; keyboard left on Hebrew/Arabic.
