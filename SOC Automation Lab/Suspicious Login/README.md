## SUSPICIOUS LOGIN - AUTOMATION RESPONSE

**Project Overview**

Automatically detects suspicious login activity with Wazuh alerts. Shuffle is used to automate the attack response process.

**Workflow Diagram**

![ssh](Images/SSH.png)

**Scenario**

The server administrator and SOC analyst receive an email notification that there was login activity on the server outside of business hours. The server administrator can confirm whether the login actually occurred. If so, the SOC analyst will receive an email notification that the activity was legitimate. If not, the SOC analyst will receive an email notification that the activity was illegitimate, and the system will automatically create an incident ticket.

