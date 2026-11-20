# SOC-Automated-Workflow

# SOC Automated Workflow (automated-soc-project)

An end-to-end **Automated SOC Alert Triage & Threat Enrichment Pipeline** integrating **Windows Sysmon**, **Splunk Enterprise (SIEM)**, **n8n (SOAR)**, **AbuseIPDB**, **OpenAI**, and **Slack (ChatOps)**.

This lab demonstrates how to reduce Tier 1 alert triage time from ~15 minutes to under 10 seconds by automating log correlation, threat intelligence enrichment, MITRE ATT&CK mapping, and incident reporting while preserving a **Human-in-the-Loop** validation model.

---

## Architecture Overview

![Architecture Diagram](architecture/workflow-diagram.png)

### End-to-End Pipeline Flow
1. **Telemetry Collection (Windows 10 + Sysmon + Splunk Universal Forwarder):** Captures authentication failures (`EventCode=4625`) and forwards logs over TCP port `9997` to Splunk Enterprise.
2. **SIEM Detection & Correlation (Splunk Enterprise):** A scheduled correlation search (`* * * * *`) aggregates failed login attempts and triggers a real-time **Production Webhook** to n8n.
3. **SOAR Orchestration (n8n on Docker):** Receives the alert JSON payload and orchestrates external enrichment and AI analysis.
4. **Threat Intelligence Enrichment (AbuseIPDB API):** Queries IP reputation, abuse confidence score, geolocation, and historical attack reports.
5. **Tier 1 AI Triage (OpenAI LLM):** Correlates raw SIEM telemetry with Threat Intelligence to produce an incident summary, severity rating, **MITRE ATT&CK (`T1110` - Brute Force)** mapping, and containment steps.
6. **ChatOps Notification (Slack):** Delivers the formatted, actionable report directly to the `#alerts` channel.

---

## 🛠️ Tech Stack & Infrastructure

* **Virtualization:** VMware Workstation Pro 17
* **Endpoint Target:** Windows 10 + Splunk Universal Forwarder
* **SIEM & SOAR Server:** Ubuntu Server hosting Splunk Enterprise & Dockerized n8n
* **Threat Intelligence:** AbuseIPDB REST API v2
* **AI Triage Engine:** OpenAI API (Tier 1 SOC Analyst System Prompt)
* **Alerting / ChatOps:** Slack Incoming App (`#alerts`)

---

## 🔍 Splunk Detection Query (`splunk/brute_force_alert.spl`)

```spl
index="automated-soc-project" EventCode=4625 
| fillnull value="N/A" ComputerName, user, src_ip 
| stats count by _time, ComputerName, user, src_ip
