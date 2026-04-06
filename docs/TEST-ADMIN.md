# Admin System Test Script -- Full E2E

**System:** WIP Manufacturing Blockchain
**URL:** https://wip.rodrigoandremarques.com
**Last updated:** 2026-04-06

This document is the authoritative end-to-end test script for system administrators.
It covers the complete lifecycle of a deployment: infrastructure validation, environment
configuration, reference data setup, user provisioning, functional validation of every
system module, blockchain integrity verification, and final sign-off by executing the
full end-user workflow.

The Admin E2E is designed to be executed on every new deployment or after any significant
system change. When all sections pass, the system is considered production-ready.

**Tools required:**
- A web browser (Chrome, Firefox, or Safari)
- A terminal with `curl` and `jq` installed
- Access to the Render.com dashboard for this service
- Admin credentials (ADMIN_PASSWORD from Render environment variables)

**Notation:**
- `$BASE` = https://wip.rodrigoandremarques.com
- `$ADMIN` = admin:YOUR_ADMIN_PASSWORD
- Commands marked with `[BROWSER]` are performed in the browser
- Commands marked with `[TERMINAL]` use curl in a terminal window

---

# SECTION 1 -- Infrastructure Validation

Before testing the application, validate that the hosting environment, DNS, and SSL
are correctly configured.

---

## A-01 -- Render service is live and healthy [TERMINAL]

```bash
curl -s -o /dev/null -w "HTTP %{http_code} | Time: %{time_total}s\n" \
  https://wip.rodrigoandremarques.com/api/health
```

**Expected:** `HTTP 200 | Time: <number>s`

