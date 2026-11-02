# Step 07 — Self-test

> 🧪 Do this **after** you finish step 07, on your saved file `my-work/step07.pkt`.
> It takes about 20 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Do the tests **in order**, top to bottom.
2. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
3. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
4. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
5. Every box ticked → step 07 works. Go to step 08.

From now on the console asks for **Username:** `admin` / **Password:** `Intel#Adm1n`.
"M…" numbers are the basic moves in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — The admin PC

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | A1 | PC-HFA-ADMIN-01 → IP Configuration | Look | **Static**, `10.10.19.100`, `255.255.255.0`, gateway `10.10.19.1` | Mission 1 |
| ☐ | A2 | PC-HFA-ADMIN-01 → Command Prompt | `ping 10.10.19.11` | ✅ replies | Step 05 Mission 8 |

## Part B — SSH works from the admin PC

From PC-HFA-ADMIN-01 → Command Prompt, for each row: type the command → `Password:` →
`Intel#Adm1n` → check the prompt → `exit`.

| ☐ | # | Type | Must see | If not |
|---|---|---|---|---|
| ☐ | B1 | `ssh -l admin 10.10.19.11` | The banner, then `SW-HFA-A-ACC1#` | `Connection refused` → D3 (the permit line must be exactly `permit 10.10.19.0 0.0.0.255`) and D4 (the name after `access-class` must be exactly `MGMT-ONLY`). `timed out` → A2 |
| ☐ | B2 | `ssh -l admin 10.10.19.12` | `SW-HFA-A-ACC2#` | Mission 5 on ACC2 |
| ☐ | B3 | `ssh -l admin 10.10.19.2` | `SWL3-HFA-A-DIST1#` | Mission 5 on DIST1 |
| ☐ | B4 | `ssh -l admin 10.10.19.3` | `SWL3-HFA-A-DIST2#` | Mission 5 on DIST2 |

## Part C — The refusals (they prove the security works)

| ☐ | # | From | Type | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | PC-HFA-ADMIN-01 | `telnet 10.10.19.11` | ❌ refused — **correct!** | `transport input ssh` missing (Mission 3c) |
| ☐ | C2 | An ENG PC | `ssh -l admin 10.10.19.11` | ❌ refused — **correct!** (source 10.10.16.x isn't on the list) | `access-class MGMT-ONLY in` missing, or the ACL permits too much (Mission 3b) |
| ☐ | C3 | An ENG PC | `ssh -l admin 10.10.16.2` (DIST1, in the user VLAN) | ❌ refused — **correct!** | DIST1's `line vty` (Mission 5) |

## Part D — Check on the switch

Click **SW-HFA-A-ACC1** → CLI → **Enter**.

| ☐ | # | Type / do | Must see | If not |
|---|---|---|---|---|
| ☐ | D1 | Log in at the console | `Username:` → `admin` → `Password:` → `Intel#Adm1n` → `SW-HFA-A-ACC1#` (privilege 15, no `enable` needed) | Asks only `Password:` → `login local` missing under `line console 0` (Mission 3d) |
| ☐ | D2 | `show ip ssh` | `SSH Enabled - version 2.0` | `ip ssh version 2` (Mission 3a). Disabled → no RSA keys (Mission 2b) |
| ☐ | D3 | `show access-lists MGMT-ONLY` | `permit 10.10.19.0 0.0.0.255` and `deny any log`. The deny line shows `(N match(es))` after C2 | Mission 3b |
| ☐ | D4 | `show running-config` → the `line vty` part | `transport input ssh` · `login local` · `access-class MGMT-ONLY in` | Mission 3c |
| ☐ | D5 | Repeat D2–D4 on ACC2, DIST1, DIST2 (through SSH from the admin PC is fine) | The same | Mission 5 |

## Part E — Login blocking (only if `login block-for` was accepted)

| ☐ | # | From | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | E1 | PC-HFA-ADMIN-01 | `ssh -l admin 10.10.19.11` with a **wrong** password 3 times quickly | Further logins are refused for **120 seconds**, even with the right password | `login block-for 120 attempts 3 within 60` (Mission 3a) |
| ☐ | E2 | Wait 2 minutes (or Fast Forward Time) | `ssh -l admin 10.10.19.11` with `Intel#Adm1n` | Works again | — |

## Part F — Saved

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | F1 | Each of the 4 switches | `copy running-config startup-config` → **Enter** | `[OK]` | — |
| ☐ | F2 | **File → Save** | — | Title shows `step07.pkt` | M1 |

---

## 🛠 Your own project
Run the same test in your own project. Replace:

| In the test | Use from your plan |
|---|---|
| `admin` / `Intel#Adm1n` | your admin user and password |
| `intel.lab` | your domain |
| `10.10.19.0 0.0.0.255`, `10.10.19.x` | your management subnet and addresses |
| An ENG PC | any PC that is **not** in your management VLAN |

Your project passes when every row matches with **your** values. The two most important
checks are B1 and C2: the admin PC gets in, and a user PC is refused.
