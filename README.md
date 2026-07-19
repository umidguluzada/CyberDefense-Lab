# Cyber Security Lab: Blue Team Infrastructure & Network Defense

This laboratory project demonstrates the creation of an integrated "Blue Team" infrastructure, focusing on centralized network security, Intrusion Detection/Prevention Systems (IDS/IPS), and Security Information and Event Management (SIEM) using **pfSense**.

---

## 1. Environment Configuration and Network Architecture

The laboratory environment is designed in a virtualized setting (VMware) using network segmentation principles:

### Virtual Machine Specifications
*   **pfSense (Firewall):** 1 GB RAM, 4 vCPU | Network Adapters: Bridged + VMnet1 + VMnet2
*   **Ubuntu Server (ELK):** 192.168.10.55 | 2-4 GB RAM (with minimum resource allocation for Elasticsearch)
*   **Windows (Victim):** 4 GB RAM, 4 vCPU | Network Adapter: VMnet1 (LAN)
*   **Kali Linux (Attacker):** 2 GB RAM, 4 vCPU | Network Adapter: VMnet2 (OPT1/Attack)

### Network Interfaces and IP Allocation
To ensure security, the attacker and victim machines are placed in separate subnets:

| Device / Interface | IP Address / Network | Purpose |
| :--- | :--- | :--- |
| **pfSense WAN** | 192.168.1.69/24 | External traffic interface |
| **pfSense LAN1 (LAN)** | 192.168.10.1/24 | Internal secure network |
| **pfSense OPT1 (ATTACK)** | 192.168.20.1/24 | Isolated attack network |
| **Ubuntu Server (ELK)** | 192.168.10.55 | SIEM monitoring center |
| **Windows (Victim)** | 192.168.10.50 | Target system in the internal network |
| **Kali Linux (Attacker)** | 192.168.20.50 | Simulated external attacker |

---

## 2. Security and Defense Mechanisms

### Suricata IDS/IPS Configuration
*   **Operational Mode:** Suricata is enabled in `Inline IPS` (netmap) mode to block threats. For stability, the VM adapter type was set to `E1000` with `Promiscuous mode` enabled.
*   **Rulesets:** Deep Packet Inspection (DPI) is applied using `emerging-exploit`, `emerging-scan`, and `emerging-malware` rule sets.
*   **Custom Rule:** If 20+ SYN connection attempts towards the target are detected within 10 seconds, an "Nmap Scan" alert is triggered, and the attacker's IP (`192.168.20.50`) is automatically added to the "Block Offenders" list.

### SIEM (Elastic Stack) and Log Management
*   **Log Forwarding:** pfSense logs are forwarded via `Remote Syslog` to the server at `192.168.10.55` on port 5140. Suricata logs are exported in `EVE JSON` format and ingested into ELK via `Filebeat`.
*   **Windows Audit Policy:** "Audit Logon Events" (Event ID 4625) was activated on the Windows machine to enable tracking of failed login (Brute-Force) attempts within the SIEM.

---

## 3. Testing, Validation, and Troubleshooting

Key challenges and implemented solutions during the lab setup include:

*   **Issue:** Attacks were not visible in the firewall or Suricata.
    *   *Solution:* The attacker (Kali) and victim (Windows) were on the same subnet, preventing traffic from passing through the firewall. Kali was moved to a separate isolated interface (`OPT1`).
*   **Issue:** Brute-force attacks not appearing in the SIEM.
    *   *Solution:* Audit policies were enabled in Windows local security policy to ensure `Event ID 4625` logs were generated.
*   **Issue:** ELK server crashing during data influx.
    *   *Solution:* RAM allocation for the host machine and the Java Virtual Machine (JVM) for Elasticsearch was increased to 2-4 GB.

---

## 4. Project Verification Scenarios
1.  **Network Connectivity:** Routing was verified via pfSense, confirming inter-network traffic and isolation rules.
2.  **Attack Simulation:** `nmap -sS` scans from Kali Linux were instantly detected by pfSense, and the attacker's IP was successfully blocked.
3.  **Monitoring:** Brute-force simulations using `Hydra` and Suricata alerts were successfully monitored in real-time via the Kibana dashboard.

---
*Note: This project is a complete cybersecurity laboratory environment built strictly for educational and practical purposes.*
