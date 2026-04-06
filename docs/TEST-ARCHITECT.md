# End-to-End Test Script -- Architect / Technical Evaluator

**System under test:** WIP Manufacturing Blockchain
**Base URL:** https://wip.rodrigoandremarques.com
**Last updated:** 2026-04-06

This document provides a complete API-level end-to-end test suite for all three user roles. Tests are executed via `curl` and cover authentication, authorization enforcement, full manufacturing workflows, blockchain integrity, and edge cases. No browser is required.

---

## Setup

Define these variables once before running any test block:

```bash
BASE="https://wip.rodrigoandremarques.com"
VISITOR="demo:WIP-Demo-2026!"
OPERATOR="test-operator:TestOp-2026!"   # Replace after creating via Admin
ADMIN="admin:YOUR_ADMIN_PASSWORD"       # From Render env var ADMIN_PASSWORD
```

All curl commands use `-s` (silent) and pipe through `jq` for readability. Install jq if needed: `brew install jq` (macOS) or `apt install jq` (Linux).

---

## Section 1 -- Health and Connectivity

### T-01 -- Health check (no authentication required)

```bash
curl -s "$BASE/api/health" | jq '.'
```

**Expected response:**
```json
{ "status": "ok", "uptime": <number> }
```

**Pass criteria:** HTTP 200. `status` is `"ok"`. `uptime` is a positive number.

**Fail if:** Connection refused (cold start -- wait 30-50s and retry), or HTTP 5xx.

---

### T-02 -- Server statistics (no authentication required)

```bash
curl -s "$BASE/api/stats" | jq '.'
```

**Expected response:** JSON object with fields such as `totalBlocks`, `totalOrders`, `chainValid`.

**Pass criteria:** HTTP 200. `chainValid` is `true`. `totalBlocks` >= 1 (genesis block always exists).

**Fail if:** `chainValid` is `false`, or response is not valid JSON.

---

## Section 2 -- Authentication

### T-03 -- Visitor authentication (valid credentials)

```bash
curl -s -u "$VISITOR" "$BASE/api/auth/me" | jq '.'
```

**Expected response:**
```json
{ "success": true, "data": { "username": "demo", "role": "visitor" } }
```

**Pass criteria:** HTTP 200. `role` is `"visitor"`.

---

### T-04 -- Invalid credentials (wrong password)

```bash
curl -s -o /dev/null -w "%{http_code}" -u "demo:WRONG_PASSWORD" "$BASE/api/auth/me"
```

**Expected response:** `401`

**Pass criteria:** HTTP 401. No session is established.

---

### T-05 -- No credentials on protected endpoint

```bash
curl -s -o /dev/null -w "%{http_code}" "$BASE/api/orders"
```

**Expected response:** `401`

**Pass criteria:** HTTP 401. Endpoint is not accessible without authentication.

---

### T-06 -- Admin authentication

```bash
curl -s -u "$ADMIN" "$BASE/api/auth/me" | jq '.'
```

**Expected response:**
```json
{ "success": true, "data": { "username": "admin", "role": "admin" } }
```

**Pass criteria:** HTTP 200. `role` is `"admin"`.

---

## Section 3 -- Authorization Matrix

The following tests verify that each role is correctly restricted. Every test in this section should return HTTP 403.

### T-07 -- Visitor cannot create a production order

```bash
curl -s -o /dev/null -w "%{http_code}" -u "$VISITOR" \
  -X POST "$BASE/api/orders" \
  -H "Content-Type: application/json" \
  -d '{"product":"TEST","quantity":1}'
```

**Expected response:** `403`

---

### T-08 -- Visitor cannot post a goods movement

```bash
curl -s -o /dev/null -w "%{http_code}" -u "$VISITOR" \
  -X POST "$BASE/api/goods-receipt" \
  -H "Content-Type: application/json" \
  -d '{"materialCode":"RM-0001","quantity":10,"supplier":"TEST"}'
```

**Expected response:** `403`

---

### T-09 -- Visitor cannot access user management

```bash
curl -s -o /dev/null -w "%{http_code}" -u "$VISITOR" "$BASE/api/admin/users"
```

**Expected response:** `403`

---

### T-10 -- Operator cannot access user management

```bash
curl -s -o /dev/null -w "%{http_code}" -u "$OPERATOR" "$BASE/api/admin/users"
```

**Expected response:** `403`

