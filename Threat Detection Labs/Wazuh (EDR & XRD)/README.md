# SOC Lab: End-to-End Threat Detection & Incident Response with Wazuh SIEM and Suricata
![wazuh](images.png)

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


## Incident Response Guidelines
* **Potential Network Scanning**
  * Source check : internal IP adress or external IP address?
  * Validation : if internal check vulnerability scanning schedule with IT team
  * Reputation : if external, check IP address reputation in Threat Intelligence (Virustotal/AbuseIPDB)
* **Failed Login / Brute Force**
  * Pattern check : Is the account being targeted continuously, or are multiple accounts being targeted (password spraying)?
  * Contextual Check: If the issue is internal, make sure it’s not because a new user changed their password, but rather because an old app is stuck trying to log in automatically.
* **Unknown Account Successfully Logged In**
  * Account Status: Check with the HR/Admin team to see if this is a new employee, a vendor account, or a trial account.
  * Access Anomaly: Check the login location (e.g., VPN from abroad) and the access rights associated with that account.
* **Modifikasi/Penambahan File di Server**
  * Check for Ticket Changes: Align with the Change Management schedule or the Developer/SysAdmin team’s deployment documentation.
  * Location & Reputation: Check whether the file is located in a sensitive directory (e.g., the web root folder) and verify the file's hash on VirusTotal.

**The Golden Rule Before Taking Action:**
Always answer these 4 questions first: Who (Who is the perpetrator?), What (What are they doing?), Where (What is the target?), and When (When did it happen, and was it outside of business hours?). Do not block the activity until you have confirmed that it is a false positive.

---

## 📌 Scenario
> In this case, the alert indicates a true positive: an attacker is attempting to take over the server and add malicious files
---


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
  
## 2.1 Alert Verification & Incident Triage (Critical Analysis Phase)
When the multiple failed login alerts (Rule ID: 5712 & 5551) triggered simultaneously, a professional SOC analyst does not immediately jump to conclusions or initiate drastic network isolation. Two initial hypotheses must be verified to eliminate false alarms:

 * **False Positive Check:** Could this alert be caused by a misconfigured internal script, system application, or an automated monitoring tool trying to authenticate?
 * **Authorized Activity Check:** Could this be an authorized system administrator attempting to remote via SSH, perhaps having forgotten their password or using an outdated credential manager?

####  Triage and Verification Process:
* **Change Management Review:** The analyst cross-checked the internal IT maintenance logs and active tickets. There was no scheduled maintenance window or authorized remote task assigned for the Linux Mint Server during this period.
* **Log Pattern & Behavior Analysis:** The analyst scrutinized the volume and velocity of the failed attempts. If an authorized human administrator forgot a password, the logs would typically show 3 to 5 failed attempts within a reasonable human typing interval, followed by a pause. 
* **Conclusion:** In this case, the SIEM captured **hundreds of authentication failures within seconds** targeting generic and non-existent usernames (e.g., `root`, `admin`). This pattern confirms the use of an automated brute-force tool (e.g., Hydra). Therefore, the authorized activity hypothesis was discarded, and the incident status was elevated to a verified **True Positive**.

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

**4. Long-Term System Hardening (Post-Incident Preventive Actions)**
To ensure the infrastructure is resilient against similar credential access attacks in the future, the following strategic hardening steps were implemented on the Linux Mint Server:

* **Disabling Password Authentication in Favor of SSH Keys:**
  The server configuration in `/etc/ssh/sshd_config` was updated to disallow password-based logins entirely, forcing the use of secure cryptographic SSH key pairs.
  ```text
  PasswordAuthentication no
  PubkeyAuthentication yes

Security Impact: This completely neutralizes automated brute-force tools (like Hydra) because there are no text passwords to guess.

* **Implementing IP Whitelisting (TCP Wrappers / Firewall Rules):**
SSH access was restricted only to authorized management IP addresses (e.g., the SOC Analyst workstation or Jump Box network). All other IP addresses attempting to access Port 22 are dropped by default.
```
Example
# Configuration in /etc/hosts.allow
sshd : 192.168.1.50 : ALLOW

# Configuration in /etc/hosts.deny
sshd : ALL : DENY

```

## Key Takeaways
* **1. Defense-in-Depth:** The combination of network monitoring (Suricata) and host monitoring (Wazuh FIM/Log Analysis) provides 360-degree visibility into unknown activity.

* **2. Log Correlation Is Key:** Early detection was successful because the analyst was able to correlate scanning activity, followed by a series of SSH failures, and finally a successful login from the same IP address—an indicator of compromise (IoC).

