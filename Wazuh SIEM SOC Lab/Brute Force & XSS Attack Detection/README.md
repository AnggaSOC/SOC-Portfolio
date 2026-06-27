# SOC Lab: End-to-End Threat Detection & Incident Response with Wazuh SIEM and Suricata

## Project Overview
This project documents the development of a small-scale SOC (Security Operations Center) laboratory and the simulation of real-world attacks (Cyber Attack Lifecycle). The main focus of this project is to integrate **Suricata (NIDS)** and **Wazuh (SIEM/EDR)** to detect attacker tactics, ranging from the reconnaissance phase, access attempts (brute force), system intrusion (initial access), to post-compromise activities (unauthorized file modification).
This project also documents the **Incident Response (IR)** actions taken by SOC analysts to tactically mitigate attacks and perform system hardening.
## Network Architecture & Simulation Workflow
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
| Host Name | Operating System | Role | IP Address | Security Tools Installed |
| :--- | :--- | :--- | :--- | :--- |
| **Wazuh-Server** | Ubuntu Server 22.04 | Central SIEM Manager | `192.168.1.100` | Wazuh Indexer, Dashboard, Manager |
| **Mint-Client** | Linux Mint |  Web Server | `192.168.1.11` | Wazuh Agent, Suricata NIDS, Nginx |
| **Kali-Attacker**| Kali Linux | Threat Actor / Attacker | `192.168.1.12` | Nmap, Hydra, SSH Client | 

## 1. SOC Detection:
IDS detected a potential network scan conducted by an IP address outside the network. 

Wazuh alert:
| Timestamp (26 Jun 2026) | Target Agent | Rule Description | Rule Level | Rule ID |
| :--- | :--- | :--- | :---: | :---: |
| 10:45:08.657 | Server02 | IDS event | 6 | **20101** |

Full_log :

06/26/2026-10:45:06.916961  [**] [1:2022973:1] ET INFO Possible Kali Linux hostname in DHCP Request Packet [**] [Classification: Potential Corporate Privacy Violation] [Priority: 1] {UDP} 0.0.0.0:68 -> 255.255.255.255:67

An external IP address using a **Kali Linux** device is scanning the server's network

## 2. Credential Access (SSH Brute Force)
Within a short period of time, there was a potential network scanning incident followed by a brute-force attack using an IP address outside the network 

*   **Target IP:** `192.168.1.11` (Linux Mint Server)
*   **Attacker IP:** `192.168.1.12` (Kali Linux)
  
| Timestamp (26 Jun 2026) | Target Agent | Rule Description | Rule Level | Rule ID |
| :--- | :--- | :--- | :---: | :---: |
| 10:52:09.098 | Server02 | sshd: brute force trying to get access to the system. Non existent user. | 10 | **5712** |
| 10:52:09.171 | Server02 | PAM: Multiple failed logins in a small period of time. | 10 | **5551** |
  


## 3. Initial Access (Successful Compromise)
An anomaly was detected because a successful login occurred immediately after hundreds of failure alerts from the same attacker's IP address (192.168.1.12).

| Timestamp (26 Jun 2026) | Target Agent | Rule Description | Rule Level | Rule ID |
| :--- | :--- | :--- | :---: | :---: |
| 10:53:57.262 | Server02 | Multiple authentication failures followed by a success. | 12 | **40112** |

An attempt to previlege escalation by accessing the **“sudo”** command was detected but failed.

| Timestamp (26 Jun 2026) | Target Agent | Rule Description | Rule Level | Rule ID |
| :--- | :--- | :--- | :---: | :---: |
| 10:55:53.316| Server02 | Command not allowed. | 5 | **5406** |

## 4. Persistence & Tampering (Malicious File Creation)
Attackers target the web server directory **/var/www/html/** to plant malicious files (potentially a web shell or backdoor) in order to maintain remote access.

| Timestamp (26 Jun 2026) | Target Agent | Rule Description | Rule Level | Rule ID |
| :--- | :--- | :--- | :---: | :---: |
| 10:59:43.808| Server02 | File added to the system. | 5 | **554** |
| 10:58:21.786| Server02 | File added to the system. | 5 | **554** |
| 11:01:32.310| Server02 | Integrity checksum changed. | 7 | **550** |

```
full_log :

File '/var/www/html/uploads/update.php' added
Mode: whodata

full_log :

File '/var/www/html/uploads/config_backup.txt' added
Mode: whodata

full_log :

File '/var/www/html/uploads/update.php' modified
Mode: whodata
Changed attributes: size,mtime,md5,sha1,sha256
Size changed from '13' to '12'
Old modification time was: '1782446383', now it is '1782446492'
Old md5sum was: '81d325b5c1288a4fa62b59d69fcf670b'
New md5sum is : 'cbe9b844bcce20c70821ae1db0f0efdf'
Old sha1sum was: 'c991a7cc7e1369c8bffbaac05593de0db19bab1c'
New sha1sum is : 'c42ade45806a8c4d46f16b2ce0c916ee43ccb27d'
Old sha256sum was: '99bbf7c32d534bb2e3141260b32c1443b0831b1be4e373954b4b6ca9680aa218'
New sha256sum is : '68704ae924b1d7237d34acb2e439d09f904f2931f5048049e2b61d3f92b72efb'
```
## Incident Response & System Hardening (SOC Playbook)

**1. Implementation of the Account Lockout Policy**
* To prevent future brute-force attacks, the SOC configured the Pluggable Authentication Modules (PAM) architecture using `pam_faillock` on Linux Server.
Configuration on **/etc/pam.d/common-auth:**
```
auth required pam_faillock.so preauth silent audit deny=3 unlock_time=600
auth [default=die] pam_faillock.so authfail audit deny=3 unlock_time=600
```

**Security Impact:** If there are three consecutive failed login attempts, the target account will be **automatically locked** for 10 minutes, thwarting automated brute-force attacks.

**2. Network Containment**
* Tactical isolation (containment) of the attacker's IP address (192.168.1.12) to terminate all active connections and prevent lateral movement.

```
sudo iptables -A INPUT -s 192.168.1.12 -j DROP
```

**3. Remediation**
* A local forensic analysis of the suspicious files. The contents of the **config_backup.txt** and **update.php** files were examined, their cryptographic hash values were compared, and after they were confirmed to be malicious code, the files were isolated and removed from the web server.

## Key Takeaways
* **1. Defense-in-Depth:** The combination of network monitoring (Suricata) and host monitoring (Wazuh FIM/Log Analysis) provides 360-degree visibility into unknown activity.

* **2. Log Correlation Is Key:** Early detection was successful because the analyst was able to correlate scanning activity, followed by a series of SSH failures, and finally a successful login from the same IP address—an indicator of compromise (IoC).
