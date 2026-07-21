# AI-Powered Endpoint Management Platform — Interview Revision Notes

Project: AI-Powered Endpoint Management & Monitoring Platform
My Role: Backend APIs + Agent Script (Windows/macOS) + AI/RAG Integration

---

### Q1. Explain the problem and solution in brief.

**Problem:** Our company works remotely. Whenever a new employee joined, IT had to manually connect to their laptop and install everything one by one — VPN, Git, Node.js, Docker, VS Code, Office, Teams, depending on their role. This took several hours per employee and didn't scale as the company grew.

**Solution:** We built an AI-powered endpoint management platform. Instead of IT doing this manually, the employee just downloaded an agent from the HR portal. This agent automatically registered the device, checked the employee's role, installed the right software for that role, and kept monitoring the device health after that — CPU, RAM, disk, VPN status, and so on.

If something went wrong, like high CPU or VPN disconnecting, the system used AI to summarize the issue, look into company documents using RAG, and give a diagnosis with a suggested fix, shown on a dashboard for the IT team.

Installation and monitoring were handled by normal automation. AI was used only for smart diagnostics and troubleshooting.

**Result:** Onboarding time reduced by around 50%.

---

### Q2. How did you integrate AI and RAG in this project?

When the Rule Engine detected an issue — like high CPU, VPN down, or Docker crashing — I first summarized the telemetry into a short report instead of sending raw logs directly to OpenAI. This kept token usage low and response fast.

For RAG, I took our company documents — VPN guide, installation manual, troubleshooting SOPs, security policies — split them into chunks, converted them into embeddings, and stored them in a vector database.

When an issue came in, I did a similarity search on the vector DB to pull the most relevant document chunks related to that specific issue. Then I built a prompt combining three things — the device summary, the relevant company docs, and the actual question — and sent it to the OpenAI API.

OpenAI returned a diagnosis with a suggested fix, like "restart Docker" or "renew VPN certificate," and I passed that back through the backend so it could show up on the dashboard for the IT team to see and act on.

---

### Q3. How did you handle device configuration, app installation, and checking pending updates/installations?

Device configuration and app installation was handled by the **agent** — a script I wrote in Node.js and TypeScript that runs on the employee's laptop as a Windows Service or macOS Launch Agent.

**Flow:**

1. When the employee installs the agent, it collects basic device info — CPU, RAM, disk, OS, installed apps, device ID, logged-in user — using a library called `systeminformation`. This is sent to the backend through `POST /register-device`.

2. Backend checks the employee's role (Engineering, HR, Manager) from the database, and the **Policy Engine** decides what software that role needs — e.g., Engineering gets Git, Docker, Node.js, VS Code, VPN; HR/Manager gets Office, Teams, VPN.

3. This list is sent back to the agent, which installs the software using OS-specific tools — **Winget** via **PowerShell** on Windows, and **Homebrew** on macOS — both triggered from Node.js using the `child_process` module.

**For pending updates/installations:**

The agent runs checks on a schedule (every few minutes), comparing what's actually installed against what's required for the role, and also checks Windows Update status. If something is missing or an update is pending, it's flagged and sent to the backend as part of telemetry.

Backend stores this in MongoDB. The **Rule Engine** checks this data — if an installation failed or a critical update is pending, it raises an alert on the dashboard, and can trigger the AI diagnosis flow if needed.

---

### Q4. What API endpoints did you work on?

**Device & Registration**

- `POST /register-device` — registers a new device, stores CPU, RAM, disk, OS, device ID, logged-in user
- `GET /device/:deviceId` — fetch current status of a specific device
- `PUT /device/:deviceId/heartbeat` — periodic ping from agent to mark device online/active

**Policy & Installation**

- `GET /policy/:role` — returns list of required software/config based on employee role
- `POST /installation-status` — agent reports back which apps installed successfully or failed

**Telemetry & Monitoring**

- `POST /telemetry` — agent sends CPU, RAM, disk, VPN status, running services on a schedule
- `GET /telemetry/:deviceId/history` — fetch historical telemetry for a device (used on dashboard)

**Alerts & Rule Engine**

- `POST /alert` — created internally when Rule Engine detects anomaly (high CPU, VPN down, disk low, update failed)
- `GET /alerts` — list active alerts for IT dashboard

**AI / RAG Diagnosis**

