# Step 10 — Wireless: employees + guests

## Goal
Building A has two WiFi networks: **INTEL-CORP** for employees (full access) and
**INTEL-GUEST** for visitors (internet only — **blocked** from every internal network).

Ministry requirement: **18**

## Concepts
- An access point **bridges** WiFi into a VLAN. One SSID → one VLAN.
  - `INTEL-CORP` → VLAN 110 (10.10.20.0/24)
  - `INTEL-GUEST` → VLAN 120 (10.10.21.0/24)
- **WPA2-PSK (AES)** — encrypted, one shared key. Fine for guests.
  **WPA2-Enterprise** — each employee logs in with their own username via a RADIUS
  server (better for employees — see the challenge).
- **Guest isolation** is done with an **ACL on the guest VLAN's gateway** (both DIST
  switches): guests may get DHCP and DNS, may go to the internet, may **not** reach any
  10.x.x.x address.
- Guests get the ISP's DNS (200.1.100.10) from DHCP — never the internal one.

## Build

| Device | Name | Connect to |
|---|---|---|
| AccessPoint-PT | `AP-HFA-A-EMP` | SW-HFA-A-ACC2 Fa0/22 (VLAN 110 — done in step 03) |
| AccessPoint-PT | `AP-HFA-A-GUEST` | SW-HFA-A-ACC2 Fa0/23 (VLAN 120) |
| Laptop-PT ×2 | `LT-HFA-EMP-01`, `LT-HFA-EMP-02` | wireless |
| Laptop-PT ×1 | `LT-HFA-GUEST-01` | wireless |

**Laptops need a WiFi card**: Physical tab → power off → drag the existing Ethernet module
out → drag **WPC300N** in → power on.

## Configure

### Access points (Config tab → Port 1)
| AP | SSID | Authentication | Key | Encryption |
|---|---|---|---|---|
| AP-HFA-A-EMP | `INTEL-CORP` | WPA2-PSK | `IntelCorp#2026` | AES |
| AP-HFA-A-GUEST | `INTEL-GUEST` | WPA2-PSK | `WelcomeGuest2026` | AES |

Use **different channels** on the two APs (e.g. 1 and 6) so they don't interfere.

### Laptops
*Desktop → PC Wireless → Connect* (or *Config → Wireless0*): pick the SSID, WPA2-Personal,
enter the key. *IP Configuration → DHCP*.

### Guest isolation ACL — on SWL3-HFA-A-DIST1 **and** DIST2
```
configure terminal
ip access-list extended GUEST-IN
 remark --- DHCP to the local gateway ---
 permit udp any any eq bootps
 remark --- DNS to the ISP resolver only ---
 permit udp any host 200.1.100.10 eq domain
 remark --- nothing internal ---
 deny ip any 10.0.0.0 0.255.255.255
 deny ip any 172.16.0.0 0.15.255.255
 deny ip any 192.168.0.0 0.0.255.255
 remark --- internet ---
 permit ip any any
 exit
interface vlan 120
 ip access-group GUEST-IN in
end
wr
```

> Why `in` on VLAN 120? Packets **from** guests enter the switch **in**to the VLAN 120
> SVI. Filtering there drops them at the first hop, before they go anywhere.

> Note that the guest's own gateway (10.10.21.1) is also 10.x — pings to it are denied.
> That's intentional; DHCP still works because the `bootps` line comes first.

## Verify

| From | Test | Expected |
|---|---|---|
| LT-HFA-EMP-01 | `ipconfig` | 10.10.20.x, gw 10.10.20.1 |
| LT-HFA-EMP-01 | `ping 10.10.1.10` | ✅ employees reach servers |
| LT-HFA-GUEST-01 | `ipconfig` | 10.10.21.x, DNS 200.1.100.10 |
| LT-HFA-GUEST-01 | `ping 10.10.16.1`, `ping 10.10.1.10` | ❌ blocked |
| LT-HFA-GUEST-01 | `ping 200.1.100.11` | ✅ after step 12 (internet) |

```
SWL3-HFA-A-DIST2# show access-lists GUEST-IN     <- deny counters increase
SW-HFA-A-ACC2#    show port-security interface fa0/23   <- laptop MACs counted on the AP port
```
(Guest traffic enters at DIST**2** — it's the HSRP active for VLAN 120.)

## Checkpoint
`my-work/step10.pkt` + commit.

## Common mistakes
- Laptop shows no WiFi → you didn't replace the Ethernet module with WPC300N.
- Laptop associates but gets 169.254.x.x → AP port is in the wrong VLAN, or the AP port
  hit the port-security maximum (step 06 sets 30).
- ACL only on DIST1 → when DIST2 is active for VLAN 120 (it is!), guests are not filtered.

## Challenge — WPA2-Enterprise for employees
Switch INTEL-CORP to **WPA2-Enterprise** with RADIUS on SRV-HFA-AAA, so each employee has a
personal login. What do you have to configure on the AP, the AAA server and the laptop?

<details><summary>Answer</summary>

1. AP-HFA-A-EMP → Port 1 → **WPA2** (Enterprise), RADIUS server `10.10.1.11`, shared
   secret `Intel#Rad`, AES.
2. SRV-HFA-AAA → AAA → *Network Configuration*: client `AP-HFA-A-EMP`, the IP the AP's
   RADIUS requests come from, secret `Intel#Rad`, type **Radius**. *User Setup*: `emp1` /
   `Emp#2026`, etc.
3. Laptop → Wireless → WPA2-Enterprise, user `emp1`, password `Emp#2026`.

Packet Tracer's AccessPoint-PT has no management IP, so this setup can be finicky; if it
won't authenticate, try a **WLC-2504 + LAP** design or a **WRT300N** (which has its own
IP). Document whichever you get working.
</details>
