# Project 2: Enterprise EDR & Threat Hunting Grid (Sentient Shield)

**Objective:**  
Build an enterprise-grade SOC framework for real-time detection and automated response to brute-force and ransomware attacks. Includes File Integrity Monitoring (FIM), MITRE ATT&CK mapping, and active response automation.

**Use Case:**  
Infotact servers are under constant brute-force and ransomware threats. This project demonstrates detection, automated response, and executive-level reporting with MITRE ATT&CK correlation.

---
## Lab Architecture

The SOC lab consists of three machines:

1. Wazuh Server (Ubuntu)
   - Wazuh Manager
   - Wazuh Dashboard
   - Log analysis and correlation

2. Windows Endpoint
   - Wazuh Agent
   - Sysmon installed
   - File integrity monitoring

3. Attacker Machine (Kali Linux) 
   - Hydra
   - Attack simulation

Attack traffic → Wazuh detection → Alert generation → Active response
---
## Tools Used

- Wazuh SIEM
- Sysmon
- Kali Linux
- Hydra
- Atomic Red Team
---


## Week 1 – Infrastructure & Agent Deployment

**Objective:**  
Deploy Wazuh Manager and agents on Linux and Windows servers, install Sysmon, and validate heartbeat/log reporting.

### Steps & Commands

**1. Wazuh Manager on Linux**
```bash
sudo apt update
sudo apt install wazuh-manager -y
sudo systemctl enable --now wazuh-manager
sudo systemctl status wazuh-manager
```
<img width="966" height="595" alt="week1 agent" src="https://github.com/user-attachments/assets/3d73cd3b-9ba6-4acc-9da7-b2a654e68815" />


**2. Linux Agent Deployment**
```bash
sudo apt install wazuh-agent -y
sudo nano /var/ossec/etc/ossec.conf   # configure Wazuh Manager IP
sudo systemctl enable --now wazuh-agent
sudo systemctl status wazuh-agent
```
<img width="752" height="571" alt="12" src="https://github.com/user-attachments/assets/5d8f7507-4cfc-4cd4-acaf-aa935a12d09a" />


**3. Windows Agent & Sysmon**

Install Wazuh Agent MSI and configure Manager IP

Install Sysmon:
```bash
Sysmon64.exe -accepteula -i sysmonconfig.xml
```      

Explanation:

Agents collect real-time system and log data.

Sysmon enhances Windows telemetry with process, network, and file events.

Continuous heartbeat monitoring ensures agent health.

Gate Check: Verify all agents show Active in Wazuh dashboard.

## Week 2 – Detection Rules (FIM & Custom Logic)

Objective:
Configure File Integrity Monitoring (FIM) and custom XML rules for proprietary logs. Enable Vulnerability Detector.

Steps & Commands

1. FIM Configuration
```bash
<syscheck>
  <directories check_all="yes">/etc,/var/www/html,/opt/app/config</directories>
  <frequency>600</frequency>  
</syscheck>
```
2. Custom XML Rule
```bash
<group name="custom-app-logs">
  <rule id="100001" level="10">
    <decoded_as>custom-app</decoded_as>
    <description>Critical configuration change detected</description>
    <frequency>1</frequency>
  </rule>
</group>
```
3. Enable Vulnerability Detector

sudo nano /var/ossec/etc/ossec.conf
sudo systemctl restart wazuh-agent

4. Test Detection

echo "Test modification" >> /var/www/html/index.html

<img width="1917" height="962" alt="week2(failed)" src="https://github.com/user-attachments/assets/1cfc5e32-acb5-4af9-8c0a-a48f1373e8ed" />

<img width="1918" height="927" alt="week2(user group)" src="https://github.com/user-attachments/assets/5727c99a-342a-4e38-91bf-6330fdd8ffc1" />



Explanation:

FIM monitors critical files for unauthorized changes.

Custom rules generate high-severity alerts for sensitive application logs.

Vulnerability Detector scans for known CVEs automatically.

Gate Check: High-severity alert should appear within seconds in the Wazuh dashboard.

MITRE ATT&CK Mapping Example:

File modification → T1070 (Indicator Removal)

Unauthorized access → T1078 (Valid Accounts)

## Week 3 – Active Response (IPS)

**Objective:** Detect SSH brute-force attempts and automatically block attacker IP.

**Steps:**

1. **Enable Active Response in Wazuh Manager**

