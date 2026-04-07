# Changelog

All documentation and system changes are recorded here.
Format: [YYYY-MM-DD] Type -- Description

---

## 2026-04-07

**[DOCS] Rewrite** -- Corrected all API endpoint names, request body field names, and
blockchain event type codes across API-REFERENCE.md, ARCHITECTURE.md, TEST-ADMIN.md,
TEST-ENDUSER.md, and USER-GUIDE.md to match the actual server.js implementation.
Previous versions used generic names (GOODS_RECEIPT, WORK_ORDER_CREATED etc.);
corrected to SAP-style codes (GOODS_RECEIPT_101, WO_CREATED, GOODS_ISSUE_261, etc.).

**[DOCS] Reorganize** -- Restructured docs/ folder into subfolders by type:
guides/ (USER-GUIDE, CONFIGURATION), technical/ (ARCHITECTURE, API-REFERENCE),
testing/ (TEST-ENDUSER, TEST-ADMIN), logs/ (this file).

**[DOCS] Add** -- TEST-ADMIN.md: complete admin E2E test script covering infrastructure
validation, user provisioning, reference data setup, full production order lifecycle
via API with correct curl commands, blockchain integrity checks, ERP bridge validation,
and resilience testing.

**[DOCS] Add** -- TEST-ENDUSER.md: detailed browser-based step-by-step test script for
Visitor and Operator roles with exact form fields, status codes, and expected results.

---

## 2026-04-06

**[DOCS] Add** -- Initial public documentation suite:
README.md, USER-GUIDE.md, ARCHITECTURE.md, API-REFERENCE.md, CONFIGURATION.md.

**[DOCS] Add** -- Problem statement, industries, blockchain rationale, key features,
data flow example, and license section to README.md.

**[SYSTEM] Fix** -- Added VISITOR_PASSWORD env var in Render dashboard to seed demo
account on startup. Confirmed demo credentials working via /api/auth/me.

**[SYSTEM] Fix** -- Added wip.rodrigoandremarques.com as custom domain in Render.
SSL certificate provisioned via Let's Encrypt. HTTPS confirmed working.

---

## Change Request Log

Use this section to record requested changes before implementation.

| Date       | Requested by | Description                        | Status   |
|------------|--------------|------------------------------------|----------|
| 2026-04-07 | R. Marques   | Fix encoding issues, remove emoji  | Done     |
| 2026-04-07 | R. Marques   | Add problem/industries to README   | Done     |
| 2026-04-07 | R. Marques   | Rewrite test scripts (E2E)         | Done     |
| 2026-04-07 | R. Marques   | Fix all event types and endpoints  | Done     |
| 2026-04-07 | R. Marques   | Reorganize docs folder structure   | Done     |
