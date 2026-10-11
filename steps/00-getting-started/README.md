# Step 00 — Getting started

## Goal
Packet Tracer installed, you understand the **basic words** of networking (switch, router,
IP, MAC, mask, gateway), you know the 4 CLI modes, and you have a place to save your work.

---

## Understand it first — what is a network? (simple English)

A **network** is a group of devices connected so they can send data to each other.
The whole project is about connecting devices *correctly*, *redundantly* (if one thing
breaks, another takes over) and *securely*.

### The devices you will use

| Device | What it does | Everyday comparison |
|---|---|---|
| **PC / laptop** | Creates and uses data (browsing, email). Called an **end device**. | A person sending and receiving letters |
| **Switch** | Connects devices **in the same place** (same office / building). Knows each device by its **MAC address**. | The mail room inside one office building — delivers letters between rooms |
| **Router** | Connects **different networks** together (building ↔ building, company ↔ internet). Works with **IP addresses**. | The post office between cities — decides which road a letter takes |
| **Layer-3 switch** | A switch that can *also* route. Fast and has many ports. We use it a lot. | A mail room that can also send mail to other cities |
| **Server** | A computer that gives a **service** to others: web pages, email, DNS, DHCP… | A shop or a government office: you go there to get something |
| **Firewall** | Checks traffic and blocks what is not allowed. | The security guard at the front door |
| **Access point** | Connects wireless devices to the wired network. | A wireless "socket" |

### Two kinds of addresses