**Note:** Run this test after creating the Operator user in T-26.

---

### T-11 -- Operator cannot access ERP Bridge export

```bash
curl -s -o /dev/null -w "%{http_code}" -u "$OPERATOR" "$BASE/api/erp/export"
```

**Expected response:** `403`

---

## Section 4 -- Visitor E2E (read-only API)

### T-12 -- List production orders

```bash
curl -s -u "$VISITOR" "$BASE/api/orders" | jq '.'
```

**Expected response:** JSON array of production orders (may be empty if none exist). HTTP 200.

---

### T-13 -- View blockchain chain

```bash
curl -s -u "$VISITOR" "$BASE/api/chain" | jq '. | length'
```

**Expected response:** A positive integer (at least 1 -- genesis block).

---

### T-14 -- View inventory

```bash
curl -s -u "$VISITOR" "$BASE/api/inventory" | jq '.'
```

**Expected response:** HTTP 200. JSON array of inventory positions.

---

### T-15 -- View material master

```bash
curl -s -u "$VISITOR" "$BASE/api/materials" | jq '.'
```

**Expected response:** HTTP 200. JSON array of material definitions.

---

### T-16 -- View work centers

```bash
curl -s -u "$VISITOR" "$BASE/api/work-centers" | jq '.'
```

**Expected response:** HTTP 200. JSON array of work center definitions.

---

### T-17 -- View routings

```bash
curl -s -u "$VISITOR" "$BASE/api/routings" | jq '.'
```

**Expected response:** HTTP 200. JSON array of routing definitions.

---

### T-18 -- View WIP report

```bash
curl -s -u "$VISITOR" "$BASE/api/reports/wip" | jq '.'
```

**Expected response:** HTTP 200. JSON WIP summary.

---

### T-19 -- View movement history report

```bash
curl -s -u "$VISITOR" "$BASE/api/reports/movements" | jq '.'
```

**Expected response:** HTTP 200. JSON array of movement records.

---

## Section 5 -- Operator Full E2E (write workflow)

This section executes a complete production order lifecycle via API. Run these tests in sequence. Each step uses the output of the previous step.

### T-20 -- Post Goods Receiving (inbound raw material)

```bash
curl -s -u "$OPERATOR" \
  -X POST "$BASE/api/goods-receipt" \
  -H "Content-Type: application/json" \
  -d '{
    "materialCode": "RM-0001",
    "quantity": 100,
    "unit": "KG",
    "supplier": "TEST-SUPPLIER-01",
    "purchaseOrder": "PO-TEST-001",
    "location": "WH-RAW"
  }' | jq '.'
```

**Expected response:** HTTP 200 or 201. Response contains event type `GOODS_RECEIPT` or equivalent. Blockchain block index increments.

**Capture:** Note the new inventory quantity at WH-RAW for use in T-21.

---

### T-21 -- Stage to Production

```bash
curl -s -u "$OPERATOR" \
  -X POST "$BASE/api/inventory/stage" \
  -H "Content-Type: application/json" \
  -d '{
    "materialCode": "RM-0001",
    "quantity": 50,
    "fromLocation": "WH-RAW",
    "toLocation": "PROD-AREA-01"
  }' | jq '.'
```

**Expected response:** HTTP 200. Blockchain event recorded. WH-RAW stock decreases by 50. PROD-AREA-01 stock increases by 50.

**Verify:**
```bash
curl -s -u "$OPERATOR" "$BASE/api/inventory" | jq '.[] | select(.materialCode == "RM-0001")'
```

---

### T-22 -- Create Production Order

```bash
ORDER=$(curl -s -u "$OPERATOR" \
  -X POST "$BASE/api/orders" \
  -H "Content-Type: application/json" \
  -d '{
    "product": "FG-0001",
    "productSku": "FINISHED-GOOD-01",
    "quantity": 10,
    "priority": "NORMAL",
    "costCenter": "CC-TEST-01",
    "createdBy": "test-operator",
    "dueDate": "2026-12-31"
  }')
echo "$ORDER" | jq '.'
ORDER_ID=$(echo "$ORDER" | jq -r '.data.id // .id')
echo "Order ID: $ORDER_ID"
```

**Expected response:** HTTP 201. Order status is `"Created"`. Order ID is captured for subsequent steps.

---

### T-23 -- Release Production Order

