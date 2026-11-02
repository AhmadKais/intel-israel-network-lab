# Step 06 — Self-test

> 🧪 Do this **after** you finish step 06, on your saved file `my-work/step06.pkt`.
> It takes about 30 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Press **Fast Forward Time** 2–3 times first (M18).
2. Do the tests **in order**, top to bottom.
3. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
4. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
5. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
6. Every box ticked → step 06 works. Go to step 07.

Part C **attacks your own network** and repairs it. Do every repair line before you go on.
"M…" numbers are the basic moves in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — Everything still works

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | Each of the 6 user PCs → Command Prompt | `ipconfig /renew` | A real address `.11`–`.254` with gateway `.1`. **Not** `169.254.x.x` | Uplinks not trusted (B2), or option 82 still on (B1). Troubleshooting table |
| ☐ | A2 | Each user PC | `ping` its gateway (`10.10.16.1` ENG, `10.10.17.1` FIN) | ✅ replies | A1 |
| ☐ | A3 | PC-HFA-ADMIN-01 | `ping 10.10.19.1` | ✅ replies | Fa0/22 in shutdown? `show port-security interface fa0/22` |
| ☐ | A4 | An ENG PC | `ping 10.10.18.10` (printer) | ✅ replies | Fa0/21 in shutdown? → recover it (C9) |

## Part B — DHCP snooping and port-security

| ☐ | # | On | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | ACC1 | `show ip dhcp snooping` | `Switch DHCP snooping is enabled` · VLANs `10,20,30,99,110,120` · `Insertion of option 82 is disabled` | Mission 2. Option 82 enabled → `no ip dhcp snooping information option` |
| ☐ | B2 | ACC1 | the same output, the interface list | `GigabitEthernet0/1` and `0/2` → Trusted `yes`, `unlimited` · `FastEthernet0/1` → `no`, rate `10` | Mission 2 (`ip dhcp snooping trust` on gi0/1–2) / 3a (`limit rate 10`) |
| ☐ | B3 | ACC2 | `show ip dhcp snooping` | The same as B1 and B2 | Mission 4a |
| ☐ | B4 | ACC1 | `show ip dhcp snooping binding` | One line per PC on ACC1 that got an address (MAC, IP, VLAN, port) | Do A1 again |
| ☐ | B5 | ACC1 | `show port-security` | `Fa0/1` … max `2` · `Restrict` · `Fa0/21` and `Fa0/22` max `1` · `Shutdown` | Mission 3 |
| ☐ | B6 | ACC2 | `show port-security` | `Fa0/1` … max `2` `Restrict` · `Fa0/21` max `1` `Shutdown` · `Fa0/22`, `Fa0/23` max `30` `Restrict` | Mission 4 |
| ☐ | B7 | ACC1 | `show port-security interface fa0/1` | `Port Status : Secure-up` | Mission 3a |
| ☐ | B8 | ACC1 | `show running-config \| include sticky` | Lines with `mac-address sticky` **and** a MAC address (the learned MACs) | Do A1–A4 again (sticky needs traffic), then `copy running-config startup-config` |

## Part C — Prove it: two attacks (then repair)

**Attack 1: a rogue DHCP server**

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | Workspace | Add a **Server** `SRV-ROGUE`, Straight-Through: **FastEthernet0** → ACC1 **Fa0/9**. Static IP `10.10.16.250` / `255.255.255.0` | Green cable | Mission 6a |
| ☐ | C2 | SRV-ROGUE → Services → DHCP | **On**, Default Gateway `10.10.16.250`, Start IP `10.10.16.200`, mask `255.255.255.0` → **Save** | — | Mission 6b |
| ☐ | C3 | An ENG PC → IP Configuration | **Static**, then **DHCP** | An address `.11`–`.254` with gateway **`10.10.16.1`**. The attack failed | `.200` / gateway `.250` → Fa0/9 is trusted, or VLAN 10 isn't snooped (B1, B2) |
| ☐ | C4 | Repair | Delete SRV-ROGUE (M7) | — | — |

**Attack 2: an unknown laptop on the printer port**

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C5 | Workspace | Delete the printer's cable. Add a **Laptop**, Straight-Through → ACC1 **Fa0/21** | — | — |
| ☐ | C6 | Laptop | Static `10.10.18.50` / `255.255.255.0` → Command Prompt → `ping 10.10.18.1` | ACC1 shows `%PORT_SECURITY-2-PSECURE_VIOLATION` and Fa0/21 goes down (red dot) | The printer's MAC wasn't learned (B8), or Fa0/21 isn't `maximum 1` / `shutdown` (B5) |
| ☐ | C7 | ACC1 | `show port-security interface fa0/21` | `Port Status : Secure-shutdown` | C6 |
| ☐ | C8 | Repair | Delete the laptop. Straight-Through: printer **FastEthernet0** → ACC1 **Fa0/21** | — | — |
| ☐ | C9 | Repair, ACC1 | `configure terminal` → `interface fa0/21` → `shutdown` → `no shutdown` → `end` | — | — |
| ☐ | C10 | An ENG PC | `ping 10.10.18.10` | ✅ replies again | Do C9 again |

## Part D — Saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | D1 | ACC1 and ACC2 | `copy running-config startup-config` → **Enter** | `[OK]` | — |
| ☐ | D2 | **File → Save** | — | Title shows `step06.pkt` | M1 |

---

## 🛠 Your own project
Run the same test in your own project. Replace:

| In the test | Use from your plan |
|---|---|
| Snooped VLANs `10,20,30,99,110,120` | every VLAN of your building that has users or devices |
| Uplinks Gi0/1–2 | your uplink ports (the only trusted ports) |
| User ports `fa0/1 - 20`, printer `fa0/21`, admin `fa0/22`, AP ports | your port plan |
| `10.10.16.x`, `10.10.18.10`, `10.10.19.1` | your PC, printer and management addresses |

Your project passes when every row matches with **your** values. C3 is the most important
test: a rogue DHCP server must **never** win.
