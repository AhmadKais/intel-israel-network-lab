# Step 02 — First switch: hardening & VLANs

> ⏱ 1–2 lessons · 🎯 ministry requirements 22.1, 22.2, 22.3, 22.6 · 📁 start from: `my-work/step01.pkt` · 💾 save as: `my-work/step02.pkt`

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

---

## Part 2 — Prepare
1. Open Packet Tracer → **File → Open** → `my-work/step01.pkt`.
2. Immediately **File → Save As** → `my-work/step02.pkt`.
   (Now step01 stays untouched — if you break something you can always go back.)

---

## Part 3 — Build (click by click)

### 3.1 Place the switch
1. Device-type box (bottom left) → **Network Devices** → **Switches**.
2. Drag a **2960** (`2960-24TT`) into the **Building A** rectangle.
3. Click it → **Config** tab → **Display Name**: type `SW-HFA-A-ACC1` → close the window.

### 3.2 Place the end devices
1. **End Devices** → **End Devices**. Hold **Ctrl**, click the **PC** icon, and click 5 times
   under the switch (5 PCs). Press **Esc**.
2. Drag one **Printer** next to them.
3. Rename each one (click → **Config** → **Display Name**):

| Rename | to |
|---|---|
| first PC | `PC-HFA-ENG-01` |
| second PC | `PC-HFA-ENG-02` |
| third PC | `PC-HFA-FIN-01` |
| fourth PC | `PC-HFA-FIN-02` |
| fifth PC | `PC-HFA-ADMIN-01` |
| printer | `PRN-HFA-A-01` |

### 3.3 Cable them
Use **Connections → Copper Straight-Through** each time (end device ↔ switch = different devices).

| Click first | Port | Click second | Port |
|---|---|---|---|
| PC-HFA-ENG-01 | FastEthernet0 | SW-HFA-A-ACC1 | **FastEthernet0/1** |
| PC-HFA-ENG-02 | FastEthernet0 | SW-HFA-A-ACC1 | **FastEthernet0/2** |
| PC-HFA-FIN-01 | FastEthernet0 | SW-HFA-A-ACC1 | **FastEthernet0/11** |
| PC-HFA-FIN-02 | FastEthernet0 | SW-HFA-A-ACC1 | **FastEthernet0/12** |
| PRN-HFA-A-01 | FastEthernet0 | SW-HFA-A-ACC1 | **FastEthernet0/21** |
| PC-HFA-ADMIN-01 | FastEthernet0 | SW-HFA-A-ACC1 | **FastEthernet0/22** |

Press **Fast Forward Time** until all lights are green. Thanks to the port labels
(step 00, Options → Preferences) you can check every port number on the screen.

### 3.4 The port plan (why these ports?)
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

## Part 4 — Configure the switch (type it — don't paste)

### 4.1 Open the CLI
1. Click **SW-HFA-A-ACC1** → **CLI** tab.
2. Click in the black area and press **Enter**. You see `Switch>`.

> Type **each line** and press **Enter**. The lines starting with `!` are **comments** — don't
> type them, they explain the next command. If you mistype, IOS shows
> `% Invalid input detected at '^' marker` — press **↑**, fix the line, press **Enter**.

### 4.2 Hardening
```
enable
configure terminal
```
The prompt is now `Switch(config)#`.
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
Now the banner. Type `banner motd #` and press Enter. The switch answers
`Enter TEXT message. End with the character '#'.` — type the text, and finish with a line
containing only `#`:
```
banner motd #
*********************************************************
*  Intel Israel - AUTHORIZED ACCESS ONLY                *
*  All activity is logged. Disconnect now if you are    *
*  not an authorized network administrator.             *
*********************************************************
#
```
Console line:
```
line console 0
 password Intel#Con
 login
 logging synchronous
 exec-timeout 10 0
 exit
```
The prompt went `(config-line)#` and back to `(config)#` after `exit`.

### 4.3 Create the VLANs
Each `vlan` command enters the VLAN's sub-mode `(config-vlan)#`; the `name` line labels it.
You can go from one `vlan` straight to the next without `exit`.
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

### 4.4 Put the ports in their VLANs
`interface range` configures many ports at once — the prompt becomes `(config-if-range)#`.
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
> ⚠️ Type `fa0/1 - 10` with **spaces around the dash**. Without spaces PT rejects it.

### 4.5 Park and shut the unused ports
```
interface range fa0/23 - 24
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
 end
```
You see messages like `%LINK-5-CHANGED: Interface FastEthernet0/23, changed state to
administratively down` — that's the `shutdown` working.

### 4.6 Save
```
copy running-config startup-config
```
It asks `Destination filename [startup-config]?` → press **Enter**. You see `[OK]`.

### 4.7 What the new commands mean
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

