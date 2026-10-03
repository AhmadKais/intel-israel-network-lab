# Step 00 — Getting started

## Goal
Packet Tracer installed, you know the 4 CLI modes, and you have a place to save your work.

## 1. Install Packet Tracer
1. Log in at **netacad.com** (free account) → *Resources → Download Packet Tracer*.
2. Linux: `sudo apt install ./CiscoPacketTracer_*_Ubuntu_64bit.deb`
3. Start it with `packettracer`, log in with the same netacad account.

## 2. The 4 IOS modes — learn these by heart

```
Switch>                      user EXEC        – look only (ping, show a few things)
Switch> enable
Switch#                      privileged EXEC  – show everything, save, reload
Switch# configure terminal
Switch(config)#              global config    – settings for the whole device
Switch(config)# interface fa0/1
Switch(config-if)#           sub-mode         – settings for one interface/line/vlan/router
Switch(config-if)# exit      → one level up
Switch(config-if)# end       → straight back to Switch#
```

## 3. Your first 5 minutes in Packet Tracer
1. Bottom-left: **Network Devices → Switches → 2960**. Drag one into the workspace.
2. **End Devices → PC**. Drag two PCs.
3. **Connections → lightning bolt** (automatic cable). Click PC0 → switch, PC1 → switch.
4. Wait for the link lights to turn green (or press **Fast Forward Time**).
5. PC0 → *Desktop → IP Configuration*: `192.168.1.1 / 255.255.255.0`. PC1: `192.168.1.2`.
6. PC0 → *Desktop → Command Prompt* → `ping 192.168.1.2`. 🎉

Now click the switch → **CLI** tab and type `enable`, then `show mac address-table`. You see
the two PCs' MAC addresses — the switch *learned* them from the ping. That is how a switch
works.

## 4. How to use this repo
- Read the step's **Concepts** before touching Packet Tracer.
- **Type** the commands. Every line has a comment (`!` lines) telling you *why*.
- Always run the **Verify** commands and compare with the expected output.
- Save after every step: `File → Save As → my-work/stepNN.pkt`, then commit it.
- Stuck? → [docs/pt-gotchas.md](../../docs/pt-gotchas.md), then [docs/cheatsheet.md](../../docs/cheatsheet.md).

## Checkpoint
- [ ] Packet Tracer runs and you are logged in
- [ ] You did the 2-PC ping above
- [ ] You can explain the difference between `Switch>` and `Switch#`

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
</details>