- `POST /diagnose` — triggered when an alert is raised; takes summarized telemetry, runs RAG lookup, calls OpenAI, returns diagnosis + fix
- `GET /diagnosis/:deviceId` — fetch past AI diagnoses for a device

---

## Tech Stack Used

**Agent (Windows/macOS):**

- Node.js, TypeScript
- Windows Service, macOS Launch Agent
- `systeminformation` (device metrics)
- `child_process` (running system commands)
- PowerShell, Winget (Windows installs)
- Homebrew (macOS installs)

**Backend:**

- Node.js, Express.js
- MongoDB
- Redis
- BullMQ (job queues)
- REST APIs
- JWT (auth)
- WebSockets (real-time updates)

**AI / RAG:**

- OpenAI API
- RAG (Retrieval-Augmented Generation)
- Vector Database (Pinecone/FAISS/Milvus)

**Frontend (colleague's part):**

- React, TypeScript

---

---

### Q5. What technical issues did you face and how did you resolve them?

**Issue 1 — Sending raw telemetry to OpenAI was expensive and slow**

Initially, the idea was to send device data directly to OpenAI whenever we wanted diagnostics. But telemetry was collected every minute — CPU, RAM, disk, logs, running apps — so sending all of this raw data to OpenAI every time was expensive, slow, and would hit the context window limit. It also raised privacy concerns since raw logs contain sensitive device/user data.

**Fix:** I separated data collection from AI analysis. Raw telemetry was stored in MongoDB. A Rule Engine checked this data using simple thresholds (CPU > 90%, disk < 10%, VPN down, install failed). Only when an actual issue was detected did I summarize the relevant telemetry into a short report and send that to OpenAI — not the raw data. This cut down cost, latency, and kept prompts within the context window.

**Issue 2 — Cross-platform installation differences (Windows vs macOS)**

Windows and macOS use completely different tools to install software — Winget/PowerShell on Windows, Homebrew on macOS. Writing separate installation logic for each OS inside the agent made the code harder to maintain and test.

**Fix:** I abstracted the installation logic behind a common interface in the agent code, so the rest of the agent (registration, policy fetch, telemetry) stayed OS-agnostic, and only a thin OS-specific layer handled the actual install commands via `child_process`.

**Issue 3 — Generic AI answers without company context**

When I first tested the AI diagnosis without any context, OpenAI gave generic troubleshooting steps that didn't match our actual company setup — like generic VPN advice instead of steps specific to our VPN provider and internal policies.

**Fix:** I implemented RAG — indexed our company documents (VPN guide, installation manual, SOPs, security policies) into a vector database, and retrieved only the most relevant chunks for each issue to include in the prompt. This made the AI's answers specific and accurate to our actual environment instead of generic.

**Issue 4 — Agent needed to detect pending updates/failed installs reliably**

Just installing once wasn't enough — devices could have failed installs, or Windows Updates pending, and this needed to be tracked continuously without overloading the backend with requests.

**Fix:** I built a scheduled check inside the agent (running every few minutes) that compared installed software against the required policy list and checked update status, then sent only the delta/status as part of telemetry — rather than re-sending full device state every time.

---

## Quick Recap (30-second version)

- Built an AI-powered endpoint management platform to automate remote employee onboarding across Windows/macOS
- Wrote the agent (Node.js/TypeScript) to register devices, install role-based software (Winget/PowerShell on Windows, Homebrew on macOS), and continuously monitor device health
- Built a Policy Engine (decides what to install by role) and relied on a Rule Engine (detects anomalies like high CPU, VPN down, failed updates) — both rule-based, no AI
- Built the AI/RAG pipeline: summarized telemetry → embedded company docs in a vector DB → similarity search → combined summary + relevant docs + question into a prompt → OpenAI → diagnosis + fix
- Built backend APIs for device registration, policy, telemetry, alerts, and AI diagnosis
- Result: ~50% reduction in onboarding time
- Key issues solved: avoided sending raw telemetry to OpenAI (cost/latency/privacy) by using Rule Engine + summarization; handled cross-platform install differences (Winget vs Homebrew) with an abstracted install layer; used RAG to fix generic AI answers; built scheduled checks for pending updates/failed installs
- My role: backend APIs + agent script + AI/RAG integration. Colleague (Prateek Raja): frontend dashboard only

![alt text](Device_mangement_agent_workflow.png)
