# 🤖 AI-Powered SOC Automation Project

## Project Objective

The main goal of this project is to construct a fully automated Security Operations Center (SOC) workflow that leverages **Artificial Intelligence (AI)** for alert triage and enrichment.

This project demonstrates a modern, skill-testing pipeline that:
1.  Ingests security telemetry from an endpoint into a SIEM.
2.  Generates a security alert based on the logs.
3.  Automatically sends the alert to an automation tool (SOAR-like).
4.  Utilizes a Large Language Model (LLM) like **ChatGPT** to analyze, enrich, and summarize the security alert.
5.  Notifies a Security Analyst via a communication platform like **Slack** with a clear, AI-summarized report, ready for action..

---

## 🏗️ Architecture and Components

This setup is primarily **on-premises** and uses Virtual Machines for all core services.

### Core Components

| Component | Role | OS/Platform |
| :--- | :--- | :--- |
| **Splunk Enterprise** | Security Information and Event Management (SIEM) | Ubuntu Server 24.04+ (Recommended 8GB RAM) |
| **N8N** | Automation/Workflow Engine (Self-hosted SOAR) | Ubuntu Server (Installed via Docker Compose) |
| **Windows 10 Pro** | Endpoint/Log Source | Windows 10 VM (Recommended 4GB RAM)  |
| **Splunk Universal Forwarder** | Log Agent |Installed on Windows 10 VM  |
| **ChatGPT / OpenAI** | Alert Processing and Enrichment (Requires API credits) | External Cloud Service |
| **Slack** | Communication/Analyst Triage Output | External Cloud Service  |
| **Hypervisor** | Virtualization Platform | VMware Workstation Pro (or VirtualBox)  |

### System Requirements

A physical machine with at least **32 GB of RAM** is recommended for smooth operation.16 GB or less may cause performance issues.

---

## ⚙️ Core Setup Workflow

### 1. Virtual Machine Setup

The project requires several VMs, all connected on the same network (e.g., **NAT** mode).

| VM Name | Purpose | Recommended Specs | OS/Image |
| :--- | :--- | :--- | :--- |
| `Thecyberine_Splunk` | SIEM | 8 GB RAM, 2 Cores, 100 GB Disk  |Ubuntu Server (24.04+)  |
| `Thecyberine_N8N` | Automation | 4 GB RAM, 2 Cores, 50 GB Disk | Ubuntu Server (24.04+) |
| `Thecyberine_Windows10` | Endpoint | 4 GB RAM, 2 Cores, 60 GB Disk | Windows 10 Pro ISO  |

### 2. Splunk Enterprise Installation (on `Thecyberine_Splunk` VM)

1.  **Update OS:** Establish an SSH session to your Splunk VM (e.g., using `ssh Thecyberine@<SPLUNK_IP>`)
    ```bash
    sudo apt-get update && sudo apt-get upgrade -y
    ```
2.  **Install Splunk:** Download the Splunk `.deb` file (Linux 64-bit) from the Splunk website.
    ```bash
    # Download using wget (example link will vary by version)
    wget -O splunk.deb 'PASTE_SPLUNK_DOWNLOAD_LINK_HERE'

    # Install the package
    sudo dpkg -i splunk.deb
    ```
3.  **Start and Configure:**
    ```bash
    # Enable Splunk to start on boot
    sudo /opt/splunk/bin/splunk enable boot-start -user splunk

    # Start Splunk for the first time, accept license, and set admin credentials
    sudo /opt/splunk/bin/splunk start
    ```
4.  **Web Configuration (http://<SPLUNK_IP>:8000)**:
    * **Add Receiving Port:** `Settings` > `Forwarding and receiving` > `Configure receiving` > `New receiving port`. Set port to **`9997`**.
    * **Create Index:** `Settings` > `Indexes` > `New Index`. Name it **`Thecyberine_project`**.
    * **Install App:** `Apps` > `Find More Apps`. Search and install **`Splunk Add-on for Microsoft Windows`**.

### 3. Windows 10 Universal Forwarder (UF) Setup

1.  **Install UF:** Download and install the **Splunk Universal Forwarder** (64-bit Windows) on the Windows 10 VM.
    * During installation, configure the receiving indexer to be your Splunk VM IP address (`<SPLUNK_IP>`) and the receiving port as **`9997`**.
2.  **Configure `inputs.conf`:** Navigate to `C:\Program Files\SplunkUniversalForwarder\etc\system\local\` and create or paste the `inputs.conf` file. This file tells the UF what logs to collect.
    *The configuration should be set to send logs to the `Thecyberine_project` index.
3.  **Restart Service:** Go to Windows Services, find the **SplunkForwarder Service**, and restart it to apply the new configuration.

### 4. N8N Automation Server Setup (on `Thecyberine_N8N` VM)

1.  **Install Docker and Docker Compose:** SSH into the N8N VM.
    ```bash
    # Install Docker Engine
    sudo apt install docker.io -y
    # Install Docker Compose
    sudo apt install docker-compose -y
    ```
2.  **Create Docker Compose File:**
    ```bash
    mkdir n8n-compose
    cd n8n-compose
    sudo nano docker-compose.yaml
    ```
3.  **Paste the following configuration into `docker-compose.yaml`:**
    * **NOTE:** Replace `<N8N_IP>` with the actual IP of your N8N VM.
    ```yaml
    version: '3.8'

    services:
      n8n:
        image: n8nio/n8n:latest
        restart: always
        ports:
          - 5678:5678
        environment:
          # IMPORTANT: Use the IP address of your N8N VM
          - N8N_HOST=192.168.136.139 # Replace with <N8N_IP> 
          - N8N_PORT=5678
          - N8N_PROTOCOL=http
        volumes:
          - ./n8n_data:/home/node/.n8n
    ```
    * *(Source for environment variables and structure )*
4.  **Deploy N8N:**
    ```bash
    # Pull the image
    sudo docker-compose pull

    # Start the container in detached mode
    sudo docker-compose up -d
    ```
5.  **Access N8N:** You can now access the N8N web interface from your host machine: **`http://<N8N_IP>:5678`**.

---

## 🚀 Advanced / Bonus Features

The project can be expanded with the following advanced features for a complete, production-ready simulation (mentioned as community/classroom bonuses):

* **Ticketing Integration:** Integrating a case management system like **DFIR-IRIS** to automatically create tickets from N8N workflows.
* **Threat Intelligence Enrichment:** Enriching file hashes or other indicators with platforms like **VirusTotal**.
* **AI-to-SIM Querying (MCP Server):** Setting up an intermediate server (MCP) that allows the LLM (AI) to execute direct queries against Splunk for real-time investigation and threat identification.

---

## ⚠️ Important Considerations

* **Privacy Risk:** This project feeds data into an external AI service (ChatGPT). It is explicitly for **learning and portfolio purposes only** and should **not** be deployed into a production environment due to inherent privacy risks.
* **Cost:** Using AI (ChatGPT/OpenAI) for processing alerts requires purchasing **API credits**.
