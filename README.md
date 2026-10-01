# SOC Detection \& Incident Response Lab



A hands-on Security Operations Center (SOC) home lab built using Wazuh SIEM, Windows, Ubuntu Server, and Kali Linux. This project focuses on security monitoring, log analysis, custom detection rules, threat hunting, and incident investigation.



## &#x20;Project Overview



The objective of this project is to build a functional SOC environment to simulate security events, detect suspicious activities, analyze alerts, and document incident investigations.



The lab uses Wazuh as its SIEM platform to collect and analyze logs from a Windows endpoint. Kali Linux is used to simulate network scanning activity and test custom detection rules.



### &#x20;Lab Architecture



|Component|Configuration|
|-|-|
|SIEM|Wazuh 4.14|
|SIEM Server|Ubuntu Server 24.04.5 LTS|
|Endpoint|Windows|
|Attack Simulation|Kali Linux|

|Virtualization|Oracle VirtualBox|
|-|-|
|Remote Access|PuTTY|
|Log Collection|wazuh Agent, Windows Event Logs, Sysmon|
|Network|NAT and Host-only Adapter|



### &#x20;Network Configuration

|Machine|IP Address|
|-|-|
|Wazuh Server|192.168.56.10|
|Windows Host|192.168.56.1|
|Kali Linux|192.168.56.20|





### &#x20;Tools and Technologies



\- Wazuh SIEM: Security event collection, alert monitoring, and threat detection.

\- Ubuntu Server: Hosts the Wazuh manager and dashboard.

\- Windows: Monitored endpoint generating security and Sysmon events.

\- Sysmon: Provides detailed Windows process creation and system activity logs.

\- Kali Linux: Used for controlled network scanning and attack simulation.

\- Nmap: Network scanning and reconnaissance simulation.

\- VirtualBox: Virtualized lab environment.

\- PuTTY: SSH access to the Ubuntu server.



### &#x20;Implemented Detection Rules



Custom Wazuh rules were created and tested to detect specific security events.

|Rule.id|Detection|Severity|
|-|-|-|
|100100|Notepad process creation|5|
|100101|PowerShell process creation|7|
|100102|Windows failed login|8|
|100103|Blocked network connection from Kali|8|



### &#x20;Detection Details



###### 1\. Process Creation Monitoring



\- Monitors Windows Sysmon process creation events.

\- Detects Notepad and PowerShell execution.

\- Uses custom rules 100100 and 100101.



###### 2\. Failed Login Detection



\- Detects Windows failed authentication events.

\- Uses Windows Event ID 4625.

\- Custom rule: 100102.

\- Severity: Level 8.



###### 3\. Network Scan Detection



\- Simulates network scanning from Kali Linux using Nmap.

\- Monitors Windows Filtering Platform Event ID 5157.

\- Detects blocked connection attempts originating from Kali Linux.

\- Custom rule: 100103.

\- Severity: Level 8.

### 

### Investigations



##### 1\. Windows Failed Login Investigation



A failed login scenario was generated on the Windows endpoint to test authentication monitoring.



###### Detection:

\- Windows Event ID: 4625

\- Custom Wazuh Rule: 100102

\- Severity: Level 8



The alert was verified in the Wazuh dashboard and documented in the investigation report.



[Failed Login Investigation](investigation/failed-login-investigation-report.md)


##### 2\. Nmap Network Scan Investigation



A controlled Nmap scan was performed from Kali Linux against the Windows host.



The scan targeted ports 135, 139, 445, and 3389. The ports were reported as filtered.



Windows Filtering Platform generated Event ID 5157 for blocked connections. Wazuh collected these events and successfully triggered custom rule 100103.



###### &#x20;Detection:

\- Source IP: 192.168.56.20

\- Windows Event ID: 5157

\- Custom Wazuh Rule: 100103

\- Severity: Level 8



\[Failed Login Investigation](investigation/failed-login-investigation.md)



#### &#x20;Wazuh Dashboard



A custom SOC dashboard was created to visualize security alerts and monitor endpoint activity.



The dashboard includes:



\- Wazuh Alerts Over Time

\- Alerts by Detection Rule

\- Alerts by Severity Level

\- Custom Detection Alerts

\- Alerts by Agent



##### &#x20;Dashboard Screenshots




## Dashboard Screenshots

### Wazuh SOC Dashboard
![Wazuh SOC Dashboard](investigation/Screenshots_evidence/Wazuh-dashboard.png)

### Custom Detection Rules
![Custom Detection Rules](investigation/Screenshots_evidence/custom-rules.png)

## Detection Evidence

### Notepad Process Detection
![Notepad Detection](investigation/Screenshots_evidence/notepad-detection.png.png)

### Notepad Alert
![Notepad Alert](investigation/Screenshots_evidence/notepad-alert.png.png)

### PowerShell Detection
![PowerShell Alert](investigation/Screenshots_evidence/powershell-alert.png.png)

### Failed Login Detection
![Failed Login Alert](investigation/Screenshots_evidence/failed-login-alert.png.png)

### Nmap Detection
![Nmap Detection Alert](investigation/Screenshots_evidence/nmap-detection-alert.png.png)

### Nmap Scan
![Nmap Scan](investigation/Screenshots_evidence/nmap_scan.png)

### Wazuh Alert Details
![Wazuh Alert Details](investigation/Screenshots_evidence/wazuh-alert-details.png.png)

###### 

###### Project Structure






## Project Structure

```text
SOC-Home-Lab/
├── README.md
├── setup-guide.md
└── investigation/
    ├── failed-login-investigation-report.md
    ├── nmap-scan-investigation.md
    └── Screenshots_evidence/
        ├── Wazuh-dashboard.png
        ├── custom-rules.png
        ├── failed-login-alert.png.png
        ├── nmap-detection-alert.png.png
        ├── nmap_scan.png
        ├── notepad-alert.png.png
        ├── notepad-detection.png.png
        ├── powershell-alert.png.png
        └── wazuh-alert-details.png.png
```



#### &#x20;Skills Demonstrated



\- SIEM deployment and configuration

\- Security event monitoring

\- Windows Event Log analysis

\- Sysmon log analysis

\- Custom Wazuh detection rule creation

\- Network reconnaissance simulation

\- Firewall event investigation

\- Alert triage and incident documentation

\- Virtual machine and network configuration



#### &#x20;Conclusion



This project demonstrates a hands-on SOC environment built to practice security monitoring, detection engineering, and incident investigation.



It combines endpoint telemetry, network scanning simulations, custom Wazuh rules, and dashboard visualizations to demonstrate a basic security monitoring and incident response workflow.



#### &#x20;Disclaimer



This project was developed for educational purposes in a controlled lab environment. All simulations and testing were performed on systems under my control.

