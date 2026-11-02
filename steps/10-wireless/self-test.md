# Step 10 — Self-test

> 🧪 Do this **after** you finish step 10, on your saved file `my-work/step10.pkt`.
> It takes about 20 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Press **Fast Forward Time** 2–3 times first (M18).
2. Do the tests **in order**, top to bottom.
3. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
4. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
5. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
6. Every box ticked → step 10 works. Go to step 11.

Log in everywhere with `admin` / `Intel#Adm1n`. "M…" numbers are the basic moves in
[docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — Access points and laptops

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | A-ACC2 | `show vlan brief` | Fa0/22 in VLAN **110**, Fa0/23 in VLAN **120** | Step 02/03 port plan of ACC2 |
| ☐ | A2 | AP-HFA-A-EMP → Config → Port 1 | Look | SSID `INTEL-CORP`, channel `1`, **WPA2-PSK**, key `IntelCorp#2026`, **AES** | Mission 4 |
| ☐ | A3 | AP-HFA-A-GUEST → Config → Port 1 | Look | SSID `INTEL-GUEST`, channel `6`, **WPA2-PSK**, key `WelcomeGuest2026`, **AES** | Mission 5 |
| ☐ | A4 | Each laptop → Config | Look at the left list | **Wireless0**, no FastEthernet0 | Mission 3b (power off before swapping the card) |
| ☐ | A5 | Workspace | Look | A dashed radio line from each laptop to its AP | SSID / key / encryption differ by one letter (Mission 6) |
| ☐ | A6 | LT-HFA-EMP-01 and EMP-02 | `ipconfig` | `10.10.20.x`, gateway `10.10.20.1` | Mission 6. `169.254…` → `ipconfig /renew` |
| ☐ | A7 | LT-HFA-GUEST-01 | `ipconfig /all` | `10.10.21.x`, gateway `10.10.21.1`, DNS `200.1.100.10` | Mission 6. DNS wrong → step 05 pool HFA-WIFI-GUEST |

## Part B — The guest-isolation ACL

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | A-DIST2 | `show access-lists GUEST-IN` | 6 lines in this order: `permit udp any any eq bootps` · `permit udp any host 200.1.100.10 eq domain` · `deny ip any 10.0.0.0 0.255.255.255` · `deny ip any 172.16.0.0 0.15.255.255` · `deny ip any 192.168.0.0 0.0.255.255` · `permit ip any any` | Wrong order → `no ip access-list extended GUEST-IN` and type it again (Mission 8a) |
| ☐ | B2 | A-DIST1 | `show access-lists GUEST-IN` | The same 6 lines | Mission 8b |
| ☐ | B3 | A-DIST2 | `show running-config interface vlan 120` | `ip access-group GUEST-IN in` | Mission 8a. `out` instead of `in` is wrong |
| ☐ | B4 | A-DIST1 | `show running-config interface vlan 120` | `ip access-group GUEST-IN in` | Mission 8b |

## Part C — Who can reach what

| ☐ | # | From | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | LT-HFA-EMP-01 | `ping 10.10.1.10` | ✅ replies — employees reach the servers | A6 |
| ☐ | C2 | LT-HFA-EMP-01 | `ping` an Engineering PC | ✅ replies | — |
| ☐ | C3 | LT-HFA-GUEST-01 | `ipconfig /renew` | Still gets `10.10.21.x` (DHCP passes the ACL) | `permit udp any any eq bootps` missing or below a deny |
| ☐ | C4 | LT-HFA-GUEST-01 | `ping 10.10.1.10` | ❌ fails — **correct!** Guests are blocked | B3 / B4: ACL not applied, or applied `out` |
| ☐ | C5 | LT-HFA-GUEST-01 | `ping` an Engineering PC | ❌ fails — **correct!** | The same |
| ☐ | C6 | A-DIST2 | `show access-lists GUEST-IN` | The `deny ip any 10.0.0.0` line shows `(N match(es))` | C4 went through A-DIST1? Run B2 and look there |
| ☐ | C7 | A-ACC2 | `show port-security interface fa0/23` | The guest laptop's MAC is counted on the AP port (`Total MAC Addresses` ≥ 1) | — |

> `ping 200.1.100.11` from the guest laptop works **only after step 12**. The internet doesn't
> exist yet, so don't test it now.

## Part D — Physical view and saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | D1 | **Physical** → Navigation | Look under **HFA Building A** | AP-HFA-A-EMP, AP-HFA-A-GUEST, LT-HFA-EMP-01, LT-HFA-EMP-02, LT-HFA-GUEST-01 | Mission 10 |
| ☐ | D2 | **Logical** tab | LT-HFA-EMP-01 → `ping 10.10.1.10` | ✅ still replies | Laptop out of WiFi range → in HFA Building A drag it next to its AP |
| ☐ | D3 | A-DIST1, A-DIST2 | `copy running-config startup-config` | `[OK]` | — |
| ☐ | D4 | **File → Save** | — | Title shows `step10.pkt` | M1 |

---

## 🛠 Your own project
Run the same test in your own project. Replace:

| In the test | Use from your plan |
|---|---|
| SSIDs `INTEL-CORP` / `INTEL-GUEST`, keys | your SSIDs and keys |
| VLANs 110 / 120, `10.10.20.x` / `10.10.21.x` | your WiFi VLANs and subnets |
| `200.1.100.10` | your guest DNS (the ISP's) |
| `10.10.1.10`, an Engineering PC | one of your servers and one of your PCs |

Your project passes when every row matches with **your** values. C4 is the most important
test: a guest must **never** reach anything inside the company.
