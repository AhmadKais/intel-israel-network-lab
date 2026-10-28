# Step 00 — Getting started: networking basics + a tour of Packet Tracer

> ⏱ about 2 lessons (2 × 90 min) · 📁 start from: nothing · 💾 save as: `my-work/step00.pkt`

## 🗺 Your missions in this step
Tick each box when you finish it. Do them **in order**.

- [ ] **Mission 1** — Download the course files (10 min)
- [ ] **Mission 2** — Install Packet Tracer and log in (15 min)
- [ ] **Mission 3** — Tour the window and turn on two settings (15 min)
- [ ] **Mission 4** — Build your first network: 2 PCs + 1 switch (15 min)
- [ ] **Mission 5** — Give the PCs addresses and ping (10 min)
- [ ] **Mission 6** — Your first CLI: look inside the switch (10 min)
- [ ] **Mission 7** — Watch it in slow motion (Simulation mode) (15 min)
- [ ] **Mission 8** — Prove that a router is needed (5 min)
- [ ] **Mission 9** — Practise the CLI modes (15 min)
- [ ] **Mission 10** — Save your work (5 min)
- [ ] **Exercises** A and B
- [ ] **🛠 Your own project** — check your proposal, create your progress table

## What you will have at the end
- Packet Tracer installed and you know **every button** you will use in this project.
- You understand the basic words: switch, router, IP, MAC, mask, gateway, ping.
- You built your first tiny network and watched it work:

```
   PC0 ──────── [ Switch0 ] ──────── PC1
192.168.1.1                      192.168.1.2
```

## What you hand in at the end of this step
1. The file `my-work/step00.pkt`.
2. A screenshot of a successful ping (Mission 5) and a screenshot of the MAC table (Mission 6).
3. A one-sentence answer to each: what is the difference between `Switch>` and `Switch#`?
   What is the difference between a MAC address and an IP address?
4. Your progress table `my-project/progress.md`, with step 00 ticked (see 🛠 Your own project).

---

## How to read every step of this course

Every step looks the same. Learn these signs once:

| Sign | Means |
|---|---|
| 🗺 **Your missions** | The to-do list of the step. Tick each box when the mission is finished |
| **Mission N** | One small job (5–20 min). Finish it and check it **before** you start the next one |
| 🧠 **Why** | One line: why we do this mission |
| 👉 **Where** | Which device to click, and which prompt you must see before you type |
| ⌨️ **Type** | The commands. Type them one line at a time |
| ✅ **You should see** | What the screen shows when the mission worked. **Compare carefully** |
| ❌ **If not** | The usual reasons it didn't work, and the fix |
| **(how? → M5)** | A Packet Tracer action explained click by click in [docs/basic-moves.md](../../docs/basic-moves.md). Open it in a second tab |
| 🏋 **Exercise** | A task you do alone. Try first, then open **Hint 1**, then **Hint 2**, and only then the **Answer** |
| 🛠 **Your own project** | The same step again, in your own company's project |
| 🎓 / 🔁 | Exam preparation: explain it, then do it again without the guide |

> **The golden rule:** never start mission N+1 while mission N shows ❌. Fix it first — a
> mistake you carry forward becomes ten mistakes later.

---

## Part 1 — Understand it first (simple English)

A **network** is a group of devices connected so they can send data to each other.
The whole project is about connecting devices *correctly*, *redundantly* (if one thing
breaks, another takes over) and *securely*.

### 1.1 The devices you will use

| Device | What it does | Everyday comparison |
|---|---|---|
| **PC / laptop** | Creates and uses data (browsing, email). Called an **end device**. | A person sending and receiving letters |
| **Switch** | Connects devices **in the same place** (same office / building). Knows each device by its **MAC address**. | The mail room inside one office building — delivers letters between rooms |
| **Router** | Connects **different networks** together (building ↔ building, company ↔ internet). Works with **IP addresses**. | The post office between cities — decides which road a letter takes |
| **Layer-3 switch** | A switch that can *also* route. Fast and has many ports. We use it a lot. | A mail room that can also send mail to other cities |
| **Server** | A computer that gives a **service** to others: web pages, email, DNS, DHCP… | A shop or a government office: you go there to get something |
| **Firewall** | Checks traffic and blocks what is not allowed. | The security guard at the front door |
| **Access point** | Connects wireless devices to the wired network. | A wireless "socket" |

