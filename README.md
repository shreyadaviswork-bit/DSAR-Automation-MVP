# DSAR-Automation-MVP
DSAR Automation App

Automate Data Subject Access Requests (DSARs) with Google Sheets and Apps Script.

**Overview**
This project demonstrates a lightweight DSAR automation prototype for GDPR compliance. It allows organizations to:

Capture DSARs from users
Automatically send acknowledgment emails
Track requests in a Google Sheet
Categorize requests by type (Access, Deletion, Correction)

This is a working prototype that doesn’t require backend hosting, making it ideal for demos or MVPs.

**Problem Statement**
Organizations must respond to DSARs under GDPR within strict timelines. Manual handling of requests can be slow, error-prone, and difficult to track.
Solution: Automate DSAR handling with Google Sheets + Apps Script to streamline intake, acknowledgment, and tracking.

**Key Features**

DSAR Intake Form: Capture user name, email, and request type.
Automatic Acknowledgment Emails: Sends confirmation to the user when a request is submitted.
Request Tracking: Google Sheet stores all requests with status updates.
Categorization: Requests are labeled as Access, Deletion, or Correction for easier workflow management.
Automation: Apps Script automatically processes pending requests and updates the sheet.

**Tech Stack**

Frontend / Input: Google Forms (optional, linked to Sheets)
Backend / Processing: Google Apps Script
Database / Storage: Google Sheets
Email Automation: Google Apps Script MailApp

**User stories **

Intake: “As a data subject, I can submit a DSAR with my email and request type so I can exercise my rights.”
AC: Form required fields; confirmation email sent.

Workflow: “As a compliance officer, I can move a request through New → In Review → Completed to track progress.”
AC: Status visible in the Requests sheet; DueDate auto-set to +30 days.

Discovery/Collection (simulated): “As a compliance officer, when I complete a request, the system gathers subject data from source systems.”
AC: Matching rows by Email are included in the package.

Redaction: “As a compliance officer, the output redacts PII.”
AC: Email masked, names partially masked, phone masked (all but last 2 digits).

Delivery: “As a data subject, I receive a secure data package.”
AC: Email with ZIP containing PDF + JSON.

Audit: “As a privacy lead, I can see a log of key actions.”
AC: Audit tab records RequestCreated and PackageSent with timestamps.

8) Success metrics you can claim

Lead time per request (form → delivery) under 1 day (demo).
Zero manual copy/paste from source tabs (scripted).
100% auditable (Audit tab entries).
Basic redaction applied consistently.

**How It Works (Workflow)**

User submits a DSAR via Google Form → Google Sheet
Apps Script checks for pending requests
Script sends an acknowledgment email
Updates the sheet: Status → Completed, Response Sent → Yes
Requests can be manually or automatically marked Completed

  <img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/e41bd11b-8040-4261-a406-56172f3bd42d" />

Setup Instructions

Create Google Sheet
Name: DSAR_MVP\
Columns: Timestamp | Name | Email | Request Type | Status | Response Sent

Create Google Form 

Fields: Name, Email, Request Type
Link to the Google Sheet

Add Apps Script
Go to Extensions → Apps Script
Paste the following code:code.gs and add triggers
  
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
