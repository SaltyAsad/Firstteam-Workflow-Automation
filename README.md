<div align="center">
  <h1>🏢 First Team Real Estate</h1>
  <h3>Workflow Automation API Backbone</h3>
  <p>Automated integration logic for a fast, frictionless agent onboarding experience!</p>
</div>

---

## 🚀 Overview

Welcome to the **Firstteam-Workflow-Automation** repository! 

This project orchestrates the complete lifecycle of onboarding a new real estate agent to the First Team ecosystem. It functions as a powerful middleware/integration layer, built predominantly with **n8n**, to connect custom frontend forms with various third-party APIs and microservices. 

By automating manual, multi-step processes across different platforms, this tool ensures speed, compliance, and an error-free onboarding experience.

## 🔄 The Automated Onboarding Flow

We've completely automated the onboarding pipeline. Here's how the magic works under the hood:

1. 🔐 **SSO Authentication**: First Team administrators log into the master dashboard seamlessly using **Microsoft SSO**.
2. 📝 **Form Generation**: The Admin initiates an onboarding packet, creating a dynamic form entry that triggers downstream microservices.
3. 🎟️ **Ticketing & Comms Setup**: A support ticket is logged instantly (e.g., in **Zendesk**), and initial notification emails are dispatched via integrated email services like Gmail/SMTP.
4. 🧑‍💻 **Initial Data Collection**: The Office Administrator (OA) completes the initial prerequisites on the form and electronically hands it off to the prospective agent. Data is secured and authorized via JWTs.
5. ✍️ **Agent Input & E-Signatures**: The agent fills in their details. Once saved, **DocuSign** integrations automatically trigger. Our workflow manages the routing of complex multi-signature contracts.
6. 🎉 **Finalization**: As soon as all signatures are successfully collected and verified, the agent is 100% onboarded into the First Team database.

---

## 🛠️ Deep Dive: Architecture & Endpoints

This repository structurally mirrors the visual n8n workflows that power this ecosystem. Below is a comprehensive breakdown of exactly what each microservice endpoint does under the hood.

### 📥 1. Data Retrieval (`Get APIs`)
These GET endpoints are responsible for securely serving data to the frontend dashboards.

* **`Get All Forms`**
  * **Flow**: Webhook ➡️ Verify & Code ➡️ Script ➡️ MongoDB (Find) ➡️ Respond
  * **Description**: Receives a secure GET request, validates the caller, executes a preparation script, and queries the MongoDB database to return a list of all active or generated onboarding forms.

* **`Get FormData`**
  * **Flow**: Webhook ➡️ JWT Decode ➡️ MongoDB (Find) ➡️ Code/Map ➡️ Switch (OA vs. Agent) ➡️ Respond
  * **Description**: Retrieves the state and schema of a specific onboarding form. Crucially, it decodes the JWT to determine if the requester is the Office Admin (OA) or the Agent, routing the logic through a switch node to serve the appropriate data payload for that specific user's view.

* **`Get Offices`**
  * **Flow**: Webhook ➡️ JWT Decode ➡️ MongoDB (Find) ➡️ Respond
  * **Description**: A streamlined endpoint that verifies the user token and fetches the current directory of active First Team branch offices so the frontend dropdowns are always accurate.

### 📤 2. Data Submission (`Post APIs`)
These POST endpoints handle state changes, microservice triggers, and incoming form actions.

* **`Post Generate Form`**
  * **Flow**: Webhook ➡️ Verify/Code ➡️ Switch ➡️ MongoDB (Insert) ➡️ Generate JWTs (OA & Agent) ➡️ Merge & Update Tokens ➡️ Zendesk (Create Ticket) ➡️ Respond
  * **Description**: The kick-off endpoint. When an Admin generates a new onboarding flow, this saves the initial record to the database, immediately generates secure JWTs for both the OA and the Agent, links those tokens back to the record, and opens a new onboarding tracking ticket in Zendesk.

* **`Post Submit Step`**
  * **Flow**: Webhook ➡️ JWT Decode ➡️ Map Model ➡️ MongoDB (Update) ➡️ Respond
  * **Description**: Acts as an autosave mechanism. As users (OA or Agent) progress through large, multi-step forms, this endpoint continuously decrypts their JWT, remaps the incoming JSON schema, and securely updates their partial progress in the database.

* **`Post Submit`**
  * **Flow**: Webhook ➡️ JWT Decode ➡️ Map Model ➡️ MongoDB (Update) ➡️ Switch Node (OA vs. Agent)
    * **OA Path**: Edit Fields ➡️ Gmail (Send Message) ➡️ Zendesk (Update Ticket) ➡️ Wait
    * **Agent Path**: DocuSign (Send Envelope) ➡️ Zendesk (Update Ticket)
  * **Description**: The heavy lifter for finalizing form submissions. It routes the final logic based on the submitter:
    * If the **OA** submits their half, the system sends an automated "Your Turn" email to the Agent via Gmail, updates the Zendesk tracker, and goes into a wait state.
    * If the **Agent** submits their final fields, the system fires off a DocuSign envelope to collect legal signatures and updates Zendesk to mark the paperwork phase as pending signatures.

### ⏲️ 3. Scheduled Tasks (Cron Jobs)

* **`Sync Offices`**
  * **Flow**: Schedule Trigger ➡️ Get Branches (API) ➡️ Map Schema ➡️ MongoDB (Update) ➡️ Wait (Loop)
  * **Description**: A background cron job that continuously polls external data sources to retrieve branch data, remaps it into our system's JSON schema, and pushes updates straight to MongoDB. This keeps office mappings in perfect sync without manual intervention.

---

## 💻 Tech Stack
- **Integration Engine**: [n8n](https://n8n.io/) (Visual Node-Based Workflows)
- **Primary Database**: MongoDB (Tracking application state, branch details, and workflow metadata)
- **Security & Auth**: JWT (JSON Web Tokens) for stateless session handling & role-routing
- **Integrations Orchestrated**:
  - **Microsoft Azure AD** (SSO & Identity)
  - **Zendesk** (Support & Internal Ticketing)
  - **DocuSign** (E-Signatures and Legal Contracts)
  - **Gmail/SMTP** (Automated comms and handoffs)

## 💡 Potential Future Enhancements

Because this is a decoupled, workflow-based architecture, adding new logic steps is highly modular:
1. **ChatOps Notifications**: Automatically drop a message into a Slack or Microsoft Teams channel whenever a new agent is fully onboarded.
2. **Automated Background Checks**: Insert an API call node right before the DocuSign trigger to securely interface with a 3rd-party background check service.
3. **Analytics Sync**: Add a daily chron job that pipes onboarding duration metrics into a reporting tool (like PowerBI or Metabase) to help optimize processing times and spot bottlenecks.

---

*Built for [First Team Real Estate](https://www.firstteam.com/) to handle the heavy lifting, so agents can focus on real estate.* 🏡