If the response time exceeds 5 seconds, the server may be waking from sleep.
Wait 40 seconds and retry. If it does not respond within 90 seconds, check
the Render dashboard (https://dashboard.render.com) for service errors.

---

## A-02 -- Health endpoint content [TERMINAL]

```bash
curl -s https://wip.rodrigoandremarques.com/api/health | jq '.'
```

**Expected:**
```json
{ "status": "ok", "uptime": <positive number> }
```

`uptime` is the number of seconds the process has been running since the last
deploy or cold start. Any positive number is acceptable.

**Fail if:** `status` is not `"ok"`, or the response is not valid JSON.

---

## A-03 -- SSL certificate is valid [BROWSER]

1. Navigate to https://wip.rodrigoandremarques.com in a browser.
2. In the browser address bar, click the padlock icon.
3. Verify: "Connection is secure" or "Certificate is valid".
4. Check the certificate details: issued to `wip.rodrigoandremarques.com`,
   issued by Let's Encrypt, not expired.

**Expected:** Valid SSL certificate. No browser security warnings.

**Fail if:** Browser shows "Your connection is not private" (ERR_CERT_*),
or certificate is expired or issued to a different domain.

**Fix if failing:** Go to Render dashboard -> your service -> Settings -> Custom Domains.
Verify that `wip.rodrigoandremarques.com` is listed and shows "Certificate: Active".
If the domain is listed but the cert is pending, wait up to 10 minutes for provisioning.

---

## A-04 -- DNS resolution is correct [TERMINAL]

```bash
dig wip.rodrigoandremarques.com CNAME +short
```

**Expected:** Output points to a Render domain, e.g.:
`wip-blockchain.onrender.com.` (exact subdomain may differ)

Also verify the CNAME from GoDaddy side:
```bash
nslookup wip.rodrigoandremarques.com
```

**Expected:** Non-authoritative answer shows address resolving to Render's IP ranges.

**Fail if:** No CNAME record found, or the CNAME points to an incorrect target.
Check the GoDaddy DNS panel: the `wip` CNAME record must point to the Render service URL.

---

## A-05 -- Render environment variables are set [BROWSER]

1. Log in to https://dashboard.render.com
2. Navigate to your WIP Blockchain service.
3. Click **Environment** in the left sidebar.
4. Verify the following variables are present and have non-empty values:

   | Variable         | Required | Expected value / notes                   |
   |------------------|----------|------------------------------------------|
   | PORT             | Yes      | 3000                                     |
   | NODE_ENV         | Yes      | production                               |
   | ADMIN_USER       | Yes      | admin (or your chosen admin username)    |
   | ADMIN_PASSWORD   | Yes      | A strong password -- this is your admin key |
   | VISITOR_USER     | Yes      | demo                                     |
   | VISITOR_PASSWORD | Yes      | WIP-Demo-2026! (or your chosen value)    |

**Fail if:** ADMIN_PASSWORD is missing -- the server runs in open dev mode with no
authentication, which is not acceptable in production.
VISITOR_PASSWORD missing -- the demo/visitor account will not be seeded on startup.

---

## A-06 -- Last deployment is current [BROWSER]

1. In the Render dashboard, click **Events** for your service.
2. The most recent event should be a successful deploy: "Deploy live for <commit hash>".
3. Verify the deploy timestamp matches the expected last code update.

**Fail if:** The most recent event is "Deploy failed" -- check the build logs.

---

# SECTION 2 -- Authentication and Authorization Validation

Set up your environment variables before running terminal tests:

```bash
BASE="https://wip.rodrigoandremarques.com"
ADMIN="admin:YOUR_ADMIN_PASSWORD"        # Replace with actual password from Render
VISITOR="demo:WIP-Demo-2026!"           # Or your configured VISITOR_PASSWORD
```

---

## A-07 -- Admin authentication works [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/auth/me" | jq '.'
```

**Expected:**
```json
{ "success": true, "data": { "username": "admin", "role": "admin" } }
```

**Fail if:** HTTP 401 -- check that ADMIN_PASSWORD in Render matches what you set in `$ADMIN`.

---

## A-08 -- Visitor authentication works [TERMINAL]

```bash
curl -s -u "$VISITOR" "$BASE/api/auth/me" | jq '.'
```

**Expected:**
```json
{ "success": true, "data": { "username": "demo", "role": "visitor" } }
```

**Fail if:** HTTP 401 -- verify VISITOR_PASSWORD in Render equals the password in `$VISITOR`.
Then trigger a manual redeploy from Render so the server re-seeds the visitor account.

---

## A-09 -- Invalid credentials are rejected [TERMINAL]

```bash
curl -s -o /dev/null -w "%{http_code}" -u "admin:WRONGPASSWORD" "$BASE/api/auth/me"
```

**Expected:** `401`

---

## A-10 -- Unauthenticated access is rejected [TERMINAL]

```bash
curl -s -o /dev/null -w "%{http_code}" "$BASE/api/orders"
```

**Expected:** `401`

---

## A-11 -- Authorization matrix: Visitor blocked from write endpoints [TERMINAL]

Each of the following must return HTTP 403:

```bash
# Visitor cannot create an order
curl -s -o /dev/null -w "Create order: %{http_code}\n" -u "$VISITOR" \
  -X POST "$BASE/api/orders" -H "Content-Type: application/json" \
  -d '{"product":"TEST","quantity":1}'

# Visitor cannot post goods receipt
curl -s -o /dev/null -w "Goods receipt: %{http_code}\n" -u "$VISITOR" \
  -X POST "$BASE/api/goods-receipt" -H "Content-Type: application/json" \
  -d '{"materialCode":"RM-0001","quantity":1}'

# Visitor cannot access user management
curl -s -o /dev/null -w "User mgmt: %{http_code}\n" -u "$VISITOR" \
  "$BASE/api/admin/users"
```

**Expected:** All three lines print `403`.

---

## A-12 -- Authorization matrix: Operator blocked from admin endpoints [TERMINAL]

Run this after creating the Operator user in Section 3.

```bash
OPERATOR="test-operator:TestOp-2026!"

# Operator cannot list users
curl -s -o /dev/null -w "User list: %{http_code}\n" -u "$OPERATOR" \
  "$BASE/api/admin/users"

# Operator cannot access ERP Bridge
curl -s -o /dev/null -w "ERP export: %{http_code}\n" -u "$OPERATOR" \
  "$BASE/api/erp/export"

# Operator cannot create a user
curl -s -o /dev/null -w "Create user: %{http_code}\n" -u "$OPERATOR" \
  -X POST "$BASE/api/admin/users" -H "Content-Type: application/json" \
  -d '{"username":"hacker","password":"x","role":"admin"}'
```

**Expected:** All three lines print `403`.

---

# SECTION 3 -- User Provisioning

---

## A-13 -- List current users [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/admin/users" | jq '.'
```

**Expected:** A JSON array containing at minimum:
- `admin` with role `admin`
- `demo` with role `visitor`

If VISITOR_PASSWORD was not set in Render, `demo` may not appear. Fix by setting
VISITOR_PASSWORD in Render Environment and triggering a redeploy.

---

## A-14 -- Create Operator user [TERMINAL]

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

**Expected:** HTTP 200 or 201. Response confirms user created with role `operator`.

---

## A-15 -- Verify Operator user was created [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/admin/users" | jq '.[] | select(.role == "operator")'
```

**Expected:** At least one entry with role `operator` and username `test-operator`.

---

## A-16 -- Verify Operator user can authenticate [TERMINAL]

```bash
curl -s -u "test-operator:TestOp-2026!" "$BASE/api/auth/me" | jq '.'
```

**Expected:**
```json
{ "success": true, "data": { "username": "test-operator", "role": "operator" } }
```

---

## A-17 -- Create a second Operator for concurrent testing (optional) [TERMINAL]

If you need to test with multiple operators simultaneously:

```bash
curl -s -u "$ADMIN" \
  -X POST "$BASE/api/admin/users" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "operator-02",
    "password": "Op02-2026!",
    "role": "operator"
  }' | jq '.'
