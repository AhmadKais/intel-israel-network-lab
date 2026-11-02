# Step 01 — Self-test

> 🧪 Do this **after** you finish step 01. It takes about 15 minutes.
> Part A is a paper test: write your answers **first**, then open the answers to check.
> These questions are **new**. They are not the exercises from the step, so you can't
> remember the answers; you have to work them out.

## How to use this test
1. Do the tests **in order**, top to bottom.
2. Write your answer, then open **✔ Answer** and compare.
3. If it matches, tick the box ☐ → ☑ (on paper, or in your copy of this file).
4. If it doesn't match, read the section named in the answer again, then try the next question.
5. You pass Part A with **at least 8 of 10**. Part B and Part C must be **all** ticked.

---

## Part A — Plan questions (no looking at design.md!)

☐ **A1.** Haifa building C is OSPF area 3. Which /20 is it, and what is its last address?
<details><summary>✔ Answer</summary>

`10.10.48.0/20`, last address `10.10.63.255` (48 + 16 − 1 = 63). → section 1.11, exercise 3.1
</details>

☐ **A2.** Marketing is the 2nd VLAN in building B. Which /24 does it get, and what is the
real IP of DIST1 in it?
<details><summary>✔ Answer</summary>

`10.10.33.0/24`. DIST1 = `.2` → `10.10.33.2`. → sections 1.8 and Mission 2
</details>

☐ **A3.** How many PCs can one /24 hold? How many devices does a /30 hold?
<details><summary>✔ Answer</summary>

/24 → 254 hosts. /30 → 2 (exactly the two ends of one cable). → section 1.7
</details>

☐ **A4.** A PC in the WIFI-GUEST VLAN of building A. What is its gateway, and which switch
is the "boss" of that VLAN?
<details><summary>✔ Answer</summary>

VLAN 120, `10.10.21.0/24` → gateway `10.10.21.1`, boss **DIST2**. → Mission 2, building A VLANs
</details>

☐ **A5.** In which range does DHCP give addresses to users? And which addresses are kept for
servers and printers?
<details><summary>✔ Answer</summary>

Users: `.11`–`.254`. Servers, printers, network devices: `.4`–`.10`. → section 1.8
</details>

☐ **A6.** Why does VLAN 998 exist, and why does it have no ports?
<details><summary>✔ Answer</summary>

It is the **native** VLAN of the trunks. It has no ports and no IP so no device can send
untagged frames into it. This blocks the VLAN-hopping attack (step 03). → section 1.10
</details>

☐ **A7.** What happens to an unused switch port in our design?
<details><summary>✔ Answer</summary>

It goes into VLAN **999** (PARKING) and is **shut down**. → section 1.10
</details>

☐ **A8.** Which network is the management VLAN of building B?
<details><summary>✔ Answer</summary>

VLAN 99, `10.10.35.0/24` (MGMT-B). → Mission 2, building B and C VLANs
</details>

☐ **A9.** Write the hostname for: the 1st distribution switch (a Layer-3 switch) of building B
in Haifa; the 3rd PC in Haifa's HR department.
<details><summary>✔ Answer</summary>

`SWL3-HFA-B-DIST1`, `PC-HFA-HR-03`. → Mission 2, naming rule
</details>

☐ **A10.** Kiryat Gat is the 3rd city. Following the recipe, which /16 does it get? Why are
the links between routers **not** inside a building block?
<details><summary>✔ Answer</summary>

`10.30.0.0/16` (10.10 Haifa, 10.20 Jerusalem, 10.30 Kiryat Gat). Links, loopbacks and servers
go in the **core** piece, so each building block holds only its own VLANs. That keeps each
building summarizable. → section 1.11
</details>

## Part B — Your Packet Tracer file

| ☐ | # | Type / do | Must see | If not |
|---|---|---|---|---|
| ☐ | B1 | **File → Open** → `my-work/step01.pkt` | Title shows `step01.pkt` | Mission 5a / 5d |
| ☐ | B2 | Look at the workspace | 3 rectangles side by side, empty space above and on the right | Mission 5b |
| ☐ | B3 | Read the labels | `HAIFA - BUILDING A (OSPF area 1)`, `… B (OSPF area 2)`, `… C (OSPF area 3)` inside the rectangles; `HAIFA HQ - CORE (OSPF area 0)` above them | Mission 5c — double-click a note to fix its text |

## Part C — Your own plan

| ☐ | # | Check in `my-project/plan.md` | Must be true |
|---|---|---|---|
| ☐ | C1 | Parts A–G | Every part is filled in. No empty cells, no `…` |
| ☐ | C2 | Your blocks | Every /24 lies **inside** its building's block (do A1/A2 again with **your** numbers) |
| ☐ | C3 | Your values | No Intel name, no `10.10.x.x` / `10.20.x.x` / `10.30.x.x`, no `Intel#…` password |
| ☐ | C4 | Approval | Your teacher approved the plan (signature or message). **Don't start step 02 without it** |