```bash
curl -s -u "$OPERATOR" \
  -X PUT "$BASE/api/orders/$ORDER_ID/release" | jq '.'
```

**Expected response:** HTTP 200. Order status is `"Released"`. Blockchain event `WORK_ORDER_RELEASED` recorded.

---

### T-24 -- Goods Issue (consume components against order)

```bash
curl -s -u "$OPERATOR" \
  -X POST "$BASE/api/orders/$ORDER_ID/goods-issue" \
  -H "Content-Type: application/json" \
  -d '{
    "materialCode": "RM-0001",
    "quantity": 10,
    "fromLocation": "PROD-AREA-01"
  }' | jq '.'
```

**Expected response:** HTTP 200. RM-0001 stock in PROD-AREA-01 decreases by 10. Blockchain event `GOODS_ISSUE` recorded and linked to `$ORDER_ID`.

---

### T-25 -- Confirm Operation

```bash
curl -s -u "$OPERATOR" \
  -X POST "$BASE/api/orders/$ORDER_ID/operations/1/confirm" \
  -H "Content-Type: application/json" \
  -d '{
    "yieldQuantity": 9,
    "scrapQuantity": 1,
    "workCenter": "WC-ASSEMBLY-01",
    "operator": "test-operator"
  }' | jq '.'
```

**Expected response:** HTTP 200. Blockchain event `OPERATION_CONFIRMED` recorded with yield=9, scrap=1.

**Note:** If the order has no routing defined, the operation ID may need to be omitted or set to `1`. Check the order details via `GET /api/orders/$ORDER_ID` for the operation structure.

---

### T-25b -- Post Quality Inspection

```bash
curl -s -u "$OPERATOR" \
  -X POST "$BASE/api/quality" \
  -H "Content-Type: application/json" \
  -d '{
    "orderId": "'"$ORDER_ID"'",
    "inspectionType": "FINAL_QC",
    "result": "PASS",
    "inspector": "test-operator",
    "observations": "All dimensions within tolerance"
  }' | jq '.'
```

**Expected response:** HTTP 200 or 201. Blockchain event `QM_INSPECTION` recorded. Quality record linked to `$ORDER_ID`.

---

### T-25c -- Goods Receipt (finished goods)

```bash
curl -s -u "$OPERATOR" \
  -X POST "$BASE/api/orders/$ORDER_ID/goods-receipt" \
  -H "Content-Type: application/json" \
  -d '{
    "quantity": 9,
    "location": "FG-WAREHOUSE"
  }' | jq '.'
```

**Expected response:** HTTP 200. FG-WAREHOUSE inventory increases by 9. Blockchain event `GOODS_RECEIPT` (finished goods) recorded.

---

### T-25d -- Transfer finished goods

```bash
curl -s -u "$OPERATOR" \
  -X POST "$BASE/api/inventory/transfer" \
  -H "Content-Type: application/json" \
  -d '{
    "materialCode": "FG-0001",
    "quantity": 5,
    "fromLocation": "FG-WAREHOUSE",
    "toLocation": "DISPATCH-AREA"
  }' | jq '.'
```

**Expected response:** HTTP 200. Stock balances updated. Blockchain event `TRANSFER` recorded.

---

### T-25e -- Scrap posting

```bash
curl -s -u "$OPERATOR" \
  -X POST "$BASE/api/inventory/scrap" \
  -H "Content-Type: application/json" \
  -d '{
    "materialCode": "RM-0001",
    "quantity": 1,
    "location": "PROD-AREA-01",
    "reasonCode": "DAMAGED"
  }' | jq '.'
```

**Expected response:** HTTP 200. PROD-AREA-01 stock decreases by 1. Blockchain event `SCRAP` recorded with reason code.

---

### T-25f -- Close Production Order

```bash
curl -s -u "$OPERATOR" \
  -X PUT "$BASE/api/orders/$ORDER_ID/close" | jq '.'
```

**Expected response:** HTTP 200. Order status is `"Closed"`. Blockchain event `WORK_ORDER_CLOSED` recorded.

---

## Section 6 -- Admin E2E

### T-26 -- Create Operator user

```bash
curl -s -u "$ADMIN" \
  -X POST "$BASE/api/admin/users" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "test-operator",
    "password": "TestOp-2026!",
    "role": "operator"
  }' | jq '.'
```

**Expected response:** HTTP 200 or 201. User created with role `"operator"`.

**Note:** Run this test before Section 5 to enable Operator tests.

