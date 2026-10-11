# Step 00 — Getting started: networking basics + a tour of Packet Tracer

> ⏱ about 2 lessons (2 × 90 min) · 📁 start from: nothing · 💾 save as: `my-work/step00.pkt`

## What you will have at the end
- Packet Tracer installed and you know **every button** you will use in this project.
- You understand the basic words: switch, router, IP, MAC, mask, gateway, ping.
- You built your first tiny network and watched it work:

```
   PC0 ──────── [ Switch0 ] ──────── PC1
192.168.1.1                      192.168.1.2
```

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

---

## Part 2 — Install Packet Tracer

1. Go to **netacad.com** and create a free account (or log in).
2. Click **Resources → Download Packet Tracer**. Choose the version for your computer.
3. Install it:
   - **Windows:** double-click the `.exe`, click *Next* until it finishes.
   - **Linux (Ubuntu):** open a terminal in the download folder and type
     `sudo apt install ./CiscoPacketTracer_*_Ubuntu_64bit.deb`
4. Start Packet Tracer. The first time, it asks you to **log in** with your netacad account.
   Tick "Keep me logged in".

---

## Part 3 — A tour of the Packet Tracer window

Open Packet Tracer. You see an empty white area. Here is every part you will use.

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
| ③ | **Common tools** | **Select** (arrow — the normal mouse), **Inspect** (magnifying glass — look at tables inside a device), **Delete** (X — delete a device or cable), **Note** (write a text label on the workspace), **Draw** (rectangles/ellipses to mark buildings), **Add Simple PDU** (closed envelope — a quick ping between two devices) |
| ④ | **Logical / Physical** | *Logical* = the network diagram (you work here 99% of the time). *Physical* = cities, buildings, racks |
| ⑤ | **Workspace** | Drag devices here and connect them |
| ⑥ | **Time bar** | **Fast Forward Time** — makes simulated time pass quickly (links come up, protocols finish). Press it whenever you are waiting for something to turn green. **Power Cycle Devices** — reboots *all* devices (careful: unsaved configs are lost) |
| ⑦ | **Device-type box** (bottom left) | Categories: *Network Devices* (routers, switches, wireless…), *End Devices* (PCs, servers, printers…), *Connections* (cables), *Components*, *Miscellaneous*… |
| ⑧ | **Device-model box** | The models inside the chosen category (e.g. 2960, 3650-24PS) — **drag** from here into the workspace |
| ⑨ | **Realtime / Simulation** | *Realtime* = normal mode. *Simulation* = slow motion: you see every packet as an envelope and can open it |

### 3.1 Two settings to turn on once (they make life much easier)
1. **Options → Preferences** → **Interface** tab.
2. Tick **"Always Show Port Labels in Logical Workspace"** — every cable now shows the port
   names at both ends (e.g. `Fa0/1`, `Gi0/1`). Essential for checking your cabling.
3. Make sure **"Show Device Name Labels"** is ticked.
4. Close the window.

### 3.2 The device window — what opens when you click a device
Click any device once (with the **Select** arrow) → a window opens with tabs:

| Tab | On which devices | What you do there |
|---|---|---|
| **Physical** | all | See the real device, its power switch, add modules (cards, power supplies) |
| **Config** | all | Change the **Display Name** (the label on the workspace); simple settings with buttons |
| **CLI** | routers, switches | The **command line** — where you type IOS commands. This is where you'll spend most of the time |
| **Desktop** | PCs, laptops | Small apps: **IP Configuration**, **Command Prompt**, **Web Browser**, **Email**, **PC Wireless**, **VPN** |
| **Services** | servers | Turn services on/off: HTTP, DHCP, DNS, EMAIL, FTP, NTP, SYSLOG, AAA |
| **Attributes** | all | Info only — you can ignore it |

### 3.3 Cables — which one to choose
Click **Connections** (the lightning bolt ⚡ in the device-type box). The cable types:

| Icon / name | Use it for |
|---|---|
| ⚡ **Automatic** | PT chooses the cable. Fast, but it chooses the **port** itself too — so for this project **don't** use it; we must use exact ports |
| **Copper Straight-Through** (solid black line) | **Different** kinds of devices: PC ↔ switch, switch ↔ router, server ↔ switch, printer ↔ switch |
| **Copper Cross-Over** (dashed black line) | **Same** kind of devices: switch ↔ switch, router ↔ router, PC ↔ PC |
| **Fiber** (orange) | Fiber ports (we use it only if PT says a port is fiber) |
| **Console** (light blue) | PC RS-232 ↔ device console port (not needed — we use the CLI tab) |
| **Phone** | DSL modem ↔ cloud (step 12) |

How to connect a cable, every time:
1. Click the cable type (e.g. **Copper Straight-Through**).
2. Click the **first device** → a small list of its ports appears → click the **port** you need.
3. Click the **second device** → choose its port.
4. Done. The dots at each end show the link state:

| Link light | Meaning |
|---|---|
| 🟢 green | Link up and forwarding |
| 🟠 orange | Link up but not forwarding yet (switch is still checking for loops — wait ~30 s or press **Fast Forward Time**). If it stays orange forever, Spanning Tree is blocking it (normal for backup links) |
| 🔴 red | Link down — wrong cable type, port shut down, or device off |

