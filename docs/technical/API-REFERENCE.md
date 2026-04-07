# API Reference

WIP (Work In Progress) System API
Base URL: https://wip.rodrigoandremarques.com

This document describes all available endpoints in the WIP system, organized by functional area.
Throughout examples, use BASE=https://wip.rodrigoandremarques.com


AUTHENTICATION & SYSTEM
========================

GET /api/health
  Auth required: None
  Description: System health check
  Response: {status:"ok", uptime:number}
  
  Example:
  ```
  curl -X GET $BASE/api/health
  ```


GET /api/auth/me
  Auth required: Yes (any authenticated user)
  Description: Get current authenticated user info
  Response: {success:true, data:{username:string, role:string}}
  
  Example:
  ```
  curl -X GET $BASE/api/auth/me \
    -H "Authorization: Bearer TOKEN"
  ```


GET /api/stats
  Auth required: None
  Description: Full system statistics including blockchain validity
  Response: Full system stats object with chainValid boolean
  
  Example:
  ```
  curl -X GET $BASE/api/stats
  ```


GET /api/chain/status
  Auth required: None
  Description: Blockchain validity check
  Response: Blockchain status and validation results
  
  Example:
  ```
  curl -X GET $BASE/api/chain/status
  ```


ADMIN: USER MANAGEMENT
======================

GET /api/admin/users
  Auth required: Yes (Admin role)
  Description: List all users in system
  Response: Array of user objects
  
  Example:
  ```
  curl -X GET $BASE/api/admin/users \
    -H "Authorization: Bearer TOKEN"
  ```


POST /api/admin/users
  Auth required: Yes (Admin role)
  Description: Create a new user
  Required fields: username, password, role
  Response: {success:true, user:{username, role}}
  
  Example:
  ```
  curl -X POST $BASE/api/admin/users \
    -H "Authorization: Bearer TOKEN" \
    -H "Content-Type: application/json" \
    -d '{
      "username": "operator1",
      "password": "secure123",
      "role": "operator"
    }'
  ```


DELETE /api/admin/users/:username
  Auth required: Yes (Admin role)
  Description: Delete a user by username
  URL parameter: username
  Response: {success:true, message:string}
  
  Example:
  ```
  curl -X DELETE $BASE/api/admin/users/operator1 \
    -H "Authorization: Bearer TOKEN"
  ```


PUT /api/admin/users/:username/password
  Auth required: Yes (Admin role)
  Description: Reset password for a user
  URL parameter: username
  Required fields: password
  Response: {success:true, message:string}
  
  Example:
  ```
  curl -X PUT $BASE/api/admin/users/operator1/password \
    -H "Authorization: Bearer TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"password": "newpassword123"}'
  ```


ADMIN: BLOCKCHAIN & AUDIT
==========================

GET /api/chain
  Auth required: Yes (Admin role)
  Description: Get full blockchain array
  Response: Array of blockchain blocks with all transactions
  
  Example:
  ```
  curl -X GET $BASE/api/chain \
    -H "Authorization: Bearer TOKEN"
  ```


GET /api/audit-trail
  Auth required: No
  Description: Get all system transactions and audit trail
  Response: Array of transaction objects with timestamps and events
  
  Example:
  ```
  curl -X GET $BASE/api/audit-trail
  ```


WORK ORDERS (SAP Production Orders)
===================================

Work Order Status Flow:
  CRTD (Created)
    |
    v
  REL (Released)
    |
    v
  CNF (Confirmed)
    |
    v
  DLV (Delivered)
    |
    v
  TECO (Technically Complete)
    |
    v
  CLSD (Closed)


GET /api/work-orders
  Auth required: No
  Description: List all work orders
  Response: Array of work order objects with current status
  
  Example:
  ```
  curl -X GET $BASE/api/work-orders
  ```


