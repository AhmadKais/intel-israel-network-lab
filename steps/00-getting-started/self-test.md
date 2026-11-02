# Step 00 — Self-test

> 🧪 Do this **after** you finish step 00, on your saved file `my-work/step00.pkt`.
> It takes about 10 minutes. It tells you whether your work is really correct before you go on.

## How to use this test
1. Do the tests **in order**, top to bottom.
2. For each test, do what the **Type / do** column says, then compare your screen with the
   **Must see** column. ❌ in "Must see" means the test **must fail**. If it fails, that's a pass.
3. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
4. If it doesn't match, do what the **If not** column says, then run the same test again.
   Don't skip a test you couldn't pass.
5. Every box ticked → step 00 works. Go to step 01.

"M…" numbers (M1, M9, M17 …) are the basic moves in [docs/basic-moves.md](../../docs/basic-moves.md).

---

## Part A — Your file

| ☐ | # | Type / do | Must see | If not |
|---|---|---|---|---|
| ☐ | A1 | **File → Open** → `my-work/step00.pkt` | The window title shows `step00.pkt`. You see Switch0, PC0 and PC1 | Mission 10 — the file wasn't saved where you think. Search your computer for `step00.pkt` |
| ☐ | A2 | Look at the two cables | **Green** dots at both ends. Labels `Fa0/1` and `Fa0/2` next to the switch | Red dot → Mission 4c (Straight-Through). No labels → Mission 3b |
| ☐ | A3 | Hover the mouse over PC0, then over PC1 | `192.168.1.1` and `192.168.1.2` | Mission 5a. Did you put PC1 back after Mission 8? |

## Part B — It works

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | B1 | PC0 → Desktop → Command Prompt (M17) | `ping 192.168.1.2` | `Reply from 192.168.1.2` (at least 3 replies — the first may time out) | Mission 5 — check both addresses and both cables |
| ☐ | B2 | Switch0 → CLI (M9) → **Enter** | — | `Switch>` | Click inside the black area and press Enter again |
| ☐ | B3 | Switch0 | `enable` | `Switch#` | You typed it wrong — try again |
| ☐ | B4 | Switch0 | `show mac address-table` | 2 lines, `DYNAMIC`, ports `Fa0/1` and `Fa0/2` | The table is empty → do B1 again, then repeat B4 (entries expire after 5 minutes) |

## Part C — Prove that a router is needed (then undo it)

| ☐ | # | On | Type / do | Must see | If not |
|---|---|---|---|---|---|
| ☐ | C1 | PC1 → Desktop → IP Configuration | Change the IPv4 address to `192.168.2.2`, close | Hover over PC1 → `192.168.2.2` | Mission 8 step 1 |
| ☐ | C2 | PC0 → Command Prompt | `ping 192.168.2.2` | ❌ `Request timed out` ×4 — **correct!** Different network, no router | If it replies, PC1's address didn't change — repeat C1 |
| ☐ | C3 | PC1 → IP Configuration | Put it back: `192.168.1.2` | B1 works again | — |
| ☐ | C4 | **File → Save** | — | Title still `step00.pkt` | M1 |

## Part D — Explain it (say the answer out loud, in one sentence)

| ☐ | # | Question | A good answer contains |
|---|---|---|---|
| ☐ | D1 | `Switch>` vs `Switch#`? | `>` = user mode, look only a little · `#` = privileged mode, see everything (`enable`) |
| ☐ | D2 | MAC vs IP address? | MAC = burned into the network card, used inside one network · IP = given by the admin, used to reach other networks |
| ☐ | D3 | Why did C2 fail? | The two PCs are on different networks (different "streets"); without a router no packet crosses |
| ☐ | D4 | What was the first envelope in Simulation mode, and why did it go to every port? | ARP — "who has 192.168.1.2?" — a broadcast, so the switch floods it to every port |

---

## 🛠 Your own project
Step 00 has nothing to configure in your project yet. Check instead:
- ☐ Your company / topic is written down and your teacher approved it.
- ☐ `my-project/progress.md` exists (copied from [docs/progress-template.md](../../docs/progress-template.md)).
