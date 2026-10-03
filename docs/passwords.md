# Password table (טבלת סיסמאות)

The project book must include a password table. These are **lab** passwords — never reuse
them on a real network.

| Purpose | Where | Value |
|---|---|---|
| Privileged EXEC | `enable secret` on every router/switch | `Intel#En4ble` |
| Console line (before AAA) | `line console 0` | `Intel#Con` |
| Local admin user (SSH + AAA fallback) | `username admin privilege 15 secret` | `Intel#Adm1n` |
| TACACS+ shared key | devices ↔ AAA server | `Intel#Tac` |
| RADIUS shared key (optional WPA2-Enterprise) | AP ↔ AAA server | `Intel#Rad` |
| VTP password | all switches | `Intel#Vtp` |
| OSPF MD5 key 1 | Haifa routed links | `Intel#Ospf` |
| EIGRP MD5 key chain | Jerusalem | `Intel#Eigrp` |
| WiFi — employees (`INTEL-CORP`) | AP-HFA-A-EMP | `IntelCorp#2026` |
| WiFi — guests (`INTEL-GUEST`) | AP-HFA-A-GUEST | `WelcomeGuest2026` |
| ASA enable | ASA-HFA-FW1 | `Intel#En4ble` |
| Site-to-site VPN pre-shared key | R-HFA-EDGE ↔ R-JLM-EDGE | `Intel#S2S-Key` |
| Remote-access VPN group `HQ-RA` key | R-HFA-EDGE | `Intel#RA-Group` |
| Remote-access VPN user | `remote1` | `Intel#Remote1` |
| Email users (all) | SRV-HFA-DMZ-MAIL / SRV-KG-MAIL | `Mail#2026` |
| FTP user `ftpadmin` | SRV-HFA-DMZ-FTP | `Intel#Ftp` |