GET /api/work-order/:id
  Auth required: No
  Description: Get single work order by ID
  URL parameter: id (work order number)
  Response: Work order object with full details
  
  Example:
  ```
  curl -X GET $BASE/api/work-order/4000001
  ```


POST /api/work-order
  Auth required: No
  Description: Create a new work order
  Required fields: material, quantity
  Optional fields: unit, slocGI, slocGR, startDate, finishDate, mrpController, plant
  
  Defaults:
    - plant: 1000
    - unit: UN
    - slocGI (from storage): 1000
    - slocGR (to storage): 3000
  
  Response: {woNumber:string, status:"CRTD", transaction:object, ethereum:object}
  Blockchain events: WO_CREATED
  
  Example:
  ```
  curl -X POST $BASE/api/work-order \
    -H "Content-Type: application/json" \
    -d '{
      "material": "MAT-001",
      "quantity": 100,
      "unit": "UN",
      "slocGI": 1000,
      "slocGR": 3000,
      "startDate": "2026-04-07",
      "finishDate": "2026-04-14",
      "plant": 1000
    }'
  ```


POST /api/work-order/:id/release
  Auth required: No
  Description: Release a work order (CRTD -> REL)
  URL parameter: id
  Required fields: operator
  Response: {woNumber:string, status:"REL", transaction:object}
  Blockchain events: WO_RELEASED
  
  Example:
  ```
  curl -X POST $BASE/api/work-order/4000001/release \
    -H "Content-Type: application/json" \
    -d '{"operator": "op123"}'
  ```


POST /api/work-order/:id/teco
  Auth required: No
  Description: Mark work order as Technically Complete (DLV -> TECO)
                Requires status DLV first (must post goods receipt before TECO)
  URL parameter: id
  Required fields: operator
  Response: {woNumber:string, status:"TECO", transaction:object}
  Blockchain events: ORDER_TECO
  
  Example:
  ```
  curl -X POST $BASE/api/work-order/4000001/teco \
    -H "Content-Type: application/json" \
    -d '{"operator": "op123"}'
  ```


POST /api/work-order/:id/close
  Auth required: No
  Description: Close a work order (TECO -> CLSD)
                Requires TECO status first
  URL parameter: id
  Required fields: operator
  Response: {woNumber:string, status:"CLSD", transaction:object}
  Blockchain events: ORDER_CLOSED
  
  Example:
  ```
  curl -X POST $BASE/api/work-order/4000001/close \
    -H "Content-Type: application/json" \
    -d '{"operator": "op123"}'
  ```


MATERIAL MOVEMENTS (Goods Issue, Goods Receipt, Backflush)
==========================================================

POST /api/goods-issue
  Auth required: No
  Description: Post goods issue (SAP MVT 261) from raw materials
  Required fields: woNumber, material, quantity
  Optional fields: slocFrom, operator, batch, postingDate
  
  Default slocFrom: 1000 (Raw Material storage location)
  
  Response: {docNumber:string, woNumber:string, status:"POSTED", transaction:object}
  Blockchain events: GOODS_ISSUE_261
  
  Example:
  ```
  curl -X POST $BASE/api/goods-issue \
    -H "Content-Type: application/json" \
    -d '{
      "woNumber": "4000001",
      "material": "MAT-001",
      "quantity": 100,
      "slocFrom": 1000,
      "operator": "op123",
      "postingDate": "2026-04-07"
    }'
  ```


