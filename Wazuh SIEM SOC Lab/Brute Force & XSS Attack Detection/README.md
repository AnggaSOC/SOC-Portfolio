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
An intrusion detection system (IDS) detects a spike in network activity and interprets it as a network scan. It sends an alert, but it's considered a **false positive** due to the high network traffic.

| Timestamp (24 Jun 2026) | Target Agent | Rule Description | Rule Level | Rule ID |
| :--- | :--- | :--- | :---: | :---: |
| 20:59:56.458 | Server02 | IDS event | 6 | **20101** |

## 2. Brute Force Detection
Wazuh detects repeated attempts to log into a server via SSH. This detection is strengthened by matching with MITRE ATT&CK.

*   **Target IP:** `192.168.56.20` (Ubuntu Server)
*   **Attacker IP:** `192.168.56.10` (Kali Linux)
*   **Perintah Hydra yang dieksekusi:**
```bash
    hydra -l target_user -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.20 -t 4 -V
    ```

## 2. Analisis Log Mentah (Raw Log Analysis)
Sebelum melihat dasbor SIEM, berikut adalah bukti log otentikasi gagal yang terekam secara lokal di target server pada file `/var/log/auth.log`:

```text
# Pasang potongan raw log Anda di sini. Contoh:
Jun 22 20:05:12 ubuntu-server sshd[3142]: Failed password for invalid user admin from 192.168.56.10 port 43210 ssh2
Jun 22 20:05:13 ubuntu-server sshd[3145]: Failed password for invalid user admin from 192.168.56.10 port 43212 ssh2
