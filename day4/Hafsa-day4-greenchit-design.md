# GreenChit — Architecture Design Pack

## 1. System Context (one-paragraph recap of Day 1-style context)

GreenChit is a reimbursement platform for BISTEC staff. Employees submit expense claims with receipts, line managers approve or reject them via Microsoft Teams, and Finance exports approved claims to payroll. The system authenticates via Microsoft Entra ID, stores files in Azure Blob Storage, and is hosted entirely on BISTEC's Azure tenant.

## 2. Containers (C4 Level 2) — embedded PNG + table of containers

# Table of Containers
| # | Name | Type | Technology | Responsibility |
|---|---|---|---|---|
| 1 | Web Application | Internal | Blazor WebAssembly, Azure Static Web Apps | Renders UI |
| 2 | API Service | Internal | ASP.NET Core .NET 8, Azure Container Apps | Orchestrates claim lifecycle, enforces access control, and triggers notifications |
| 3 | Database | Internal | Azure SQL | Stores claims, line items, user-role mappings, and tamper-evident audit log with row-level security |
| 4 | Blob Storage | Internal | Azure Blob Storage | Stores receipt images. Access gated by short-lived SAS URLs |
| 5 | Microsoft Entra ID | External | Microsoft | Issues and validates JWT token for SSO |
| 6 | Microsoft Teams | External | External System | Displays claim status notifications to staff and managers |
| 7 | SharePoint | External | External System | Receives approved-claims CSV |
| 8 | Email | External | External System | Fallback notification channel when Teams webhook fails |
| 9 | Payroll System | External | External System | Watches SharePoint folder and reads approved-claims CSV |

# Embedded PNG
![Container Diagram](Container.drawio.png)

## 3. Components (C4 Level 3) for the API service — embedded PNG + table of components

# Table of Componenets
**1. Component table**

| # | Component | What it does |
|---|---|---|
| 1 | Auth | The gatekeeper. Every request passes through here first. It checks the user's login token to confirm who they are and what role they have (staff, manager, finance). If the token is invalid, the request gets rejected before it reaches anything else. |
| 2 | Claims | The core of the system. Handles creating, viewing, updating, and moving claims through their lifecycle — Draft → Submitted → Approved/Rejected → Paid. When a claim changes state, it tells the Notifications and Audit components. |
| 3 | Receipts | Manages receipt image uploads. Instead of pushing images through the API (which would be slow), it generates a temporary secure link (SAS URL) that lets the browser upload directly to Blob Storage. Also checks that files aren't too big (≤10 MB) and there aren't too many (≤5 per claim). |
| 4 | Notifications | Sends alerts when something happens to a claim. Posts a formatted message (Adaptive Card) to the person's Microsoft Teams. If Teams is down, it sends an email instead. |
| 5 | Audit | Records everything. Every time a claim changes state, this component writes a log entry — who did it, when, what changed, and why (if rejected). The log is append-only, meaning entries can never be edited or deleted, which is required for financial compliance. |
| 6 | Export | When Finance clicks the export button, this component queries the database for all approved claims, formats them into a CSV file, and uploads it to the SharePoint folder where the payroll system picks it up. |
| 7 | Authorization | Auth confirms *who you are* (authentication). Authorization decides *what you can see and do*. For example: a manager can only approve claims from their own team, staff can only see their own claims, finance can see all approved claims but can't approve them. This is the component that enforces the privacy NFR. |

# Embedded PNG
![Component Diagram](Component.drawio.png)

## 4. Reading order — how a reviewer should walk through the diagrams

2. Container diagram 
Start with the three actors at the top — Staff, Manager, and Finance. All three interact with the same client application, which is the only thing users touch directly. The client application authenticates against Microsoft Entra ID and sends requests to the API service. The API service is the centre of the diagram — everything flows through it. From there, follow the arrows to the two data stores: Azure SQL DB for structured data and Blob Storage for receipt images. Then follow the arrows to to see the outbound integrations (yellow): Teams for notifications, Email as a fallback, and SharePoint for CSV exports. At the bottom, SharePoint connects to the Payroll system, which watches the folder and picks up the CSV files that the API dropped there.

2. Component diagram 
Start at the top with the client application  where every request begins. The first component it hits is Auth, which validates the user's login token. Once the token is confirmed, the request passes to Authorization, which checks whether that user is allowed to do what they're asking. Only then does the request reach one of the four business components — Claims for anything related to creating or changing a claim, Receipts for uploading receipt images, Export for generating the payroll CSV, or Notifications which gets triggered by Claims whenever a claim changes state.The arrows pointing outward from each component show which external system it talks to and over what protocol.