POST /api/goods-receipt
  Auth required: No
  Description: Post goods receipt (SAP MVT 101) of finished goods
                Sets WO status to DLV. Creates THREE blockchain events plus optional scrap posting.
  Required fields: woNumber, material, qtyReceived
  Optional fields: slocTo, qtyScrap, operator, postingDate
  
  Default slocTo: 3000 (Finished Goods storage location)
  
  Response: {docNumber:string, woNumber:string, status:"DLV", inspLotCreated:true, 
             transactions:array, ethereum:object}
  
  Blockchain events:
    - GOODS_RECEIPT_101 (always)
    - WO_DELIVERED (always)
    - INSP_LOT_CREATED (always)
    - SCRAP_POSTING_551 (if qtyScrap > 0)
  
  Example:
  ```
  curl -X POST $BASE/api/goods-receipt \
    -H "Content-Type: application/json" \
    -d '{
      "woNumber": "4000001",
      "material": "MAT-001",
      "qtyReceived": 98,
      "slocTo": 3000,
      "qtyScrap": 2,
      "operator": "op123",
      "postingDate": "2026-04-14"
    }'
  ```


POST /api/backflush
  Auth required: No
  Description: Post backflush (SAP MVT 543) - consumption by yield
  Required fields: woNumber, yieldQty
  Optional fields: operationNumber, operator
  
  Response: {transaction:object}
  Blockchain events: BACKFLUSH_543
  
  Example:
  ```
  curl -X POST $BASE/api/backflush \
    -H "Content-Type: application/json" \
    -d '{
      "woNumber": "4000001",
      "yieldQty": 95,
      "operationNumber": "0010",
      "operator": "op123"
    }'
  ```


OPERATION CONFIRMATION (CO11N - Feedback)
==========================================

POST /api/confirmation
  Auth required: No
  Description: Confirm operation execution on work order
                Sets WO status to CNF
  Required fields: woNumber, operationNumber, yieldQty
  Optional fields: workCenter, confirmationType, scrapQty, giMode, 
                   actualLaborMin, machineTimeMin, operator, postingDate
  
  Response: {wtId:string, woNumber:string, status:"CNF", transaction:object}
  Blockchain events: OP_CONFIRMED
  
  Example:
  ```
  curl -X POST $BASE/api/confirmation \
    -H "Content-Type: application/json" \
    -d '{
      "woNumber": "4000001",
      "operationNumber": "0010",
      "yieldQty": 98,
      "workCenter": "WC-SMT-01",
      "scrapQty": 2,
      "actualLaborMin": 120,
      "machineTimeMin": 150,
      "operator": "op123",
      "postingDate": "2026-04-10"
    }'
  ```


QUALITY MANAGEMENT (QM)
=======================

POST /api/qm-result
  Auth required: No
  Description: Post Quality Management inspection result
  Required fields: woNumber, usageDecision
  Optional fields: grDocNumber, sampleSize, inspector, comments
  
  usageDecision values and stock effects:
    - PASS    -> stockEffect: UNRESTRICTED_3000
    - FAIL    -> stockEffect: BLOCKED_344_4000
    - QI      -> stockEffect: QI_STOCK_3000
  
  Response: {transaction:object, inspection:object}
  Blockchain events: INSP_GR_RESULT
  
  Example:
  ```
  curl -X POST $BASE/api/qm-result \
    -H "Content-Type: application/json" \
    -d '{
      "woNumber": "4000001",
      "usageDecision": "PASS",
      "grDocNumber": "5000001",
      "sampleSize": 10,
      "inspector": "qa-001",
      "comments": "All tests passed"
    }'
  ```


MATERIAL MOVEMENTS - REVERSALS & TRANSFERS
===========================================

POST /api/reversal
  Auth required: No
  Description: Reverse a material movement document
                MVT codes: 261->262, 101->102, 543->552
  Required fields: originalDocNumber
  Optional fields: reason, operator
  
  Response: {transaction:object}
  Blockchain events: REVERSAL_262 or REVERSAL_102 or REVERSAL_552
  
  Example:
  ```
  curl -X POST $BASE/api/reversal \
    -H "Content-Type: application/json" \
    -d '{
      "originalDocNumber": "5000001",
      "reason": "Quantity correction",
      "operator": "op123"
    }'
  ```


