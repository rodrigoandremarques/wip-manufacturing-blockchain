# Test Administration Guide

Administrator testing procedures for validating complete production order workflows, ERP integration, and system resilience.

## Section 1: Infrastructure Validation

### 1.1 Render Deployment Health Check

```bash
BASE="https://your-render-app.onrender.com"
HEALTH_CHECK=$(curl -s -o /dev/null -w "%{http_code}" "$BASE/health")
if [ "$HEALTH_CHECK" -eq 200 ]; then
  echo "Render deployment is healthy"
else
  echo "Render health check failed: $HEALTH_CHECK"
fi
```

### 1.2 DNS Resolution

```bash
nslookup your-render-app.onrender.com
# Expected: Resolves to Render-owned IP space
```

### 1.3 SSL Certificate Validation

```bash
echo | openssl s_client -servername your-render-app.onrender.com \
  -connect your-render-app.onrender.com:443 2>/dev/null | \
  openssl x509 -noout -text | grep -E "Subject:|Issuer:|Not After"
```

### 1.4 API Readiness

```bash
BASE="https://your-render-app.onrender.com"
curl -s "$BASE/api/work-orders" -u "$ADMIN" | jq '.status'
# Expected: "success"
```

## Section 2: Authentication and Authorization Matrix

### 2.1 User Roles and Permissions

| Role          | Work Order | Confirmation | QM Review | Close WO |
|---------------|-----------|--------------|-----------|----------|
| Operator      | No        | Yes          | No        | No       |
| QM Inspector  | No        | No           | Yes       | No       |
| Planner       | Yes       | No           | No        | No       |
| Administrator | Yes       | Yes          | Yes       | Yes      |

### 2.2 API Authentication Setup

```bash
# Set environment variables for testing
export BASE="https://your-render-app.onrender.com"
export ADMIN="admin@company.com:AdminPassword123"
export OPERATOR="operator@company.com:OpPassword123"
export QM="qminspector@company.com:QMPassword123"

# Verify admin credentials
curl -s -u "$ADMIN" "$BASE/api/work-orders" | jq '.status'
```

### 2.3 Authorization Test Cases

**Admin can create work orders:**
```bash
curl -X POST "$BASE/api/work-order" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{"material":"MAT-001","quantity":100,"unit":"EA","slocGI":"1000","slocGR":"3000","startDate":"2026-04-08","finishDate":"2026-04-15","mrpController":"PLANNER01","plant":"1000"}'
```

**Operator cannot create work orders (401 expected):**
```bash
curl -w "\nStatus: %{http_code}\n" \
  -X POST "$BASE/api/work-order" \
  -u "$OPERATOR" \
  -H "Content-Type: application/json" \
  -d '{"material":"MAT-001","quantity":100,"unit":"EA","slocGI":"1000","slocGR":"3000","startDate":"2026-04-08","finishDate":"2026-04-15","mrpController":"PLANNER01","plant":"1000"}'
```
## Section 3: User Provisioning

### 3.1 Create Operator User

```bash
curl -X POST "$BASE/api/users" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{
    "email":"test-operator@company.com",
    "name":"Test Operator",
    "password":"TestOp123!",
    "role":"OPERATOR"
  }'
# Expected response: {"status":"success","userId":"OP-001"}
```

### 3.2 Verify User Creation

```bash
curl -s "$BASE/api/users/test-operator@company.com" \
  -u "$ADMIN" | jq '.data | {email, name, role}'
# Expected: email: "test-operator@company.com", role: "OPERATOR"
```

### 3.3 Delete Test User

```bash
curl -X DELETE "$BASE/api/users/test-operator@company.com" \
  -u "$ADMIN" \
  -H "Content-Type: application/json"
# Expected response: {"status":"success"}
```

## Section 4: Reference Data

### 4.1 Materials Master Data

Pre-load test materials:

