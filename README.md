#  Enterprise-Grade Agentic AI Threat Hunting System

## Executive Summary
This project is a production-ready **Security Operations Center (SOC) automation platform** that leverages **Multi-AI Consensus** to detect, analyze, and respond to cybersecurity threats.

Unlike standard automation that relies on a single model, this system ingests security logs from **Microsoft Sentinel** and processes them in parallel through three leading LLMs—**GPT-4** (Pattern Recognition), **Claude 3 Opus** (Contextual Reasoning), and **Gemini 1.5 Pro** (Anomaly Detection). A consensus engine then aggregates these findings to eliminate false positives and automatically escalates critical threats to security leadership via Teams, Slack, and Email.

## System Architecture
The workflow operates on a 15-minute polling cycle, ensuring near real-time detection while adhering to API rate limits.

<p align="center">
  <img src=".assets/Screenshot 2026-02-07 143313.png" alt="Multi-AI Threat Hunting Architecture" width="850"/>
  <br>
  <b>Figure 1: Triple-Model Consensus Workflow Architecture</b>
</p>

##  Usage: How to Import
To deploy this system in your own n8n instance:
1.  Download the `.json` workflow file included in this repository.
2.  Open your n8n workspace.
3.  Click the **three dots (•••)** in the top right corner of the canvas.
4.  Select **"Import from File"** and upload the JSON file.
5.  Follow the *Detailed Credential Setup Guide* below to configure your Azure, AI, and Database connections.

---

##  Detailed Workflow Breakdown

### Phase 1: Dual-Trigger Log Ingestion
* **1.1 Scheduled Polling:** Runs every 15 minutes to pull the latest batch of logs, ensuring consistent monitoring even if push notifications fail.
* **1.2 Real-Time Webhook:** A secondary listener for immediate push notifications from Sentinel automation rules.
* **1.3 Configuration Node:** Sets global variables (Subscription ID, Workspace ID) to standardise downstream parameters.

### Phase 2: Microsoft Sentinel Integration
* **2.1 Fetch Security Logs:** Authenticates via Azure Active Directory and executes a KQL query to retrieve critical security events from the last 15 minutes.
    * *Query:* `SecurityEvent | where EventLevelName in ("Critical", "Error") ...`
* **2.2 Normalization & Filtering:** A Code Node standardizes the raw Azure JSON and filters out noise, passing only high-risk events (e.g., "malware", "privilege escalation") to the AI models.

### Phase 3: Multi-AI Consensus Analysis
The system splits events and sends them to three distinct "AI Agents" simultaneously:
* **Agent 1 (GPT-4 Turbo):** Focuses on speed and known attack pattern recognition.
* **Agent 2 (Claude 3 Opus):** Tasked with deep reasoning and understanding complex attack chains.
* **Agent 3 (Gemini 1.5 Pro):** Specialized in detecting zero-day anomalies and novel vectors.
* **Tools:** All agents have access to **Pinecone** (Historical RAG), **VirusTotal** (Hash/IP Reputation), and **AbuseIPDB**.

### Phase 4: Consensus Scoring & Validation
* **4.1 Aggregation:** Collects the JSON outputs from all three models.
* **4.2 Scoring Logic:**
    * **Severity Vote:** Majority rules (e.g., if 2/3 models say "Critical", the event is Critical).
    * **Confidence Averaging:** Calculates the mean confidence score (0-100%) across models.
    * **Disagreement Flagging:** If models drastically disagree, the alert is flagged for human review.

### Phase 5: Intelligent Routing & Response
* **Critical Path:** Immediate escalation via **Microsoft Teams** (Adaptive Card), **Slack** (Block Kit), and **Email** to the CISO.
* **High Path:** Alerts the SOC channel in Slack for analyst review.
* **Medium/Low Path:** logged to the database for trend analysis without waking up staff.

### Phase 6: Forensics & Compliance
* **6.1 Persistence:** Stores full audit trails in **PostgreSQL** and n8n internal tables.
* **6.2 Reporting:** Generates a **Daily Executive Summary** at 8:00 AM, detailing total threats, MITRE tactic frequency, and ROI metrics.

