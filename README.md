# Enterprise EDR & Threat Hunting Grid (Sentient Shield)

**Objective:**  
Build an enterprise-grade SOC framework for real-time detection and automated response to brute-force and ransomware attacks. Includes File Integrity Monitoring (FIM), MITRE ATT&CK mapping, and active response automation.

**Use Case:**  
Infotact servers are under constant brute-force and ransomware threats. This project demonstrates detection, automated response, and executive-level reporting with MITRE ATT&CK correlation.

---
## Lab Architecture

The Wazuh manager is deployed on a centralized Linux server. 
Agents are installed on both Windows and Linux endpoints.

The Linux system acts as the SSH target for brute-force attack simulation, while the Windows system is used for ransomware simulation using Sysmon.

Active response for SSH brute-force detection must be configured on the Linux agent, as firewall rules are enforced on the target system.

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

3. Enable Vulnerability Detector
``` bash
sudo nano /var/ossec/etc/ossec.conf
sudo systemctl restart wazuh-agent
```

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
    <rules_id>5716,5715,5710,5712</rules_id>
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
<img width="1567" height="202" alt="image" src="https://github.com/user-attachments/assets/2c7e32ec-5d3b-43a9-9e8e-6db4984d1e0e" />


<img width="1523" height="496" alt="image" src="https://github.com/user-attachments/assets/15699a77-ebe3-489b-895e-8523ae601b62" />


**Verify Firewall Block**
```
sudo iptables -L --line-numbers
```

A brute-force attack was simulated using Hydra from Kali Linux.
Wazuh successfully detected multiple failed login attempts.

Active response was configured; however, IP blocking requires the Wazuh agent to be installed on the Linux target system. In this setup, detection was verified, while automated blocking can be achieved with proper agent deployment.

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
## Week 4: Threat Simulation 

### Objective
Simulate ransomware behavior using Atomic Red Team techniques such as shadow copy deletion.

### Attack Simulation
- Created shadow copy
- Deleted using: vssadmin delete shadows /all /quiet

### Detection Results
- Wazuh detected process creation
- Custom rules triggered for ransomware behavior

### Custom Detection Rules   👈 HERE

Custom Wazuh rules were created to detect:

- SSH brute-force attempts
- Suspicious ransomware file extensions (.locked)
- Mass file modifications
- Shadow copy deletion (ransomware behavior)

Configuration file:
configs/local_rules.xml

---

### Steps

1. **Run Atomic Red Team Technique T1490 – Delete Shadow Volume Copies**  

Open PowerShell on the Windows endpoint:

```powershell
vssadmin delete shadows /all /quiet
```

<img width="1917" height="858" alt="image" src="https://github.com/user-attachments/assets/817232bf-89b5-4150-8c39-d56e33877f9e" />

---

<img width="1897" height="293" alt="image" src="https://github.com/user-attachments/assets/bcf5dabc-8bff-4cbe-b1b0-105f731c4f71" />

MITRE ATT&CK Mapping:
- T1486 – Data Encrypted for Impact
- T1490 – Inhibit System Recovery

---

Ransomware behavior was simulated by creating and deleting volume shadow copies using the vssadmin utility.

A custom Wazuh rule was developed to detect shadow copy deletion activity via Sysmon logs and map it to MITRE ATT&CK technique T1490 (Inhibit System Recovery).

High-severity alerts were successfully generated and visualized in the Wazuh dashboard.

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