To delete a cable: choose the **Delete** tool (X) and click the cable, or select it and press **Delete** on the keyboard.

### 3.4 Useful mouse & keyboard tricks
| Trick | How |
|---|---|
| Place many devices of the same type | Hold **Ctrl** while clicking the model, then click in the workspace several times. Press **Esc** to stop |
| Copy devices (with their settings) | Select them with a box (drag a rectangle with the mouse), **Ctrl+C**, **Ctrl+V** |
| Move the workspace | Drag on an empty area, or use the scroll bars |
| Zoom | Ctrl + mouse wheel, or the zoom buttons in the toolbar |
| Rename a device | Click it → **Config** tab → **Display Name** (or double-click the name label under the icon) |

---

## Part 4 — Your first network (click by click)

### 4.1 Place the devices
1. In the device-type box (bottom left), click **Network Devices** → **Switches**.
2. In the model box, find **2960** (its label says `2960-24TT`). **Drag** it into the middle of
   the workspace. It's called `Switch0`.
3. Click **End Devices** → **End Devices** (the first icon). Drag a **PC** to the left of the
   switch, then another **PC** to the right. They are called `PC0` and `PC1`.

### 4.2 Connect the cables
1. Click **Connections** (⚡) → choose **Copper Straight-Through** (PC ↔ switch = different devices).
2. Click **PC0** → choose **FastEthernet0**.
3. Click **Switch0** → choose **FastEthernet0/1**.
4. Again: **Copper Straight-Through** → **PC1** → **FastEthernet0** → **Switch0** → **FastEthernet0/2**.
5. The lights are orange at first. Press **Fast Forward Time** two or three times → green.

### 4.3 Give the PCs IP addresses
1. Click **PC0** → **Desktop** tab → **IP Configuration**.
2. Make sure **Static** is selected.
3. **IPv4 Address:** `192.168.1.1`
4. Click in **Subnet Mask** → PT fills in `255.255.255.0` automatically. Keep it.
5. Leave **Default Gateway** empty (there is no router).
6. Close the window (X at the top right). PT saves the settings immediately.
7. Do the same on **PC1** with `192.168.1.2`.

### 4.4 Test with ping
1. Click **PC0** → **Desktop** → **Command Prompt**.
2. Type: `ping 192.168.1.2` and press Enter.
3. You should see:
```
Pinging 192.168.1.2 with 32 bytes of data:

Request timed out.                          <- maybe one: ARP (explained below)
Reply from 192.168.1.2: bytes=32 time<1ms TTL=128
Reply from 192.168.1.2: bytes=32 time<1ms TTL=128
Reply from 192.168.1.2: bytes=32 time<1ms TTL=128
```
🎉 Your first working network!

### 4.5 Look inside the switch — your first CLI
1. Click **Switch0** → **CLI** tab.
2. Click in the black area and press **Enter**. You see `Switch>`.
3. Type `enable` and press Enter → the prompt becomes `Switch#`.
4. Type `show mac address-table` and press Enter:
```
          Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0001.4211.aa01    DYNAMIC     Fa0/1
   1    0060.2f3b.cc02    DYNAMIC     Fa0/2
```
The switch **learned** which PC sits on which port.

### 4.6 What just happened? (explain this to students)
1. PC0 knew PC1's **IP** (you typed it) but not its **MAC**. The switch delivers by MAC, so
   PC0 first shouted to everyone: *"Who has 192.168.1.2? Tell me your MAC."*
   This shout is called an **ARP broadcast**.
2. The switch sent the shout out of every port. PC1 answered: *"That's me, my MAC is …"*
3. While this happened, the switch **wrote down** which MAC lives on which port — that's the
   **MAC address table** you just saw.
4. From now on the switch sends PC0↔PC1 traffic **only** to the right port.

That's why the **first** ping sometimes times out — PC0 was still waiting for the ARP answer.

