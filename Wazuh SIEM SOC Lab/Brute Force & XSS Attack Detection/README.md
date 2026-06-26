# Technical Walkthrough: SSH Brute Force, XSS Detection & Response

This technical documentation explains the process of simulating a brute force attack, xss attack, and malware distribution on Ubuntu Server's SSH service and how Wazuh SIEM performs detection and log analysis.

## 🗺️ Network Architecture & Simulation Workflow
```
[ Kali Linux ] --(Network Scanning/Host)--> [ Linux Mint Server ]
                                                     |
                                        +------------+------------+
                                        |                         |
                                 [ Network Traffic ]       [ System Logs ]
                                        |                         |
                                 (Monitored by IDS)        (Monitored by FIM/OSSEC)
                                        |                         |
                                 [ /var/log/suricata/eve.json ]   |
                                        |                         |
                                        +------------+------------+
                                                     |
                                              [ Wazuh Agent ]
                                                     |  (Send logs via Port 1514/TCP)
                                                     v
                                             [ Wazuh Server ]

```
## 1. Alert From IDS (Suricata)
IDS detected a potential network scan conducted by an IP address outside the network

| Timestamp (26 Jun 2026) | Target Agent | Rule Description | Rule Level | Rule ID |
| :--- | :--- | :--- | :---: | :---: |
| 10:45:08.657 | Server02 | IDS event | 6 | **20101** |

Full_log :

06/26/2026-10:45:06.916961  [**] [1:2022973:1] ET INFO Possible Kali Linux hostname in DHCP Request Packet [**] [Classification: Potential Corporate Privacy Violation] [Priority: 1] {UDP} 0.0.0.0:68 -> 255.255.255.255:67

An external IP address using a **Kali Linux** device is scanning the server's network

## 2. Brute Force Detection
Within a short period of time, there was a potential network scanning incident followed by a brute-force attack using an IP address outside the network 

*   **Target IP:** `192.168.1.11` (Linux Mint Server)
*   **Attacker IP:** `192.168.1.12` (Kali Linux)
  
| Timestamp (26 Jun 2026) | Target Agent | Rule Description | Rule Level | Rule ID |
| :--- | :--- | :--- | :---: | :---: |
| 10:52:09.098 | Server02 | sshd: brute force trying to get access to the system. Non existent user. | 10 | **5712** |
  


## 2. Analisis Log Mentah (Raw Log Analysis)
Sebelum melihat dasbor SIEM, berikut adalah bukti log otentikasi gagal yang terekam secara lokal di target server pada file `/var/log/auth.log`:

```text
# Pasang potongan raw log Anda di sini. Contoh:
Jun 22 20:05:12 ubuntu-server sshd[3142]: Failed password for invalid user admin from 192.168.56.10 port 43210 ssh2
Jun 22 20:05:13 ubuntu-server sshd[3145]: Failed password for invalid user admin from 192.168.56.10 port 43212 ssh2
