# Network-Based-Intrusion-Detection-System-
# Network-Based Intrusion Detection System (NIDS)

### Design, Deployment & Live Threat Monitoring with Suricata

![Cybersecurity](https://img.shields.io/badge/Domain-Cybersecurity-red)
![Suricata](https://img.shields.io/badge/IDS-Suricata-orange)
![Linux](https://img.shields.io/badge/OS-Ubuntu%20Linux-purple)
![Nmap](https://img.shields.io/badge/Testing-Nmap-blue)
![Wireshark](https://img.shields.io/badge/Analysis-Wireshark-blue)
![Status](https://img.shields.io/badge/Project-Completed-brightgreen)

## Project Overview

This project demonstrates the design, deployment, configuration, and testing of a **Network-Based Intrusion Detection System (NIDS)** using **Suricata**.

The system was developed in an isolated virtual laboratory to continuously monitor network traffic, detect suspicious activity, generate real-time alerts, and implement an automated response mechanism.

The project demonstrates a practical SOC-style workflow:

**Network Traffic → Packet Inspection → Detection Rules → Alert Generation → Analysis → Automated Response → Visualization**

The completed system was capable of capturing live traffic, applying community and custom detection rules, generating alerts, responding to repeated malicious activity, and presenting security events through a visualization layer.

---

## Objectives

The primary objectives of this project were to:

* Deploy a Network-Based Intrusion Detection System.
* Configure Suricata for continuous network monitoring.
* Implement community-maintained and custom detection rules.
* Detect reconnaissance and suspicious network activity.
* Generate and investigate real-time security alerts.
* Implement automated response against repeated malicious activity.
* Visualize detected security events.
* Validate the IDS using controlled network scanning activity.

---

## Lab Architecture

The project was implemented inside an isolated virtual lab environment consisting of:

```text
                    ┌──────────────────────┐
                    │   Attacker / Tester  │
                    │                      │
                    │       Nmap           │
                    └──────────┬───────────┘
                               │
                               │ Network Traffic
                               ▼
                    ┌──────────────────────┐
                    │    Suricata NIDS     │
                    │                      │
                    │  Packet Inspection   │
                    │  Detection Rules     │
                    │  Alert Generation    │
                    └──────────┬───────────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
                ▼              ▼              ▼
        ┌────────────┐  ┌─────────────┐  ┌──────────────┐
        │ Suricata   │  │  Fail2Ban   │  │ Filebeat /  │
        │ Alert Logs │  │  + iptables │  │ ELK / EveBox │
        └────────────┘  └─────────────┘  └──────────────┘
                               │
                               ▼
                       Automated Blocking
```

The architecture follows a real-world NIDS deployment model where a monitoring sensor observes traffic entering or leaving a protected network segment.

---

## Technologies & Tools

| Tool                    | Purpose                                |
| ----------------------- | -------------------------------------- |
| **Suricata**            | Core IDS/IPS engine                    |
| **Ubuntu Server 22.04** | NIDS sensor operating system           |
| **Nmap**                | Controlled reconnaissance and scanning |
| **Wireshark**           | Packet capture and traffic analysis    |
| **Fail2Ban**            | Automated response mechanism           |
| **iptables**            | Firewall-based IP blocking             |
| **Filebeat**            | Log collection and forwarding          |
| **ELK Stack / EveBox**  | Alert visualization and analysis       |

---

## Installation & Deployment

### 1. Update the system

```bash
sudo apt update
```

### 2. Install Suricata

```bash
sudo apt install suricata -y
```

### 3. Identify the network interface

```bash
ip addr
```

or:

```bash
ip route
```

Example interface:

```text
ens33
```

### 4. Configure Suricata

Open the configuration file:

```bash
sudo nano /etc/suricata/suricata.yaml
```

Configure the appropriate network interface under the `af-packet` section.

Example:

```yaml
af-packet:
  - interface: ens33
```

### 5. Test the configuration

Before starting the service, validate the configuration:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

A successful configuration test confirms that Suricata can load its configuration without errors.

### 6. Start Suricata

```bash
sudo systemctl enable suricata
sudo systemctl start suricata
```

Check the service:

```bash
sudo systemctl status suricata
```

Monitor alerts:

```bash
sudo tail -f /var/log/suricata/fast.log
```

---

## Detection Rules

The project used both **Emerging Threats community rules** and custom Suricata rules.

### Update the rule set

```bash
sudo suricata-update
```

### Custom ICMP Detection Rule

A custom rule was created to identify potential ICMP reconnaissance activity:

```text
alert icmp any any -> 192.168.91.4 any
(msg:"Possible ICMP Ping Sweep/Flood to 192.168.91.4";
itype:8;
threshold:type both, track by_src, count 5, seconds 10;
sid:1000011;
rev:1;)
```

This rule looks for repeated ICMP echo requests from a source within a defined time window.

### Suspicious Executable Transfer Detection

A second rule was created to detect a suspicious executable file transfer:

```text
alert http any any -> $192.168.91.4 any
(msg:"Suspicious EXE Download Detected";
flow:established,to_client;
file_data;
content:"MZ";
sid:1000002;
rev:1;)
```

The rule looks for the `MZ` executable signature within HTTP file data.

---

## Detection Validation

To validate the NIDS, a controlled Nmap SYN scan was performed against the protected host.

```bash
nmap -sS 192.168.91.4
```

The scan generated network traffic that was inspected by Suricata.

### Expected Workflow

```text
Nmap SYN Scan
      ↓
Network Packets
      ↓
Suricata Packet Inspection
      ↓
Detection Rule Matching
      ↓
Security Alert
      ↓
Suricata Log
      ↓
Analyst Investigation
```

The project successfully detected and logged the controlled reconnaissance activity in real time.

---

## Automated Response

The project implemented two response levels.

### Passive Response

Suricata records detected events in its alert logs for analyst investigation.

### Active Response

**Fail2Ban + iptables** were configured to respond to repeated malicious activity.

Installation:

```bash
sudo apt install fail2ban -y
```

The Fail2Ban configuration was designed to monitor the Suricata alert log and trigger a firewall block when the configured threshold was exceeded.

Example firewall action:

```bash
iptables -I INPUT -s <OFFENDING_IP> -j DROP
```

This demonstrates the transition from a traditional **IDS** model toward an automated defensive response.

---

## Security Event Visualization

Filebeat was used as the log-shipping component for visualization.

Installation:

```bash
sudo apt install filebeat -y
```

Enable the Suricata module:

```bash
sudo filebeat modules enable suricata
```

Set up Filebeat:

```bash
sudo filebeat setup
```

Start the service:

```bash
sudo systemctl start filebeat
```

The visualization layer was designed to provide visibility into:

* Alert volume over time
* Top source IP addresses
* Alert severity
* Detected security events
* Network intrusion trends

This provides an analyst-friendly view similar to the monitoring workflow used in a Security Operations Center (SOC).

---

## Challenges & Solutions

| Challenge                      | Resolution                          |
| ------------------------------ | ----------------------------------- |
| False positives                | Tuned detection thresholds          |
| Suricata not capturing traffic | Corrected the `af-packet` interface |
| Outdated detection rules       | Regularly ran `suricata-update`     |
| Large volume of alerts         | Applied rule tuning and thresholds  |
| Need for automated response    | Integrated Fail2Ban and iptables    |

---

## Security Concepts Demonstrated

This project demonstrates practical knowledge of:

* Network Intrusion Detection
* Network Traffic Monitoring
* Signature-Based Detection
* Custom IDS Rule Development
* Network Reconnaissance Detection
* ICMP Monitoring
* HTTP Traffic Inspection
* Security Alert Generation
* Log Analysis
* Automated Incident Response
* Firewall-Based Blocking
* Packet Analysis
* SOC Monitoring
* Security Event Visualization
* Detection Rule Tuning
* Incident Response

---

## Project Results

The implementation successfully demonstrated the complete NIDS lifecycle:

```text
                ┌─────────────────┐
                │ Network Traffic │
                └────────┬────────┘
                         ↓
                ┌─────────────────┐
                │ Traffic Capture │
                └────────┬────────┘
                         ↓
                ┌─────────────────┐
                │ Rule Evaluation │
                └────────┬────────┘
                         ↓
                ┌─────────────────┐
                │ Alert Generation│
                └────────┬────────┘
                         ↓
                ┌─────────────────┐
                │ Threat Analysis │
                └────────┬────────┘
                         ↓
                ┌─────────────────┐
                │ Automated       │
                │ Response        │
                └────────┬────────┘
                         ↓
                ┌─────────────────┐
                │ Visualization   │
                └─────────────────┘
```

The project demonstrates the same fundamental workflow used in professional network defense operations:

**Collect → Detect → Alert → Analyze → Respond → Tune**

---

## Future Improvements

Future versions of the project could include:

* Deploying Suricata in full inline IPS mode.
* Integrating external threat-intelligence feeds.
* Enriching alerts with IP reputation information.
* Implementing automated email/SMS notifications.
* Integrating Suricata with a SIEM such as Splunk.
* Adding additional custom detection signatures.
* Simulating brute-force attacks in the isolated lab.
* Detecting malware command-and-control traffic.
* Building automated incident-response playbooks.
* Integrating MITRE ATT&CK mapping into alerts.

---

## Suggested Repository Structure

```text
NIDS-Suricata/
│
├── README.md
│
├── config/
│   └── suricata.yaml
│
├── rules/
│   └── custom.rules
│
├── scripts/
│   └── monitoring.sh
│
├── logs/
│   └── .gitkeep
│
├── screenshots/
│   ├── suricata-status.png
│   ├── nmap-detection.png
│   ├── alerts.png
│   └── dashboard.png
│
├── documentation/
│   └── NIDS_Project_Report.pdf
│
└── LICENSE
```

> **Important:** Do not upload real IP addresses, credentials, API keys, tokens, private network information, or sensitive log data to a public GitHub repository.

---

## Key Learning Outcomes

Through this project, I developed practical experience in:

1. Deploying and configuring Suricata.
2. Monitoring live network traffic.
3. Writing custom Suricata detection rules.
4. Detecting reconnaissance activity.
5. Validating IDS functionality using Nmap.
6. Analyzing Suricata alerts.
7. Automating defensive responses with Fail2Ban and iptables.
8. Forwarding security logs for visualization.
9. Troubleshooting IDS configuration and network interfaces.
10. Applying SOC-style detection and response workflows.

---

## Author

### Adabanija Toheeb

**Cybersecurity Professional | Penetration Tester | SOC Analyst | DFIR**

This project was developed as part of my practical cybersecurity portfolio to demonstrate hands-on capability in **network security monitoring, intrusion detection, threat detection, incident response, and security automation**.

---

## References

* Suricata Official Documentation
* Emerging Threats Open Rule Set
* Elastic / ELK Stack Documentation
* Fail2Ban Documentation

---

## Disclaimer

This project was developed and tested in an **isolated virtual laboratory environment**. All scanning and simulated malicious activity was performed for authorized security testing and educational purposes.

Do not use the techniques demonstrated in this repository against systems or networks without explicit authorization.

---

**If you find this project useful, feel free to explore the repository and connect with me for cybersecurity collaboration and learning.**