```

---

# SECTION 4 -- Reference Data Validation

Reference data (Material Master, Work Centers, Routings, BOMs) must be configured
before Operators can execute production workflows. Validate that all required master
data is present.

---

## A-18 -- Material Master is populated [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/materials" | jq '. | length'
```

**Expected:** A number greater than 0. If 0, materials have not been loaded.

List all materials to verify:
```bash
curl -s -u "$ADMIN" "$BASE/api/materials" | jq '.[] | {code: .code, description: .description, type: .type}'
```

**Required materials for this test session:**

| Material Code | Description          | Type          | Unit |
|---------------|----------------------|---------------|------|
| RM-0001       | Raw Material 01      | RAW           | KG   |
| FG-0001       | Finished Good 01     | FINISHED      | EA   |

If these materials are missing, create them via the Admin Panel [BROWSER]:
1. Navigate to https://wip.rodrigoandremarques.com/admin/prototype.html
2. Use the Material Master section to add the missing materials.

Or via API [TERMINAL]:
```bash
curl -s -u "$ADMIN" \
  -X POST "$BASE/api/materials" \
  -H "Content-Type: application/json" \
  -d '{"code":"RM-0001","description":"Raw Material 01","type":"RAW","unit":"KG"}' | jq '.'

curl -s -u "$ADMIN" \
  -X POST "$BASE/api/materials" \
  -H "Content-Type: application/json" \
  -d '{"code":"FG-0001","description":"Finished Good 01","type":"FINISHED","unit":"EA"}' | jq '.'
```

---

## A-19 -- Work Centers are configured [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/work-centers" | jq '.[] | {code: .code, description: .description}'
```

**Required work centers for this test:**

| Code              | Description        |
|-------------------|--------------------|
| WC-ASSEMBLY-01    | Assembly Line 01   |

If missing, create via API:
```bash
curl -s -u "$ADMIN" \
  -X POST "$BASE/api/work-centers" \
  -H "Content-Type: application/json" \
  -d '{"code":"WC-ASSEMBLY-01","description":"Assembly Line 01","capacityUnit":"HR","costRate":50}' | jq '.'
```

---

## A-20 -- Routings are configured [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/routings" | jq '.[] | {material: .materialCode, operations: (.operations | length)}'
```

**Expected:** A routing for FG-0001 with at least 1 operation.