```bash
sudo nano /var/ossec/etc/ossec.conf
```

```XML
  <!-- Active Response (Week-3) -->

  <active-response>
    <command>firewall-drop</command>
    <location>local</location>
    <rules_id>5710,5712,5501,5760,1002</rules_id>
    <timeout>3600</timeout>
  </active-response>
```
```
**Restart Wazuh Manager**
```
sudo systemctl restart wazuh-manager
```

**Simulate SSH Brute-Force**
``` bash
sudo hydra -l username -P /usr/share/wordlists/rockyou.txt ssh:Target_Ip
```
<img width="1003" height="405" alt="w3(hydra)" src="https://github.com/user-attachments/assets/45cc4ca4-38ea-4324-8835-88eded8aa03e" />


<img width="895" height="202" alt="w-3" src="https://github.com/user-attachments/assets/32ae5f1c-d8d5-4cb2-a9e2-71e309073952" />



**Verify Firewall Block**
```
sudo iptables -L --line-numbers
```

Screenshots:
�
Wazuh detected multiple failed login attempts triggered by Hydra.
�
Attacker IP automatically blocked by Wazuh active response.

Windows default logs provide limited visibility. Sysmon (System Monitor) from Microsoft Sysinternals generates detailed logs such as:

- Process creation  
- Network connections  
- File creation events  
- DNS queries  

These logs help the SIEM platform detect suspicious activities more effectively.

---

### Step 1 – Download Sysmon

Download Sysmon from Microsoft Sysinternals.

Official Source:  
https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

After downloading, extract the ZIP file.

#### Extracted Files

```
Sysmon64.exe
Sysmon.exe
Eula.txt
```

---

### Step 2 – Install Sysmon

Open **PowerShell as Administrator** and run:

```powershell
.\Sysmon64.exe -i sysmonconfig.xml
```
<img width="1918" height="932" alt="Screenshot 2026-03-12 202029" src="https://github.com/user-attachments/assets/9c636068-f600-4cf7-a3da-9a42cc13b53f" />

#### Explanation

This command installs Sysmon using a configuration file (`sysmonconfig.xml`) which defines which system activities should be monitored.

Once installed, Sysmon begins monitoring system behavior.

#### Expected Output

```
Sysmon installed
SysmonDrv installed
```

---

### Step 3 – Verify Sysmon Configuration

Run the following command:

```powershell
.\Sysmon64.exe -c
```
<img width="862" height="542" alt="week3()" src="https://github.com/user-attachments/assets/40378857-0d8a-48fa-b29b-81f842c12517" />

#### Explanation

This command displays the current Sysmon configuration and shows which monitoring rules are enabled.

#### Example Output

```
ProcessCreate enabled
NetworkConnect enabled
FileCreate enabled
DNSQuery enabled
```

These events help detect suspicious activities such as malware execution or unauthorized network connections.

---
## Week 4 – Threat Simulation

**Objective:** Simulate ransomware activity on the Windows endpoint and detect it using Wazuh.

---

### Steps

1. **Run Atomic Red Team Technique T1490 – Delete Shadow Volume Copies**  

Open PowerShell on the Windows endpoint:

```powershell
vssadmin delete shadows /all /quiet
```
<img width="928" height="983" alt="w4(2)" src="https://github.com/user-attachments/assets/9bd1ca5e-621c-4970-a565-4751986a1c55" />



<img width="1366" height="868" alt="w4(3)" src="https://github.com/user-attachments/assets/b8558e98-8d60-4d8a-978e-68f509ef65b9" />


<img width="1891" height="921" alt="w4(1)" src="https://github.com/user-attachments/assets/658b091d-bd37-4c08-bd37-654af4270fe6" />





---
🔎 Detection Workflow
Atomic Red Team Attack Executed
        │
        ▼
Sysmon Logs Event
        │
        ▼
Wazuh Agent Collects Log
        │
        ▼
Wazuh Manager Analyzes Event
        │
        ▼
Security Alert Generated
---
## Conclusion

This project successfully demonstrates a working SOC environment capable of detecting and responding to security threats in real time.

Key capabilities implemented:

- Centralized log monitoring
- Threat detection
- Automated incident response
- MITRE ATT&CK mapping
- Ransomware attack simulation

The project replicates real-world SOC operations and provides hands-on experience with enterprise security monitoring.
