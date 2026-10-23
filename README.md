# 🛡️ Enterprise SIEM & EDR Implementation: Wazuh Security Operations

## 📖 Project Overview & Narrative

### What I Did
In this project, I engineered and deployed a functional **Security Information and Event Management (SIEM)** and **Endpoint Detection and Response (EDR)** environment from the ground up using **Wazuh**. 

* **Central SIEM Management:** Provisioned a hardened Linux-based Wazuh Manager node within a virtualized environment to ingest, correlate, and analyze security telemetry.
* **Endpoint Enrollment:** Configured and deployed the Wazuh Agent on a Windows 11 endpoint host, establishing secure communications over TCP ports 1514 (telemetry) and 1515 (enrollment).
* **Real-Time Monitoring Modules:** Configured **File Integrity Monitoring (FIM)** to audit sensitive system directories in real time, and accelerated the automated **Vulnerability Detector** engine to perform hourly cross-referencing of installed software against National Vulnerability Database (NVD) CVE records.
* **System & Administrative Hardening:** Executed administrative credential updates via direct OpenSearch Indexer security tooling, resolved complex network interface binding issues, and mapped security telemetry directly against the **MITRE ATT&CK Enterprise Framework**.

---

### Why I Did It
The primary objective of this project was to transition from theoretical cybersecurity knowledge to hands-on, enterprise-grade Security Operations Center (SOC) engineering. 

1. **Gaining Centralized Visibility:** Disparate endpoint logs are ineffective without centralized correlation. Building this SIEM layer provided a "single pane of glass" to monitor endpoint health, security events, and unauthorized system modifications in real time.
2. **Proactive Risk Reduction:** Rather than passively waiting for security incidents, configuring active vulnerability scanning allowed me to identify unpatched software dependencies and prioritize system updates before exploitation.
3. **Real-World Engineering Experience:** Setting up this environment provided practical experience in network interface troubleshooting, Windows service management, administrative credential rotation, and SIEM tuning—skills directly applicable to enterprise SOC and SysAdmin roles.

---

## 🛠️ Technical Infrastructure Baseline

| Host / Node | Operating System | Role / Component | Specifications & Service Ports |
| :--- | :--- | :--- | :--- |
| **Wazuh Manager** | RHEL / AlmaLinux OVA | Central SIEM / Indexer / Dashboard | VirtualBox VM, HTTPS (Port 443), Ingestion (1514/1515 TCP) |
| **Primary Workstation** | Windows 11 Pro | Monitored Client Endpoint | Wazuh Agent Daemon (`wazuh`), Syscollector, FIM |
| **Server Host (Planned)** | Windows 11 Pro (Inspiron 5310) | 24/7 Decoupled Server Node | Docker Desktop (WSL2), Hyper-V, External Storage |

---

## 🏗️ Architecture Flow

```text
┌───────────────────────────────────────────────────────────────┐
│                    Wazuh Central Manager                       │
│  • Web Dashboard & REST API (Port 443)                        │
│  • NVD / CVE Feed Correlation (Accelerated 1-Hour Cycle)      │
│  • Decoupled Ingestion Pipeline (TCP 1514 / 1515)             │
└───────────────────────────────┬───────────────────────────────┘
                                │
                                ▼
┌───────────────────────────────────────────────────────────────┐
│                  Enrolled Windows Endpoint                    │
│  • Telemetry Daemon (`wazuh` background service)             │
│  • Real-Time File Integrity Monitoring (Syscheck)             │
│  • System Collector (`syscollector` OS & Software Inventory)  │
└───────────────────────────────────────────────────────────────┘
```

---

## 🔧 Engineering Resolutions & System Hardening

### 1. Web UI Addressing & TLS Connectivity
- **Issue:** Web Dashboard unreachable via standard web browser URL entry; initial attempts resulted in routing timeouts.
- **Root Cause:** Inadvertent inclusion of CIDR prefix notation (`/24`) in URL formatting and targeting network broadcast addresses (`.255`).
- **Resolution:** Evaluated interface bindings via `ip -4 addr show`, confirmed the exact IPv4 host address, and verified direct TLS communications over port 443 (`https://<MANAGER_IP>:443`).

### 2. Automated Vulnerability Detector Optimization
- **Configuration:** Updated `/var/ossec/etc/ossec.conf` to optimize CVE database synchronization:
  ```xml
  <vulnerability-detection>
    <enabled>yes</enabled>
    <index-status>yes</index-status>
    <feed-update-interval>1h</feed-update-interval>
  </vulnerability-detection>
  ```
- **Validation:** Confirmed Windows endpoint `syscollector` inventory logging via `C:\Program Files (x86)\ossec-agent\ossec.conf` and cycled daemon state via PowerShell:
  ```powershell
  NET STOP Wazuh
  NET START Wazuh
  ```

### 3. Windows Service Key Identification & Ingestion
- **Issue:** Windows Service Control Manager returned invalid service name errors when executing standard execution strings (`wazuhsvc`).
- **Resolution:** Enumerated active Windows services via elevated PowerShell to locate the registered service key (`wazuh`) and initialized log ingestion:
  ```powershell
  Get-Service | Where-Object { $_.DisplayName -like "*Wazuh*" }
  NET START Wazuh
  ```