```bash
# Material 1: Raw Material
curl -X POST "$BASE/api/materials" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{
    "materialCode":"MAT-001",
    "description":"Electronic Component A",
    "baseUnit":"EA",
    "type":"RAW_MATERIAL"
  }'

# Material 2: Semi-Finished
curl -X POST "$BASE/api/materials" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{
    "materialCode":"MAT-002",
    "description":"Assembly B",
    "baseUnit":"EA",
    "type":"SEMI_FINISHED"
  }'

# Verify materials loaded
curl -s "$BASE/api/materials" -u "$ADMIN" | jq '.data | length'
```

### 4.2 Work Centers (Pre-Configured)

The following work centers are pre-configured in the system and require no creation:

- WC-SMT-01 (Surface Mount Technology)
- WC-AOI-01 (Automated Optical Inspection)
- WC-PTH-01 (Plated Through Hole)
- WC-WAVE (Wave Soldering)
- WC-TEST (Functional Testing)
- WC-VISUAL-INSP (Visual Inspection)
- WC-COAT (Coating)
- WC-ASSY (Assembly)
- WC-PACK (Packaging)

Verify pre-configured work centers:

```bash
curl -s "$BASE/api/work-centers" -u "$ADMIN" | jq '.data[] | {code, description}'
```

### 4.3 Bill of Materials (BOM)

```bash
curl -X POST "$BASE/api/bom" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{
    "materialCode":"MAT-002",
    "bomItems":[
      {"componentMaterial":"MAT-001","qty":2,"unit":"EA"}
    ]
  }'

# Verify BOM
curl -s "$BASE/api/bom/MAT-002" -u "$ADMIN" | jq '.data.bomItems'
```

### 4.4 Routing (Operation Sequence)

```bash
curl -X POST "$BASE/api/routing" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{
    "materialCode":"MAT-002",
    "operations":[
      {"operationNumber":"0010","workCenter":"WC-ASSY","standardTime":30},
      {"operationNumber":"0020","workCenter":"WC-TEST","standardTime":15}
    ]
  }'
```
## Section 5: Full Production Order E2E via API

### 5.1 Step 1: Create Work Order

```bash
WO_RESPONSE=$(curl -s -X POST "$BASE/api/work-order" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{
    "material":"MAT-002",
    "quantity":50,
    "unit":"EA",
    "slocGI":"1000",
    "slocGR":"3000",
    "startDate":"2026-04-08",
    "finishDate":"2026-04-15",
    "mrpController":"PLANNER01",
    "plant":"1000"
  }')

WO_NUM=$(echo "$WO_RESPONSE" | jq -r '.data.woNumber')
echo "Work Order Created: $WO_NUM"
# Expected status in response: CRTD
```

### 5.2 Step 2: Release Work Order

```bash
curl -s -X POST "$BASE/api/work-order/$WO_NUM/release" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{"operator":"PLANNER01"}' | jq '.data.status'
# Expected status: REL
```

### 5.3 Step 3: Post Goods Issue (Raw Material)

```bash
curl -s -X POST "$BASE/api/goods-issue" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{
    "woNumber":"'"$WO_NUM"'",
    "material":"MAT-001",
    "quantity":100,
    "slocFrom":"1000",
    "operator":"OPERATOR01",
    "postingDate":"2026-04-08"
  }' | jq '.data | {docNumber, status}'
```

### 5.4 Step 4: Confirm Operation (Assembly)

```bash
curl -s -X POST "$BASE/api/confirmation" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{
    "woNumber":"'"$WO_NUM"'",
    "operationNumber":"0010",
    "workCenter":"WC-ASSY",
    "yieldQty":48,
    "scrapQty":2,
    "operator":"OPERATOR01",
    "confirmationType":"Final",
    "actualLaborMin":1500,
    "machineTimeMin":1200
  }' | jq '.data | {confirmationNumber, status}'
# Expected work order status: CNF (set automatically)
```

### 5.5 Step 5: Post Goods Receipt (Finished Goods)

```bash
curl -s -X POST "$BASE/api/goods-receipt" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{
    "woNumber":"'"$WO_NUM"'",
    "material":"MAT-002",
    "qtyReceived":48,
    "qtyScrap":0,
    "slocTo":"3000",
    "operator":"OPERATOR01"
  }' | jq '.data | {docNumber, events}'
# Expected: Creates GOODS_RECEIPT_101, WO_DELIVERED, INSP_LOT_CREATED
# Expected work order status: DLV
```