POST /api/transfer
  Auth required: No
  Description: Transfer material between storage locations (SAP MVT 311)
  Required fields: material, quantity
  Optional fields: slocFrom, slocTo, operator, reason
  
  Defaults:
    - slocFrom: 1000 (Raw Material)
    - slocTo: 2000 (Semi-Finished)
  
  Response: {docNumber:string, transaction:object}
  Blockchain events: TRANSFER_311
  
  Example:
  ```
  curl -X POST $BASE/api/transfer \
    -H "Content-Type: application/json" \
    -d '{
      "material": "MAT-001",
      "quantity": 50,
      "slocFrom": 1000,
      "slocTo": 2000,
      "operator": "op123",
      "reason": "Move to production"
    }'
  ```


POST /api/scrap
  Auth required: No
  Description: Post scrap material (SAP MVT 551)
  Required fields: material, quantity
  Optional fields: woNumber, slocFrom, scrapReason, operator
  
  Default slocFrom: 3000 (Finished Goods)
  
  Response: {transaction:object}
  Blockchain events: SCRAP_POSTING_551
  
  Example:
  ```
  curl -X POST $BASE/api/scrap \
    -H "Content-Type: application/json" \
    -d '{
      "material": "MAT-001",
      "quantity": 2,
      "woNumber": "4000001",
      "slocFrom": 3000,
      "scrapReason": "Quality failure",
      "operator": "op123"
    }'
  ```


INVENTORY MANAGEMENT (Legacy System)
====================================

GET /api/inventory
  Auth required: No
  Description: List all inventory items
  Response: Array of inventory objects
  
  Example:
  ```
  curl -X GET $BASE/api/inventory
  ```


GET /api/inventory/:id
  Auth required: No
  Description: Get single inventory item
  URL parameter: id
  Response: Inventory object with details
  
  Example:
  ```
  curl -X GET $BASE/api/inventory/INV-001
  ```


GET /api/inventory/:id/movements
  Auth required: No
  Description: Get movement history for inventory item
  URL parameter: id
  Response: Array of movement transactions
  
  Example:
  ```
  curl -X GET $BASE/api/inventory/INV-001/movements
  ```


POST /api/inventory/receive
  Auth required: No
  Description: Receive inventory into system
  Required fields: material, quantity
  Response: {transaction:object}
  
  Example:
  ```
  curl -X POST $BASE/api/inventory/receive \
    -H "Content-Type: application/json" \
    -d '{
      "material": "MAT-001",
      "quantity": 500
    }'
  ```


POST /api/inventory/transfer
  Auth required: No
  Description: Transfer inventory between locations
  Required fields: material, quantity, slocFrom, slocTo
  Response: {transaction:object}
  
  Example:
  ```
  curl -X POST $BASE/api/inventory/transfer \
    -H "Content-Type: application/json" \
    -d '{
      "material": "MAT-001",
      "quantity": 100,
      "slocFrom": "LOC-A",
      "slocTo": "LOC-B"
    }'
  ```


POST /api/inventory/issue
  Auth required: No
  Description: Issue inventory to work order
  Required fields: material, quantity, woNumber
  Response: {transaction:object}
  
  Example:
  ```
  curl -X POST $BASE/api/inventory/issue \
    -H "Content-Type: application/json" \
    -d '{
      "material": "MAT-001",
      "quantity": 100,
      "woNumber": "4000001"
    }'
  ```


POST /api/inventory/adjust
  Auth required: No
  Description: Post physical inventory adjustment
  Required fields: material, quantity, reason
  Response: {transaction:object}
  
  Example:
  ```
  curl -X POST $BASE/api/inventory/adjust \
    -H "Content-Type: application/json" \
    -d '{
      "material": "MAT-001",
      "quantity": -5,
      "reason": "Physical count variance"
    }'
  ```


GET /api/locations
  Auth required: No
  Description: Get stock aggregated by storage location
  Response: Object with stock quantities by storage location code
  
  Storage locations:
    - 1000: Raw Material
    - 2000: Semi-Finished
    - 3000: Finished Goods
    - 4000: Scrap / Reject
  
  Example:
  ```
  curl -X GET $BASE/api/locations
  ```