If missing, create a routing with one operation:
```bash
curl -s -u "$ADMIN" \
  -X POST "$BASE/api/routings" \
  -H "Content-Type: application/json" \
  -d '{
    "materialCode": "FG-0001",
    "operations": [
      {
        "sequence": 10,
        "name": "Assembly",
        "workCenter": "WC-ASSEMBLY-01",
        "standardTimeHours": 0.5
      }
    ]
  }' | jq '.'
```

---

## A-21 -- Bill of Materials is configured [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/bom" | jq '.[] | {product: .productCode, components: (.components | length)}'
```

**Expected:** A BOM for FG-0001 listing RM-0001 as a component.

If missing, create the BOM:
```bash
curl -s -u "$ADMIN" \
  -X POST "$BASE/api/bom" \
  -H "Content-Type: application/json" \
  -d '{
    "productCode": "FG-0001",
    "components": [
      {
        "materialCode": "RM-0001",
        "quantity": 1,
        "unit": "KG"
      }
    ]
  }' | jq '.'
```

---

## A-22 -- Storage locations are available [TERMINAL]

The system uses string-based location codes. Verify that inventory entries reference
consistent location codes. The following locations are used in this test:

| Location Code  | Purpose                       |
|----------------|-------------------------------|
| WH-RAW         | Raw material warehouse        |
| PROD-AREA-01   | Production staging area       |
| FG-WAREHOUSE   | Finished goods warehouse      |
| DISPATCH-AREA  | Dispatch / outbound           |

No explicit creation step is needed -- locations are created on first use.
Verify naming consistency across all Operator steps by checking that the same
codes appear in inventory records after movements are posted.

---

# SECTION 5 -- Functional Module Validation (API Level)

These tests validate every functional module of the system via API before handing
off to end users for browser testing.

---

## A-23 -- Blockchain genesis block exists [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/chain" | jq '.[0]'
```

**Expected:** The genesis block at index 0. Fields: `index: 0`, `previousHash: "0"` or
a defined seed string, `type: "GENESIS"` or similar, `hash: <non-empty string>`.

---

## A-24 -- Chain is valid before any test data [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/stats" | jq '{totalBlocks: .totalBlocks, chainValid: .chainValid}'
```

**Expected:** `chainValid: true`. `totalBlocks` >= 1.

This is the baseline. Record the block count: `BASELINE_BLOCKS = <value>`.

---

## A-25 -- Full production order E2E via API [TERMINAL]

This sequence mirrors the Operator browser workflow but executes entirely via API.
It validates that every endpoint responds correctly and that each step produces
a blockchain event.

**Step 1: Post Goods Receiving**
```bash
curl -s -u "test-operator:TestOp-2026!" \
  -X POST "$BASE/api/goods-receipt" \
  -H "Content-Type: application/json" \
  -d '{"materialCode":"RM-0001","quantity":100,"unit":"KG","supplier":"TEST-SUPPLIER-01","purchaseOrder":"PO-TEST-001","location":"WH-RAW"}' \
  | jq '{success: .success, event: .data.type}'
```
**Expected:** `success: true`. Event type includes `GOODS_RECEIPT` or `MATERIAL_RECEIVED`.

**Step 2: Stage to Production**
```bash
curl -s -u "test-operator:TestOp-2026!" \
  -X POST "$BASE/api/inventory/stage" \
  -H "Content-Type: application/json" \
  -d '{"materialCode":"RM-0001","quantity":50,"fromLocation":"WH-RAW","toLocation":"PROD-AREA-01"}' \
  | jq '{success: .success}'
```
**Expected:** `success: true`.

**Step 3: Create Production Order**
```bash
ORDER_RESP=$(curl -s -u "test-operator:TestOp-2026!" \
  -X POST "$BASE/api/orders" \
  -H "Content-Type: application/json" \
  -d '{"product":"FG-0001","productSku":"FG-0001","quantity":10,"priority":"NORMAL","costCenter":"CC-TEST-01","createdBy":"test-operator","dueDate":"2026-12-31"}')
