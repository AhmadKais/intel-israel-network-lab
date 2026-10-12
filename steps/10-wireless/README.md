# Step 10 — Wireless: employees + guests

> ⏱ 1–2 lessons · 🎯 ministry requirement 18 · 📁 start from: `my-work/step09.pkt` · 💾 save as: `my-work/step10.pkt`

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
and it puts their traffic onto the wired network — into **one VLAN**. Each AP broadcasts a
network name, the **SSID**.

| SSID | AP | Switch port | VLAN | Subnet |
|---|---|---|---|---|
| INTEL-CORP | AP-HFA-A-EMP | ACC2 Fa0/22 | 110 WIFI-EMP | 10.10.20.0/24 |
| INTEL-GUEST | AP-HFA-A-GUEST | ACC2 Fa0/23 | 120 WIFI-GUEST | 10.10.21.0/24 |

The VLANs, gateways (HSRP) and DHCP pools for 110 and 120 already exist since steps 02–05 —
we prepared for this moment.

### 1.2 WiFi security
Radio goes through walls — anyone in the car park can try to connect. So WiFi must be
**encrypted** and need a **key**.

| Security | How it works | We use it for |
|---|---|---|
| Open | No password, no encryption | Never |
| WEP | Old encryption, broken in minutes | Never |
| **WPA2-Personal (PSK)** + AES | One shared password for everyone | **Guests**, and employees in this lab |
| **WPA2-Enterprise** | Every user logs in with their **own** username via a RADIUS (AAA) server | Real company employees (see the challenge) |

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

---

## Part 2 — Prepare
**File → Open** → `my-work/step09.pkt` → **File → Save As** → `my-work/step10.pkt`.

---

## Part 3 — Build (click by click)

### 3.1 The access points
1. **Network Devices → Wireless Devices** → drag an **AccessPoint-PT** next to ACC2.
   Rename it `AP-HFA-A-EMP`.
2. Drag a second **AccessPoint-PT** → rename `AP-HFA-A-GUEST`.
3. Cable with **Copper Straight-Through**:

| AP | AP port | Switch | Switch port |
|---|---|---|---|
| AP-HFA-A-EMP | Port 0 | SW-HFA-A-ACC2 | **Fa0/22** |
| AP-HFA-A-GUEST | Port 0 | SW-HFA-A-ACC2 | **Fa0/23** |

### 3.2 The laptops — and their WiFi card
1. **End Devices → End Devices** → drag 3 **Laptop** icons near the APs.
2. Rename: `LT-HFA-EMP-01`, `LT-HFA-EMP-02`, `LT-HFA-GUEST-01`.
3. A laptop comes with a **wired** card. For **each** laptop:
   1. Click it → **Physical** tab.
   2. Click the laptop's **power button** (on the picture) → it turns off.
   3. Drag the existing module (**PT-LAPTOP-NM-1CFE**, at the side of the laptop) **out** to the
      MODULES list on the left.
   4. From MODULES, drag **WPC300N** (wireless card) into the empty slot.
   5. Click the **power button** again → on.
   6. Close the window.

---

## Part 4 — Configure

### 4.1 AP-HFA-A-EMP
Click it → **Config** tab → in the left list click **Port 1** (the wireless side):
| Field | Value |
|---|---|
| SSID | `INTEL-CORP` |
| 2.4 GHz Channel | `1` |
| Authentication | **WPA2-PSK** |
| PSK Pass Phrase | `IntelCorp#2026` |
| Encryption Type | **AES** |

### 4.2 AP-HFA-A-GUEST
**Config → Port 1**:
| Field | Value |
|---|---|
| SSID | `INTEL-GUEST` |
| 2.4 GHz Channel | `6` (different channel = no interference with the other AP) |
| Authentication | **WPA2-PSK** |
| PSK Pass Phrase | `WelcomeGuest2026` |
| Encryption Type | **AES** |

### 4.3 The laptops
For **LT-HFA-EMP-01** and **LT-HFA-EMP-02**: click → **Config** tab → **Wireless0** (left list):
| Field | Value |
|---|---|
| SSID | `INTEL-CORP` |
| Authentication | **WPA2-PSK** |
| PSK Pass Phrase | `IntelCorp#2026` |
| Encryption Type | **AES** |
| IP Configuration | **DHCP** |

For **LT-HFA-GUEST-01**: the same with SSID `INTEL-GUEST` and `WelcomeGuest2026`.

On the workspace you'll see dashed "radio" lines between each laptop and its AP. ✅
(Alternative: **Desktop → PC Wireless → Connect** tab → choose the network → **Connect**.)