---

### T-27 -- List all users

```bash
curl -s -u "$ADMIN" "$BASE/api/admin/users" | jq '.'
```

**Expected response:** HTTP 200. JSON array containing at least `admin`, `demo`, and `test-operator`.

---

### T-28 -- ERP Bridge export

```bash
curl -s -u "$ADMIN" "$BASE/api/erp/export" | jq '.'
```

**Expected response:** HTTP 200. JSON payload containing production orders and movements in ERP-compatible structure. At minimum, includes the order created in T-22.

---

### T-29 -- Delete Operator user

```bash
curl -s -u "$ADMIN" \
  -X DELETE "$BASE/api/admin/users/test-operator" | jq '.'
```

**Expected response:** HTTP 200. User deleted.

**Verify deletion:**
```bash
curl -s -o /dev/null -w "%{http_code}" -u "test-operator:TestOp-2026!" "$BASE/api/auth/me"
```

**Expected:** `401` -- deleted user can no longer authenticate.

---

## Section 7 -- Blockchain Integrity Verification

### T-30 -- Download full chain and verify block count

```bash
CHAIN=$(curl -s -u "$ADMIN" "$BASE/api/chain")
BLOCK_COUNT=$(echo "$CHAIN" | jq '. | length')
echo "Total blocks in chain: $BLOCK_COUNT"
```

**Expected:** `$BLOCK_COUNT` equals the total number of events generated throughout this test session (at minimum: genesis + goods receipt + stage + order created + released + goods issue + operation confirmed + QM inspection + finished GR + transfer + scrap + order closed = 12+ blocks).

---

### T-31 -- Verify hash chain linkage

```bash
CHAIN=$(curl -s -u "$ADMIN" "$BASE/api/chain")
echo "$CHAIN" | jq -r 'to_entries[] | "\(.key): index=\(.value.index) hash=\(.value.hash[0:12])... prev=\(.value.previousHash[0:12])..."'
```

**Manual verification:** For each block at index N, its `previousHash` must match the `hash` of the block at index N-1. Block 0 (genesis) has `previousHash` set to `"0"` or a defined genesis string.

---

### T-32 -- Chain integrity endpoint

```bash
curl -s -u "$ADMIN" "$BASE/api/stats" | jq '.chainValid'
```

**Expected response:** `true`

**Fail if:** `false`. This indicates a tampered or corrupted block in the chain.

---

### T-33 -- Audit trail completeness

```bash
curl -s -u "$ADMIN" "$BASE/api/reports/movements" | jq '[.[] | .type] | group_by(.) | map({type: .[0], count: length})'
```

**Expected:** Event types generated during this test session are all present: GOODS_RECEIPT, STAGING_TO_PRODUCTION (or equivalent), WORK_ORDER_CREATED, WORK_ORDER_RELEASED, GOODS_ISSUE, OPERATION_CONFIRMED, QM_INSPECTION, TRANSFER, SCRAP, WORK_ORDER_CLOSED.

**Fail if:** Any expected event type is missing from the movement history.

---

## Section 8 -- Cold Start and Resilience

### T-34 -- Cold start recovery

**Precondition:** The server has been idle for at least 15 minutes (Render free tier sleep).

```bash
time curl -s -o /dev/null -w "%{http_code}" "$BASE/api/health"
```

**Expected:** HTTP 200 returned within 60 seconds. The `time` output shows wall clock elapsed time between 30-60 seconds on first request after sleep.

**Pass criteria:** System recovers without manual intervention and returns a valid response.

---

### T-35 -- Data persistence after cold start

**Precondition:** T-34 passed (server woke up). Run T-22 through T-25f before the sleep period.

```bash
curl -s -u "$ADMIN" "$BASE/api/orders/$ORDER_ID" | jq '.status'
```

**Expected:** The order created in T-22 and closed in T-25f still exists with status `"Closed"`. All blockchain events are still present.

**Pass criteria:** Data written before the sleep period is fully intact after the server wakes up. This confirms that the JSON persistence layer survived the container restart.

**Note:** On Render's free tier, the filesystem is ephemeral -- data persists within a deployment but is lost if a new deployment is triggered. This is documented system behavior, not a bug.

---

## Section 9 -- Authorization Edge Cases

### T-36 -- Visitor cannot call Operator endpoint via direct URL