echo "$ORDER_RESP" | jq '{success: .success, orderId: (.data.id // .id), status: (.data.status // .status)}'
ORDER_ID=$(echo "$ORDER_RESP" | jq -r '.data.id // .id')
echo "Captured ORDER_ID: $ORDER_ID"
```
**Expected:** `success: true`. Status is `Created`. ORDER_ID is captured for subsequent steps.

**Step 4: Release Order**
```bash
curl -s -u "test-operator:TestOp-2026!" \
  -X PUT "$BASE/api/orders/$ORDER_ID/release" \
  | jq '{success: .success, status: (.data.status // .status)}'
```
**Expected:** `success: true`. Status is `Released`.

**Step 5: Goods Issue**
```bash
curl -s -u "test-operator:TestOp-2026!" \
  -X POST "$BASE/api/orders/$ORDER_ID/goods-issue" \
  -H "Content-Type: application/json" \
  -d '{"materialCode":"RM-0001","quantity":10,"fromLocation":"PROD-AREA-01"}' \
  | jq '{success: .success}'
```
**Expected:** `success: true`. Inventory at PROD-AREA-01 decreases.

**Step 6: Confirm Operation**
```bash
curl -s -u "test-operator:TestOp-2026!" \
  -X POST "$BASE/api/orders/$ORDER_ID/operations/1/confirm" \
  -H "Content-Type: application/json" \
  -d '{"yieldQuantity":9,"scrapQuantity":1,"workCenter":"WC-ASSEMBLY-01","operator":"test-operator"}' \
  | jq '{success: .success}'
```
**Expected:** `success: true`. Operation status confirmed.

**Step 7: Quality Inspection**
```bash
curl -s -u "test-operator:TestOp-2026!" \
  -X POST "$BASE/api/quality" \
  -H "Content-Type: application/json" \
  -d "{\"orderId\":\"$ORDER_ID\",\"inspectionType\":\"FINAL_QC\",\"result\":\"PASS\",\"inspector\":\"test-operator\",\"observations\":\"All dimensions within tolerance\"}" \
  | jq '{success: .success}'
```
**Expected:** `success: true`.

**Step 8: Goods Receipt -- Finished Goods**
```bash
curl -s -u "test-operator:TestOp-2026!" \
  -X POST "$BASE/api/orders/$ORDER_ID/goods-receipt" \
  -H "Content-Type: application/json" \
  -d '{"quantity":9,"location":"FG-WAREHOUSE"}' \
  | jq '{success: .success}'
```
**Expected:** `success: true`. FG-WAREHOUSE inventory increases by 9 EA.

**Step 9: Close Order**
```bash
curl -s -u "test-operator:TestOp-2026!" \
  -X PUT "$BASE/api/orders/$ORDER_ID/close" \
  | jq '{success: .success, status: (.data.status // .status)}'
```
**Expected:** `success: true`. Status is `Closed`.

---

## A-26 -- Verify inventory consistency after E2E [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/inventory" | \
  jq '.[] | select(.materialCode == "RM-0001" or .materialCode == "FG-0001") | {material: .materialCode, location: .location, qty: .quantity}'
```

**Expected stock levels after A-25:**

| Material | Location     | Expected Qty | Explanation                          |
|----------|--------------|--------------|--------------------------------------|
| RM-0001  | WH-RAW       | 50           | 100 received - 50 staged             |
| RM-0001  | PROD-AREA-01 | 40           | 50 staged - 10 issued to order       |
| FG-0001  | FG-WAREHOUSE | 9            | 9 confirmed yield received into stock|

**Fail if:** Any quantity does not match. Investigate the movement history report to
identify which step produced an unexpected result.

---

## A-27 -- Transfer and Scrap [TERMINAL]

```bash
# Transfer 4 units of FG-0001 from FG-WAREHOUSE to DISPATCH-AREA
curl -s -u "test-operator:TestOp-2026!" \
  -X POST "$BASE/api/inventory/transfer" \
  -H "Content-Type: application/json" \
  -d '{"materialCode":"FG-0001","quantity":4,"fromLocation":"FG-WAREHOUSE","toLocation":"DISPATCH-AREA"}' \
  | jq '{success: .success}'

# Scrap 2 KG of RM-0001 from WH-RAW
curl -s -u "test-operator:TestOp-2026!" \
  -X POST "$BASE/api/inventory/scrap" \
  -H "Content-Type: application/json" \
  -d '{"materialCode":"RM-0001","quantity":2,"location":"WH-RAW","reasonCode":"DAMAGED"}' \
  | jq '{success: .success}'
```

