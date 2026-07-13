## Enterprise Threat Monitoring with Splunk SIEM and Sysmon ##
![sysplunk](Images/Sysplunk1.png)

## Project Overview
Monitor and detect suspicious activity using Splunk Enterprise integrated with Sysmon. Monitoring and detection operations use the Splunk dashboard.

## Project Environment
* Splunk Enterprise Server
* PC Desktop - Windows 10
* Attacker - Kali Linux

## Monitoring With Splunk Dashboard 
The SOC team detected an alert on the dashboard regarding a malicious application running on an employee’s workstation, followed by an alert regarding data exfiltration

## Cyber Kill Chain Mapping
* **Delivery**
  
  The employee downloaded the file via Google Drive as a .zip archive, extracted it, and then opened it.
  - Raw file : Finance_Report2026.zip
  - Browser : Microsoft Edge.exe
  - Source : Google Drive
  - File extraction : Finance_Report2026.pdf
  - Original file extension : Finance_Report2926.pdf.exe (hidden double extension)


## Incident Report

  Confidentialy : Internal Only
  Incident ID : INC-20260711-005
  Severity Level : Critical
  Detection Time :

  **1. Incident Summary**
  | Parameter | Data |
  | -- | -- |
  | Incident Status | Closed (mitigated) |
  | Attack Category | Phising/Unauthorized access/Data theft |
  | Affected Assets | Hostname : `Desktop01 - Windows 10` |
  | Threat Actor IP | Kali linux : `192.168.1.11` |
  | Entry Point | Phising attachment with download via Google Drive |

  **2. Chronological Logs**
  - 21:11:11 | Delivery
    Process : msedge.exe write file `Finance_Report2026.zip` to directory : `C:\Users\Desktop01\Downloads\`
    Evidence log : Sysmon Event ID 15
    (add here)
  - 21:22:56 | Exploitation & Execution
    Process : The user extracts the .zip archive and finds the Finance Report 2026.pdf file (the original file is hidden with a double extension .pdf.exe) and opens it. The process of giving birth to a child is a command line terminal process.
    Evidence log : Sysmon Event ID 1
    (add here)
  - 22:22:34 | Installation (Reverse shell)
    Process : Finance_Report.pdf.exe opened an outbound connection to an uncommon port.
    Evidence log : Sysmon Event ID 3
    (add here)
  - 23:34:34 | Command & Control
    Process : The threat actor executed local reconnaissance commands via a shell interface.
    Evidence log : Sysmon Event ID 1
    (add here)
  - 24:08:28 | Action On Objective (Data Exfiltration)
    Process : Detected a spike in file reading activity and sending large volumes of encrypted data from victim hosts to the attacker's IP.
    Evidence log : Sysmon ID 3
    (add here)

**3. Indicator Of Compromise (IOC)**
  * File-based IOC
    - File name : Finance_Report2026.pdf.exe
    - Path : C:\Users\Desktop01\Downloads\Finance_Report2026.pdf.exe
    - 256SHA Hash: add here

  * Network-based IOC
    - IP Attacker : 192.168.1.11
    - Port : 4444
    - Source domain : `drive.google.com`

**4. Remediation**

* Host Isolation: Disconnects the network connection of VM WIN-VICTIM-01 to prevent lateral movement.
* C2 Session Termination: Blocks all incoming and outgoing traffic to and from IP `192.168.1.11` on the firewall perimeter.
* Artifact Cleanup: Force-kills the Finance_Report2026.pdf.exe process and deletes the associated binary files from the victim's storage directory. 
