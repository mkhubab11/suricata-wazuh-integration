## Suricata Integration with Wazuh

Network Intrusion Detection through SIEM and IDS/IPS Integration

---
## 📌 Project Overview

This project demonstrates the integration of Suricata, an open-source Network Intrusion Detection and Prevention System (IDS/IPS), with Wazuh, a Security Information and Event Management (SIEM) platform.

The integration forwards Suricata is eve.json alert logs to the Wazuh agent, allowing network security events to be centrally collected, parsed, searched, and visualized in the Wazuh dashboard alongside host-based telemetry.

The project validates the detection pipeline by attacker machine Kali Linux against an Ubuntu endpoint.

**Project type**: Cybersecurity / SOC Lab
**Focus**: SIEM Administration, Network Intrusion Detection, Log Integration, and Threat Hunting

---
## 🎯 Objectives

* Deploy and configure a Wazuh manager and dashboard.
* Register an Ubuntu endpoint as a monitored Wazuh agent.
* Install and configure Suricata on the Ubuntu endpoint.
* Configure the Emerging Threats open ruleset.
* Forward Suricata's `eve.json` alert log to Wazuh.
* Visualize and analyze Suricata alerts in the Wazuh Threat Hunting module.
* Examine structured alert fields and network traffic metadata.
---
## 🏗️ Lab Architecture

                    ┌──────────────────────────┐
                    │     Kali Linux           │
                    │     Attacker Machine     │
                    │                          │
                    │  Nmap Service Scan       │
                    └────────────┬─────────────┘
                                 │
                                 │ Network Traffic
                                 ▼
                    ┌──────────────────────────┐
                    │     Ubuntu Endpoint      │
                    │      Ubuntu 20.04        │
                    │                          │
                    │  ┌────────────────────┐  │
                    │  │     Suricata       │  │
                    │  │     IDS/IPS        │  │
                    │  └─────────┬──────────┘  │
                    │            │             │
                    │       eve.json           │
                    │            │             │
                    │  ┌─────────▼──────────┐  │
                    │  │   Wazuh Agent      │  │
                    │  └─────────┬──────────┘  │
                    └────────────┼─────────────┘
                                 │
                                 │ Log Forwarding
                                 ▼
                    ┌──────────────────────────┐
                    │    Wazuh Manager         │
                    │    + Wazuh Dashboard     │
                    │                          │
                    │  Alert Parsing           │
                    │  Threat Hunting          │
                    │  Log Analysis            │
                    └──────────────────────────┘
---
## 🛠️ Tools and Environment

| Component           | Details                       |
| ------------------- | ----------------------------- |
| SIEM Platform       | Wazuh Manager + Dashboard     |
| Network IDS/IPS     | Suricata 6.0.8                |
| Ruleset             | Emerging Threats Open Ruleset |
| Monitored Endpoint  | Ubuntu 20.04.4 LTS            |
| Wazuh Agent         | ubuntu_agent                  |
| Attacker Machine    | Kali Linux                    |
| Attack Simulation   | Nmap service/version scan     |
| Log Source          | /var/log/suricata/eve.json    |
| Network Interface   | ens33                         |
| Configuration Files | suricata.yaml, ossec.conf     |

---
 ## ⚙️ Implementation
**1. Install the Wazuh Agent on Ubuntu**

Install and register the Wazuh agent on the Ubuntu endpoint, then verify that the agent is active and connected to the Wazuh manager.
<p>
<img width="1331" height="632" alt="Screenshot 2026-09-16 185303" src="https://github.com/user-attachments/assets/c5c9026f-18ba-408d-86f2-f1138a07b5cd" /> 

  
The endpoint should appear in the Wazuh dashboard's agent list.
 
<img width="1330" height="472" alt="Screenshot 2026-09-17 115955" src="https://github.com/user-attachments/assets/466a2f6a-e27e-4252-867f-cfe5b4f4498f" />


```bash
sudo systemctl status wazuh-agent
```


<img width="728" height="410" alt="Screenshot 2026-09-17 123415" src="https://github.com/user-attachments/assets/b9d2a0b1-68fe-4427-a154-4cbda5e3b3ca" />
---
**2. Install Suricata**

Add the official Suricata stable repository and install Suricata:
```bash
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata -y
```
<img width="728" height="290" alt="Screenshot 2026-09-07 130644" src="https://github.com/user-attachments/assets/eb6d902f-caeb-4d29-9387-b85c7f446a09" />


Start and verify the Suricata service:
```bash
sudo systemctl start suricata
sudo systemctl status suricata
```
<img width="737" height="259" alt="Screenshot 2026-09-07 130550" src="https://github.com/user-attachments/assets/3158845c-7e69-4b4e-9689-47b5551ea3c5" />

---
**3. Download the Emerging Threats Ruleset**