**Expected:** Both return `success: true`. Re-check inventory:
- FG-0001 at FG-WAREHOUSE: 5 (9 - 4)
- FG-0001 at DISPATCH-AREA: 4
- RM-0001 at WH-RAW: 48 (50 - 2)

---

# SECTION 6 -- Blockchain Integrity Validation

---

## A-28 -- Block count increased correctly [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/stats" | jq '.totalBlocks'
```

**Expected:** Block count is higher than the baseline recorded in A-24.
Minimum expected new blocks from A-25 and A-27:
Goods Receipt + Stage + Create Order + Release + Goods Issue +
Operation Confirm + QM Inspection + FG Receipt + Close + Transfer + Scrap = 11 new blocks.

Total = BASELINE_BLOCKS + 11 (minimum).

---

## A-29 -- Hash chain is intact [TERMINAL]

```bash
CHAIN=$(curl -s -u "$ADMIN" "$BASE/api/chain")
TOTAL=$(echo "$CHAIN" | jq '. | length')
echo "Total blocks: $TOTAL"

# Verify each block's previousHash matches the prior block's hash
echo "$CHAIN" | jq -r 'to_entries[] |
  if .key > 0 then
    {
      index: .value.index,
      hash_prefix: .value.hash[0:16],
      prev_prefix: .value.previousHash[0:16],
      match: (.value.previousHash == (.key - 1 | tostring | . as $i | $CHAIN | fromjson[$i | tonumber].hash))
    }
  else
    {index: 0, note: "genesis"}
  end'
```

**Simplified manual check** (if jq cross-reference is not available):
```bash
curl -s -u "$ADMIN" "$BASE/api/chain" | jq '.[-3:] | .[] | {index, hash: .hash[0:20], prev: .previousHash[0:20]}'
```

Visually confirm that `prev` of block N matches `hash` of block N-1.

**Fail if:** Any block's `previousHash` does not match the previous block's `hash`.
This indicates chain tampering or a data corruption issue.

---

## A-30 -- Chain integrity API confirmation [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/stats" | jq '.chainValid'
```

**Expected:** `true`

---

## A-31 -- Audit trail contains all expected event types [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/reports/movements" | \
  jq '[.[] | .type] | unique | sort'
```

**Expected event types** (based on A-25 and A-27):
```
GOODS_ISSUE
GOODS_RECEIPT
OPERATION_CONFIRMED
QM_INSPECTION
SCRAP
STAGING_TO_PRODUCTION (or TRANSFER)
TRANSFER
WORK_ORDER_CLOSED
WORK_ORDER_CREATED
WORK_ORDER_RELEASED
```

**Fail if:** Any event type from the list above is missing.

---

# SECTION 7 -- ERP Bridge Validation

---

## A-32 -- ERP Bridge export is accessible to Admin only [TERMINAL]

```bash
# Admin can access
curl -s -o /dev/null -w "Admin: %{http_code}\n" -u "$ADMIN" "$BASE/api/erp/export"

# Operator cannot access
curl -s -o /dev/null -w "Operator: %{http_code}\n" -u "test-operator:TestOp-2026!" "$BASE/api/erp/export"

