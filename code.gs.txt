const SHEET_REQUESTS = 'Requests';
const SHEET_AUDIT = 'Audit';
const SHEET_CRM = 'Mock_CRM';
const SHEET_BILLING = 'Mock_Billing';

function onFormSubmit(e) {
  const ss = SpreadsheetApp.getActive();
  const sh = ss.getSheetByName(SHEET_REQUESTS);
  
  // e.values contains the submitted row (array)
  const values = e.values; // Timestamp | FullName | Email | RequestType

  // Determine row that was just added
  const row = sh.getLastRow();

  const timestamp = values[0];
  const fullName = values[1];
  const email = values[2];
  const requestType = values[3];

  // Generate TicketID
  const ticketId = 'DSAR-' + Utilities.formatString('%04d', row - 1);

  // Fill in TicketID, Status, DueDate
  sh.getRange(row, 5).setValue(ticketId);       // TicketID (col B)
  sh.getRange(row, 6).setValue('New');          // Status (col F)
  sh.getRange(row, 7).setValue(new Date(new Date(timestamp).getTime() + 30*24*60*60*1000)); // DueDate

  logAudit(ticketId, 'RequestCreated', 'System', `Type=${requestType}; Email=${email}`);
}


function logAudit(ticketId, action, actor, notes) {
  const sh = SpreadsheetApp.getActive().getSheetByName(SHEET_AUDIT);
  sh.appendRow([new Date(), ticketId, action, actor, notes || '']);
}
function onEdit(e) {
  const sh = e.range.getSheet();
  if (sh.getName() !== SHEET_REQUESTS) return;

  const editedCol = e.range.getColumn();
  const STATUS_COL = 6; // F
  if (editedCol !== STATUS_COL) return;

  const row = e.range.getRow();
  const values = sh.getRange(row, 1, 1, sh.getLastColumn()).getValues()[0];

 const fullName = values[1];     // Full Name
const email = values[2];        // Email
const requestType = values[3];  // Request Type
const ticketId = values[4];     // TicketID
const status = values[5];       // Status

if (status === 'Completed') {
  buildAndSendPackage({ ticketId, fullName, email, requestType });
}
}

function buildAndSendPackage(ctx) {
  const ss = SpreadsheetApp.getActive();
  const crm = ss.getSheetByName(SHEET_CRM).getDataRange().getValues();
  const billing = ss.getSheetByName(SHEET_BILLING).getDataRange().getValues();

  const crmHeader = crm.shift();
  const billingHeader = billing.shift();

  const crmMatches = crm.filter(r => (r[0] + '').toLowerCase() === ctx.email.toLowerCase());
  const billingMatches = billing.filter(r => (r[0] + '').toLowerCase() === ctx.email.toLowerCase());

  const redactedCrm = crmMatches.map(r => ({
    Email: maskEmail(r[0]),
    FullName: maskName(r[1]),
    Phone: maskPhone(r[2]),
    Address: r[3] ? 'REDACTED' : ''
  }));

  const redactedBilling = billingMatches.map(r => ({
    Email: maskEmail(r[0]),
    InvoiceId: r[1],
    Amount: r[2],
    Last4: r[3],
    InvoiceDate: r[4]
  }));

  const payload = {
    request: {
      ticketId: ctx.ticketId,
      type: ctx.requestType,
      subjectEmailMasked: maskEmail(ctx.email),
      completedAt: new Date()
    },
    sources: {
      crm: redactedCrm,
      billing: redactedBilling
    }
  };

  // Create JSON blob
  const jsonBlob = Utilities.newBlob(JSON.stringify(payload, null, 2), 'application/json', `${ctx.ticketId}.json`);

  // Create a PDF summary (Doc -> export as PDF)
  const doc = DocumentApp.create(`${ctx.ticketId}_Summary`);
  const body = doc.getBody();
  body.appendParagraph('DSAR Fulfillment Summary').setHeading(DocumentApp.ParagraphHeading.HEADING1);
  body.appendParagraph(`Ticket: ${ctx.ticketId}`);
  body.appendParagraph(`Type: ${ctx.requestType}`);
  body.appendParagraph(`Subject (masked): ${maskEmail(ctx.email)}`);
  body.appendParagraph(`CRM records: ${redactedCrm.length}`);
  body.appendParagraph(`Billing records: ${redactedBilling.length}`);
  body.appendParagraph(`Generated: ${new Date().toLocaleString()}`);
  doc.saveAndClose();
  const pdfBlob = DriveApp.getFileById(doc.getId()).getAs(MimeType.PDF).setName(`${ctx.ticketId}_Summary.pdf`);

  // Zip JSON + PDF
  const zipBlob = Utilities.zip([jsonBlob, pdfBlob], `${ctx.ticketId}_Package.zip`);

  // Email to requester
  MailApp.sendEmail({
    to: ctx.email,
    subject: `Your DSAR package – ${ctx.ticketId}`,
    htmlBody: `Hello ${ctx.fullName || ''},<br><br>Your DSAR package is attached.<br><br>Regards,<br>Privacy Team`,
    attachments: [zipBlob],
    name: 'Privacy Team'
  });

  logAudit(ctx.ticketId, 'PackageSent', 'System', `Emailed ZIP to ${maskEmail(ctx.email)}`);
}

// --- Redaction helpers (simple demo logic) ---
function maskEmail(s) {
  if (!s) return '';
  const parts = s.split('@');
  return parts[0].slice(0, 1) + '***@' + parts[1];
}
function maskName(s) {
  if (!s) return '';
  return s.slice(0, 1) + '***';
}
function maskPhone(s) {
  if (!s) return '';
  s = String(s).replace(/\D/g, '');
  if (s.length <= 2) return '**';
  return '*'.repeat(s.length - 2) + s.slice(-2);
}