### 1.2 Two kinds of addresses

| | **MAC address** | **IP address** |
|---|---|---|
| Example | `00E0.F7A1.3B2C` | `10.10.16.25` |
| Who sets it | Burned into the network card at the factory | The admin, or DHCP (automatic) |
| Changes? | Never | Yes — when the device moves to another network |
| Used by | **Switches** — to deliver inside the local network | **Routers** — to deliver between networks |
| Comparison | Your **ID number** (ת"ז) — always the same | Your **home address** — changes when you move |

### 1.3 Subnet mask — "which part is the street, which part is the house"
An IPv4 address is 4 numbers (0–255). The **subnet mask** says how many of them describe
the **network** (the street) and how many describe the **host** (the house on that street).

```
IP:    192.168.1.25
Mask:  255.255.255.0      -> the first 3 numbers are the "street"
         street = 192.168.1      house = 25
```
- `192.168.1.25` and `192.168.1.40` → **same street** → they talk directly through the switch.
- `192.168.1.25` and `192.168.2.40` → **different streets** → they need a **router**.

### 1.4 Default gateway — "where do I send mail for other streets?"
The **default gateway** is the router's address on *your* street. When a PC wants to reach
an address on another network, it hands the packet to the gateway, and the router takes it
from there. No gateway → the PC can only talk to its own street.

### 1.5 Ping — "are you there?"
`ping 192.168.1.2` sends a small "are you there?" message. If a reply comes back, the path
works **in both directions**. It is the test you will use most in this project.

### 1.6 Packet Tracer, IOS and the CLI
- **Packet Tracer (PT)** — Cisco's free simulator. Devices behave like real Cisco equipment,
  so you can build a 150-device company network on a laptop.
- **IOS** — the operating system inside Cisco routers and switches.
- **CLI** (command-line interface) — you configure IOS by **typing commands**. Real network
  engineers work this way, the exams test it, and the project is graded on it.

### 1.7 How this course works
The course teaches every topic on **one worked example** — the network of Intel Israel. Your
final project is **your own** company (the one in the project proposal you submitted). In every
step you do two things:

| | What you do | Where you save |
|---|---|---|
| **1. Learn** | Follow the step exactly, with the Intel values. Type every command, check it works, do the exercises | `my-work/` |
| **2. Build your own** | Do the same step in your project, with your values. The **🛠 Your own project** section at the end of every step says exactly what to change | `my-project/` |

Why this way? You first see the topic working on an example that has already been checked.
Then, when something doesn't work in your project, you know the problem is in your values,
not in the commands.

---

## Mission 1 — Download the course files
🧠 **Why:** you need the steps, the reference docs, and a place to save your work.

1. Open the course link your teacher gave you (the GitHub page of this course).
2. Click the green **Code** button (top right of the file list) → **Download ZIP**.
3. Open your **Downloads** folder. Right-click the ZIP → **Extract All…** → choose your
   **Documents** folder → **Extract**.
4. Open the new folder. Its name starts with `intel-israel-network-lab`. This is your
   **course folder**. Inside it you see:

| Folder | What is in it |
|---|---|
| `steps/` | The steps (lessons). One per topic, in order |
| `docs/` | Reference: the example's plan, passwords, **basic moves**, the plan template for your project |
| `my-work/` | Save your practice files on the example here (`step00.pkt`, `step01.pkt`…) |
| `my-project/` | Save your own project here |

✅ You can see the four folders `steps`, `docs`, `my-work`, `my-project`.

> **Tip:** read the steps on the website (in the browser) — tables and pictures display
> properly there. Save your `.pkt` files in the course folder on your computer.

---

## Mission 2 — Install Packet Tracer and log in
🧠 **Why:** you build the whole project in Packet Tracer.

1. Go to **netacad.com** and create a free account (or log in).
2. After logging in, look in the top menu for **Resources → Download Packet Tracer**.
   If you can't find it, go straight to `https://www.netacad.com/resources/lab-downloads`.
3. Choose the version for your computer (8.2 or newer) and download it.
4. Install it:
   - **Windows:** double-click the downloaded `.exe` → click **Next** on every screen →
     **Finish**.
   - **Linux (Ubuntu):** open a terminal in the download folder and type
     `sudo apt install ./CiscoPacketTracer*.deb`
5. Start Packet Tracer. The first time, it asks you to **log in** with your netacad account.
   Tick **Keep me logged in**.

✅ Packet Tracer is open and shows an empty white area.
❌ It asks for a login every time → you didn't tick "Keep me logged in". It's not a problem —
just log in.

---

## Mission 3 — Tour the window and turn on two settings
🧠 **Why:** every later step says "click …" with the name of a button. Know them now, and you
won't get stuck hunting for buttons when you need to concentrate on the network.

### 3a — Find every part of the window
Look at your screen and find each part below. Point at it with the mouse.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ ① Menu bar:  File  Edit  Options  View  Tools  Extensions  Window  Help  │
│ ② Main toolbar:  New Open Save Print ...  Zoom  ...                       │
│ ③ Common tools: Select  Inspect  Delete  Resize  Note  Draw  ✉  ✉+     │
│ ④ [Logical] [Physical]                                                   │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                ⑤ WORKSPACE  (where you build the network)                │
│                                                                          │
├──────────────────────────────────────────────────────────────────────────┤
│ ⑥ Time: 00:12:31  [Power Cycle Devices] [Fast Forward Time]              │
│ ⑦ Device-type box  │  ⑧ Device-model box             ⑨ [Realtime][Simulation] │
└──────────────────────────────────────────────────────────────────────────┘
```
(The exact position of some parts changes a little between versions — look for the names.)

| # | Part | What it is for |
|---|---|---|
| ① | **Menu bar** | *File → Save As* (save your work), *Options → Preferences* (settings) |
| ② | **Main toolbar** | Shortcuts: New, Open, Save, Zoom in/out |
| ③ | **Common tools** | **Select** (arrow — the normal mouse), **Inspect** (magnifying glass), **Delete** (X), **Place Note** (write a label), **Drawing Palette** (rectangles to mark buildings), **Add Simple PDU** (closed envelope — a quick ping) |
| ④ | **Logical / Physical** | *Logical* = the network diagram (you work here 99% of the time). *Physical* = cities, buildings, racks |
| ⑤ | **Workspace** | Drag devices here and connect them |
| ⑥ | **Time bar** | **Fast Forward Time** — makes simulated time pass quickly. **Power Cycle Devices** — reboots *all* devices (careful: unsaved configs are lost) |
| ⑦ | **Device-type box** (bottom left) | Categories: *Network Devices*, *End Devices*, *Connections* (cables)… |
| ⑧ | **Device-model box** | The models inside the chosen category — **drag** from here into the workspace |
| ⑨ | **Realtime / Simulation** | *Realtime* = normal. *Simulation* = slow motion |

✅ You can point at: the **Connections** lightning bolt, the **Fast Forward Time** button, and
the **Simulation** button.

### 3b — Turn on two settings (once, forever)
1. Top menu: **Options** → **Preferences**.
2. Click the **Interface** tab.
3. Tick **Always Show Port Labels in Logical Workspace**.
4. Make sure **Show Device Name Labels** is ticked.
5. Close the window with the **X**.

✅ Nothing visible changes yet — you will see the port labels in Mission 4.

### 3c — The device window
Every device opens a window when you click it. The tabs:

| Tab | On which devices | What you do there |
|---|---|---|
| **Physical** | all | See the real device, its power switch, add modules |
| **Config** | all | Change the **Display Name**; simple settings with buttons |
| **CLI** | routers, switches | The **command line** — where you type IOS commands |
| **Desktop** | PCs, laptops, servers | Small apps: **IP Configuration**, **Command Prompt**, **Web Browser**, **Email**… |
| **Services** | servers | Turn services on/off: HTTP, DHCP, DNS, EMAIL, FTP, NTP, SYSLOG, AAA |
| **Attributes** | all | Info only — ignore it |

Now open [docs/basic-moves.md](../../docs/basic-moves.md) in a **second browser tab** and read
the list of moves M1–M23. You don't need to remember them — just know they are there.

---

## Mission 4 — Build your first network: 2 PCs + 1 switch
🧠 **Why:** the smallest possible network shows the ideas the whole project is built on.

### 4a — Place the switch (how? → M2)
1. Bottom left, top row: click **Network Devices**. Bottom row: click **Switches**.
2. In the model box, hover until you find **2960** (`2960-24TT`).
3. **Drag** it into the middle of the workspace.

✅ A switch named `Switch0` is on the workspace.

### 4b — Place two PCs (how? → M2)
1. Bottom left: **End Devices** → **End Devices**.
2. Drag a **PC** to the **left** of the switch. Then drag another **PC** to the **right**.

✅ You see `PC0` on the left and `PC1` on the right.

### 4c — Connect the cables (how? → M5)
PC ↔ switch = **different** device types → **Copper Straight-Through**.
1. Bottom left: click **Connections** (⚡) → click **Copper Straight-Through** (solid black line).
2. Click **PC0** → in the list click **FastEthernet0**.
3. Click **Switch0** → in the list click **FastEthernet0/1**.
4. Again: **Copper Straight-Through** → click **PC1** → **FastEthernet0** → click **Switch0** →
   **FastEthernet0/2**.
5. The dots are orange at first. Click **Fast Forward Time** 2–3 times (how? → M18).

✅ Both cables have **green** dots at both ends, and the labels show `Fa0/1` and `Fa0/2` next
to the switch.
❌ A dot is red → you used the wrong cable type. Delete it (M7) and use Straight-Through.
❌ No port labels → do Mission 3b again.

---

## Mission 5 — Give the PCs addresses and ping

### 5a — Addresses (how? → M14)
1. Click **PC0** → **Desktop** tab → **IP Configuration**.
2. Select **Static** at the top.
3. **IPv4 Address:** type `192.168.1.1`
4. Click in **Subnet Mask** → PT fills in `255.255.255.0` by itself. Keep it.
5. Leave **Default Gateway** empty (there is no router).
6. Close the window (X at the top right). PT keeps the settings immediately.
7. Do the same on **PC1** with `192.168.1.2`.

✅ Hover the mouse over PC1 — the box shows `192.168.1.2`.

### 5b — Ping (how? → M17)
1. Click **PC0** → **Desktop** → **Command Prompt**.
2. Type `ping 192.168.1.2` and press **Enter**.

✅ You should see:
```
Pinging 192.168.1.2 with 32 bytes of data:

Request timed out.                          <- maybe one: ARP (explained below)
Reply from 192.168.1.2: bytes=32 time<1ms TTL=128
Reply from 192.168.1.2: bytes=32 time<1ms TTL=128
Reply from 192.168.1.2: bytes=32 time<1ms TTL=128
```
(The first line may or may not be `Request timed out` — both are fine.)

❌ `Request timed out` four times → check both addresses (5a) and that both cables are green.

📸 Take a screenshot of the replies for your hand-in (how? → M22).

🎉 Your first working network!

---

## Mission 6 — Your first CLI: look inside the switch
1. Click **Switch0** → **CLI** tab (how? → M9).
2. Click inside the black area and press **Enter**.

✅ You see `Switch>`.

3. Type `enable` and press **Enter**.

✅ The prompt is now `Switch#`.

4. Type `show mac address-table` and press **Enter**.

✅ You should see:
```
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----

   1    0001.4211.aa01    DYNAMIC     Fa0/1
   1    0060.2f3b.cc02    DYNAMIC     Fa0/2
```
Your MAC addresses will be different — that's normal. What matters: two lines, one with
`Fa0/1` and one with `Fa0/2`.

❌ The table is empty → the switch forgot (entries expire after 5 minutes). Ping again from PC0,
then run the command again.

📸 Screenshot this table for your hand-in.

### What just happened? Why did it work?
1. PC0 knew PC1's **IP** (you typed it) but not its **MAC**. The switch delivers by MAC, so
   PC0 first shouted to everyone: *"Who has 192.168.1.2? Tell me your MAC."*
   This shout is called an **ARP broadcast**.
2. The switch sent the shout out of every port. PC1 answered: *"That's me, my MAC is …"*
3. While this happened, the switch **wrote down** which MAC lives on which port — that's the
   **MAC address table** you just saw.
4. From now on the switch sends PC0↔PC1 traffic **only** to the right port.

That's why the **first** ping sometimes times out — PC0 was still waiting for the ARP answer.

---

## Mission 7 — Watch it in slow motion (Simulation mode)
🧠 **Why:** see with your own eyes the ARP shout and the ping you just read about.
(how? → M21)

1. Click **PC0** → **Desktop** → **Command Prompt** → type `arp -d` → **Enter**.
   (This makes PC0 forget PC1's MAC, so it has to ask again.) Close the window.
2. Bottom right: click **Simulation**. The *Simulation Panel* opens.
3. In the panel: click **Show All/None** (all protocols disappear from the list).
4. Click **Edit Filters** → tick **ICMP** and **ARP** (they may be on different tabs of the
   filter window, depending on the version) → close the filter window.
5. In the toolbar, click the **closed envelope** (*Add Simple PDU*) → click **PC0** → click **PC1**.
6. In the Simulation Panel press **Capture / Forward** (▶|) once. Look at the drawing.
7. Keep pressing ▶| one at a time and watch:
   - the **ARP** envelope goes from PC0 to the switch, then to **all** other ports (broadcast);
   - PC1 answers;
   - then the **ICMP** (ping) envelope goes only to PC1, and back.
8. Click one envelope on the drawing → a window shows its layers. Look for the MAC and IP
   addresses inside. Close it.
9. Click **Realtime** to go back to normal.

✅ You saw an ARP envelope go out to **every** port, and the ICMP envelope go **only** to PC1.
❌ No envelopes move → you didn't press ▶|, or the filters hide them (repeat 3–4).

---

## Mission 8 — Prove that a router is needed
1. Click **PC1** → **Desktop** → **IP Configuration** → change the IPv4 address to
   `192.168.2.2`. Close.
2. Click **PC0** → **Desktop** → **Command Prompt** → `ping 192.168.2.2`.

✅ The ping **fails** (`Request timed out` ×4). That is the expected result!

Same switch, same cable — but now they are on **different streets** (networks). Without a
router, they can't talk. We'll solve this properly in step 05.

3. Put PC1 back to `192.168.1.2` (so your saved file works).

---

## Mission 9 — Practise the CLI modes
🧠 **Why:** from now on, every router and switch is configured by typing commands. If you don't
know which "mode" you are in, correct commands will fail.

Think of a building with doors you open one after another:

| Prompt | Mode | Building comparison | What you can do |
|---|---|---|---|
| `Switch>` | **User EXEC** | The lobby | Look a little (ping, a few `show` commands). Can't change anything |
| `Switch#` | **Privileged EXEC** | The manager's office (needs a key = `enable`) | See **everything**, save, reboot |
| `Switch(config)#` | **Global configuration** | The control room | Change settings for the **whole device** |
| `Switch(config-if)#` | **Sub-mode** (interface, line, VLAN, router…) | One machine inside the control room | Change settings for **one part** |

👉 **Where:** Switch0 → CLI. Type each line and **watch the prompt** after every Enter:

| Type | The prompt becomes |
|---|---|
| (press Enter) | `Switch>` |
| `enable` | `Switch#` |
| `configure terminal` | `Switch(config)#` |
| `interface fa0/1` | `Switch(config-if)#` |
| `exit` | `Switch(config)#` (one level back) |
| `interface fa0/2` | `Switch(config-if)#` |
| `end` | `Switch#` (straight back) |
| `disable` | `Switch>` |

✅ Every prompt matched the table.

### Keys that save you time — try each one now
| Key / command | What it does | Try it |
|---|---|---|
| **?** | Shows what you can type here | In `Switch#` type `show ?` |
| **Tab** | Completes the word | Type `conf` then press **Tab** → `configure` |
| Short forms | IOS accepts any unique start | `conf t` = `configure terminal`, `sh run` = `show running-config` |
| **↑ / ↓** | Previous / next command | Press ↑ a few times |
| **Ctrl+Shift+6** | Stops a long ping or a stuck lookup | — |
| `do show ...` | Run a `show` command while in config mode | In `(config)#` type `do show vlan brief` |
| `no <command>` | Removes a setting | `no shutdown` |

### Running vs startup config
| | Running-config | Startup-config |
|---|---|---|
| Where | RAM (memory) | NVRAM (permanent storage) |
| What | What the device is doing **right now** | What it loads when it **boots** |
| Comparison | A whiteboard | A notebook |

Every command changes the **running-config** immediately. If the device restarts, the
whiteboard is wiped — unless you copied it into the notebook (how? → M13):
`copy running-config startup-config` (short: `copy run start` or `wr`).
**Save every device at the end of every step.**

---

## Mission 10 — Save your work (how? → M1)
🧠 **Why:** every step continues from the file of the step before. A file you didn't save =
building everything again.

1. **File → Save As**.
2. Go to the course folder (Mission 1) → open **my-work** → name it `step00.pkt` → **Save**.
3. (Optional, if you use git) in a terminal in the course folder:
   `git add my-work/step00.pkt` → `git commit -m "Step 00 done"` → `git push`.

✅ The window title shows `step00.pkt`.

> Saving the `.pkt` file saves the whole drawing. Saving inside each device (`wr`) makes the
> device keep its config after a reboot. **Do both.**

---

## Checkpoint
- [ ] Packet Tracer runs, port labels are switched on
- [ ] You did the 2-PC ping and saw the MAC table
- [ ] You watched ARP in Simulation mode
- [ ] You did the "different network" test
- [ ] You can explain `Switch>` vs `Switch#`, and MAC vs IP, in one sentence each
- [ ] You have prepared what you hand in (see "What you hand in" at the top)

## If something doesn't work
| Problem | Fix |
|---|---|
| Cable dots red | Wrong cable type (switch ↔ switch needs **Cross-Over**), or the device is off (Physical tab → power button) |
| Dots stay orange | Press **Fast Forward Time** a few times |
| Ping says `Request timed out` every time | Check both IPs and masks; check the cable goes to the right PC |
| CLI shows nothing | Click inside the black area and press **Enter** |
| `% Invalid input detected at '^' marker` | The `^` points to the mistake. Press **↑** to get the line back and fix it (M12) |
| A typo in `Switch#` waits 30 s ("Translating…") | It thinks the typo is a hostname. Press **Ctrl+Shift+6**. (We switch this off in step 02) |
| No command works and the letters look strange | The keyboard is set to Hebrew/Arabic. Switch to English (Alt+Shift) |

---

## 🏋 Exercises

### Exercise A — Give the switch a name ⭐
- 🎯 **Task:** change the switch's name from `Switch` to `MY-FIRST-SWITCH`, and save it so it
  survives a restart.
- 📁 **Start from:** your `step00.pkt`.
- 👣 **Steps:**
  1. Open the CLI of Switch0 and reach the `Switch#` prompt.
  2. Run `show running-config` and find the line that starts with `hostname`.
  3. Go into configuration mode and change the hostname.
  4. Save the configuration.
- ✅ **Done when:** the prompt shows `MY-FIRST-SWITCH#`, and `show startup-config` also contains
  `hostname MY-FIRST-SWITCH`.

<details><summary>💡 Hint 1</summary>

The command that sets the name is called `hostname`. It is typed in `(config)#` mode.
</details>

<details><summary>💡 Hint 2</summary>

`configure terminal` → `hostname ...` → `end` → save with M13.
</details>

<details><summary>✔ Answer</summary>

```
enable
configure terminal
hostname MY-FIRST-SWITCH
end
copy running-config startup-config
```
The prompt changes to `MY-FIRST-SWITCH#` immediately — proof that the running-config
changed the moment you pressed Enter. When `copy` asks `Destination filename [startup-config]?`
just press **Enter**.
</details>

### Exercise B — A third PC ⭐
- 🎯 **Task:** add `PC2` with address `192.168.1.3` on port **Fa0/3**, and prove it can reach PC0.
- 📁 **Start from:** your `step00.pkt`.
- 👣 **Steps:**
  1. Place a PC (M2).
  2. Connect it to Switch0 Fa0/3 with the right cable (M5).
  3. Give it the address (M14).
  4. Ping PC0 from PC2 (M17).
  5. Show the MAC table on the switch.
- ✅ **Done when:** the ping replies, and `show mac address-table` shows **three** lines
  (Fa0/1, Fa0/2, Fa0/3).

<details><summary>💡 Hint 1</summary>

PC ↔ switch = Copper **Straight-Through**. The mask is the same as the other PCs.
</details>

<details><summary>✔ Answer</summary>

Copper Straight-Through: PC2 FastEthernet0 → Switch0 FastEthernet0/3. IP `192.168.1.3`, mask
`255.255.255.0`, no gateway. From PC2: `ping 192.168.1.1` → replies. On the switch: three MAC
lines. (If you see only two, ping from PC1 too — a line appears only after the PC sent something.)
</details>

---

## 🛠 Your own project

In this step you don't build your project yet — you prepare its foundation.

- [ ] **1.** Open the **project proposal** you submitted. Your company, its 3 sites and its
  departments are already written there.
- [ ] **2.** Check that the proposal follows the rules, and tick each one:
  - [ ] 3 sites, one of them defined as the **main branch** (requirement 19)
  - [ ] **6 different departments** in every site, with at least **8 PCs** in each
    (requirement 20.2)
  - [ ] No site is a copy of another — each site has its own design model and features (rule ג)
  - [ ] One site is **collapsed core** (requirement 5) and one site is entirely **IPv6**
    (requirement 7)
- [ ] **3.** If something is missing or unclear — write it down and ask your teacher next lesson.
- [ ] **4.** Open [docs/project-plan-template.md](../../docs/project-plan-template.md) and read it
  once. In step 01 you will fill it in from your proposal.
- [ ] **5.** Copy [docs/progress-template.md](../../docs/progress-template.md) to
  `my-project/progress.md`: open it, select all (**Ctrl+A**), copy (**Ctrl+C**), create a new
  file `progress.md` in `my-project`, paste (**Ctrl+V**), save. Tick step 00 when you finish
  it, and show it to your teacher at the end of every step.
- [ ] **6.** Read [docs/exam-guide.md](../../docs/exam-guide.md) once, so you know from today
  what the examiner will ask.

> **Important:** don't save files containing an ID number (such as the proposal form) in the
> course folder. This folder may be shared.

---

## 🎓 Explain it to the examiner

At the exam, the examiner points at **your** project and asks you to **explain** and to **show**.
Practise every question **out loud**, then show the answer in your own project — not in the
example. Cover the answer, say yours, then open it and compare.

★1–★3 = a question from the ministry question bank (difficulty 1–3).
◆ = not in the bank, but examiners often ask it.

**1. ◆ What is the difference between a MAC address and an IP address?**

<details><summary>Answer</summary>

A MAC address is burned into the network card and never changes; switches use it to deliver frames **inside** one network. An IP address is given by the admin or DHCP and changes when the device moves; routers use it to deliver packets **between** networks.

**Show it:** PC → Desktop → Command Prompt → `ipconfig /all` (shows both). On the switch: `show mac address-table`.
</details>

**2. ★1 What is a default gateway?**

<details><summary>Answer</summary>

The address of the router on the PC's own network. When the destination is on another network, the PC sends the packet to the gateway, and the router forwards it. Without a gateway the PC can only reach its own network.

**Show it:** PC → Desktop → IP Configuration (the Default Gateway field).
</details>

**3. ◆ How does a switch learn where each device is? What does it do with a frame for an unknown MAC?**

<details><summary>Answer</summary>

It reads the **source** MAC of every frame and writes "this MAC is on this port" in its MAC address table. A frame for a MAC that isn't in the table is **flooded** out of every port except the one it came in on.

**Show it:** `show mac address-table`
</details>

**4. ◆ What is ARP and why did the first ping sometimes fail?**

<details><summary>Answer</summary>

ARP finds the MAC address that belongs to an IP address: the PC broadcasts "who has 192.168.1.2?" and the owner replies with its MAC. The first ping can time out while the PC is still waiting for that reply.

**Show it:** Simulation mode with the ARP and ICMP filters; PC → `arp -a`.
</details>

**5. ◆ What is the difference between running-config and startup-config?**

<details><summary>Answer</summary>

running-config is in RAM — what the device is doing now; every command changes it immediately. startup-config is in NVRAM — what the device loads when it boots. `copy running-config startup-config` saves the first into the second.

**Show it:** `show running-config`, `show startup-config`
</details>

**6. ◆ What are the CLI modes and how do you move between them?**

<details><summary>Answer</summary>

User EXEC `>` (look only) → `enable` → privileged EXEC `#` (see everything, save) → `configure terminal` → global config `(config)#` → e.g. `interface fa0/1` → sub-mode `(config-if)#`. `exit` goes one level down, `end` goes straight back to `#`.

**Show it:** Type them on any switch and watch the prompt change.
</details>

---

## 🔁 Do it again — without the guide

The examiner may ask you to configure something again, in front of them. You can only do that
if you have done it **without the guide** before. Do this drill in your own project:

1. **File → New**. Without opening this page, build PC0 – Switch0 – PC1 with the correct cable
   and ports, give the PCs `192.168.1.1` and `192.168.1.2`, ping, and show the MAC table.
2. Target: **5 minutes**. Say out loud what you are doing and why at every click.
3. Then change PC1 to another network, show that the ping fails, and explain why.

---

## 👩‍🏫 Teacher notes
- **Lesson 1 (90 min):** Missions 1–3 — download the course files, explain the learn /
  build-your-own workflow and the "how to read a step" table (15 min) → device table, MAC vs IP
  (ID number vs home address), mask and gateway on the board (30 min) → install + tour of the PT
  window, preferences (20 min) → Missions 4–6: 2-PC lab and MAC table (25 min).
- **Lesson 2 (90 min):** Mission 7 Simulation mode ARP demo on the projector (20 min) → students
  repeat it (20 min) → Mission 8 "different network" test and discussion (15 min) → Mission 9 CLI
  modes, keys, running vs startup (25 min) → save the file (10 min).
- **Teach the mission rhythm from day one:** do → compare with ✅ → only then continue. Walk
  around and ask "show me your ✅" before students move on. Weak students skip the check, not the
  commands.
- **Common student questions**
  - *"Why do we need MAC **and** IP?"* — MAC delivers inside one network (one hop); IP gets
    you across many networks. A letter needs the city address to cross the country and the
    room number to reach the right desk in the building.
  - *"Why does my first ping fail?"* — ARP. Ping again.
  - *"I typed a command and the device forgot it."* — they didn't save (`wr`).
  - *"Why not the automatic cable?"* — it picks ports itself; the project needs exact ports.
  - *"Nothing is accepted."* — almost always the keyboard is on Hebrew or Arabic.
- **Check understanding:** each student explains to a neighbour why the 192.168.1.x ↔
  192.168.2.x ping fails.
