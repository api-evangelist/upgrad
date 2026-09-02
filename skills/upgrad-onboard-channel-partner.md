---
name: Onboard a channel partner
description: Register a new upGrad channel partner, upload and verify their KYC documents, and move them through
  the partner status pipeline.
api: openapi/upgrad-partner-openapi.yml
operations:
- savePartnerDetails
- getPartnerInfo
- uploadPartnerDocument
- getPartnerDocuments
- verifyDocument
- updateAddressStatus
- updatePartnerStage
- updateVendorCode
---

# Onboard a channel partner

## When to use this
Use this skill to take a new upGrad channel partner from first record to a verified, transactable state on the Partner Service.

## Before you start
- Base URL: `https://partner.upgrad.com`
- Every operation requires the header `AUTH-TOKEN: <key>`. The key is issued by upGrad; there is no self-serve signup.
- There is **no sandbox and no test mode**. Every call below writes to production.
- There is **no idempotency key**. If a call times out, do NOT blindly retry a write — read back with `getPartnerInfo` first and only re-issue if the record is genuinely absent.
- There is **no reversal operation** for anything in this flow. Document verification and stage transitions cannot be undone through the API.

## Steps
1. **Create the partner record** — `POST /user/{userId}/partner-detail` (`savePartnerDetails`). `userId` is a path parameter and is required.
2. **Read it back** — `GET /user/{userId}/partner-detail` (`getPartnerInfo`). Confirm the write landed before doing anything irreversible.
3. **Upload each KYC document** — `POST /user/{userId}/partner/document` (`uploadPartnerDocument`), once per document.
4. **List what is on file** — `GET /user/{userId}/partner/documents` (`getPartnerDocuments`) to confirm every expected document is present before verification.
5. **Verify** — `POST /user/{userId}/verify/document` (`verifyDocument`). Irreversible: no un-verify operation exists.
6. **Set the address status** — `PUT /user/{userId}/address-status` (`updateAddressStatus`).
7. **Advance the partner stage** — `PUT /user/{userId}/partner/stage/{type}` (`updatePartnerStage`). Forward-only; there is no rollback operation.
8. **Attach the finance vendor code** — `PUT /vpc-only/user/{userId}/vendorCode` (`updateVendorCode`), required before the partner can be invoiced.

## Errors
The Partner Service declares the same eight statuses on every operation — 400, 403, 404, 405, 409, 429, 500, 503 — and returns a vendor `ErrorContext` body (`errorCode`, `messages`, `data`) under media type `*/*`. There is no RFC 9457 problem document and no published error-code registry, so branch on the HTTP status and surface `messages` verbatim rather than trying to parse `errorCode`.

On **429**, back off exponentially: upGrad publishes no limit, no window and no `Retry-After` header, so you cannot compute a correct delay.