STOCK (SAP-Style by Storage Location)
=====================================

GET /api/stock
  Auth required: No
  Description: Get current stock by storage location (sloc)
  Response: Stock quantities by sloc with material details
  
  Storage location codes:
    - 1000: Raw Material
    - 2000: Semi-Finished
    - 3000: Finished Goods
    - 4000: Scrap / Reject
  
  Example:
  ```
  curl -X GET $BASE/api/stock
  ```


DOCUMENTS
=========

GET /api/documents
  Auth required: No
  Description: Get all posted material documents, sorted by date (most recent first)
  Response: Array of document objects with types, quantities, and dates
  
  Example:
  ```
  curl -X GET $BASE/api/documents
  ```


MATERIAL MASTER
===============

GET /api/materials
  Auth required: No
  Description: List all materials in system
  Response: Array of material objects
  
  Example:
  ```
  curl -X GET $BASE/api/materials
  ```


GET /api/materials/:id
  Auth required: No
  Description: Get single material by ID
  URL parameter: id (material code)
  Response: Material object with full details
  
  Example:
  ```
  curl -X GET $BASE/api/materials/MAT-001
  ```


POST /api/materials
  Auth required: No
  Description: Create new material master record
  Required fields: code, description, type, unit
  Response: {success:true, material:object}
  
  Example:
  ```
  curl -X POST $BASE/api/materials \
    -H "Content-Type: application/json" \
    -d '{
      "code": "MAT-NEW-001",
      "description": "Electronic Component Type X",
      "type": "component",
      "unit": "UN"
    }'
  ```


PUT /api/materials/:id
  Auth required: Yes (Admin role)
  Description: Update material master record
  URL parameter: id
  Optional fields: description, type, unit
  Response: {success:true, material:object}
  
  Example:
  ```
  curl -X PUT $BASE/api/materials/MAT-001 \
    -H "Authorization: Bearer TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"description": "Updated description"}'
  ```


DELETE /api/materials/:id
  Auth required: Yes (Admin role)
  Description: Delete material from system
  URL parameter: id
  Response: {success:true, message:string}
  
  Example:
  ```
  curl -X DELETE $BASE/api/materials/MAT-001 \
    -H "Authorization: Bearer TOKEN"
  ```


GET /api/materials/:id/where-used
  Auth required: No
  Description: Get where-used list (all BOMs containing this material)
  URL parameter: id
  Response: Array of BOM references
  
  Example:
  ```
  curl -X GET $BASE/api/materials/MAT-001/where-used
  ```


BILL OF MATERIALS (BOM)
=======================

GET /api/bom
  Auth required: No
  Description: List all BOMs in system
  Response: Array of BOM objects
  
  Example:
  ```
  curl -X GET $BASE/api/bom
  ```


GET /api/bom/:materialId
  Auth required: No
  Description: Get BOM for specific product material
  URL parameter: materialId (product code)
  Response: BOM object with component list
  
  Example:
  ```
  curl -X GET $BASE/api/bom/PROD-001
  ```


POST /api/bom
  Auth required: No
  Description: Create new BOM (bill of materials)
  Required fields: productCode, components
  components structure: [{materialCode, quantity, unit}, ...]
  
  Response: {success:true, bom:object}
  
  Example:
  ```
  curl -X POST $BASE/api/bom \
    -H "Content-Type: application/json" \
    -d '{
      "productCode": "PROD-001",
      "components": [
        {"materialCode": "MAT-001", "quantity": 10, "unit": "UN"},
        {"materialCode": "MAT-002", "quantity": 5, "unit": "UN"}
      ]
    }'
  ```


