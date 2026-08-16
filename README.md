# Home-SIEM-Lab
Wazuh Agents for home LAN
# 🛡️ Enterprise SIEM & EDR Implementation: Wazuh Security Operations

## 📖 Project Overview & Narrative

### What I Did
In this project, I engineered and deployed a functional **Security Information and Event Management (SIEM)** and **Endpoint Detection and Response (EDR)** environment from the ground up using **Wazuh**. 

* **Central SIEM Management:** Provisioned a hardened Linux-based Wazuh Manager node within a virtualized environment to ingest, correlate, and analyze security telemetry.
* **Endpoint Enrollment:** Configured and deployed the Wazuh Agent on a Windows 11 endpoint host, establishing secure communications over TCP ports 1514 (telemetry) and 1515 (enrollment).
* **Real-Time Monitoring Modules:** Configured **File Integrity Monitoring (FIM)** to audit sensitive system directories in real time, and accelerated the automated **Vulnerability Detector** engine to perform hourly cross-referencing of installed software against National Vulnerability Database (NVD) CVE records.
* **System & Administrative Hardening:** Executed administrative credential updates via direct OpenSearch Indexer security tooling, resolved complex network interface binding issues, and mapped security telemetry directly against the **MITRE ATT&CK Enterprise Framework**.

---

## Technical Infrastructure Baseline

| Host / Node | Operating System | Role / Component | Specifications & Service Ports |
| :--- | :--- | :--- | :--- |
| **Wazuh Manager** | RHEL / AlmaLinux OVA | Central SIEM / Indexer / Dashboard | VirtualBox VM, HTTPS (Port 443), Ingestion (1514/1515 TCP) |
| **Primary Workstation** | Windows 11 Pro | Monitored Client Endpoint | Wazuh Agent Daemon (`wazuh`), Syscollector, FIM |
| **Server Host (Planned)** | Windows 11 Pro (Inspiron 5310) | 24/7 Decoupled Server Node | Docker Desktop (WSL2), Hyper-V, External Storage |


### Why I Did It
The primary objective of this project was to transition from theoretical cybersecurity knowledge to hands-on, enterprise-grade Security Operations Center (SOC) engineering. 

1. **Gaining Centralized Visibility:** Disparate endpoint logs are ineffective without centralized correlation. Building this SIEM layer provided a "single pane of glass" to monitor endpoint health, security events, and unauthorized system modifications in real time.
2. **Proactive Risk Reduction:** Rather than passively waiting for security incidents, configuring active vulnerability scanning allowed me to identify unpatched software dependencies and prioritize system updates before exploitation.
3. **Real-World Engineering Experience:** Setting up this environment provided practical experience in network interface troubleshooting, Windows service management, administrative credential rotation, and SIEM tuning—skills directly applicable to enterprise SOC and SysAdmin roles.

---

## Architecture Flow

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


---
### 🚀 Future Roadmap & Planned Configurations

This initial deployment represents Phase 1 of a multi-tier infrastructure strategy. Planned upcoming projects include:

* **24/7 Server Decoupling & Containerization (Phase 2):** 
  Migrate the central Wazuh Manager stack from a local workstation VM to a dedicated, always-on server host (Dell Inspiron 13 5310) running **Docker Desktop via WSL2**. This ensures continuous 24/7 security ingestion regardless of the client workstation’s power state.
* **Network Segmentation & Perimeter Isolation:**
  Implement managed VLAN subnets to strictly isolate trusted workstations, server management interfaces, and smart home/IoT devices (e.g., IP cameras). Deploy an **Nginx Reverse Proxy** to handle edge traffic routing, SSL/TLS termination, and secure dashboard access over port 443.
* **Hybrid Cloud Telemetry Ingestion (AWS EC2):**
  Provision remote Linux instances in **Amazon Web Services (AWS EC2)** to host public-facing game servers and applications. Enroll these cloud nodes into the primary Wazuh Manager via encrypted tunnels to centralize hybrid-cloud telemetry.
* **LLM & Model Context Protocol (MCP) Integration:**
  Deploy a lightweight local Model Context Protocol (MCP) middleware server to allow AI tools to securely query SIEM API endpoints for automated log analysis and threat report generation.
