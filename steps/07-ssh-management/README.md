# Step 07 — Secure management (SSH + management VLAN)

## Goal
Every device in building A can be managed **only** with SSH, **only** from the management
network, with a personal username. Telnet is gone.

Ministry requirements: **13, 22.5**

## Concepts
- **Telnet** sends the password in clear text (look at a Telnet packet in Simulation mode
  — you can read it). **SSH** encrypts the session. SSH needs: a hostname, a domain name
  (they form the RSA key name), RSA keys, and a username.
- **`transport input ssh`** — the VTY lines refuse Telnet.
- **`access-class MGMT-ONLY in`** — a standard ACL on the VTY lines: only sources in the ACL
  may even try to log in. This is the "prevent management from other IPs" part of 22.5.
- **`login block-for`** — after 3 failed logins in 60 s, refuse all logins for 120 s
  (brute-force protection).
- Who may manage: the **MGMT VLANs** of all buildings (`10.10.19.0/24`, `.35`, `.51`) and
  later the **IT department** (`10.10.49.0/24`). Right now only `10.10.19.0/24` exists.

## Configure — on all 4 building-A switches (ACC1, ACC2, DIST1, DIST2)

```
configure terminal
ip domain-name intel.lab
username admin privilege 15 secret Intel#Adm1n

! RSA keys — when asked "How many bits in the modulus", answer 1024
crypto key generate rsa
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
login block-for 120 attempts 3 within 60

ip access-list standard MGMT-ONLY
 remark --- who may SSH to network devices ---
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
wr
```

> After `line console 0 / login local`, the console asks for **username admin** — the old
> console password is no longer used.

> The 2960 only has `line vty 0 15`; if PT says `0 4`, configure `0 4` and `5 15` separately.

## Verify

From **PC-HFA-ADMIN-01** (10.10.19.100) → Command Prompt:
```
C:\> ssh -l admin 10.10.19.11
Password: Intel#Adm1n
SW-HFA-A-ACC1#              <- you're in, encrypted
```
Repeat for 10.10.19.12 (ACC2), 10.10.19.2 (DIST1), 10.10.19.3 (DIST2).

| Test | Expected |
|---|---|
| ADMIN-01 → `ssh -l admin 10.10.19.11` | ✅ login |
| ADMIN-01 → `telnet 10.10.19.11` | ❌ refused (`transport input ssh`) |
| ENG-01 → `ssh -l admin 10.10.19.11` | ❌ refused (ACL) |
| ENG-01 → `ssh -l admin 10.10.16.2` (DIST1's user-VLAN IP!) | ❌ refused — the ACL checks the **source**, so any of the switch's IPs is protected |

```
SWL3-HFA-A-DIST1# show ip ssh               <- SSH Enabled - version 2.0
SWL3-HFA-A-DIST1# show access-lists MGMT-ONLY  <- deny counter goes up after ENG-01's try
SWL3-HFA-A-DIST1# show users                <- who is logged in
```

## Checkpoint
`my-work/step07.pkt` + commit.

## Common mistakes
- `crypto key generate rsa` fails → you forgot `ip domain-name` (or hostname is still `Switch`).
- SSH works from nowhere → `access-class` with a typo in the ACL name = **deny all**.
- Locked yourself out? Console still works with `admin` / `Intel#Adm1n`.

## Challenge
Open **Simulation mode**, filter on SSH and Telnet only. Temporarily allow Telnet on ACC1
(`transport input ssh telnet`), telnet to it from ADMIN-01, and inspect a Telnet packet's
payload. Then do the same with SSH. What's the difference? Remove Telnet again afterwards.

<details><summary>Answer</summary>

The Telnet packet's application layer shows the characters you typed — including the
password. The SSH packet only shows encrypted data. This is exactly why req. 13 asks for
SSH.
</details>
