# 🏗️ Project Epic: Container-Based (Docker) Runtipi Home Server Architecture

> **More Than an Internship Notebook: A Production-Grade Self-Hosted Infrastructure Case Study Built From Scratch**

[![Status](https://img.shields.io/badge/Status-Active%20%26%20Operational-success?style=for-the-badge)]()
[![Platform](https://img.shields.io/badge/Platform-WSL2%20%7C%20Ubuntu-orange?style=for-the-badge&logo=linux)]()
[![Orchestration](https://img.shields.io/badge/Orchestration-Docker%20Compose-blue?style=for-the-badge&logo=docker)]()
[![Core](https://img.shields.io/badge/Core-Runtipi%20v4.10.1-purple?style=for-the-badge)]()

---

## 👤 Project Owner

**Alemdar** — Within the scope of a 19-day internship program, abandoning the traditional "observation log" format, an end-to-end self-hosted ecosystem was built that brings together the disciplines of *Infrastructure Engineering*, *Network Architecture*, and *Site Reliability Engineering (SRE)*.

---

## 📜 Executive Summary

This document narrates the architectural journey of a self-hosted, isolated, and observable container orchestration environment built entirely from scratch. The project is not a mere "installation" process; it is the practical application of **Infrastructure Design**, **Network Security**, **Service Orchestration**, **Observability**, and **Disaster Recovery** principles on a live system.

The system was designed on top of an Ubuntu distribution running on WSL2 (Windows Subsystem for Linux), running nine different microservices side by side—each isolated from one another yet integrated—under a single roof, using Docker containerization technology. Critical system failures encountered throughout the process were resolved via Root Cause Analysis, and each one is documented in this report as a distinct engineering case study.

---

## 🗂️ Table of Contents

- [Phase 0: Architectural Vision and Design Philosophy](#-phase-0-architectural-vision-and-design-philosophy)
- [Phase 1: Building the Foundation Layer](#-phase-1-building-the-foundation-layer)
- [Phase 2: Orchestration Core — Runtipi Integration](#-phase-2-orchestration-core--runtipi-integration)
- [Phase 3: Network Architecture and Security Layer](#-phase-3-network-architecture-and-security-layer)
- [Phase 4: The Microservice Ecosystem](#-phase-4-the-microservice-ecosystem)
- [Phase 5: Monitoring and Observability](#-phase-5-monitoring-and-observability)
- [Phase 6: Disaster Recovery and Data Resilience](#-phase-6-disaster-recovery-and-data-resilience)
- [🔥 Critical Engineering Cases (Troubleshooting Cases)](#-critical-engineering-cases-troubleshooting-cases)
- [Engineering Competencies Acquired](#-engineering-competencies-acquired)
- [Conclusion and Future Vision](#-conclusion-and-future-vision)

---

## 🎯 Phase 0: Architectural Vision and Design Philosophy

The starting point of this project was a simple question: *"Without depending on cloud providers, can I build a self-sufficient digital ecosystem that keeps my data sovereignty in my own hands?"*

The answer took shape as an architecture that honors **High Availability** principles, weaves security rings using a **Defense in Depth** logic, and minimizes the risk of a **Single Point of Failure**.

The three pillars of the design philosophy were defined as follows:

1. **Isolation:** Each microservice operates within its own container boundaries; a vulnerability in one service cannot propagate to the others.
2. **Observability:** Turning the system from a "black box" into something where every layer (CPU, RAM, network traffic, container logs) can be monitored in real time.
3. **Resilience:** Integrating automatic, encrypted, and scheduled backup strategies into the system from the outset, as a defense against data-loss scenarios.

---

## 🧱 Phase 1: Building the Foundation Layer

### 1.1. Operating System Layer Selection

The project is deployed on **WSL2 (Windows Subsystem for Linux 2)**, a hybrid approach that runs within the Windows environment while not compromising on the native performance of the Linux kernel. This choice was made to merge development-environment flexibility with a production-like Linux runtime in the same crucible.

```bash
# Example terminal session (with placeholder username/host information)
ornek_kullanici@ornek-sunucu:~$ lsb_release -a
Distributor ID: Ubuntu
Description:    Ubuntu 22.04 LTS
```

### 1.2. Installing the Containerization Engine

**Docker Engine** and **Docker Compose**, which form the backbone of the system, were installed with careful verification of package dependencies and version-compatibility matrices. At this stage, the compatibility of the `cgroups` (control groups) configuration with the WSL2 kernel was verified, confirming that container resource limiting (CPU/RAM throttling) functioned correctly.

```bash
ornek_kullanici@ornek-sunucu:~$ docker --version
ornek_kullanici@ornek-sunucu:~$ docker compose version
```

### 1.3. File System Hierarchy and Permission Architecture

The Linux file permission structure (the `rwx` permission triad) was carefully planned for each service's volume mounts. This is a forward-looking step that would later become the foundation of a critical engineering case (see *Case 1*) in subsequent phases.

---

## ⚙️ Phase 2: Orchestration Core — Runtipi Integration

Instead of managing microservices manually one by one through `docker-compose.yml` files, **Runtipi (v4.10.1)** was chosen as a central **Orchestration Layer**. Runtipi is a self-hosted PaaS (Platform as a Service) solution that unifies container lifecycle management, network configuration, and an application store (*App Store*) concept under a single control plane.

### 2.1. Why Runtipi?

| Criterion | Traditional Docker Compose | Runtipi Orchestration |
|---|---|---|
| Service Management | Manual YAML editing | Centralized CLI + Web Interface |
| Network Isolation | Manually defined `networks` | Automatic internal bridge network |
| Update Management | Manual `pull` + `up -d` | Single-command CLI-based upgrade |
| Reverse Proxy Integration | Requires external configuration | Built-in support |

### 2.2. Bringing the Core Service Online

```bash
ornek_kullanici@ornek-sunucu:~/runtipi$ sudo ./runtipi-cli start
```

This command triggered Runtipi's own internal containers (reverse proxy, worker, dashboard), spinning up the isolated network environment (*internal Docker bridge network*) in which all upper-layer microservices would live.

---

## 🔐 Phase 3: Network Architecture and Security Layer

This phase is where the project's **Defense in Depth** strategy becomes concrete. Network security was achieved not through a single tool, but through a layered security stack.

### 3.1. Perimeter Security — UFW (Uncomplicated Firewall)

All ports of the server exposed to the outside world were configured following a **default-deny** approach as a guiding principle. Only explicitly permitted ports (SSH, HTTP/HTTPS, Runtipi reverse proxy ports) were left open to traffic, while all remaining ports were locked down under a `DENY` policy.

```bash
ornek_kullanici@ornek-sunucu:~$ sudo ufw status verbose
Status: active
Default: deny (incoming), allow (outgoing)

To                         Action      From
--                         ------      ----
22/tcp (SSH)               ALLOW       Anywhere
80,443/tcp (HTTP/HTTPS)    ALLOW       Anywhere
```

This approach minimized the attack surface against a potential port-scanning attack.

### 3.2. Traffic Routing and SSL/TLS Termination — Nginx Proxy Manager

**Nginx Proxy Manager (NPM)** was deployed to provide access to all microservices via **subdomains**, through a single IP address. NPM functioned as a **Reverse Proxy** layer that routed incoming requests to the relevant container based on the `Host` header, while also taking on automatic SSL/TLS certificate issuance and renewal through **Let's Encrypt** integration.
Client

│

▼

[ UFW Firewall — Port Filtering ]

│

▼

[ Nginx Proxy Manager — SSL Termination + Subdomain Routing ]

│

├──▶ nextcloud.ornek-domain.local

├──▶ jellyfin.ornek-domain.local

├──▶ vault.ornek-domain.local

└──▶ dashboard.ornek-domain.local
Thanks to this architecture, a single entry point was presented to the outside world, with each service now serving over its own encrypted (HTTPS) and human-readable subdomain.

---

## 🧩 Phase 4: The Microservice Ecosystem

The table below details the architectural role and strategic purpose of every microservice integrated into the system:

| # | Service | Category | Architectural Role |
|---|---|---|---|
| 1 | ☁️ **Nextcloud** | Data Sovereignty | Self-hosted file sync and storage, independent of third-party cloud providers |
| 2 | 🎬 **Jellyfin** | Media Delivery | Hardware-accelerated transcoding media streaming |
| 3 | 🔑 **Vaultwarden** | Cryptography & Security | End-to-end encrypted password vault (Bitwarden-compatible API) |
| 4 | 📊 **Glances** | Telemetry | Real-time CPU/RAM/Disk/Network resource monitoring |
| 5 | 📜 **Dozzle** | Observability | Live monitoring of all container logs via a web interface |
| 6 | 🗄️ **Adminer** | Database Management | Isolated, lightweight database management panel |
| 7 | 🏠 **Homepage** | Centralized Dashboard | Single control-center screen integrating all services via API |
| 8 | 💾 **Duplicati** | Disaster Recovery | Scheduled, encrypted, incremental backup orchestration |
| 9 | 🛡️ **Pi-hole** | Network Security & DNS | Network-wide DNS-level ad and tracker blocking |

### 4.1. Inter-Service Integration Logic

These nine services were not designed as disconnected islands, but as an integrated ecosystem brought together through API keys via the **Homepage** dashboard. For example, Homepage gathers live system metrics from Glances, log statuses from Dozzle, and health-check results from other services into a single visual panel — bringing the **Single Pane of Glass** principle to life.

---

## 👁️ Phase 5: Monitoring and Observability

One of the core principles of modern DevOps practice — *"If you can't measure it, you can't manage it"* — was implemented in this project through a two-layered observability architecture:

- **Metrics Layer — Glances:** CPU load, memory consumption, disk I/O, and network traffic at the system level were converted into real-time telemetry data.
- **Logging Layer — Dozzle:** The `stdout`/`stderr` outputs of every container were made live-traceable, filterable, and searchable through a centralized web interface.

The combination of these two layers formed the diagnostic infrastructure for root cause analysis that enabled the resolution of the project's most critical troubleshooting cases (see *Case 2*).

---

## 🛟 Phase 6: Disaster Recovery and Data Resilience

The greatest risk for a self-hosted system is data loss caused by hardware failure or human error. To mitigate this risk, the **Duplicati** service was integrated following this Disaster Recovery strategy:

- ✅ **Scheduled Backups:** Backup jobs automatically triggered at defined intervals.
- ✅ **Encryption at Rest:** Backup data sets are protected via AES-based encryption before being written to disk.
- ✅ **Incremental Backup:** Rather than repeatedly copying the entire data set, only changed blocks are backed up, ensuring storage efficiency.

This strategy represents an enterprise-grade backup philosophy — one aimed at minimizing **RPO (Recovery Point Objective)** and **RTO (Recovery Time Objective)** metrics — applied at the scale of a home server.

---

## 🔥 Critical Engineering Cases (Troubleshooting Cases)

> *This section is the heart of the project. Anyone can set up a system; but it's the engineer who stays standing when the system goes down.*

### 🚨 Case #1: The "Permission Denied" Crisis During the Runtipi v4.10.1 Update

**Symptom:**
During the upgrade process to Runtipi `v4.10.1`, the system threw a `Permission Denied` error while attempting to operate on the `docker-compose.yml` file.

```bash
ornek_kullanici@ornek-sunucu:~/runtipi$ ./runtipi-cli update
Error: EACCES: permission denied, open 'docker-compose.yml'
```

**Root Cause Analysis:**
Investigation revealed that, during a prior update/installation operation, certain files had been created by the `root` user, and file ownership had not been transferred back to the standard user account. Due to Linux's strict permission hierarchy (UID/GID-based access control), this prevented the CLI tool from gaining write access to the file.

**Engineering Solution Applied:**

1. The update process was re-triggered with elevated privileges to force the operation through:

```bash
ornek_kullanici@ornek-sunucu:~/runtipi$ sudo ./runtipi-cli update
```

2. The file ownership locked by root was reclaimed by intervening in the file system hierarchy and reassigning it to the standard user:

```bash
ornek_kullanici@ornek-sunucu:~/runtipi$ sudo chown -R $USER:$USER .
```

3. The system was successfully restarted with zero data loss, and service integrity was verified.

**Engineering Takeaway:**
This case demonstrated just how critical the Linux file permission model (`rwx` + UID/GID) is in containerized environments. The solution was not simply running a command — it required a systemic understanding of **file ownership hierarchy** and a permanent fix grounded in that understanding.

---

### 🚨 Case #2: DNS Port 53 Conflict During Pi-hole Installation

**Symptom:**
During the deployment of the Pi-hole service, the installation wizard returned a `"Log check required"` warning, indicating that the service had not started in a healthy state.

**Root Cause Analysis:**
Since the error message alone wasn't sufficiently informative, real-time access to the Pi-hole container's live logs was obtained via **Dozzle**. Upon inspection, the actual underlying problem became clear:
ERROR: failed to bind host port for 0.0.0.0:53: address already in use
At this point, the analysis was pushed down from the container level to the **operating system level**. Further investigation revealed that Ubuntu's built-in **`systemd-resolved`** service was already listening on port `53` as the system's default DNS resolver. Since Pi-hole was requesting the same port, a classic **port conflict** scenario had emerged.

**Engineering Solution Applied:**

1. Confirming the port conflict:

```bash
ornek_kullanici@ornek-sunucu:~$ sudo lsof -i :53
systemd-resolved   853   root   13u  IPv4  LISTEN
```

2. Stopping and disabling the conflicting system service:

```bash
ornek_kullanici@ornek-sunucu:~$ sudo systemctl stop systemd-resolved
ornek_kullanici@ornek-sunucu:~$ sudo systemctl disable systemd-resolved
```

3. Reconfiguring `/etc/resolv.conf` so that the system's DNS resolution would now flow through Pi-hole.

4. Verifying that the Pi-hole container could now successfully bind to ports `53/tcp` and `53/udp`, and confirming that network-wide DNS traffic was being routed correctly.

**Engineering Takeaway:**
This case proved how critical observability tools (Dozzle) are not merely for "monitoring," but as an **active diagnostic instrument in root cause analysis**. The problem originated at the container layer but was resolved at the operating system layer (*host-level networking*) — underscoring the importance of cross-layer systemic thinking.

---

## 🎓 Engineering Competencies Acquired

The technical and analytical competencies gained at the end of this intensive 19-day process can be summarized as follows:

- 🐳 **Container Orchestration:** Multi-service management with Docker & Docker Compose
- 🌐 **Network Architecture:** Designing Reverse Proxy, Subdomain Routing, and SSL/TLS Termination
- 🔥 **Security Hardening:** Applying perimeter security and default-deny principles with UFW
- 👁️ **Observability Engineering:** Using log and metric layers together for integrated root cause analysis
- 🛟 **Disaster Recovery Planning:** Designing encrypted, scheduled, and incremental backup strategies
- 🧩 **Systemic Troubleshooting:** A multi-layered problem-solving methodology descending from the container layer down to the operating system layer
- 📁 **Linux Filesystem Permission Management:** UID/GID-based access control and file ownership management

---

## 🚀 Conclusion and Future Vision

This project is concrete proof of an intern's transition from the role of "observer" to the identity of an **infrastructure engineer who makes their own decisions, diagnoses their own failures, and produces their own solutions**. Over the course of 19 days, the system that was built became not just a "working" infrastructure, but an architecture that is **understood, observable, and maintainable**.

Planned improvements for future phases:

- [ ] Transition to **Kubernetes (K3s)** for true *High Availability* and automatic scaling
- [ ] **Prometheus + Grafana** integration for advanced metric visualization and alerting mechanisms
- [ ] **CI/CD Pipeline** (GitHub Actions) integration to connect infrastructure changes to automated testing and deployment workflows
- [ ] Automated provisioning based on **Infrastructure as Code (IaC)** principles using Ansible/Terraform

---

<div align="center">

### 📌 This project is not an "internship notebook"; it is the documentation of an engineering journey.

**Prepared by: RadmelaSub**

</div>

Turkish= https://github.com/radmela5461/kendi-windows-bilgisayar-nda-sunucu-kur-ve-runtipi-ile-geli-tir