# Visitor cannot access
curl -s -o /dev/null -w "Visitor: %{http_code}\n" -u "$VISITOR" "$BASE/api/erp/export"
```

**Expected:** `Admin: 200`, `Operator: 403`, `Visitor: 403`

---

## A-33 -- ERP export contains the test order [TERMINAL]

```bash
curl -s -u "$ADMIN" "$BASE/api/erp/export" | jq '.'
```

**Expected:** JSON payload containing the production order created in A-25 (ORDER_ID).
The payload should include: order number, product code, quantity, status (Closed),
cost center, and associated movements.

---

# SECTION 8 -- Resilience and Persistence

---

## A-34 -- Data survives server sleep and wake [TERMINAL]

**Precondition:** Complete Sections 1-7 first. Note ORDER_ID and block count.

1. Leave the server idle for at least 15 minutes to trigger Render's sleep.
2. After 15 minutes, run:

```bash
time curl -s -o /dev/null -w "%{http_code}" "$BASE/api/health"
```

**Expected:** HTTP 200 returned within 60 seconds. Wake time of 30-50 seconds is normal.

3. Verify the order and inventory data persisted:

```bash
# Order must still exist and be Closed
curl -s -u "$ADMIN" "$BASE/api/orders/$ORDER_ID" | jq '{id: .id, status: .status}'

# Inventory must show same quantities as end of A-26 and A-27
curl -s -u "$ADMIN" "$BASE/api/inventory" | \
  jq '.[] | select(.materialCode == "RM-0001" or .materialCode == "FG-0001") | {material: .materialCode, location: .location, qty: .quantity}'

# Chain must still be valid
curl -s -u "$ADMIN" "$BASE/api/stats" | jq '{totalBlocks: .totalBlocks, chainValid: .chainValid}'
```

**Expected:** All data is intact. Chain remains valid. Block count unchanged.

**Note:** Data persists within a deployment. A new deployment (code push) restarts the
server and may reset the filesystem. This is documented behavior for Render's free tier.
For persistent data across deployments, configure an external DATA_DIR or database.

---

## A-35 -- Admin account cannot be deleted [BROWSER]

1. Log in to the browser as Admin.
2. Navigate to User Management.
3. Attempt to find and delete the `admin` user.

**Expected:** The system prevents deletion of the primary admin account.
An error message appears such as "Cannot delete admin user" or the delete
button is not present for the admin entry.

---

## A-36 -- Operator user cleanup [TERMINAL]

Delete the test operator created in A-14 and verify it is removed:

```bash
curl -s -u "$ADMIN" \
  -X DELETE "$BASE/api/admin/users/test-operator" | jq '.'

# Verify deletion
curl -s -o /dev/null -w "%{http_code}" \
  -u "test-operator:TestOp-2026!" "$BASE/api/auth/me"
```

**Expected:** Delete returns success. Auth attempt returns `401`.

---

# SECTION 9 -- End-User Workflow Sign-Off

After completing Sections 1-8, the system is validated at the API level. The final
step is to confirm the complete Operator workflow works correctly in the browser.

---

## A-37 -- Re-create Operator for browser test [TERMINAL]

```bash
curl -s -u "$ADMIN" \
  -X POST "$BASE/api/admin/users" \
  -H "Content-Type: application/json" \
  -d '{"username":"test-operator","password":"TestOp-2026!","role":"operator"}' | jq '.'
