# 📧 Email Security Triage & Forensics: End-to-End Phishing Investigation Report
### 🔬 Case Study: Linuxenic Security Simulation Lab

---

## 📌 Project Context & Disclaimer
> **⚠️ Educational Lab Scenario:** 
> This investigation was conducted using simulated artifacts (.eml/.msg email files, header logs, and untrusted links/files) provided by the Linuxenic Corp. cybersecurity training platform. This report has been compiled chronologically into a single comprehensive case study to demonstrate the incident response capabilities of a SOC analyst. 

---

## Manual Analysis of The Header and Body 
The SOC team received a report of a potentially malicious email. The email contained a link intended for employees


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
## 🏗️ Metodologi & Alur Investigasi
