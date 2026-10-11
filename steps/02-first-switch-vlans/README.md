# Step 02 — First switch: hardening & VLANs

## Goal
One access switch in Haifa building A, secured with the basics every device in the
project needs, with department VLANs. PCs in the same VLAN can talk; PCs in different
VLANs **cannot** (yet).

Ministry requirements: **22.1, 22.2, 22.3, 22.6**

## Understand it first (simple English)

### 1. What is a VLAN?
Without VLANs, a switch is **one big room**: every device hears every "shout" (broadcast)
of every other device, and anyone can talk to anyone.

A **VLAN (Virtual LAN)** cuts one physical switch into several **separate virtual
switches**. Imagine one office floor divided by walls into closed rooms: Engineering in
one room, Finance in another. They share the same building (switch), but:
- a broadcast in Engineering **never** reaches Finance;
- an Engineering PC **cannot** talk to a Finance PC directly — it needs a router (step 05),
  which is where you can later control who may talk to whom.

**Why companies use VLANs**

| Reason | Example |
|---|---|
| **Security** | Guests can't see the Finance PCs |
| **Less noise** | Fewer devices per broadcast domain → less wasted traffic |
| **Organisation** | Move a person's desk anywhere — just put their port in their department's VLAN |

Each VLAN has a **number** (1–4094) and a **name**. Each VLAN = one IP subnet
(VLAN 10 = `10.10.16.0/24`, VLAN 20 = `10.10.17.0/24`…).

### 2. Port names and access ports
`Fa0/1` = **F**ast **E**thernet (100 Mb/s), slot **0**, port **1**. `Gi0/1` = **Gi**gabit (1000 Mb/s).
On a 2960 the 24 Fa ports are for devices and the 2 Gi ports are for uplinks.

An **access port** belongs to exactly **one** VLAN and connects **one end device** (PC,
printer). The PC doesn't know VLANs exist — the switch decides which "room" it is in.

### 3. Hardening — the basic protection every device gets
"Hardening" = closing the easy doors before an attacker finds them.