DELETE /api/bom/:materialId
  Auth required: Yes (Admin role)
  Description: Delete BOM for a product material
  URL parameter: materialId
  Response: {success:true, message:string}
  
  Example:
  ```
  curl -X DELETE $BASE/api/bom/PROD-001 \
    -H "Authorization: Bearer TOKEN"
  ```


WORK CENTERS
============

GET /api/work-centers
  Auth required: No
  Description: List all work centers in system
  Response: Array of work center objects
  
  Example:
  ```
  curl -X GET $BASE/api/work-centers
  ```


GET /api/work-centers/:id
  Auth required: No
  Description: Get single work center
  URL parameter: id (work center code)
  Response: Work center object with details
  
  Example:
  ```
  curl -X GET $BASE/api/work-centers/WC-SMT-01
  ```


POST /api/work-centers
  Auth required: No
  Description: Create new work center
  Required fields: code, description, type, capacityMin, plant, costCenter
  Response: {success:true, workCenter:object}
  
  Example:
  ```
  curl -X POST $BASE/api/work-centers \
    -H "Content-Type: application/json" \
    -d '{
      "code": "WC-NEW-01",
      "description": "New Assembly Line",
      "type": "assembly",
      "capacityMin": 480,
      "plant": 1000,
      "costCenter": "CC-001"
    }'
  ```


PUT /api/work-centers/:id
  Auth required: Yes (Admin role)
  Description: Update work center
  URL parameter: id
  Optional fields: description, type, capacityMin, plant, costCenter
  Response: {success:true, workCenter:object}
  
  Example:
  ```
  curl -X PUT $BASE/api/work-centers/WC-SMT-01 \
    -H "Authorization: Bearer TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"capacityMin": 500}'
  ```


DELETE /api/work-centers/:id
  Auth required: Yes (Admin role)
  Description: Delete work center
  URL parameter: id
  Response: {success:true, message:string}
  
  Example:
  ```
  curl -X DELETE $BASE/api/work-centers/WC-NEW-01 \
    -H "Authorization: Bearer TOKEN"
  ```


Pre-Configured Work Centers (seeded at startup)
  WC-SMT-01       SMT Pick & Place — Line 1
  WC-AOI-01       Automated Optical Inspection
  WC-PTH-01       PTH Insertion — Manual
  WC-WAVE         Wave Soldering Oven
  WC-TEST         Functional & ICT Test Station
  WC-VISUAL-INSP  Visual Inspection Bench
  WC-COAT         Conformal Coating Station
  WC-ASSY         Final Assembly & Integration
  WC-PACK         Packaging & Labeling


ROUTINGS (Operations & Sequences)
=================================

GET /api/routings
  Auth required: No
  Description: List all routings (operation sequences) in system
  Response: Array of routing objects
  
  Example:
  ```
  curl -X GET $BASE/api/routings
  ```


GET /api/routings/:id
  Auth required: No
  Description: Get single routing by ID
  URL parameter: id (routing ID)
  Response: Routing object with operation sequence
  
  Example:
  ```
  curl -X GET $BASE/api/routings/ROUT-001
  ```


GET /api/routings/material/:materialId
  Auth required: No
  Description: Get routing for a specific material
  URL parameter: materialId
  Response: Routing object for that material
  
  Example:
  ```
  curl -X GET $BASE/api/routings/material/PROD-001
  ```


POST /api/routings
  Auth required: No
  Description: Create new routing (operation sequence)
  Required fields: materialCode, operations
  operations structure: [{sequence, name, workCenter, standardTimeHours}, ...]
  
  Response: {success:true, routing:object}
  
  Example:
  ```
  curl -X POST $BASE/api/routings \
    -H "Content-Type: application/json" \
    -d '{
      "materialCode": "PROD-001",
      "operations": [
        {
          "sequence": 10,
          "name": "SMT Assembly",
          "workCenter": "WC-SMT-01",
          "standardTimeHours": 0.5
        },
        {
          "sequence": 20,
          "name": "AOI Inspection",
          "workCenter": "WC-AOI-01",
          "standardTimeHours": 0.25
        },
        {
          "sequence": 30,
          "name": "Functional Test",
          "workCenter": "WC-TEST",
          "standardTimeHours": 0.75
        }
      ]
    }'
  ```