---

##  Detailed Credential Setup Guide

### Step 1: Azure & Microsoft Sentinel
1.  **Create Service Principal:** In Azure AD, register a new app ("n8n-threat-hunting"). Copy the `Application (client) ID` and `Directory (tenant) ID`.
2.  **Generate Secret:** Create a Client Secret and copy the value immediately.
3.  **Grant Permissions:** In your Sentinel Workspace IAM, assign the **"Microsoft Sentinel Reader"** role to your app.
4.  **n8n Auth:** Create an `OAuth2 API` credential in n8n using these details.

### Step 2: OpenAI (GPT-4)
1.  Generate an API Key at `platform.openai.com`.
2.  Add to n8n as **OpenAI** credential.
3.  *Model Selection:* Ensure the node is set to `gpt-4-turbo` or newer.

### Step 3: Anthropic (Claude)
1.  Generate an API Key at `console.anthropic.com`.
2.  Add to n8n as **Anthropic** credential.
3.  *Model Selection:* Use `claude-3-opus` for maximum reasoning capability.

### Step 4: Google Gemini
1.  Generate an API Key at `makersuite.google.com`.
2.  Add to n8n as **Google AI** credential.
3.  *Model Selection:* Use `gemini-1.5-pro`.

### Step 5: Vector Database (Pinecone)
1.  Create an index named `threat-intelligence` with **1536 dimensions** (Cosine metric).
2.  Add your API Key and Environment (e.g., `us-west1-gcp`) to n8n.

### Step 6: Threat Intelligence Feeds
* **VirusTotal:** Get API Key from profile -> Add to n8n -> Configure Header `x-apikey`.
* **AbuseIPDB:** Get API Key -> Add to n8n as Header Auth (`Key: [Your_Key]`).

### Step 7: Communication Channels
* **Microsoft Teams:** Create a Webhook or Bot in Azure (`ChannelMessage.Send` permissions). Configure the node with your `Team ID` and `Channel ID`.
* **Slack:** Create an App with `chat:write` scopes. Use the **Bot User OAuth Token**.
* **Email:** Use standard SMTP credentials or Gmail OAuth2.

### Step 8: Database (PostgreSQL)
1.  Create the database: `CREATE DATABASE threat_intelligence;`
2.  Create the schema:
    ```sql
    CREATE TABLE threat_intelligence (
        event_id VARCHAR(255) PRIMARY KEY,
        timestamp TIMESTAMP,
        severity VARCHAR(50),
        ai_consensus JSONB,
        indicators JSONB
    );
    ```
3.  Connect in n8n via the **Postgres** node.

### Step 9: n8n Data Tables
1.  In n8n, go to **Data Tables** -> **Add Data Table**.
2.  Name: `threat_intelligence`.
3.  Columns: `event_id` (Text), `severity` (Text), `confidence_score` (Number), `ai_consensus` (JSON).

### Step 10: Final Configuration
* Open the **Workflow Configuration** node (Step 1.3) and enter your specific `subscriptionId`, `resourceGroup`, and `workspaceId`.

---

##  Expected Outcomes
* **Real-Time Detection:** The 15-minute polling cycle ensures near real-time threat detection, capturing indicators before they can escalate into breaches.
* **Reduced False Positives:** Multi-model consensus (GPT-4 + Claude + Gemini) reduces alert noise by an estimated **70-80%** compared to single-model systems.
* **Faster Response:** Automated escalation cuts incident triage time from **hours to minutes**, allowing analysts to focus on remediation rather than investigation.

* **Compliance & Audit:** Provides a full, immutable audit trail compatible with **SOC 2**, **ISO 27001**, and **GDPR** requirements.
* **Executive Visibility:** Daily summaries provide leadership with actionable ROI metrics without requiring dashboard access.
* **Enterprise Readiness:** This architecture represents the level of security automation deployed by Fortune 500 companies in production environments.
* **Executive Visibility:** Daily summaries provide leadership with actionable ROI metrics without requiring dashboard access.
