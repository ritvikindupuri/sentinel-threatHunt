# 🛡️ Enterprise-Grade Agentic AI Threat Hunting System (Microsoft Sentinel)

## Executive Summary
This project is an advanced **Security Operations Center (SOC) automation platform** built in n8n. It redefines threat hunting by leveraging a **Parallel Multi-AI Consensus Engine** to ingest, analyze, and respond to cybersecurity threats from Microsoft Sentinel.

Instead of relying on a single Large Language Model (LLM) which can hallucinate or misinterpret context, this architecture branches high-risk Sentinel events to three independent AI Agents simultaneously: **GPT-4**, **Claude**, and **Gemini**. The system forces these models to compare findings, calculate a consensus risk score, and subsequently route verified alerts to incident response teams and executive leadership via Slack and Email.

## System Architecture

The workflow is highly modular, moving from deterministic log filtering into non-deterministic AI analysis, and back into deterministic alerting and reporting.

<p align="center">
  <img src=".assets/Screenshot 2026-02-07 143313.png" alt="Multi-AI Threat Hunting Architecture" width="850"/>
  <br>
  <b>Figure 1: Triple-Model Consensus Workflow Architecture</b>
</p>

## 🚀 Usage: How to Import
To deploy this system in your own n8n instance:
1.  Download the `.json` workflow file included in this repository.
2.  Open your n8n workspace.
3.  Click the **three dots (•••)** in the top right corner of the canvas.
4.  Select **"Import from File"** and upload the JSON file.
5.  Follow the *Detailed Credential Setup Guide* below to configure your connections.

---

## 🎯 Detailed Workflow Breakdown

### 🌊 FLOW 1: Sentinel Ingestion & Pre-Processing
The pipeline captures and sanitizes raw telemetry to ensure the AI agents only spend compute resources on actionable anomalies.
* **Poll Sentinel Logs / Real-time Sentinel Events:** Dual triggers ensure high availability. The system runs on a polling schedule while simultaneously listening for real-time webhook push events from Azure.
* **Workflow Configuration & Fetch Sentinel Security Logs:** Standardizes API parameters and retrieves the latest security event payloads from the Sentinel workspace.
* **Normalize Security Events & Filter High-Risk Events:** Cleans the raw JSON schema and drops low-level noise, forwarding only critical indicators.
* **Split Events for Parallel AI Analysis:** This node duplicates the filtered payload, dispatching it concurrently to the three separate AI agents.

### 🧠 FLOW 2: Parallel Multi-Agent Analysis
The core engine. Three specialized LLMs independently investigate the exact same security event using a shared suite of active OSINT tools.
* **GPT-4 Threat Hunter Agent:** Configured for rapid pattern recognition and known attack vector mapping.
* **Claude Threat Research Agent:** Configured for deep contextual reasoning and complex attack chain analysis.
* **Gemini Correlation Agent:** Configured for multi-modal analysis and anomaly detection.
* **Shared Agent Tools & Memory:** All three agents autonomously utilize:
    * *Historical Threat Intelligence DB:* A Pinecone vector store (using *Embeddings for Threat Vectors*) to recall past internal incidents.
    * *VirusTotal Intelligence Tool* & *AbuseIPDB Reputation Tool:* For real-time indicator lookup.
    * *MITRE ATT&CK Framework Tool:* To map behaviors to standardized tactics and techniques.
    * *Threat Context Memory:* To correlate multi-step actions during the session.

### ⚖️ FLOW 3: Consensus & Scoring Engine
To eliminate AI hallucinations and false positives, the system acts as a judge over the three agents.
* **Merge Multi-AI Threat Analysis:** Waits for all three agents to finish their distinct investigations and compiles their JSON outputs.
* **Consensus Threat Scoring Engine:** Mathematically evaluates the findings. A threat is only escalated if the agents reach a consensus on its severity and validity.
* **Route by Threat Severity:** A logic gate that directs the consensus report down the Critical, High, Medium, or Low paths.

### 📢 FLOW 4: Intelligent Routing & Escalation
* **Critical Path:**
    * Generates a *Format Critical Alert* and simultaneously broadcasts to:
        * *Alert Security Operations Center* (Slack)
        * *Alert Incident Response Team* (Slack)
        * *Email CISO - Critical Threat* (Direct Email)
* **High Path:**
    * Generates a *Format High Priority Alert* and routes to *Notify Security Analyst* (Slack) for standard queue review.
* **Medium/Low Path:** * Bypasses active notifications to prevent alert fatigue, funneling directly into storage.