DELETE /api/routings/:id
  Auth required: Yes (Admin role)
  Description: Delete routing
  URL parameter: id
  Response: {success:true, message:string}
  
  Example:
  ```
  curl -X DELETE $BASE/api/routings/ROUT-001 \
    -H "Authorization: Bearer TOKEN"
  ```


QUALITY RECORDS
===============

GET /api/quality
  Auth required: No
  Description: List all quality/inspection records
  Response: Array of quality record objects
  
  Example:
  ```
  curl -X GET $BASE/api/quality
  ```


POST /api/quality
  Auth required: No
  Description: Create inspection quality record
  Required fields: woNumber, inspector
  Optional fields: sampleSize, usageDecision, comments
  
  Response: {success:true, quality:object}
  
  Example:
  ```
  curl -X POST $BASE/api/quality \
    -H "Content-Type: application/json" \
    -d '{
      "woNumber": "4000001",
      "inspector": "qa-001",
      "sampleSize": 10,
      "usageDecision": "PASS",
      "comments": "Visual inspection passed"
    }'
  ```


REPORTS
=======

GET /api/reports/materials
  Auth required: No
  Description: Get material master report
  Response: Report object with material statistics
  
  Example:
  ```
  curl -X GET $BASE/api/reports/materials
  ```


GET /api/reports/bom
  Auth required: No
  Description: Get BOM usage report
  Response: Report object with BOM statistics
  
  Example:
  ```
  curl -X GET $BASE/api/reports/bom
  ```


GET /api/reports/stock
  Auth required: No
  Description: Get stock report aggregated by location
  Response: Report object with stock by sloc
  
  Example:
  ```
  curl -X GET $BASE/api/reports/stock
  ```


GET /api/reports/movements
  Auth required: No
  Description: Get material movement history report
  Response: Report object with movement statistics
  
  Example:
  ```
  curl -X GET $BASE/api/reports/movements
  ```


ERP EXPORT (IDOC Format)
========================

GET /api/erp/order/:id
  Auth required: Yes (Admin role)
  Description: Export work order in SAP IDOC LOIPRO01 format
  URL parameter: id (work order number)
  Response: IDOC XML structure for SAP production order
  
  Example:
  ```
  curl -X GET $BASE/api/erp/order/4000001 \
    -H "Authorization: Bearer TOKEN"
  ```


GET /api/erp/movement/:id
  Auth required: Yes (Admin role)
  Description: Export material movement in SAP IDOC MBGMCR02 format
  URL parameter: id (document number)
  Response: IDOC XML structure for SAP material movement
  
  Example:
  ```
  curl -X GET $BASE/api/erp/movement/5000001 \
    -H "Authorization: Bearer TOKEN"
  ```


GET /api/orders/:id/erp-export
  Auth required: Yes (Admin role)
  Description: Export order in ERP format
  URL parameter: id (order ID)
  Response: Order export object
  
  Example:
  ```
  curl -X GET $BASE/api/orders/4000001/erp-export \
    -H "Authorization: Bearer TOKEN"
  ```


GET /api/inventory/:id/erp-export
  Auth required: Yes (Admin role)
  Description: Export inventory item in ERP format
  URL parameter: id (inventory ID)
  Response: Inventory export object
  
  Example:
  ```
  curl -X GET $BASE/api/inventory/INV-001/erp-export \
    -H "Authorization: Bearer TOKEN"
  ```


LEGACY: WIP ORDERS SYSTEM
=========================

Note: The following endpoints are part of the legacy WIP orders system.
They run in parallel with the modern work order system.