### 5.6 Step 6: QM Result (Quality Decision)

```bash
curl -s -X POST "$BASE/api/qm-result" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{
    "woNumber":"'"$WO_NUM"'",
    "usageDecision":"PASS",
    "inspector":"QM-INSP-01",
    "sampleSize":5,
    "comments":"All samples passed visual and functional tests"
  }' | jq '.data | {resultNumber, usageDecision}'
```

### 5.7 Step 7: Mark for TECO (Technical Completion)

```bash
curl -s -X POST "$BASE/api/work-order/$WO_NUM/teco" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{"operator":"PLANNER01"}' | jq '.data.status'
# Expected status: TECO (allowed because status is DLV)
```

### 5.8 Step 8: Close Work Order

```bash
curl -s -X POST "$BASE/api/work-order/$WO_NUM/close" \
  -u "$ADMIN" \
  -H "Content-Type: application/json" \
  -d '{"operator":"PLANNER01"}' | jq '.data.status'
# Expected status: CLSD (final status)
```

### 5.9 E2E Validation

Verify work order status flow completed:

```bash
curl -s "$BASE/api/work-order/$WO_NUM" \
  -u "$ADMIN" | jq '.data | {woNumber, status, createdAt, closedAt}'
# Expected final status: CLSD
```
## Section 6: Stock Verification After E2E

### 6.1 Verify Stock Levels by Storage Location

After completing the full E2E flow, verify stock levels across all storage locations:

```bash
curl -s -u "$ADMIN" "$BASE/api/stock" | jq '.data.stock'
```

Expected output structure:

```json
{
  "stock": {
    "1000": {
      "description": "Raw Material",
      "materials": {
        "MAT-001": {
          "quantity": 0,
          "unit": "EA",
          "lastMoved": "2026-04-08T10:30:00Z"
        }
      }
    },
    "2000": {
      "description": "Semi-Finished",
      "materials": {}
    },
    "3000": {
      "description": "Finished Goods",
      "materials": {
        "MAT-002": {
          "quantity": 48,
          "unit": "EA",
          "lastMoved": "2026-04-08T11:45:00Z"
        }
      }
    },
    "4000": {
      "description": "Scrap/Reject",
      "materials": {}
    }
  }
}
```

### 6.2 Detailed Stock Query per Location

```bash
# Raw Material (1000)
curl -s -u "$ADMIN" "$BASE/api/stock?sloc=1000" | jq '.data'

# Semi-Finished (2000)
curl -s -u "$ADMIN" "$BASE/api/stock?sloc=2000" | jq '.data'

# Finished Goods (3000)
curl -s -u "$ADMIN" "$BASE/api/stock?sloc=3000" | jq '.data'

# Scrap/Reject (4000)
curl -s -u "$ADMIN" "$BASE/api/stock?sloc=4000" | jq '.data'
```

### 6.3 Stock Validation Checklist

After E2E execution, verify:

- [ ] SLOC 1000 (Raw Material): MAT-001 reduced by 100 units
- [ ] SLOC 3000 (Finished Goods): MAT-002 increased by 48 units
- [ ] SLOC 4000 (Scrap/Reject): Empty (no scrap recorded in QM-PASS scenario)
- [ ] All timestamps reflect E2E operation sequence
- [ ] Stock audit trail matches material document sequence

### 6.4 Material Documents List

```bash
curl -s -u "$ADMIN" "$BASE/api/documents" | \
  jq '.data[] | {docNumber, docType, woNumber, material, quantity, sloc, postingDate}'
```

Expected document sequence:

1. GOODS_ISSUE document (MAT-001, quantity 100, from SLOC 1000)
2. CONFIRMATION document (operation 0010, yield 48, scrap 2)
3. GOODS_RECEIPT document (MAT-002, quantity 48, to SLOC 3000)
4. QM_RESULT document (PASS decision)
## Section 7: Blockchain Integrity Validation

### 7.1 Verify Document Blockchain Entries