## Part 5 — Give the end devices their addresses
For each device: click it → **Desktop** → **IP Configuration** → **Static** → fill in → close.
(The printer has no Desktop tab: click it → **Config** → **FastEthernet0** → fill in IP and mask;
the gateway is under **Config → Settings**.)

| Device | IPv4 Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC-HFA-ENG-01 | 10.10.16.101 | 255.255.255.0 | (leave empty for now) |
| PC-HFA-ENG-02 | 10.10.16.102 | 255.255.255.0 | (empty) |
| PC-HFA-FIN-01 | 10.10.17.101 | 255.255.255.0 | (empty) |
| PC-HFA-FIN-02 | 10.10.17.102 | 255.255.255.0 | (empty) |
| PRN-HFA-A-01 | 10.10.18.10 | 255.255.255.0 | 10.10.18.1 (permanent) |
| PC-HFA-ADMIN-01 | 10.10.19.100 | 255.255.255.0 | 10.10.19.1 (permanent) |

The user PCs get temporary addresses now. In step 05 they switch to **DHCP** (automatic).

---

## Part 6 — Check your work

### 6.1 Check the VLANs on the switch
In the switch CLI (`SW-HFA-A-ACC1#`):
```
show vlan brief
```
You should see:
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
✅ VLAN 1 has **only** the two Gi uplinks (they become trunks in step 03).

### 6.2 Ping tests
| From (Desktop → Command Prompt) | Type | Expected | Why |
|---|---|---|---|
| PC-HFA-ENG-01 | `ping 10.10.16.102` | ✅ replies | same VLAN |
| PC-HFA-FIN-01 | `ping 10.10.17.102` | ✅ replies | same VLAN |
| PC-HFA-ENG-01 | `ping 10.10.17.101` | ❌ fails | different VLAN, no router yet |

### 6.3 Check the security settings
1. In the switch CLI type `exit` until you see `Press RETURN to get started.` Press **Enter**.
2. You see the **banner**, then `User Access Verification` → `Password:` → type `Intel#Con`
   (nothing appears while you type — that's normal).
3. Type `enable` → `Password:` → `Intel#En4ble`.
4. Type `show running-config` (press **Space** to page down). Find:
   - `enable secret 5 $1$...` → hashed ✅
   - under `line con 0`: `password 7 0822455D0A16...` → scrambled ✅

### 6.4 What just happened? Why does ENG → FIN fail?
Two reasons, both on purpose:
1. **Different VLANs** — the switch keeps rooms 10 and 20 completely apart. A frame from
   VLAN 10 is never delivered to a port in VLAN 20.
2. **Different subnets** — `10.10.16.x` and `10.10.17.x` are different "streets". The PC knows
   it must use a **gateway** (router) for another street — and there is none yet.

Departments are isolated by default. In step 05 we add routing and decide *deliberately*
which traffic may cross.

---

## Part 7 — Save
1. In the switch CLI: `copy running-config startup-config` (if you changed anything since 4.6).
2. **File → Save** (`my-work/step02.pkt`).

## Checkpoint
- [ ] `show vlan brief` matches 6.1
- [ ] Same-VLAN pings work, cross-VLAN pings fail
- [ ] Banner, console password and enable secret all work
- [ ] Device names on the workspace follow the naming rule

## If something doesn't work
| Problem | Fix |
|---|---|
| Ping in the same VLAN fails | `show vlan brief`: is the PC's port in the right VLAN? Check the cable goes to the right port (port labels!). Check the PC's IP/mask |
| `% Invalid input` on `interface range fa0/1-10` | Add spaces: `fa0/1 - 10` |
| VLAN name shows `VLAN0010` | You assigned a port before creating/naming the VLAN — go back: `vlan 10` → `name ENGINEERING` |
| Banner never ends | You must type a line with just `#` |
| Forgot the console password | Click **Physical** → power off/on won't help (it's saved). Re-open your saved `step01.pkt` and redo, or practise "password recovery" with the teacher |

## Challenge
1. Plug a new PC into **Fa0/24**. Does it work? Why not? What would an admin have to do?
2. Run `show interfaces fa0/24 switchport` and find the line that proves the port is in VLAN 999.

<details><summary>Answer</summary>

1. No — the port is `administratively down` (shut down) and in the parking VLAN. The admin
   must move it to the right VLAN and type `no shutdown`. Nobody can just walk in and plug
   a laptop in.
2. `Access Mode VLAN: 999 (PARKING)`
</details>

---

## 👩‍🏫 Teacher notes
- **Lesson plan (~90–120 min):** 20 min VLAN concept (draw one switch with coloured rooms on
  the board) → 15 min hardening table → 15 min build & cable together on the projector →
  40 min students type the config → 15 min verify + the "why does ENG → FIN fail" discussion.
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
  prompt); forgetting `switchport mode access`; not saving.