GET /api/orders
  Auth required: No
  Description: List all legacy orders
  Response: Array of legacy order objects
  
  Example:
  ```
  curl -X GET $BASE/api/orders
  ```


GET /api/orders/:id
  Auth required: No
  Description: Get single legacy order
  URL parameter: id (order ID)
  Response: Legacy order object with details
  
  Example:
  ```
  curl -X GET $BASE/api/orders/4000001
  ```


POST /api/orders
  Auth required: No
  Description: Create legacy order
  Optional fields: material, quantity (deprecated)
  Response: {success:true, order:object}
  
  Example:
  ```
  curl -X POST $BASE/api/orders \
    -H "Content-Type: application/json" \
    -d '{"material": "MAT-001", "quantity": 100}'
  ```


POST /api/orders/:id/start-operation
  Auth required: No
  Description: Start operation on legacy order (deprecated)
  URL parameter: id
  Required fields: operationNumber
  Response: {success:true}
  
  Example:
  ```
  curl -X POST $BASE/api/orders/4000001/start-operation \
    -H "Content-Type: application/json" \
    -d '{"operationNumber": "0010"}'
  ```


POST /api/orders/:id/operations/:opId/complete
  Auth required: No
  Description: Complete operation on legacy order (deprecated)
  URL parameters: id (order ID), opId (operation ID)
  Response: {success:true}
  
  Example:
  ```
  curl -X POST $BASE/api/orders/4000001/operations/0010/complete \
    -H "Content-Type: application/json" \
    -d '{}'
  ```


LEGACY: WIP BOARD SYSTEM
========================

GET /api/wip
  Auth required: No
  Description: Get WIP board state (all items on board)
  Response: Array of WIP items with status
  
  Example:
  ```
  curl -X GET $BASE/api/wip
  ```


GET /api/wip/:id
  Auth required: No
  Description: Get single WIP item
  URL parameter: id
  Response: WIP item object
  
  Example:
  ```
  curl -X GET $BASE/api/wip/WIP-001
  ```


POST /api/wip/:id/move
  Auth required: No
  Description: Move WIP item to next stage on board
  URL parameter: id
  Required fields: stage
  Response: {success:true, wipItem:object}
  
  Example:
  ```
  curl -X POST $BASE/api/wip/WIP-001/move \
    -H "Content-Type: application/json" \
    -d '{"stage": "in-process"}'
  ```


POST /api/wip-event
  Auth required: No
  Description: Post WIP event
  Required fields: wipId, event
  Response: {success:true, event:object}
  
  Example:
  ```
  curl -X POST $BASE/api/wip-event \
    -H "Content-Type: application/json" \
    -d '{
      "wipId": "WIP-001",
      "event": "moved_to_next_stage"
    }'
  ```


HTTP STATUS CODES & ERROR RESPONSES
===================================

200 OK
  Request succeeded. Response body contains result data.

201 Created
  Resource created successfully. Response includes created object.

400 Bad Request
  Invalid request. Required field missing or invalid format.
  Response: {error: "description of missing/invalid field"}

401 Unauthorized
  Missing or invalid authentication token.
  Response: {error: "Unauthorized"}

403 Forbidden
  Authenticated but insufficient role/permissions.
  Response: {error: "Forbidden - admin role required"}

404 Not Found
  Resource does not exist.
  Response: {error: "Work order 4000999 not found"}

500 Internal Server Error
  Server error during processing.
  Response: {error: "Internal server error"}


AUTHENTICATION
==============

All endpoints requiring auth expect:
  Header: Authorization: Bearer TOKEN

Roles:
  - admin: Full system access, user management, blockchain access
  - operator: Can create/confirm orders, post movements
  - viewer: Read-only access

To obtain token, authenticate via separate login endpoint (not documented here).


VERSION HISTORY
===============

API Version: 1.0 (April 2026)
Base URL: https://wip.rodrigoandremarques.com

This document covers all production endpoints as of April 7, 2026.
