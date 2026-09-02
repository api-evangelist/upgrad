---
name: Capture and progress a partner prospect
description: Capture a referred learner as a prospect against a channel partner, list and read prospects, and record
  status transitions.
api: openapi/upgrad-partner-openapi.yml
operations:
- saveProspect
- getProspects
- getProspect
- updateProspect
- saveStatusData
- fetchLabelsByStatusTypes
---

# Capture and progress a partner prospect

## When to use this
Use this skill to record and progress the learner leads a channel partner refers to upGrad.

## Before you start
- Base URL `https://partner.upgrad.com`, header `AUTH-TOKEN: <key>`.
- `getProspects` takes Spring Data `pageable` and a QueryDSL `predicate` query parameter, both marked required. Neither is decomposed in the contract, so the filterable fields are not discoverable from the spec — get the expected serialisation from your upGrad contact before relying on filtering.
- Prospect writes are not idempotent and not reversible.

## Steps
1. **Read the status vocabulary first** — `GET /partner-status` (`fetchLabelsByStatusTypes`) so you write a status value the service accepts.
2. **Create the prospect** — `POST /prospect` (`saveProspect`).
3. **List prospects** — `GET /prospect` (`getProspects`), supplying `pageable` and `predicate`.
4. **Read one prospect** — `GET /prospect/{prospectId}/lead/{leadId}` (`getProspect`). Note it needs BOTH the prospect id and the lead id.
5. **Update the prospect** — `PUT /vpc-only/prospect/{leadId}/{lsqEmail}` (`updateProspect`). Keyed on the lead id and the LeadSquared email, not the prospect id.
6. **Record a status transition** — `POST /save-status` (`saveStatusData`).

## Errors
Same eight-status contract and `ErrorContext` envelope as every other Partner Service operation. A `409` on step 2 most likely means the lead already exists — re-read with step 3 or 4 rather than retrying the create.
