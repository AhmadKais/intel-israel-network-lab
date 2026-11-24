# Osem — build a full 3-branch project together in class

A complete final project (Osem: Sakhnin, Ashdod, Eilat), built **click by click** in **5 lessons of
90 minutes**, with the theory behind each step. It is based on a real project from a previous year,
with its mistakes fixed, and every mistake became a "find the bug" moment.
Each lesson ends with **"Your own project"**: do the same in your company's network.

**Osem is only an example.** Don't copy its names, addresses or passwords into your project.

| Lesson | Topic | File to open |
|---|---|---|
| [01](lesson-01/) | First department: devices, cables, names, passwords | a new file |
| [02](lesson-02/) | Sakhnin switching: VLAN, VTP, trunks, STP, port security | [lesson02-start.pkt](lesson-02/lesson02-start.pkt) |
| [03](lesson-03/) | Sakhnin routing and services: HSRP, DHCP, DNS, web, FTP, mail | [lesson03-start.pkt](lesson-03/lesson03-start.pkt) |
| [04](lesson-04/) | Connecting the branches: WAN, OSPF, EIGRP, BGP, NAT | [lesson04-start.pkt](lesson-04/lesson04-start.pkt) |
| [05](lesson-05/) | Security: ACL, VPN, SSH, AAA, NTP, Syslog, ASA | [lesson05-start.pkt](lesson-05/lesson05-start.pkt) |

🧱 **Want to build everything alone, without start files?** [From an empty file to the whole project](from-zero/): every device, cable and command that the start files do for you, and 14 tests at the end.

The start file of each lesson is the result of the previous one, so a student who missed a lesson or
got stuck can join. **Packet Tracer 9.0.1** is recommended: that is the version we tested with.

## What you find in every lesson
| Sign | What it is | How to use it in class |
|---|---|---|
| 🌐 | the network now, with a numbered map | say where we are. The numbers show which device to click |
| ❗ | the problem | show it on the screen **before** solving it |
| 💡 | the solution, in words | after it, ask a student to explain it back |
| 📋 | do it yourself in the project | 5 minutes alone, without looking down |
| 🖱 | the full solution: what to click, what to write, what you must see | compare screen to screenshot, step by step |
| 💬 | the idea in simple words | say it before the definition |
| **Example** | a worked example with our addresses | solve it on the board, one row at a time |
| ❓ | a question for the class | wait for answers before you open the answer |
| ✅ | a check | nobody continues before it is true on their screen |
| 👀 | see the packets move, in Simulation mode | 5–10 minutes on the projector: students guess, then you click |
| 🧯 | "It doesn't work?" | the student looks here first, then calls you |
| 🔎 | find the bug | in pairs, 3 minutes |
| 🏋 | exercises, with answers | for those who finish early, or homework |
| 🎓 | examiner questions, with model answers | one student asks, another answers |
| ✏️ | exit quiz | 3 minutes at the end, on paper |

Every lesson has more material than 90 minutes: the missions are the core, and 🏋, 🎓 and ✏️ can move to homework.

## The address plan
| Branch | VLAN | Name | Network | Gateway (VIP) |
|---|---|---|---|---|
| Sakhnin | 10 | MANAGEMENT | 192.168.10.0/24 | .252 |
| Sakhnin | 20 | DISPLAY-SALARIES | 192.168.20.0/24 | .252 |
| Sakhnin | 30 | ASSEMBLY-MEETING | 192.168.30.0/24 | .252 |
| Sakhnin | 40 | IT | 192.168.40.0/24 | .252 |
| Sakhnin | 50 | SERVERS | 192.168.50.0/24 | .252 |
| Ashdod | 60–63 | 63 = servers | 192.168.60–63.0/24 | .252 |
| Eilat | 80–83 | 83 = servers | 192.168.80–83.0/24 | .252 |

In every network .254 is the main router and .253 the backup. The WAN table is in lesson 4.

## Classroom passwords
| Where | Value |
|---|---|
| Console / enable | `Osem#Con` / `Osem#En1` |
| After lesson 5, console of the main and backup routers | `admin` / `Osem#Adm1` |

For class only. In your project choose passwords of your own.
