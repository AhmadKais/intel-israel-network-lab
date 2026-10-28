# Packet Tracer basic moves — click by click

Every step in this course uses the same few actions again and again: place a device, connect
a cable to an exact port, open the CLI, give a PC an address…

They are all written here **once**, click by click, with a number (**M1**, **M2**…). When a
step says **"(how? → M5)"**, come here, follow M5, and go back to the step.

> Keep this page open in a second browser tab while you work.

| # | Move | # | Move |
|---|---|---|---|
| [M1](#m1--open-the-previous-step-and-save-it-under-the-new-name) | Open the previous step and save under the new name | [M13](#m13--save-the-device-configuration) | Save the device configuration |
| [M2](#m2--place-a-device) | Place a device | [M14](#m14--give-a-pc-server-or-laptop-a-fixed-static-address) | Give a PC / server a fixed address |
| [M3](#m3--place-many-devices-of-the-same-model) | Place many devices of the same model | [M15](#m15--give-a-printer-a-fixed-address) | Give a printer a fixed address |
| [M4](#m4--rename-a-device-display-name) | Rename a device | [M16](#m16--set-a-pc-to-get-its-address-automatically-dhcp) | Set a PC to DHCP |
| [M5](#m5--connect-a-cable-to-an-exact-port) | Connect a cable to an exact port | [M17](#m17--run-a-test-from-a-pc-ping-tracert-ipconfig) | Run a test from a PC (ping…) |
| [M6](#m6--check-a-cable-is-on-the-right-ports) | Check a cable is on the right ports | [M18](#m18--make-time-pass-fast-forward-time) | Fast Forward Time |
| [M7](#m7--delete-a-cable-or-a-device) | Delete a cable or a device | [M19](#m19--draw-a-rectangle-and-label-it) | Draw a rectangle and label it |
| [M8](#m8--add-a-module-power-supply-wifi-card-extra-ports) | Add a module (power supply, WiFi card…) | [M20](#m20--turn-a-service-on-in-a-server) | Turn a service on in a server |
| [M9](#m9--open-the-cli-and-reach-the--prompt) | Open the CLI and reach the `#` prompt | [M21](#m21--use-simulation-mode-slow-motion) | Simulation mode (slow motion) |
| [M10](#m10--type-a-block-of-commands-the-rules) | Type a block of commands — the rules | [M22](#m22--take-a-screenshot-for-your-hand-in) | Take a screenshot |
| [M11](#m11--where-am-i-read-the-prompt) | Where am I? Read the prompt | [M23](#m23--i-broke-everything--go-back-to-the-last-good-file) | I broke everything — go back |
| [M12](#m12--i-typed-something-wrong--fix-it) | I typed something wrong — fix it | | |

---

## M1 — Open the previous step and save it under the new name
Every step starts from the file you saved at the end of the step before.
1. Open Packet Tracer.
2. Top menu: click **File** → **Open**.
3. Go to your course folder → **my-work** → click the file of the **previous** step
   (for step 05 that is `step04.pkt`) → **Open**.
4. **Straight away**: **File** → **Save As** → type the **new** name (for step 05: `step05.pkt`) →
   **Save**.

✅ The title bar at the top of the window now shows the new name.
Why? The old file stays untouched. If you break something, you can always go back to it (M23).

---

## M2 — Place a device
1. Look at the **bottom-left corner** of the window. There are two rows of small icons: the
   **top row** is the *category* (Network Devices, End Devices, Components, Connections…), the
   **bottom row** is the *sub-category* (Routers, Switches, Wireless…).
2. Click the category, then the sub-category (see the table below).
3. The **models** appear in the box to the right of it. Hover over a model to see its name.
4. **Drag** the model into the white workspace (hold the mouse button, move, let go).

Where to find each device used in this course:

| Device | Top row (category) | Bottom row | Model (hover to read the name) |
|---|---|---|---|
| Access switch | Network Devices | **Switches** | `2960` (shows as `2960-24TT`) |
| Layer-3 switch | Network Devices | **Switches** | `3650` (shows as `3650-24PS`) |
| Router | Network Devices | **Routers** | `2911` |
| Firewall | Network Devices | **Security** | `5506-X` (ASA) |
| Access point | Network Devices | **Wireless Devices** | `AccessPoint-PT` |
| Cloud (internet / DSL) | Network Devices | **WAN Emulation** | `Cloud-PT` |
| DSL modem | Network Devices | **WAN Emulation** | `DSL-Modem-PT` |
| PC | End Devices | **End Devices** | `PC` |
| Laptop | End Devices | **End Devices** | `Laptop` |
| Server | End Devices | **End Devices** | `Server` |
| Printer | End Devices | **End Devices** | `Printer` |

✅ The device appears with a default name under it (e.g. `Switch0`, `PC3`, `Router1`).

> Can't find the category? The window may be too small. Maximise Packet Tracer (double-click
> its title bar) and the icons appear.

---

## M3 — Place many devices of the same model
1. Choose the category and sub-category (M2).
2. Hold **Ctrl** and **click** the model once (don't drag). The mouse pointer changes.
3. Let go of Ctrl. Now **click** in the workspace — one device appears per click.
4. Press **Esc** when you have enough.

---

## M4 — Rename a device (Display Name)
1. Click the device once → its window opens.
2. Click the **Config** tab (at the top of the window).
3. On the left, make sure **Settings** (under GLOBAL) is selected.
4. Click in the **Display Name** box, delete the old name, type the new name **exactly**
   (capital letters, dashes).
5. Close the window with the **X** at the top right.

✅ The new name is under the device on the workspace.

> **Display Name ≠ hostname.** The Display Name is only the label on the drawing. The name
> *inside* a switch or router (the prompt) is set with the `hostname` command in the CLI.
> We always make them the same.

---

## M5 — Connect a cable to an exact port
1. Bottom-left: click the **Connections** category (the orange **lightning bolt** ⚡).
2. In the model box, click the cable you need. Hover to read the names:

| Cable | Looks like | Use it between |
|---|---|---|
| **Copper Straight-Through** | solid black line | **different** device types: PC ↔ switch, server ↔ switch, printer ↔ switch, switch ↔ router, switch ↔ ASA, AP ↔ switch |
| **Copper Cross-Over** | dashed black line | **same** device types: switch ↔ switch, router ↔ router |
| **Phone** | | DSL modem ↔ cloud (step 12 only) |
| ~~Automatic (⚡)~~ | | **Never** in this course — it chooses the port by itself, and the project needs exact ports |

3. Click the **first** device. A small list of its ports pops up.
4. Click the exact port the step names (e.g. **FastEthernet0/11**). Ports already in use are
   greyed out.
5. Move the mouse to the **second** device (a line follows the mouse) and click it.
6. Click its exact port from the list.

✅ A line joins the devices. Each end has a coloured dot:

| Dot | Meaning | What to do |
|---|---|---|
| 🟢 green | Working | Nothing |
| 🟠 orange | Starting up (switch is checking for loops) | Wait, or press **Fast Forward Time** (M18). If it stays orange after that, it may be a backup link that STP blocks on purpose (from step 04) |
| 🔴 red | Not working | Wrong cable type? Port shut down (`shutdown`)? Device switched off? 3650 without a power supply (M8)? |

> **Port names:** `FastEthernet0/11` = Fa0/11, `GigabitEthernet0/1` = Gi0/1,
> `GigabitEthernet1/0/1` = Gi1/0/1, `GigabitEthernet1/1/1` = Gi1/1/1. The list shows the long
> names; the steps often use the short ones.

> **The port I need is not in the list.** You probably clicked the wrong device, or the device
> needs a module first (M8). Press **Esc** to cancel the cable and try again.

---

## M6 — Check a cable is on the right ports
Wrong ports are the **#1 reason** a step doesn't work. Check every cable after you place it.
1. Turn on port labels once: **Options** → **Preferences** → **Interface** tab → tick
   **Always Show Port Labels in Logical Workspace** → close. Now each cable shows its port name
   at both ends.
2. Or: hover the mouse over a device and wait one second. A box lists every port, its link
   state and its address.
3. In the CLI of a switch or router: `show cdp neighbors` — lists every neighbour device and
   the ports on **both** sides:
```
Device ID     Local Intrfce   Holdtme  Capability  Platform  Port ID
SW-HFA-A-ACC1 Gig 1/0/1       160      S           2960      Gig 0/1
```
   Read it as: "my port Gi1/0/1 is connected to SW-HFA-A-ACC1, on its port Gi0/1".

**Wrong port?** Delete the cable (M7) and connect it again (M5).

---

## M7 — Delete a cable or a device
1. In the toolbar at the top, click the **Delete** tool (a red **X**), or press the **Delete** key.
2. Click the cable (or device) you want to delete.
3. Press **Esc** to go back to the normal arrow (**Select** tool).

---

## M8 — Add a module (power supply, WiFi card, extra ports)
Some devices need a part added before they work. The rule: **switch off → add → switch on.**
1. Click the device → **Physical** tab. You see a picture of the device.
2. **Switch it off:** click the small **power button** on the picture (on a 3650 or laptop it may
   not be needed — see below). The green light next to it goes off.
3. On the left there is a list of **MODULES**. Click a module name to see its picture at the
   bottom.
4. **Drag** the module from the picture at the bottom into the empty slot on the device picture.
5. **Switch it on** again (click the power button).
6. Close the window.

The modules used in this course:

| Device | Module | Where it goes | Needed in |
|---|---|---|---|
| **3650** switch | `AC-POWER-SUPPLY` | the empty slot on the right of the back panel (zoom in if needed) | **every** 3650 — without it the switch is off and every link is red |
| **Laptop** | `WPC300N` (WiFi card) | first drag the old card (`PT-LAPTOP-NM-1CFE`) **out** of the laptop into the module list, then drag `WPC300N` into the empty slot | step 10 |

✅ The device is on (green light) and its links come up.

---

## M9 — Open the CLI and reach the `#` prompt
1. Click the switch / router / ASA → click the **CLI** tab.
2. Click inside the **black area** and press **Enter**.
3. What you see depends on the device:

| You see | Do this |
|---|---|
| `Would you like to enter the initial configuration dialog? [yes/no]:` (new router) | type `no` → **Enter** → press **Enter** again |
| `Switch>` or `Router>` or `ciscoasa>` | type `enable` → **Enter** |
| `Password:` after `enable` on a new ASA | just press **Enter** (empty) |
| a banner and then `Password:` / `Username:` (a device you already secured) | type the console password from [passwords.md](passwords.md) (nothing appears while you type — normal) → **Enter**; then `enable` and the enable secret |
| `SW-HFA-A-ACC1#` (any name, ending in `#`) | you are ready |

4. To change settings, type `configure terminal` → **Enter**. The prompt ends in `(config)#`.

---

## M10 — Type a block of commands: the rules
Steps show commands in grey boxes like this:
```
! this is a comment - don't type it
hostname SW-HFA-A-ACC1
interface fa0/1
 description ENGINEERING users
```
1. **One line → Enter → look → next line.** Never type the next line before you read the answer
   to the previous one.
2. Lines that start with **`!`** are comments. **Don't type them.**
3. The **space at the start** of some lines is only there to make the box easier to read. You
   can type it or not — it changes nothing.
4. Type **exactly** what is shown: same spelling, same capital letters in names and passwords.
5. Keyboard on **English** (Alt+Shift or Win+Space) and **Caps Lock off**.
6. **Look at the prompt** after every line (M11). If the step says the prompt should now be
   `(config-if)#` and it isn't — stop and fix before you continue.
7. The answer `% Invalid input detected at '^' marker` means the line was **not** accepted.
   The `^` points to the mistake. Fix it (M12) before you go on.
8. Words in a box are **values from the example**. In your own project you replace them with
   the values from your plan.

> **Type or paste?** Type every command the first time you meet it. You may paste a block you
> have already typed by hand on an earlier device: copy it from the page (**Ctrl+C**), click in
> the CLI, and **right-click → Paste**. After pasting, scroll up and check there is no
> `% Invalid input`.

---

## M11 — Where am I? Read the prompt
The text before the cursor (the **prompt**) tells you where you are. A command typed in the
wrong place fails.

| Prompt ends with | You are in | Allowed here | To get here |
|---|---|---|---|
| `>` | user mode (the lobby) | ping, a few `show` | — |
| `#` | privileged mode (manager's office) | every `show`, save, reload | `enable` |
| `(config)#` | global configuration | settings for the whole device | `configure terminal` |
| `(config-if)#` | one interface | settings for that port | `interface fa0/1` |
| `(config-if-range)#` | several interfaces | settings for all of them | `interface range fa0/1 - 10` |
| `(config-vlan)#` | one VLAN | `name` | `vlan 10` |
| `(config-line)#` | console / VTY lines | passwords, timeouts | `line console 0` / `line vty 0 15` |
| `(config-router)#` | a routing protocol | `network`, `router-id`… | `router ospf 1` |

- `exit` → **one** level back.
- `end` → straight back to `#`.
- Need a `show` command while you are in `(config…)#`? Put `do` in front: `do show vlan brief`.

---

## M12 — I typed something wrong: fix it
| What happened | Fix |
|---|---|
| `% Invalid input detected at '^' marker` | Press **↑** (the line comes back), fix it with ← → and Backspace, press **Enter** |
| `% Incomplete command` | You stopped too early. Press **↑**, add the missing word. `?` shows what can come next |
| `% Ambiguous command` | The short word matches two commands. Type more letters |
| It says `Translating "xyz"...` and hangs | You typed a typo in `#` mode. Press **Ctrl+Shift+6** and wait |
| A command **was** accepted but it's wrong (wrong IP, wrong VLAN) | Type the same command with **`no`** in front to remove it, then type the right one. E.g. `no switchport access vlan 20` → `switchport access vlan 10` |
| A wrong IP on an interface | Just type the right `ip address …` line again — it replaces the old one |
| Wrong `hostname` | Type `hostname` again with the right name |
| Many things are wrong and you're lost | M23 |

---

## M13 — Save the device configuration
Commands change the device **now** (running-config). They are lost when the device restarts —
unless you save them into the startup-config.
1. Reach the `#` prompt (type `end` if you are in `(config…)#`).
2. Type `copy running-config startup-config` → **Enter**.
3. It asks `Destination filename [startup-config]?` → press **Enter**.

✅ You see `[OK]`. (Short forms: `copy run start` or `wr`. On the ASA: `write memory`.)

> Saving the device (`wr`) and saving the file (**File → Save**) are two different things.
> **Do both** at the end of every mission.

---

## M14 — Give a PC, server or laptop a fixed (static) address
1. Click the PC → **Desktop** tab → click **IP Configuration**.
   (Server: **Desktop** → **IP Configuration** too.)
2. At the top, select **Static**.
3. Click in **IPv4 Address**, type the address.
4. Click in **Subnet Mask**. PT fills in a mask by itself — check it is the one in the step
   (usually `255.255.255.0`); if not, delete it and type the right one.
5. **Default Gateway**: type it (or leave it empty if the step says so).
6. **DNS Server**: only if the step gives one.
7. Close the window with the **X**. There is no "Save" button — PT keeps it as soon as you type.

✅ Hover over the PC: it shows the address.

---

## M15 — Give a printer a fixed address
A printer has **no Desktop tab**.
1. Click the printer → **Config** tab.
2. On the left, **Settings** (under GLOBAL) → type the **Gateway**.
3. On the left, click **FastEthernet0** (under INTERFACE) → select **Static** → type the
   **IPv4 Address** and **Subnet Mask**.
4. Close the window.

---

## M16 — Set a PC to get its address automatically (DHCP)
1. Click the PC → **Desktop** → **IP Configuration**.
2. At the top, select **DHCP**.
3. Wait. The text `Requesting IP Address...` appears, then `DHCP request successful.`

✅ IPv4 Address, Subnet Mask and Default Gateway fill in by themselves.
❌ `DHCP failed. APIPA is being used.` and an address `169.254.x.x` → the DHCP server was not
reached. Click **Static** and then **DHCP** again to retry. If it fails again, read
"If something doesn't work" in the step.

---

## M17 — Run a test from a PC (ping, tracert, ipconfig)
1. Click the PC → **Desktop** → **Command Prompt**.
2. Type the command and press **Enter**:

| Command | What it tells you |
|---|---|
| `ping 10.10.16.1` | Can I reach this address? |
| `ipconfig` | My address, mask and gateway |
| `ipconfig /all` | Also my MAC address and DNS server |
| `tracert 10.10.200.13` | Which routers the packet passes on the way |
| `nslookup www.intel.lab` | Which IP this name belongs to (DNS test) |
| `ssh -l admin 10.10.19.11` | Remote login to a switch (step 07) |

How to read a ping:

| You see | Means |
|---|---|
| `Reply from …` (4 times) | ✅ works |
| `Request timed out.` once, then `Reply from …` | ✅ works — the first try was waiting for ARP. Normal |
| `Request timed out.` 4 times | ❌ no answer. Check addresses, gateway, cables, VLAN |
| `Destination host unreachable.` | ❌ a router (or the PC itself) has no route to there |

> A long ping won't stop? Press **Ctrl+C**.

---

## M18 — Make time pass (Fast Forward Time)
Links, Spanning Tree, OSPF, BGP, DHCP… all need a little simulated time.
1. At the bottom of the window (bottom left, next to the clock) click **Fast Forward Time**
   (or press **Alt+D**).
2. Press it 2–3 times.

Use it whenever a step says "wait", when lights are orange, or before you test.

---

## M19 — Draw a rectangle and label it
We draw a rectangle around each building / site to keep the drawing readable.
1. In the toolbar at the top, click **Drawing Palette** (the icon with a pencil / shapes).
   A small palette opens.
2. Click **Rectangle**. Choose a light colour. (Optional: untick **Fill** so the devices stay
   visible.)
3. In the workspace, **drag** from the top-left corner to the bottom-right corner of the
   rectangle.
4. Close the palette.
5. In the toolbar, click **Place Note** (the icon with a sheet of paper, or press **N**).
6. Click inside the rectangle near its top, type the label (e.g. `HAIFA - BUILDING A`), and
   click somewhere empty.

> Draw the rectangle **before** you put devices in it — a rectangle drawn later can cover them.
> If it does: right-click the rectangle → **Send to back** (if your version has it), or delete
> it and draw it again bigger.

---

## M20 — Turn a service on in a server
1. Click the server → **Services** tab.
2. On the left, click the service (HTTP, DHCP, DNS, EMAIL, FTP, NTP, SYSLOG, AAA).
3. Switch it **On** (radio button at the top of that page).
4. Fill in what the step says, and press **Add** / **Save** if the page has such a button.
5. Turn **off** every service the step does not need (security: fewer open doors).

---

## M21 — Use Simulation mode (slow motion)
1. Bottom right: click **Simulation** (next to **Realtime**). The *Simulation Panel* opens.
2. In the panel: **Show All/None** (clears all filters) → **Edit Filters** → tick only the
   protocols the step names (e.g. ICMP, ARP) → close the filter window.
3. Make traffic: e.g. a ping from a PC (M17), or the **Add Simple PDU** tool (closed envelope
   in the toolbar) → click the source → click the destination.
4. Press **Capture / Forward** (▶|) once per hop. Envelopes move on the drawing.
5. Click an envelope to open it and see what is inside (layers, addresses, VLAN tag…).
6. When you finish: click **Realtime** again. **Don't save the file in Simulation mode.**

---

## M22 — Take a screenshot for your hand-in
- **Windows:** **Win+Shift+S** → drag a box around the part you want → paste it in your
  document (**Ctrl+V**).
- **Linux:** **Print Screen** (or **Shift+Print Screen** to choose an area).

Make sure the screenshot shows **the device name / prompt** and **the result** (e.g. the
`Reply from` lines). A screenshot without the device name proves nothing.

---

## M23 — I broke everything: go back to the last good file
1. **File** → **Open** → `my-work/` → the file of the **previous** step → **Open**.
   (Packet Tracer asks if you want to save the broken file — answer **No**.)
2. **File** → **Save As** → the current step's name → **Save** (replace it).
3. Start the current step again from Mission 1.

> This is why every step starts with "Save As" (M1). A step done twice is learnt twice.