Each material document creates blockchain entries. Verify integrity of E2E documents:

```bash
# Get all blockchain entries for the work order
curl -s -u "$ADMIN" "$BASE/api/blockchain/work-order/$WO_NUM" | \
  jq '.data[] | {blockNumber, hash, previousHash, documentNumber, timestamp}'
```

### 7.2 Validate Blockchain Chain Integrity

```bash
curl -s -u "$ADMIN" "$BASE/api/blockchain/validate" | jq '.data | {isValid, totalBlocks, lastBlockHash}'
# Expected: {"isValid": true, ...}
```

### 7.3 Event Sequence Validation

Verify that all expected events were recorded in blockchain order:

```bash
curl -s -u "$ADMIN" "$BASE/api/events/work-order/$WO_NUM" | \
  jq '.data[] | {timestamp, eventType, details}' | \
  head -20
```

Expected event sequence:

1. WO_CREATED (CRTD status)
2. WO_RELEASED (REL status)
3. GOODS_ISSUE_101 (material deducted from SLOC 1000)
4. OPERATION_CONFIRMED (operation 0010, CNF status)
5. GOODS_RECEIPT_101 (material added to SLOC 3000, DLV status)
6. INSP_LOT_CREATED
7. QM_RESULT_RECORDED (PASS decision)
8. WO_TECOED (TECO status)
9. WO_CLOSED (CLSD status)

### 7.4 Document Hash Verification

```bash
# Retrieve specific document and verify its blockchain hash
DOC_NUM=$(curl -s -u "$ADMIN" "$BASE/api/documents?woNumber=$WO_NUM" | \
  jq -r '.data[0].docNumber')

curl -s -u "$ADMIN" "$BASE/api/blockchain/document/$DOC_NUM" | \
  jq '.data | {docNumber, hash, chainValid}'
# Expected: chainValid: true
```

### 7.5 Anti-Tampering Verification

```bash
# Verify that blockchain hash chain is unbroken
curl -s -u "$ADMIN" "$BASE/api/blockchain/integrity-check" | \
  jq '.data | {status, brokenChains, lastValidBlock}'
# Expected status: "VALID"
```
## Section 8: ERP Bridge Validation

### 8.1 Verify IDOC LOIPRO01 (Work Order Master)

Admin-only endpoint to verify work order IDOC format sent to SAP:

```bash
curl -s -u "$ADMIN" "$BASE/api/erp/order/$WO_NUM" | \
  jq '.data | {messageType, idocType, docNumber, woNumber, status, materials, operations}'
```

Expected IDOC structure for LOIPRO01:

```json
{
  "messageType": "ORDERS01",
  "idocType": "LOIPRO01",
  "docNumber": "0000000123",
  "woNumber": "WO-002026-001",
  "status": "CLSD",
  "materials": [
    {
      "materialCode": "MAT-002",
      "quantity": 50,
      "unit": "EA",
      "slocFrom": "1000",
      "slocTo": "3000"
    }
  ],
  "operations": [
    {
      "operationNumber": "0010",
      "workCenter": "WC-ASSY",
      "yieldQty": 48,
      "scrapQty": 2
    }
  ]
}
```

### 8.2 Verify IDOC MBGMCR02 (Material Movement)

Admin-only endpoint to verify material movement IDOC format:

```bash
# Get first material document number from E2E execution
MOVEMENT_DOC=$(curl -s -u "$ADMIN" "$BASE/api/documents?woNumber=$WO_NUM" | \
  jq -r '.data[0].docNumber')

curl -s -u "$ADMIN" "$BASE/api/erp/movement/$MOVEMENT_DOC" | \
  jq '.data | {messageType, idocType, docNumber, movementType, material, quantity, sloc, postingDate}'
```

Expected IDOC structure for MBGMCR02:

```json
{
  "messageType": "MATMAS03",
  "idocType": "MBGMCR02",
  "docNumber": "0000000456",
  "movementType": "101",
  "material": "MAT-001",
  "quantity": 100,
  "sloc": "1000",
  "postingDate": "2026-04-08",
  "plant": "1000"
}
```

### 8.3 ERP Bridge Status