### 4. Administrative Security Indexer Pathing
- **Issue:** CLI credential rotation tool failed to execute due to altered pathing in Wazuh 4.x distributed indexer setups.
- **Resolution:** Bypassed the standard alias scripts and executed the OpenSearch indexer security tool directly from the absolute system path:
  ```bash
  sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh -u admin -p '<STRONG_PASSWORD>'
  sudo systemctl restart wazuh-dashboard
  ```

---

## 📊 Active Security Posture & Operational Modules

- **Log Audit & Security Events:** Ingesting real-time Windows Event Logs across Low and Medium severity levels.
- **Continuous Vulnerability Detection:** Ingestion pipeline actively matching installed endpoint software packages against NVD database records on an hourly update cycle.
- **File Integrity Monitoring (FIM):** Active syscheck rules auditing directory changes in real time.
- **MITRE ATT&CK Mapping:** Dashboard security visualizations aligned with the MITRE ATT&CK enterprise threat taxonomy.

---

## Gateway Hardening & Client DNS Integration

The primary gateway was hardened and client devices were moved to the 5310-hosted Pi-hole DNS service. Exact private addresses and device MAC addresses are intentionally omitted from this public repository; use local router and host settings for those values.

### Gateway controls

| Area | Configured state | Security purpose |
|---|---|---|
| Address management | DHCP static assignment for the 5310 host | Keeps the DNS host address stable across reboots. |
| Wi-Fi security | WPA Personal Transition Mode | Allows WPA3-SAE clients while retaining WPA2-PSK compatibility. |
| Protected Management Frames | Capable / Optional | Supports PMF for WPA3 clients without excluding WPA2-only clients. |
| WPS | Disabled | Removes WPS PIN-based onboarding exposure. |
| Firewall | Medium | Applies the gateway's selected inbound and outbound filtering. |
| WAN blocking | Enabled | Suppresses responses to unsolicited WAN probes. |
| Fragmented packets | Blocked | Reduces exposure to fragmented-packet evasion and malformed-packet attacks. |
| Port scan detection | Enabled | Detects and blocks rapid probing activity. |
| IP flood detection | Enabled | Applies gateway flood protections. |
| UPnP | Disabled | Prevents clients from automatically creating inbound port mappings. |
| SIP ALG | Disabled | Avoids rewriting SIP traffic in ways that can disrupt voice and real-time media. |
| Firewall logging | Blocked traffic and configuration events | Captures security-relevant events while limiting permitted-traffic log volume. |

### Client onboarding and DNS

The Nintendo Switch initially rejected the network under mandatory WPA3/PMF settings. WPA Personal Transition Mode restored compatibility, allowing the Switch to connect with WPA2-PSK/AES. The Switch and other clients use the 5310 host's Pi-hole DNS service; the actual LAN address is not published here. Keep client DNS pointed at the reserved host address, and leave client IP, subnet mask, and gateway assignment on DHCP unless the router administrator specifies otherwise.

The Pi-hole container publishes DNS on the Docker host's LAN interfaces. The host must remain powered on and connected for clients to resolve DNS through Pi-hole. The Pi-hole upstream resolver configuration is separate from client DNS settings.

### Security control mapping

| Boundary / capability | NIST SP 800-53 Rev. 5 | ISO/IEC 27001:2022 | Evidence |
|---|---|---|---|
| Edge gateway | SC-7 | A.8.20 | WAN blocking, port scan detection, and firewall filtering. |
| Wireless perimeter | IA-5 | A.8.5 | WPS disabled; WPA transition mode enabled. |
| Address management | CM-8 | A.8.9 | DHCP static assignment for the 5310 DNS host. |
| DNS filtering | SI-3 | A.8.23 | Client DNS routed through Pi-hole. |
| Audit logging | AU-2 / AU-6 | A.8.15 | Blocked-traffic and configuration-event logging. |

These mappings document the intended control alignment and do not constitute an independent compliance assessment.

---
## Related Docker homelab project

The companion Docker Desktop homelab is maintained separately in `Home-Docker-Lab`.

- Docker services and Pi-hole DNS: `Home-Docker-Lab`
- Wazuh VirtualBox SIEM and Windows endpoint: this repository

Keep credentials, API keys, agent keys, and private network details out of both repositories.

* **24/7 Server Decoupling & Containerization (Phase 2):** 
  Migrate the central Wazuh Manager stack from a local workstation VM to a dedicated, always-on server host (Dell Inspiron 13 5310) running **Docker Desktop via WSL2**. This ensures continuous 24/7 security ingestion regardless of the client workstation’s power state.
* **Network Segmentation & Perimeter Isolation:**
  Implement managed VLAN subnets to strictly isolate trusted workstations, server management interfaces, and smart home/IoT devices (e.g., IP cameras). Deploy an **Nginx Reverse Proxy** to handle edge traffic routing, SSL/TLS termination, and secure dashboard access over port 443.
* **Hybrid Cloud Telemetry Ingestion (AWS EC2):**
  Provision remote Linux instances in **Amazon Web Services (AWS EC2)** to host public-facing game servers and applications. Enroll these cloud nodes into the primary Wazuh Manager via encrypted tunnels to centralize hybrid-cloud telemetry.
* **LLM & Model Context Protocol (MCP) Integration:**
  Deploy a lightweight local Model Context Protocol (MCP) middleware server to allow AI tools to securely query SIEM API endpoints for automated log analysis and threat report generation.
