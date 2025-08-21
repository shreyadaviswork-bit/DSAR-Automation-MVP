# DSAR-Automation-MVP
DSAR Automation Platform – Mini Case Study
Overview

This project demonstrates a Data Subject Access Request (DSAR) Automation MVP built using Google Forms, Sheets, Apps Script, Gmail, and Drive.

Objective: Automate the DSAR workflow to comply with GDPR/CPRA, reducing manual effort and ensuring traceability.
| Feature              | Description                                                                               |
| -------------------- | ----------------------------------------------------------------------------------------- |
| **Intake Form**      | Google Form collects **Full Name, Email, Request Type** (Access / Deletion / Correction). |
| **Request Tracking** | Submissions stored in `Requests` sheet with **TicketID, Status, DueDate**.                |
| **Workflow View**    | Filter view for **New / In Review / Completed** requests.                                 |
| **Redaction**        | Sensitive fields in CRM/Billing mock data are masked automatically.                       |
| **Package Delivery** | Generates **JSON + PDF**, zips them, and emails to the requester.                         |
| **Audit Trail**      | Actions logged in `Audit` tab for compliance and traceability.                            |


[Google Form Submission] 
        ↓
  [Requests Sheet] 
        ↓
  [Apps Script Trigger]
        ↓
  [Status = Completed?] → Yes → [buildAndSendPackage]
        ↓
  [Redact Data from Mock_CRM & Mock_Billing]
        ↓
  [Generate JSON + PDF] → [Zip]
        ↓
  [Email DSAR Package]
        ↓
  [Log Audit]

  <img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/e41bd11b-8040-4261-a406-56172f3bd42d" />

  
**Key Learnings / Impact**

Automated DSAR process reduces manual work and human errors.
Fully implemented in Google Workspace, no external servers required.
Provides audit logs for compliance verification.
Demonstrates end-to-end product thinking: intake → processing → delivery → compliance.

**How to Run / Test**

Submit a new request via the Google Form.
Observe auto-generated TicketID, Status, DueDate in Requests sheet.
Change Status → Completed to trigger DSAR package creation and email delivery.
Verify Audit tab for logged actions.