Download and extract the Emerging Threats open ruleset:
```bash
cd /tmp/
curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
sudo tar -xvzf emerging.rules.tar.gz
sudo mkdir -p /etc/suricata/rules
sudo mv rules/*.rules /etc/suricata/rules/
```

> **Note**: The commands above follow the project report. For a production environment, review file permissions and avoid unnecessarily broad permissions such as 777.
---
**4. Configure Suricata**

Edit the Suricata configuration file:

```bash
sudo nano /etc/suricata/suricata.yaml
```


Configure the network and ruleset:

```yaml
HOME_NET: "UBUNTU_IP"
EXTERNAL_NET: "any"

default-rule-path: /etc/suricata/rules

rule-files:
  - "*.rules"

stats:
  enabled: yes

af-packet:
  - interface: ens33
```

Replace <UBUNTU_IP> with the actual IP address or appropriate network configuration for your lab.

Restart Suricata:
```bash
sudo systemctl restart suricata
```
---
**5. Integrate Suricata with Wazuh**

Edit the Wazuh agent configuration:
```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add the following localfile configuration inside the existing <ossec_config> element:

xml

```bash
<!-- Suricata integration with Wazuh -->
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```
<img width="486" height="136" alt="Screenshot 2026-09-07 150221" src="https://github.com/user-attachments/assets/c4de7d7d-5301-4df7-bcca-6ea2d7725983" />

Restart the Wazuh agent:

```bash
sudo systemctl restart wazuh-agent
```

This configuration enables the Wazuh agent to collect Suricata's JSON event log.
---
## 🧪 Attack Emulation and Detection

To validate the integration, an Nmap service/version scan was launched from a Kali Linux machine against the Ubuntu endpoint.

Run the scan from the authorized lab attacker machine:
<img width="634" height="270" alt="Screenshot 2026-09-16 165535" src="https://github.com/user-attachments/assets/15909109-826d-4d20-a532-c2f642296dce" />

```bash 
nmap -sS -sV <ubuntu_ip>
```

Replace <ubuntu_ip> with the IP address of the Ubuntu endpoint.

Detection Workflow

1. Kali Linux launches the Nmap scan.
2. Suricata monitors the network traffic.
3. Suricata matches the traffic against its detection rules.
4. Alerts are written to eve.json.
5. The Wazuh agent collects the JSON events.
6. Wazuh parses and displays the alerts in the dashboard.
7. Analysts inspect the events through Threat Hunting and Document Details.
---
## 🔎 Viewing Alerts in Wazuh

To inspect the generated alerts:

1. Open the Wazuh Dashboard.
2. Navigate to the Threat Hunting module.
3. Filter events related to the monitored Ubuntu agent.
4. Open individual alerts to inspect their details.
5. Review fields such as:

   * Alert classification
   * Source and destination IP addresses
   * Source and destination ports
   * Protocol
   * Flow information
   * HTTP metadata
   * Suricata signature ID
   * Rule group and severity
---
## 📊 Key Findings

The project report documented the following findings:

* Suricata detected the Nmap scan using the Emerging Threats signature ET SCAN Nmap Scripting Engine User-Agent Detected.
* The associated signature ID was 2009358.
* Wazuh successfully ingested and parsed Suricata's JSON logs.
* Structured fields included data.alert.category, data.flow, and data.http.
* Additional reconnaissance-related probes were flagged, including suspicious connections involving PostgreSQL, MSSQL, Oracle, MySQL, and VNC ports.
* The alerts were classified at rule level 3, representing informational/low-severity reconnaissance activity in the documented lab.
* Alerts retained network context, including source/destination IPs, ports, protocol information, and byte counts.

>**Important**: These results represent the behavior observed in this specific lab environment and ruleset configuration. Detection results may vary depending on Suricata versions, rules, network configuration, and traffic.
---
## 🔐 Ethical and Legal Disclaimer

This project was conducted in a controlled cybersecurity lab environment for educational and defensive security purposes.

Only perform scans and security testing against systems you own or have explicit authorization to test. Do not scan public or third-party systems without permission.

---
## 🚀 Future Improvements

Potential extensions for this project include:

* Adding automated response or active-response capabilities.
* Integrating additional network traffic analysis tools.
* Creating custom Suricata detection rules.
* Developing custom Wazuh decoders and rules.
* Building dashboards for attack trends and alert severity.
* Adding File Integrity Monitoring and endpoint security telemetry.
* Testing additional authorized attack simulations.
* Integrating the project with a broader SOC monitoring workflow.
---
## 👨‍💻 Author

**Muhammad Khubab**

Domain: SIEM Administration & Network Intrusion Detection (IDS)

LinkedIn: [Muhammad Khubab](https://www.linkedin.com/in/muhammad-khubab-475046204/)

---
## 📄 License

This project is intended for educational and portfolio purposes. You may add an open-source license such as the MIT License if you want others to reuse and modify the project.