### 4.4 Guest isolation ACL — on SWL3-HFA-A-DIST1 **and** SWL3-HFA-A-DIST2
Type on **both** (log in `admin` / `Intel#Adm1n`):
```
enable
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

### 4.5 What the new commands mean
| Command | In simple English |
|---|---|
| `ip access-list extended GUEST-IN` | A detailed rule list (source + destination + protocol) named GUEST-IN |
| `permit udp any any eq bootps` | "Allow DHCP requests" (bootps = the DHCP server port 67) |
| `permit udp any host 200.1.100.10 eq domain` | "Allow DNS (port 53) only to that one server" |
| `deny ip any 10.0.0.0 0.255.255.255` | "Block anything going to any 10.x address" |
| `permit ip any any` | "Allow the rest" (without it the invisible deny-all would block the internet too) |
| `ip access-group GUEST-IN in` | "Check packets **coming in** from the guest VLAN with this list" |

> Why on **both** DIST switches? DIST2 is the HSRP **Active** for VLAN 120 — guest traffic enters
> there. DIST1 needs it too, for when DIST2 fails.

---

## Part 5 — Check your work

| From | Type | Expected |
|---|---|---|
| LT-HFA-EMP-01 | `ipconfig` | `10.10.20.x`, gateway `10.10.20.1` |
| LT-HFA-EMP-01 | `ping 10.10.1.10` | ✅ employees reach the servers |
| LT-HFA-EMP-01 | `ping` an Engineering PC | ✅ |
| LT-HFA-GUEST-01 | `ipconfig /all` | `10.10.21.x`, DNS `200.1.100.10` |
| LT-HFA-GUEST-01 | `ping 10.10.1.10` | ❌ blocked |
| LT-HFA-GUEST-01 | `ping` an Engineering PC | ❌ blocked |
| LT-HFA-GUEST-01 | `ping 200.1.100.11` | ✅ **after step 12** (the internet doesn't exist yet) |

On **DIST2**: `show access-lists GUEST-IN` → the `deny ip any 10.0.0.0` line shows matches.
On **ACC2**: `show port-security interface fa0/23` → the laptop's MAC counted on the AP port.

---

## Part 6 — Save
**File → Save** (the DIST switches were saved with `copy run start`).

## Checkpoint
- [ ] Both SSIDs work; laptops get addresses from the right VLAN
- [ ] Employees reach internal servers; guests don't

## If something doesn't work
| Problem | Fix |
|---|---|
| Laptop has no Wireless0 in Config | The WPC300N card isn't installed (3.2) |
| No dashed radio line | SSID / key / encryption typo — they must match the AP **exactly** |
| Laptop connects but gets `169.254.x.x` | AP port in the wrong VLAN (`show vlan brief` on ACC2), or port-security on Fa0/22-23 too strict (step 06 set 30) |
| Guest reaches internal PCs | ACL applied only on DIST1, or applied `out` instead of `in` |
| Guest gets no address after the ACL | The `bootps` line is missing or below the deny lines |

## Challenge — WPA2-Enterprise for employees
Change INTEL-CORP to **WPA2-Enterprise** so each employee logs in with a personal account
checked by SRV-HFA-AAA (RADIUS). What do you configure on the AP, the AAA server and the laptop?

<details><summary>Answer</summary>

1. **AP-HFA-A-EMP → Port 1:** Authentication **WPA2** (Enterprise), RADIUS server `10.10.1.11`,
   shared secret `Intel#Rad`, AES.
2. **SRV-HFA-AAA → AAA → Network Configuration:** client `AP-HFA-A-EMP`, the IP its requests come
   from, secret `Intel#Rad`, type **Radius**. **User Setup:** `emp1` / `Emp#2026`, `emp2` / …
3. **Laptop → Wireless0:** WPA2 (Enterprise), user `emp1`, password `Emp#2026`.

Packet Tracer's AccessPoint-PT has no IP address of its own, so this can be difficult. If it
won't authenticate, use a **WLC-2504 + LAP** design or a **WRT300N**. Write in the book which one
worked — it shows understanding.
</details>

---

## 🛠 Your own project

You did this step on the Intel example. Now do it again in **your** project, with the values
from **your** plan (`my-project/plan.md`). The commands are the same — only the values change.

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

## 👩‍🏫 Teacher notes
- **Lesson plan (~90 min):** 15 min AP/SSID/VLAN mapping + WiFi security table → 15 min guest
  isolation story ("a visitor in the lobby scanning your network") → 30 min build (the laptop card
  swap takes time the first time) → 15 min ACL → 15 min tests.
- **Common student questions**
  - *"Why not one AP with two SSIDs?"* — Real APs do that (one SSID per VLAN on a trunk port). PT's
    simple AccessPoint-PT supports one SSID per AP, so we use two.
  - *"Why `in` and not `out`?"* — `in` on VLAN 120 = packets from guests, filtered at the first
    router they touch.
- **Typical errors:** forgetting the WPC300N swap; ACL only on one DIST; extended ACL line order.
