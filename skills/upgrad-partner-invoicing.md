---
name: Report on partner invoices and credit notes
description: Read the partner invoice and credit-note ledger, fetch supporting documents, and update credit-note
  details.
api: openapi/upgrad-partner-openapi.yml
operations:
- getInvoiceStatusFlow
- getPartnerInvoiceCreditNoteReport
- getPartnerInvoiceCreditNoteDashboard
- uploadInvoiceDocument
- updatePartnerInvoiceCreditDetails
- getInvoiceDocumentUrl
- getCommissionRules
- getCommissionDetail
---

# Report on partner invoices and credit notes

## When to use this
Use this skill to reconcile what upGrad owes a channel partner: commission rules, the invoice/credit-note ledger, and the documents behind each line.

## Before you start
- Base URL `https://partner.upgrad.com`, header `AUTH-TOKEN: <key>`.
- This flow touches money. There is **no idempotency key and no reversal operation**, and upGrad publishes no reversal window. A credit note is modelled as a record here, not as an operation that voids an invoice — do not treat `updatePartnerInvoiceCreditDetails` as an undo.
- Read before you write. Every step 1–5 below is safe; only steps 6 and 7 mutate.

## Steps
1. **Read the commission rules in force** — `GET /commission-rule` (`getCommissionRules`), then `GET /commission-rule/{commissionId}` (`getCommissionDetail`) for a specific rule.
2. **Read the invoice status machine** — `GET /invoice-status-flow` (`getInvoiceStatusFlow`) so you can interpret the status on each ledger row.
3. **Pull the dashboard rollup** — `GET /invoice-credit-note-dashboard` (`getPartnerInvoiceCreditNoteDashboard`).
4. **Pull the detailed report** — `GET /partner-invoice-credit-note-report` (`getPartnerInvoiceCreditNoteReport`). Optional `email`, `mobile` and `invoiceMonth` query parameters narrow it.
5. **Resolve a supporting document** — `GET /invoice-document-url/{dmsUploadId}` (`getInvoiceDocumentUrl`) returns the document URL for a ledger row.
6. **Attach a document** — `POST /invoice/{userId}/document` (`uploadInvoiceDocument`).
7. **Update credit-note details** — `PUT /update-partner-invoice-credit-note` (`updatePartnerInvoiceCreditDetails`), keyed by the `partnerInvoiceCreditNoteId` and `partnerInvoiceFlowId` query parameters.

## Errors
Eight-status contract, `ErrorContext` envelope, `*/*` media type. Because there is no idempotency contract, a timeout on step 6 or 7 is ambiguous — re-run step 4 or 5 to establish the real state before re-issuing.