### 💾 FLOW 5: Storage & Executive Reporting
* **Store Threat Intelligence:** Parses the finalized *Prepare Threat Intelligence Record* and writes it to a persistent database.
* **Daily Executive Summary Generation:** An automated sequence (*Aggregate Daily Threat Metrics* ➔ *Calculate Threat Statistics*) compiles a 24-hour retrospective.
* **Report Delivery:** The *Format Executive Report* node packages the metrics into an HTML brief and dispatches it via *Send Daily Executive Summary*, giving leadership visibility without requiring SIEM dashboard access.

---

## 🔧 Detailed Credential Setup Guide

### Step 1: Microsoft Azure & Sentinel (Critical)
**1. Register the Application:**
* Go to **Azure Portal** > **Microsoft Entra ID** (formerly Azure AD).
* Select **App registrations** > **New registration**.
* Name: `n8n-sentinel-automation`.
* Supported account types: *Accounts in this organizational directory only*.
* Click **Register**.
* **Copy & Save:** `Application (client) ID` and `Directory (tenant) ID`.

**2. Generate the Secret:**
* In your new app, go to **Certificates & secrets** > **Client secrets** > **New client secret**.
* Description: `n8n-key`. Expires: *12 months*.
* **Copy & Save:** The **Value** (not the Secret ID). You will not see this again.

**3. Grant Sentinel Permissions:**
* Go to your **Log Analytics Workspace** (where Sentinel is connected).
* Select **Access control (IAM)** > **Add** > **Add role assignment**.
* Role: **Microsoft Sentinel Reader**.
* Members: Select the `n8n-sentinel-automation` service principal you just created.
* Click **Review + assign**.

**4. Configure in n8n:**
* Create a new Credential type: **Microsoft Azure**.
* `Client ID`: Paste *Application (client) ID*.
* `Client Secret`: Paste the Secret *Value*.
* `Tenant ID`: Paste *Directory (tenant) ID*.

### Step 2: AI Model API Keys
**1. OpenAI (GPT-4):**
* Go to `platform.openai.com` > **API keys**.
* Create new key named `n8n-gpt4`.
* In n8n, create **OpenAI** credential and paste the key.

**2. Anthropic (Claude):**
* Go to `console.anthropic.com` > **Settings** > **API Keys**.
* Create key named `n8n-claude`.
* In n8n, create **Anthropic** credential and paste the key.

**3. Google Gemini:**
* Go to `aistudio.google.com` > **Get API key**.
* Create key in a new project.
* In n8n, create **Google AI** credential and paste the key.

### Step 3: Vector Database (Pinecone)
* Go to `app.pinecone.io`.
* Create Index: Name `threat-intelligence`, Dimensions **1536** (Critical for OpenAI compatibility), Metric **Cosine**.
* Go to **API Keys** > Copy the Key.
* In n8n, create **Pinecone** credential. Enter the API Key.

### Step 4: Threat Intelligence Tools
**1. VirusTotal:**
* Login to VirusTotal > Click Profile Icon > **API Key**.
* In n8n, use **Header Auth**. Name: `x-apikey`, Value: `[Your_VT_Key]`.

**2. AbuseIPDB:**
* Login to AbuseIPDB > **API** tab > **Create Key**.
* In n8n, use **Header Auth**. Name: `Key`, Value: `[Your_AbuseIPDB_Key]`.

### Step 5: Database (PostgreSQL)
* Run this SQL command in your Postgres instance to create the required schema:
    ```sql
    CREATE TABLE threat_intelligence (
        id SERIAL PRIMARY KEY,
        event_id TEXT UNIQUE NOT NULL,
        severity TEXT,
        consensus_score NUMERIC,
        ai_analysis JSONB,
        timestamp TIMESTAMP DEFAULT NOW()
    );
    ```
* In n8n, create **Postgres** credential with your Host, User, Password, and Database name.

---

## 📊 Impact & Outcomes
* **Near-Zero False Positives:** By requiring three distinct LLM architectures to reach a consensus, the system mathematically filters out single-model hallucinations and benign anomalies.
* **Automated OSINT Triage:** The agents autonomously query VirusTotal, AbuseIPDB, and MITRE before a human ever sees the alert, cutting triage time from hours to seconds.
* **Tiered Signal-to-Noise Ratio:** Critical threats immediately page IR teams and the CISO, while High/Medium threats are silently queued or logged, completely protecting analysts from alert fatigue.
* **Automated Executive Visibility:** The daily metrics compilation ensures leadership receives consistent, data-driven insights into the threat landscape and SOC performance without manual reporting overhead.
