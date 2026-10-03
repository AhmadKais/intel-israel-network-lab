# Step 02 — First switch: hardening & VLANs

## Goal
One access switch in Haifa building A, secured with the basics every device in the
project needs, with department VLANs. PCs in the same VLAN can talk; PCs in different
VLANs **cannot** (yet).

Ministry requirements: **22.1, 22.2, 22.3, 22.6**

## Concepts
- **VLAN** = a separate Layer-2 network (broadcast domain) inside one physical switch.
  Engineering broadcasts never reach Finance.
- **Access port** = belongs to exactly one VLAN; connects an end device.
- **`enable secret`** stores a hash (type 5/9). `service password-encryption` scrambles the
  *other* passwords (type 7 — weak, but better than clear text in a printout).
- **Banner MOTD** = legal warning. Needed in court to prove the intruder was warned.

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