```bash
curl -s -u "$ADMIN" "$BASE/api/erp/status" | \
  jq '.data | {bridgeStatus, lastSyncTime, pendingDocuments, errorCount}'
```

Expected response:

```json
{
  "bridgeStatus": "CONNECTED",
  "lastSyncTime": "2026-04-08T12:00:00Z",
  "pendingDocuments": 0,
  "errorCount": 0
}
```

### 8.4 IDOC Transmission Log

```bash
curl -s -u "$ADMIN" "$BASE/api/erp/idoc-log?limit=10" | \
  jq '.data[] | {timestamp, idocType, status, messageNumber, errorDetails}'
```

All entries for the E2E work order should have status "TRANSMITTED" with no errors.
## Section 9: Resilience Testing

### 9.1 Cold Start Validation

After restarting the application, verify data persistence:

```bash
# 1. Restart the Render app
# Go to Render dashboard > Services > your-service > select "Restart Service"
# Wait 30-60 seconds for service to come online

# 2. Verify health check passes
BASE="https://your-render-app.onrender.com"
curl -s "$BASE/health" | jq '.status'
# Expected: "healthy"

# 3. Verify work order still exists and has correct status
curl -s -u "$ADMIN" "$BASE/api/work-order/$WO_NUM" -u "$ADMIN" | \
  jq '.data | {woNumber, status, closedAt}'
# Expected: status: "CLSD", closedAt should be populated
```

### 9.2 Data Persistence Check

Verify that all material documents persisted:

```bash
curl -s -u "$ADMIN" "$BASE/api/documents?woNumber=$WO_NUM" | \
  jq '.data | length'
# Expected: 4 documents (goods issue, confirmation, goods receipt, qm result)
```

Verify stock levels unchanged after cold start:

```bash
curl -s -u "$ADMIN" "$BASE/api/stock" | jq '.data.stock["3000"].materials["MAT-002"].quantity'
# Expected: 48 (same as before restart)
```

### 9.3 Database Connection Recovery

```bash
# Verify database is responsive after cold start
curl -s -u "$ADMIN" "$BASE/api/health/database" | jq '.data | {status, responseTime_ms, recordCount}'
# Expected: status: "connected", recordCount > 0
```

### 9.4 Event Log Persistence

```bash
# Verify all events are still in the log
curl -s -u "$ADMIN" "$BASE/api/events/work-order/$WO_NUM" | \
  jq '.data | length'
# Expected: minimum 9 events (WO_CREATED through WO_CLOSED)
```

### 9.5 Blockchain Integrity After Cold Start

```bash
curl -s -u "$ADMIN" "$BASE/api/blockchain/integrity-check" | \
  jq '.data | {status, lastBlockNumber}'
# Expected: status: "VALID"
```

### 9.6 ERP Bridge Reconnection

```bash
curl -s -u "$ADMIN" "$BASE/api/erp/status" | \
  jq '.data.bridgeStatus'
# Expected: "CONNECTED"
```
## Section 10: End-User Sign-Off

This administrator test plan is the prerequisite for end-user acceptance testing. Refer to TEST-ENDUSER.md for user-facing validation procedures.

### 10.1 Prerequisites for End-User Testing

Before releasing to end-user testing, all of the following must be complete:

- [ ] Section 1: Infrastructure Validation (Render, DNS, SSL, health check) - PASSED
- [ ] Section 2: Authentication and Authorization Matrix - PASSED
- [ ] Section 3: User Provisioning (user create, verify, delete) - PASSED
- [ ] Section 4: Reference Data (materials, work centers, routings, BOMs) - PASSED
- [ ] Section 5: Full Production Order E2E via API - PASSED
- [ ] Section 6: Stock Verification - PASSED
- [ ] Section 7: Blockchain Integrity - PASSED
- [ ] Section 8: ERP Bridge IDOC Validation - PASSED
- [ ] Section 9: Resilience (cold start, data persistence) - PASSED

### 10.2 Test Metrics Summary

Capture the following metrics from your test run:

