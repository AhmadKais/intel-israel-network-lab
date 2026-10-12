# Preparing for the project exam

At the end of the year you defend your project in front of an examiner. Building the network is
not enough: you must be able to **explain** every part of it, **show** where it is configured
and that it works, and **configure it again** in front of the examiner.

This page tells you what to expect and how to prepare. Every step of the course ends with two
sections built for exactly this:

- **🎓 Explain it to the examiner** — the questions from the ministry question bank for that
  topic, with model answers and the command that shows the answer in your project.
- **🔁 Do it again — without the guide** — a timed drill: configure the step again from your
  plan only.

---

## 1. The three kinds of questions

The ministry question bank (מאגר שאלות פרויקט י"ד) has about 150 questions in 14 topics. They
come in three kinds:

| Kind | Example from the bank | What the examiner wants |
|---|---|---|
| **Explain a concept** | "What is HSRP?", "What is a BPDU?", "What is the difference between a standard and an extended ACL?" | A short, correct definition in your own words, and why it is used |
| **Show me in your project** | "Show me where you configured HSRP", "Show me which switch is the root bridge", "Show me the NAT configuration" | You open the right device, type the right `show` command, and explain the lines |
| **What happens if…** | "If this router fails, what happens?", "If the DR fails, who takes over?" | You understand how the network behaves, not only how it is typed |

The bank grades every question 1 (basic) to 3 (advanced). In this course they are marked
★1, ★2, ★3.

## 2. How to answer — four parts

Answer every question in this order. It takes under a minute and shows you understand:

1. **What it is** — one sentence. *"HSRP gives the PCs one virtual gateway shared by two switches."*
2. **Why** — what problem it solves. *"If one switch fails, the PCs keep their gateway."*
3. **Where in my project** — the device and the interface. *"On both distribution switches, on every SVI."*
4. **Show it** — type the command and point at the line. *`show standby brief` → "DIST1 is Active for VLAN 10."*

If you don't know an answer, say what you **do** know and show it — never guess commands.

## 3. Be ready to configure it again

The examiner may delete a line, shut an interface, or ask you to add something, and watch you
work. You can only do this calmly if you have done every step **without the guide** at least
once. That is what the 🔁 drill at the end of every step is for.

In the drill:
- work from your **plan** (`my-project/plan.md`), not from the step;
- say out loud what each command does while you type it;
- finish with the step's `show` commands and explain the output.

## 4. Which step teaches which topic of the question bank

| Topic in the bank | Where you learn it |
|---|---|
| Address planning, subnet mask, prefix, default gateway | Steps 00, 01 |
| DHCP (how hosts get an address, binding table, configuration) | Step 05 (on the L3 switch), step 13 (on a server) |
| IP helper | Step 13 |
| DHCP attacks (rogue server, starvation) and protection | Step 06 |
| Router-on-a-stick, 802.1Q | Steps 03, 05 |
| HSRP | Step 05 (IPv4), step 14 (IPv6) |
| SVI | Step 05 |
| EtherChannel | Step 04 (LACP), step 13 (PAgP) |
| STP | Step 04 |
| SSH | Step 07 |
| Servers: NTP, Syslog | Step 08 (NTP main server in step 11) |
| Servers: HTTP, DNS, FTP, Mail | Step 11 (DMZ), step 16 (email accounts) |
| AAA (RADIUS, TACACS+) | Step 08 |
| Access lists | Step 07 (standard), steps 10–11 (extended) |
| EIGRP | Step 13 |
| OSPF (areas, DR/BDR, LSAs) | Step 08 |
| OSPF (virtual link, stub / NSSA) | Step 09 |
| OSPF E1 / E2 | Step 11 |
| NAT, PAT | Step 12 (static + PAT), step 13 (dynamic), step 14 (PAT) |
| VPN, encryption, hashing, Diffie-Hellman | Step 15 |
| BGP | Step 12 |
| ASA, DMZ, security levels | Step 11 |

## 5. Your preparation plan

- **After every step:** do the 🔁 drill, answer every 🎓 question out loud, and tick the step in
  your progress table ([progress-template.md](progress-template.md)).
- **Every week:** pick 10 random questions from earlier steps and answer them to a classmate,
  in your own project.
- **Before the exam:** do the final rehearsal in step 16 twice — a classmate picks requirements
  for you to show, and breaks something for you to find and fix.