```

---

## A-38 -- Execute full browser-based user test [BROWSER]

Follow the complete End-User Test Script (docs/TEST-ENDUSER.md) from O-01 through O-15
using the test-operator credentials. This validates that the browser UI correctly
reflects all API operations and that the Operator can complete a full production
order lifecycle without any technical issues.

All 15 Operator test cases in TEST-ENDUSER.md must pass before signing off.

---

## A-39 -- Execute Visitor browser test [BROWSER]

Follow TEST-ENDUSER.md from V-01 through V-10 using the Visitor credentials
(demo / WIP-Demo-2026!) to confirm that read-only access works correctly in the browser.

---

# SECTION 10 -- Admin Panel Validation

---

## A-40 -- Admin Panel loads [BROWSER]

1. While logged in as Admin, navigate to:
   https://wip.rodrigoandremarques.com/admin/prototype.html
2. The Admin Panel should load with advanced controls.

**Expected:** Page loads without a 404 or 403 error. Admin controls are visible.

---

## A-41 -- Reports accessible to Admin [BROWSER]

1. Click **Reports** in the navigation menu.
2. Run each of the following reports and verify they return data:
   - WIP Summary
   - Movement History
   - Order Status
   - Stock Snapshot (if available)

**Expected:** All reports load. Data reflects the movements made during this test session.

---

# SIGN-OFF CHECKLIST

Complete this checklist at the end of the Admin E2E. The system is production-ready
when all items are marked Pass.

## Section 1 -- Infrastructure

| Test  | Description                              | Result | Notes |
|-------|------------------------------------------|--------|-------|
| A-01  | Service health check responds            |        |       |
| A-02  | Health endpoint returns valid JSON       |        |       |
| A-03  | SSL certificate valid (browser padlock)  |        |       |
| A-04  | DNS CNAME resolves to Render             |        |       |
| A-05  | All required env vars set in Render      |        |       |
| A-06  | Last deployment is current and live      |        |       |

## Section 2 -- Authentication and Authorization

| Test  | Description                              | Result | Notes |
|-------|------------------------------------------|--------|-------|
| A-07  | Admin login works                        |        |       |
| A-08  | Visitor login works                      |        |       |
| A-09  | Wrong password returns 401               |        |       |
| A-10  | No credentials returns 401               |        |       |
| A-11  | Visitor blocked from all write endpoints |        |       |
| A-12  | Operator blocked from admin endpoints    |        |       |

## Section 3 -- User Provisioning

| Test  | Description                              | Result | Notes |
|-------|------------------------------------------|--------|-------|
| A-13  | Admin and Visitor users exist            |        |       |
| A-14  | Create Operator user via API             |        |       |
| A-15  | Operator user appears in user list       |        |       |
| A-16  | Operator user can authenticate           |        |       |

## Section 4 -- Reference Data

| Test  | Description                              | Result | Notes |
|-------|------------------------------------------|--------|-------|
| A-18  | Material Master populated (RM-0001, FG-0001) |    |       |
| A-19  | Work Centers configured (WC-ASSEMBLY-01) |        |       |
| A-20  | Routing for FG-0001 with min 1 operation |        |       |
| A-21  | BOM for FG-0001 with RM-0001 component   |        |       |

## Section 5 -- Functional E2E (API)

| Test  | Description                              | Result | Notes |
|-------|------------------------------------------|--------|-------|
| A-25  | Full order lifecycle via API             |        |       |
| A-26  | Inventory consistency after E2E          |        |       |
| A-27  | Transfer and Scrap posting               |        |       |

## Section 6 -- Blockchain Integrity

| Test  | Description                              | Result | Notes |
|-------|------------------------------------------|--------|-------|
| A-28  | Block count increased as expected        |        |       |
| A-29  | Hash chain linkage is intact             |        |       |
| A-30  | chainValid = true                        |        |       |
| A-31  | All event types present in audit trail   |        |       |

## Section 7 -- ERP Bridge

| Test  | Description                              | Result | Notes |
|-------|------------------------------------------|--------|-------|
| A-32  | ERP export access control correct        |        |       |
| A-33  | ERP export contains test order           |        |       |

## Section 8 -- Resilience

| Test  | Description                              | Result | Notes |
|-------|------------------------------------------|--------|-------|
| A-34  | Data survives cold start                 |        |       |
| A-35  | Admin account cannot be deleted          |        |       |
| A-36  | Operator user cleanup confirmed          |        |       |

## Section 9 -- End-User Sign-Off

| Test  | Description                              | Result | Notes |
|-------|------------------------------------------|--------|-------|
| A-38  | All 15 Operator browser tests pass       |        |       |
| A-39  | All 10 Visitor browser tests pass        |        |       |

## Section 10 -- Admin Panel

| Test  | Description                              | Result | Notes |
|-------|------------------------------------------|--------|-------|
| A-40  | Admin Panel loads                        |        |       |
| A-41  | All reports accessible and return data   |        |       |

---

**Sign-off:**
Tested by: ______________________
Date: ______________________
Deployment version / commit: ______________________
All items above: PASS / FAIL (circle one)

If any item is FAIL, do not mark the system as production-ready.
Document the failure, fix the root cause, redeploy, and re-run from Section 1.