```bash
curl -s -o /dev/null -w "%{http_code}" -u "$VISITOR" \
  -X POST "$BASE/api/goods-receipt" \
  -H "Content-Type: application/json" \
  -d '{"materialCode":"RM-0001","quantity":1}'
```

**Expected:** `403`

---

### T-37 -- Visitor cannot close a production order

```bash
curl -s -o /dev/null -w "%{http_code}" -u "$VISITOR" \
  -X PUT "$BASE/api/orders/$ORDER_ID/close"
```

**Expected:** `403`

---

### T-38 -- Admin has Operator-level access (role inheritance)

```bash
curl -s -u "$ADMIN" "$BASE/api/orders" | jq '. | length'
```

**Expected:** HTTP 200. Admin can access all Operator endpoints. Same response structure as when called by Operator.

---

## Test Completion Summary

| Test ID | Section              | Description                                | Role     | Result |
|---------|----------------------|--------------------------------------------|----------|--------|
| T-01    | Connectivity         | Health check                               | None     |        |
| T-02    | Connectivity         | Server statistics                          | None     |        |
| T-03    | Authentication       | Visitor valid login                        | Visitor  |        |
| T-04    | Authentication       | Invalid credentials rejected               | None     |        |
| T-05    | Authentication       | No credentials rejected                    | None     |        |
| T-06    | Authentication       | Admin valid login                          | Admin    |        |
| T-07    | Authorization        | Visitor cannot create order                | Visitor  |        |
| T-08    | Authorization        | Visitor cannot post goods movement         | Visitor  |        |
| T-09    | Authorization        | Visitor cannot access user management      | Visitor  |        |
| T-10    | Authorization        | Operator cannot access user management     | Operator |        |
| T-11    | Authorization        | Operator cannot access ERP Bridge          | Operator |        |
| T-12    | Visitor E2E          | List production orders                     | Visitor  |        |
| T-13    | Visitor E2E          | View blockchain chain                      | Visitor  |        |
| T-14    | Visitor E2E          | View inventory                             | Visitor  |        |
| T-15    | Visitor E2E          | View material master                       | Visitor  |        |
| T-16    | Visitor E2E          | View work centers                          | Visitor  |        |
| T-17    | Visitor E2E          | View routings                              | Visitor  |        |
| T-18    | Visitor E2E          | WIP report                                 | Visitor  |        |
| T-19    | Visitor E2E          | Movement history report                    | Visitor  |        |
| T-20    | Operator E2E         | Goods Receiving                            | Operator |        |
| T-21    | Operator E2E         | Staging to Production                      | Operator |        |
| T-22    | Operator E2E         | Create Production Order                    | Operator |        |
| T-23    | Operator E2E         | Release Production Order                   | Operator |        |
| T-24    | Operator E2E         | Goods Issue                                | Operator |        |
| T-25    | Operator E2E         | Operation Confirmation                     | Operator |        |
| T-25b   | Operator E2E         | Quality Inspection                         | Operator |        |
| T-25c   | Operator E2E         | Goods Receipt (finished goods)             | Operator |        |
| T-25d   | Operator E2E         | Transfer between locations                 | Operator |        |
| T-25e   | Operator E2E         | Scrap posting                              | Operator |        |
| T-25f   | Operator E2E         | Close Production Order                     | Operator |        |
| T-26    | Admin E2E            | Create Operator user                       | Admin    |        |
| T-27    | Admin E2E            | List all users                             | Admin    |        |
| T-28    | Admin E2E            | ERP Bridge export                          | Admin    |        |
| T-29    | Admin E2E            | Delete Operator user                       | Admin    |        |
| T-30    | Blockchain Integrity | Block count verification                   | Admin    |        |
| T-31    | Blockchain Integrity | Hash chain linkage                         | Admin    |        |
| T-32    | Blockchain Integrity | Chain integrity endpoint                   | Admin    |        |
| T-33    | Blockchain Integrity | Audit trail completeness                   | Admin    |        |
| T-34    | Resilience           | Cold start recovery                        | None     |        |
| T-35    | Resilience           | Data persistence after cold start          | Admin    |        |
| T-36    | Auth Edge Cases      | Visitor blocked from Operator endpoint     | Visitor  |        |
| T-37    | Auth Edge Cases      | Visitor cannot close order                 | Visitor  |        |
| T-38    | Auth Edge Cases      | Admin inherits Operator access             | Admin    |        |