| Metric | Value | Expected |
|--------|-------|----------|
| E2E Workflow Completion Time | ___ seconds | < 60 seconds |
| API Response Time (avg) | ___ ms | < 200 ms |
| Stock Accuracy | ___ % | 100% |
| Blockchain Chain Validity | ___ blocks | Unbroken chain |
| IDOC Transmission Success | ___ / ___ | 100% |
| Cold Start Recovery Time | ___ seconds | < 120 seconds |
| Data Loss During Restart | ___ records | 0 |

### 10.3 Known Issues and Limitations

Document any deviations from expected behavior:

1. Issue: _________________________________
   Severity: [ ] Critical [ ] High [ ] Medium [ ] Low
   Status: [ ] Blocking Release [ ] Can Wait [ ] Workaround Available
   Details: _________________________________

2. Issue: _________________________________
   Severity: [ ] Critical [ ] High [ ] Medium [ ] Low
   Status: [ ] Blocking Release [ ] Can Wait [ ] Workaround Available
   Details: _________________________________

### 10.4 Sign-Off Confirmation

Upon successful completion of this test plan, provide the following sign-off:

```
Test Administrator: _____________________________
Date Completed: _____________________________
Environment: [ ] Development [ ] Staging [ ] Production

All test sections completed: [ ] Yes [ ] No
All critical issues resolved: [ ] Yes [ ] No
Ready for end-user testing: [ ] Yes [ ] No

Comments:
_________________________________________________________________
_________________________________________________________________
```

## Test Completion Checklist

### Infrastructure & Deployment
- [ ] Render deployment is healthy (HTTP 200)
- [ ] DNS resolves correctly
- [ ] SSL certificate is valid and not expired
- [ ] API health endpoint responds within 5 seconds

### Authentication & Authorization
- [ ] Admin user can create work orders
- [ ] Operator user cannot create work orders (401 rejected)
- [ ] QM Inspector can record quality results
- [ ] Role-based access control working as designed

### User Management
- [ ] New user can be created
- [ ] User creation recorded in audit log
- [ ] User can authenticate with new credentials
- [ ] User can be deleted without residual data

### Reference Data
- [ ] Materials loaded correctly
- [ ] Work centers pre-configured (9 total)
- [ ] Bill of Materials created successfully
- [ ] Routing operations defined

### Full Production Order E2E
- [ ] Work Order created with status CRTD
- [ ] Work Order released (status REL)
- [ ] Goods Issue posted (SLOC 1000 → work)
- [ ] Operation confirmed (status CNF)
- [ ] Goods Receipt posted (SLOC 3000, status DLV)
- [ ] QM Result recorded (PASS decision)
- [ ] TECO executed successfully (status TECO)
- [ ] Work Order closed (status CLSD)
- [ ] Final status flow: CRTD → REL → CNF → DLV → TECO → CLSD

### Stock Accuracy
- [ ] SLOC 1000: MAT-001 quantity reduced by 100
- [ ] SLOC 3000: MAT-002 quantity increased by 48
- [ ] SLOC 4000: Empty (no scrap in PASS scenario)
- [ ] All timestamps in sequence

### Material Documents
- [ ] 4 documents created for E2E flow
- [ ] Document numbers sequential
- [ ] All documents retrievable via API
- [ ] Posting dates and times accurate

### Blockchain Validation
- [ ] All E2E documents have blockchain entries
- [ ] Blockchain chain is unbroken
- [ ] Hash values are valid
- [ ] No tampering detected

### ERP Bridge
- [ ] LOIPRO01 IDOC generated for work order
- [ ] MBGMCR02 IDOC generated for each material movement
- [ ] IDOC transmission status is TRANSMITTED
- [ ] No IDOC errors in transmission log

### Resilience
- [ ] Application cold starts successfully
- [ ] Work order data persists after restart
- [ ] Stock levels unchanged after restart
- [ ] All events remain in log after restart
- [ ] Blockchain integrity maintained after restart
- [ ] ERP bridge reconnects after restart

### Sign-Off
- [ ] Test administrator reviews and approves all results
- [ ] All critical issues are resolved
- [ ] System ready for end-user acceptance testing (TEST-ENDUSER.md)
