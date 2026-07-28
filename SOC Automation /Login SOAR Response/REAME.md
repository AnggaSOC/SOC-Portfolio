## PROJECT OVERVIEW
Detect SSH login activity outside of business hours and automate the response using Shuffle, email, and Jira Management System for incident ticketing response

## PROJECT ENVIRONTMEN
- Wazuh server
- Enterprise server (Wazuh agent)
- SOC Analyst
- Administrator server
- Attacker
- Shuffle Automation Tools
- Jira Management System

---

## 🛠️ Infrastructure & IP Addressing

| Component | IP Address | Function / Role |
| :--- | :--- | :--- |
| **Wazuh Server** | `192.168.1.2` | Centralized SIEM & Log Analysis |
| **Administrator Server** | `192.168.1.8` | Admin Endpoint (Monitored by Wazuh Agent) |
| **Enterprise Server** | `192.168.1.11` | Target Server (Monitored by Wazuh Agent) |
| **Attacker** | `192.168.1.12` | Simulation Endpoint (Attacker Node) |
| **Shuffle Automation** | Cloud / Local | SOAR Platform (Workflow Engine) |
| **Jira Management** | Cloud / Local | Incident Ticketing System |

---

## 🏗️ Network Architecture Topology

