# 🌐 Network Traffic Forensics: Cyber Attack Investigation via Packet Analysis
![wireshark](Images/wireshark.jpg)
### 🔬 Case Study: Infected Endpoint Detection & Core .pcap Analysis

---

## 📌 Project Context & Disclaimer
> **⚠️ Educational Lab Scenario:** > This investigation was conducted using raw network traffic logs (`.pcap`) from the **Linuxenic Corp** platform. This project demonstrates a SOC analyst’s ability to perform *network forensics*, conduct in-depth packet analysis (*Deep Packet Inspection*), map the timeline of an infection, and identify indicators of compromise (IoC) directly from network cables using **Wireshark**.

---

## 📖 Incident Scenario & Threat Hypothesis
The SOC team reports that an employee’s workstation has been compromised. The team has captured network traffic in `data-leak.pcap`. Further analysis is needed to dig deeper for information. The tool used is Wireshark.

## Investigation
* **Identify the IP addresses of employee workstations**
  

Sebagai analis, saya melakukan investigasi forensik jaringan untuk membuktikan hipotesis serangan berikut:
1. **Initial Access:** Bagaimana malware pertama kali masuk ke jaringan internal?
2. **Command & Control (C2):** Ke mana perangkat korban menghubungi server luar untuk menerima instruksi?
3. **Data Exfiltration / Payload Delivery:** Apakah ada aktivitas pengunduhan file berbahaya atau pencurian data sensitif?

---

## 🏗️ Metodologi & Alur Investigasi Network Forensics