| | **MAC address** | **IP address** |
|---|---|---|
| Example | `00E0.F7A1.3B2C` | `10.10.16.25` |
| Who sets it | Burned into the network card at the factory | The admin, or DHCP (automatic) |
| Changes? | Never | Yes — when the device moves to another network |
| Used by | **Switches** — to deliver inside the local network | **Routers** — to deliver between networks |
| Comparison | Your **ID number** (ת"ז) — always the same | Your **home address** — changes when you move |

### Subnet mask — "which part is the street, which part is the house"
An IPv4 address is 4 numbers (0–255). The **subnet mask** says how many of them describe
the **network** (the street) and how many describe the **host** (the house on that street).

```
IP:    192.168.1.25
Mask:  255.255.255.0      -> the first 3 numbers are the "street"
         street = 192.168.1      house = 25
```
- `192.168.1.25` and `192.168.1.40` → **same street** → they talk directly through the switch.
- `192.168.1.25` and `192.168.2.40` → **different streets** → they need a **router**.

### Default gateway — "where do I send mail for other streets?"
The **default gateway** is the router's address on *your* street. When a PC wants to reach
an address on another network, it hands the packet to the gateway, and the router takes it
from there. No gateway → the PC can only talk to its own street.

### Ping — "are you there?"
`ping 10.10.16.25` sends a small "are you there?" message (ICMP echo). If a reply comes back,
the path works **in both directions**. It's the #1 test you will use.

### Packet Tracer, IOS and the CLI
- **Packet Tracer** — Cisco's free simulator. Devices behave like real Cisco equipment, so you
  can build a 150-device company network on a laptop. It's a simulator, so a few real
  commands are missing (see [pt-gotchas](../../docs/pt-gotchas.md)).
- **IOS** — the operating system inside Cisco routers and switches.
- **CLI** (command-line interface) — you configure IOS by **typing commands**. Real network
  engineers work this way, the exams test it, and the project is graded on it.

---

## 1. Install Packet Tracer
1. Log in at **netacad.com** (free account) → *Resources → Download Packet Tracer*.
2. Linux: `sudo apt install ./CiscoPacketTracer_*_Ubuntu_64bit.deb`
3. Start it with `packettracer`, log in with the same netacad account.

## 2. The 4 IOS modes — learn these by heart

Think of a building with doors you open one after another:

| Prompt | Mode | Building comparison | What you can do |
|---|---|---|---|
| `Switch>` | **User EXEC** | The lobby | Look a little (ping, a few `show` commands). Can't change anything. |
| `Switch#` | **Privileged EXEC** | The manager's office (needs a key = `enable` password) | See **everything**, save, reboot |
| `Switch(config)#` | **Global configuration** | The control room | Change settings for the **whole device** |
| `Switch(config-if)#` | **Sub-mode** (interface, line, VLAN, router…) | One machine inside the control room | Change settings for **one part** (one port, one line…) |

```
Switch>                      user EXEC
Switch> enable
Switch#                      privileged EXEC
Switch# configure terminal
Switch(config)#              global config
Switch(config)# interface fa0/1
Switch(config-if)#           sub-mode for port Fa0/1
Switch(config-if)# exit      -> one level up
Switch(config-if)# end       -> straight back to Switch#
```

### Two configurations: running vs startup
| | Running-config | Startup-config |
|---|---|---|
| Where | RAM (memory) | NVRAM (permanent storage) |
| What | What the device is doing **right now** | What it loads when it **boots** |
| Comparison | A whiteboard | A notebook |

Every command you type changes the **running-config** immediately. If the device restarts,
the whiteboard is wiped — unless you copied it to the notebook:
`copy running-config startup-config` (short: `copy run start` or `wr`). **Save after every step.**

## 3. Your first 5 minutes in Packet Tracer
1. Bottom-left: **Network Devices → Switches → 2960**. Drag one into the workspace.
2. **End Devices → PC**. Drag two PCs.
3. **Connections → lightning bolt** (automatic cable). Click PC0 → switch, PC1 → switch.
4. Wait for the link lights to turn green (or press **Fast Forward Time**).
5. PC0 → *Desktop → IP Configuration*: `192.168.1.1 / 255.255.255.0`. PC1: `192.168.1.2`.
6. PC0 → *Desktop → Command Prompt* → `ping 192.168.1.2`. 🎉

Now click the switch → **CLI** tab, type `enable`, then `show mac address-table`.

### What just happened? (explain this to students)
1. PC0 knew PC1's **IP** (you typed it) but not its **MAC**. The switch delivers by MAC, so
   PC0 first shouted to everyone: *"Who has 192.168.1.2? Tell me your MAC."*
   This shout is called an **ARP broadcast**.
2. The switch sent the shout out of every port. PC1 answered: *"That's me, my MAC is …"*
3. While this happened, the switch **wrote down** which MAC lives on which port — that's the
   **MAC address table** you just saw.
4. From now on the switch sends PC0↔PC1 traffic **only** to the right port, not to everyone.

That's also why the **first** ping sometimes shows a timeout — it's waiting for ARP.

### Try this — see a router is needed
Change PC1 to `192.168.2.2`. Ping again → **fails**. Same switch, same cable — but now they are
on **different streets** (networks). Without a router, they can't talk. You'll fix this kind
of problem properly in step 05.

## 4. How to use this repo
- Read the step's **Concepts** before touching Packet Tracer.
- **Type** the commands. Every line has a comment (`!` lines) telling you *why*.
- Always run the **Verify** commands and compare with the expected output.
- Save after every step: `File → Save As → my-work/stepNN.pkt`, then commit it.
- Stuck? → [docs/pt-gotchas.md](../../docs/pt-gotchas.md), [docs/cheatsheet.md](../../docs/cheatsheet.md),
  and every new word is in [docs/glossary.md](../../docs/glossary.md).

## Checkpoint
- [ ] Packet Tracer runs and you are logged in
- [ ] You did the 2-PC ping above, and the "different network" test
- [ ] You can explain the difference between `Switch>` and `Switch#`
- [ ] You can explain MAC vs IP in one sentence each

## Challenge
On the same switch, run `show running-config`. Find: the hostname, and the interface
your PC0 is plugged into. Then change the hostname to `MY-FIRST-SWITCH` and save it.

<details><summary>Answer</summary>

```
enable
configure terminal
hostname MY-FIRST-SWITCH
end
copy running-config startup-config
```
The prompt changes to `MY-FIRST-SWITCH#` immediately — proof that the running-config
changed the moment you pressed Enter.
</details>

---

## 👩‍🏫 Teacher notes
- **Lesson plan (~90 min):** 20 min the device table + MAC vs IP (use the ID-number vs
  home-address comparison), 15 min mask & gateway on the board, 30 min the 2-PC lab,
  15 min the "different network" test, 10 min modes + saving.
- **Common student questions**
  - *"Why do we need MAC **and** IP?"* — MAC delivers inside one network (one hop); IP gets
    you across many networks. A letter needs the street address to cross the country and
    the room number to reach the right desk in the building.
  - *"Why does my first ping fail?"* — ARP. Ping again.
  - *"I typed a command and the device forgot it."* — they didn't save (`wr`).
- **Check understanding:** ask each student to explain to a neighbour why the
  192.168.1.x ↔ 192.168.2.x ping fails.