| Setting | What it does | Why |
|---|---|---|
| `hostname` | Gives the device a name | Logical names (req. 22.1); also needed later for SSH keys |
| `enable secret` | Password to enter `#` mode, stored as a **hash** (scrambled one-way, can't be reversed) | Anyone reading the config can't see it (req. 22.3) |
| `service password-encryption` | Scrambles the **other** passwords (console etc.) | A printed config doesn't show them in clear text. Weak (type 7) — but better than nothing |
| `banner motd` | Message shown before login | A legal warning: "authorised access only". Needed to prosecute an intruder (req. 22.2) |
| `line console 0` | The physical console cable port | Someone standing next to the switch still needs a password |
| `no ip domain-lookup` | Stops the switch treating typos as hostnames | Saves you 30 seconds of waiting after every typo |
| `exec-timeout 10 0` | Logs you out after 10 minutes idle | An admin who walks away doesn't leave an open session |

> `secret` vs `password`: **always use `secret`** where it exists. `password` is stored in
> clear text (or weak type 7).

### 4. Unused ports — the "parking" VLAN
An empty port in a meeting room is an open door: anyone can plug in a laptop. We put every
unused port in VLAN 999 (which leads nowhere) **and** shut it down. To use it, an admin has
to deliberately enable it.

## Build

| Device (PT model) | Name |
|---|---|
| 2960-24TT | `SW-HFA-A-ACC1` |
| PC | `PC-HFA-ENG-01`, `PC-HFA-ENG-02` |
| PC | `PC-HFA-FIN-01`, `PC-HFA-FIN-02` |
| Printer | `PRN-HFA-A-01` |
| PC | `PC-HFA-ADMIN-01` (the network admin's PC, used in step 07) |

Rename devices: click the device → **Config** tab → *Display Name*.

| From | To |
|---|---|
| PC-HFA-ENG-01 | ACC1 Fa0/1 |
| PC-HFA-ENG-02 | ACC1 Fa0/2 |
| PC-HFA-FIN-01 | ACC1 Fa0/11 |
| PC-HFA-FIN-02 | ACC1 Fa0/12 |
| PRN-HFA-A-01 | ACC1 Fa0/21 |
| PC-HFA-ADMIN-01 | ACC1 Fa0/22 |

### Port plan for every building-A access switch
| Ports | VLAN | Use |
|---|---|---|
| Fa0/1–10 | 10 ENGINEERING | users |
| Fa0/11–20 | 20 FINANCE | users |
| Fa0/21 | 30 PRINTERS-A | printer |
| Fa0/22 | ACC1: 99 MGMT-A (admin PC) · ACC2: 110 WIFI-EMP (AP) | |
| Fa0/23 | ACC1: parking · ACC2: 120 WIFI-GUEST (AP) | |
| Fa0/24 | 999 PARKING (shut) | |
| Gi0/1, Gi0/2 | trunks to DIST1 / DIST2 (step 03) | |

## Configure — SW-HFA-A-ACC1

```
enable
configure terminal

! ---------- identity & basic hardening (you will do this on EVERY device) ----------
hostname SW-HFA-A-ACC1
! stop the switch trying to DNS-resolve every typo (saves you 30 s of waiting)
no ip domain-lookup
! hashed privileged password
enable secret Intel#En4ble
! scramble any clear-text passwords in the config
service password-encryption
banner motd #
*********************************************************
*  Intel Israel - AUTHORIZED ACCESS ONLY                *
*  All activity is logged. Disconnect now if you are    *
*  not an authorized network administrator.             *
*********************************************************
#

line console 0
 password Intel#Con
 login
 ! messages won't break the line you are typing
 logging synchronous
 ! auto-logout after 10 minutes idle
 exec-timeout 10 0
 exit

! ---------- VLANs ----------
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

! ---------- access ports ----------
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

! ---------- unused ports: parking VLAN + shut down ----------
interface range fa0/23 - 24
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
end

copy running-config startup-config
```

> `switchport mode access` (not leaving it on *dynamic auto*) means an attacker cannot
> negotiate a trunk on a user port with DTP. That is a Layer-2 attack protection (req. 13).

### What the new commands mean
| Command | In simple English |
|---|---|
| `vlan 10` → `name ENGINEERING` | "Create room number 10 and put a sign on the door: ENGINEERING" |
| `interface range fa0/1 - 10` | "The next commands apply to ports 1 to 10 at once" |
| `description ...` | A label for humans — shows in `show` commands, does nothing technical |
| `switchport mode access` | "This port is for one end device, never a trunk" |
| `switchport access vlan 10` | "Whatever is plugged in here is in room 10" |
| `shutdown` | Turns the port off (`no shutdown` turns it on) |
| `copy running-config startup-config` | Save the whiteboard into the notebook (step 00) |

### PCs — temporary static IPs (DHCP comes in step 05)
| PC | IP | Mask |
|---|---|---|
| PC-HFA-ENG-01 | 10.10.16.101 | 255.255.255.0 |
| PC-HFA-ENG-02 | 10.10.16.102 | 255.255.255.0 |
| PC-HFA-FIN-01 | 10.10.17.101 | 255.255.255.0 |
| PC-HFA-FIN-02 | 10.10.17.102 | 255.255.255.0 |
| PRN-HFA-A-01 | 10.10.18.10 | 255.255.255.0, gateway 10.10.18.1 (permanent) |
| PC-HFA-ADMIN-01 | 10.10.19.100 | 255.255.255.0, gateway 10.10.19.1 (permanent) |

## Verify

```
SW-HFA-A-ACC1# show vlan brief
```
Expected: VLAN 10 has Fa0/1–10, VLAN 20 has Fa0/11–20, 30 has Fa0/21, 99 has Fa0/22,
999 has Fa0/23–24, VLAN 1 has **only Gi0/1, Gi0/2**.

| Test | Expected | Why |
|---|---|---|
| ENG-01 → `ping 10.10.16.102` | ✅ replies | same VLAN |
| ENG-01 → `ping 10.10.17.101` | ❌ fails | different VLAN, no router yet |

Exit the console and log in again: you see the banner, you need `Intel#Con`, and
`show run` shows `enable secret 5 $1$...` (hashed) and `password 7 ...` (scrambled).

### What just happened? Why does ENG → FIN fail?
Two reasons, and both are on purpose:
1. **Different VLANs** — the switch keeps rooms 10 and 20 completely apart. A frame from
   VLAN 10 is never delivered to a port in VLAN 20.
2. **Different subnets** — `10.10.16.x` and `10.10.17.x` are different "streets". The PC
   knows it must use a **gateway** (router) for another street — and there is none yet.

This is **exactly** what we want: departments are isolated by default. In step 05 we add
a router function and decide *deliberately* which traffic may cross.

## Checkpoint
Save `my-work/step02.pkt` and commit.

## Common mistakes
- Typing `switchport access vlan 10` before the VLAN exists → the switch creates it with
  the name `VLAN0010`. Not a disaster, but name it.
- Forgetting `copy run start` → reload = everything gone.

## Challenge
1. Plug a new PC into Fa0/24. Does it work? Why not? What would an admin have to do?
2. Find in `show interfaces fa0/24 switchport` the line proving the port is in VLAN 999.

<details><summary>Answer</summary>

1. No — the port is `administratively down` (shutdown) and in the parking VLAN. The admin
   must deliberately move it to the right VLAN and `no shutdown`. Nobody can just walk
   in and plug a laptop in.
2. `Access Mode VLAN: 999 (PARKING)`
</details>

---

## 👩‍🏫 Teacher notes
- **Lesson plan (~90 min):** 20 min VLAN concept (draw one switch with coloured rooms on the
  board), 15 min the hardening table, 40 min students type the config, 15 min verify and the
  "why does ENG → FIN fail" discussion.
- **Make them type, not paste.** The hardening block is repeated on *every* device in the
  project; by the 3rd switch they should type it from memory. A good quiz: "write the
  hardening block on paper".
- **Common student questions**
  - *"Is a VLAN the same as a subnet?"* — Not the same thing, but we always pair them
    1:1. The VLAN is the Layer-2 room; the subnet is the Layer-3 street address of that room.
  - *"Why is VLAN 1 not used?"* — It's the default VLAN on every port of every Cisco switch,
    so attackers know about it. We leave it empty.
  - *"What's the difference between the console password and enable secret?"* — Console
    password = getting into the lobby through the physical cable; enable secret = the key
    to the manager's office (`#` mode).
- **Typical errors to watch for:** forgetting `exit`/`end` and typing a VLAN command inside
  an interface; typing `switchport access vlan 10` before `switchport mode access` (works,
  but teach the right order); not saving.