### 4.7 Watch it in slow motion (Simulation mode)
1. PC0 → Command Prompt → type `arp -d` (clears PC0's memory of MACs). Close the window.
2. Bottom right: click **Simulation**. A *Simulation Panel* opens.
3. In the panel, click **Edit Filters** (or **Show All/None**) → leave only **ARP** and **ICMP** ticked.
4. Click the closed **envelope** tool (*Add Simple PDU*) in the toolbar → click **PC0** → click **PC1**.
5. Press **Capture / Forward** (▶|) step by step. Watch:
   - the ARP envelope goes from PC0 to the switch, then to **all** other ports (broadcast);
   - PC1 answers;
   - then the ICMP (ping) envelope goes only to PC1.
6. Click an envelope on the workspace → you see its layers (OSI model). Look at the MAC and
   IP addresses inside.
7. Click **Realtime** to go back to normal.

### 4.8 See that a router is needed
1. PC1 → Desktop → IP Configuration → change the address to `192.168.2.2`. Close.
2. PC0 → Command Prompt → `ping 192.168.2.2` → **fails**.
3. Same switch, same cable — but now they are on **different streets** (networks).
   Without a router, they can't talk. We'll solve this properly in step 05.

---

## Part 5 — The CLI: modes and the basic keys

Think of a building with doors you open one after another:

| Prompt | Mode | Building comparison | What you can do |
|---|---|---|---|
| `Switch>` | **User EXEC** | The lobby | Look a little (ping, a few `show` commands). Can't change anything |
| `Switch#` | **Privileged EXEC** | The manager's office (needs a key = `enable`) | See **everything**, save, reboot |
| `Switch(config)#` | **Global configuration** | The control room | Change settings for the **whole device** |
| `Switch(config-if)#` | **Sub-mode** (interface, line, VLAN, router…) | One machine inside the control room | Change settings for **one part** (one port, one line…) |

```
Switch>                      user EXEC
Switch> enable               -> go up
Switch#                      privileged EXEC
Switch# configure terminal   -> go up (short: conf t)
Switch(config)#              global config
Switch(config)# interface fa0/1
Switch(config-if)#           sub-mode for port Fa0/1
Switch(config-if)# exit      -> one level down
Switch(config-if)# end       -> straight back to Switch#
```

### Keys that save you time
| Key / command | What it does |
|---|---|
| **?** | Shows what you can type here. `show ?` lists all show commands. |
| **Tab** | Completes the word: `conf` + Tab → `configure` |
| Short forms | IOS accepts any unique start: `conf t` = `configure terminal`, `int fa0/1` = `interface FastEthernet0/1`, `sh run` = `show running-config` |
| **↑ / ↓** | Previous / next command you typed |
| **Ctrl+Shift+6** | Stops a long ping or a stuck lookup |
| `do show ...` | Run a `show` command while you are in config mode |
| `no <command>` | Removes a setting (e.g. `no shutdown`) |

### Running vs startup config
| | Running-config | Startup-config |
|---|---|---|
| Where | RAM (memory) | NVRAM (permanent storage) |
| What | What the device is doing **right now** | What it loads when it **boots** |
| Comparison | A whiteboard | A notebook |

Every command changes the **running-config** immediately. If the device restarts, the
whiteboard is wiped — unless you copied it into the notebook:
`copy running-config startup-config` (short: `copy run start` or `wr`).
**Save every device at the end of every step.**

---

## Part 6 — Save your work

1. **File → Save As**.
2. Go to the repo folder → `my-work` → name it `step00.pkt` → **Save**.
3. (Optional, if you use git) in a terminal in the repo folder:
   `git add my-work/step00.pkt` → `git commit -m "Step 00 done"` → `git push`.

> Saving the `.pkt` file saves the whole drawing. Saving inside each device (`wr`) makes the
> device keep its config after a reboot. **Do both.**

---

## Checkpoint
- [ ] Packet Tracer runs, port labels are switched on
- [ ] You did the 2-PC ping and saw the MAC table
- [ ] You watched ARP in Simulation mode
- [ ] You did the "different network" test
- [ ] You can explain `Switch>` vs `Switch#`, and MAC vs IP, in one sentence each

## If something doesn't work
| Problem | Fix |
|---|---|
| Cable lights red | Wrong cable type (switch ↔ switch needs **Cross-Over**), or the device is off (Physical tab → power button) |
| Lights stay orange | Press **Fast Forward Time** a few times |
| Ping says `Request timed out` every time | Check both IPs and masks; check the cable goes to the right PC |
| CLI shows nothing | Click inside the black area and press **Enter** |
| Typed a wrong command → `% Invalid input detected at '^' marker` | The `^` points to the mistake. Use **↑** to get the line back and fix it |
| Typed a typo in `Switch#` and it waits for 30 s ("Translating…") | It thinks the typo is a hostname. Press **Ctrl+Shift+6**. (We switch this off in step 02) |

## Challenge
On the switch, run `show running-config`. Find the line `hostname Switch`. Change the
hostname to `MY-FIRST-SWITCH` and save it.

<details><summary>Answer</summary>

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

---

## 👩‍🏫 Teacher notes
- **Lesson 1 (90 min):** device table, MAC vs IP (ID number vs home address), mask and
  gateway on the board (30 min) → install + tour of the PT window, preferences (30 min) →
  2-PC lab and MAC table (30 min).
- **Lesson 2 (90 min):** Simulation mode ARP demo on the projector (20 min) → students repeat
  it (20 min) → "different network" test and discussion (15 min) → CLI modes, keys, running vs
  startup (25 min) → save the file (10 min).
- **Common student questions**
  - *"Why do we need MAC **and** IP?"* — MAC delivers inside one network (one hop); IP gets
    you across many networks. A letter needs the city address to cross the country and the
    room number to reach the right desk in the building.
  - *"Why does my first ping fail?"* — ARP. Ping again.
  - *"I typed a command and the device forgot it."* — they didn't save (`wr`).
  - *"Why not the automatic cable?"* — it picks ports randomly; the project needs exact ports.
- **Check understanding:** each student explains to a neighbour why the 192.168.1.x ↔
  192.168.2.x ping fails.
