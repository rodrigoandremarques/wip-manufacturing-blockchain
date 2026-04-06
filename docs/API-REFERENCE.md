# API Reference

All endpoints require **HTTP Basic Auth**. Visitor accounts may only call GET endpoints.

**Base URL:** `https://wip.rodrigoandremarques.com`

**Auth header:** `Authorization: Basic base64(username:password)`

**Response format:** All responses are JSON `{ success: true, data: ... }` on success, or `{ success: false, error: "..."}` on failure.

---

## Authentication

### GET /api/auth/me
Returns the authenticated user's profile. Use this to verify credentials are correct.

**Auth:** any role

```bash
curl -u 'demo:WIP-Demo-2026!' https://wip.rodrigoandremarques.com/api/auth/me
```

**Response:**
```json
{
  "success": true,
  "data": { "username": "demo", "role": "visitor" }
}
```

---

### GET /api/health
Health check endpoint. No auth required.

```bash
curl https://wip.rodrigoandremarques.com/api/health
```

**Response:**
```json
{ "status": "ok", "uptime": 3600.5 }
```

---

### GET /api/stats
Dashboard summary statistics.

**Auth:** any role

**Response:**
```json
{
  "success": true,
  "data": {
    "openOrders": 5,
    "totalMovements": 142,
    "blockCount": 89,
    "chainValid": true
  }
}
```

---

## Production Orders

### GET /api/work-orders
List all production orders.

**Auth:** any role

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "WO-20260406-001",
      "material": "FG-001",
      "quantity": 100,
      "status": "Released",
      "workCenter": "ASSEMBLY-01"
    }
  ]
}
```

---

### GET /api/work-order/:id
Get a specific production order with full details and operations.

**Auth:** any role

---

### POST /api/work-order
Create a new production order.

**Auth:** operator or admin

**Request body:**
```json
{
  "material": "FG-001",
  "quantity": 100,
  "unit": "EA",
  "workCenter": "ASSEMBLY-01",
  "routing": "RT-FG-001"
}
```

**Response:** Created order object + blockchain block reference.

---

### POST /api/work-order/:id/release
Release a production order to the shop floor.

**Auth:** operator or admin

---

### POST /api/work-order/:id/teco
Mark a production order as technically complete.

**Auth:** operator or admin

---

### POST /api/work-order/:id/close
Close a production order. No further movements allowed.

**Auth:** operator or admin

---

## Goods Movements

### POST /api/goods-issue
Issue materials from stock to a production order (consumption).

**Auth:** operator or admin

**Request body:**
```json
{
  "orderId": "WO-20260406-001",
  "movements": [
    { "material": "ROH-100", "quantity": 50, "unit": "KG", "location": "STAGE-01" }
  ],
  "actor": "operator1"
}
```

---

### POST /api/goods-receipt
Receive finished goods into stock on order completion.

**Auth:** operator or admin

**Request body:**
```json
{
  "orderId": "WO-20260406-001",
  "quantity": 98,
  "unit": "EA",
  "location": "FG-STORE",
  "actor": "operator1"
}
```

---

### POST /api/confirmation
Confirm completion of a routing operation.

**Auth:** operator or admin

**Request body:**
```json
{
  "orderId": "WO-20260406-001",
  "operationId": "OP-10",
  "yieldQty": 100,
  "scrapQty": 2,
  "operator": "operator1",
  "workCenter": "CUT-01"
}
```

---

### POST /api/backflush
Automatic material consumption triggered on operation confirmation.

**Auth:** operator or admin

**Request body:**
```json
{
  "orderId": "WO-20260406-001",
  "operationId": "OP-10",
  "confirmedQty": 100,
  "actor": "operator1"
}
```

---

### POST /api/transfer
Internal stock transfer between locations.

**Auth:** operator or admin

**Request body:**
```json
{
  "material": "ROH-100",
  "quantity": 50,
  "fromLocation": "WH-01",
  "toLocation": "STAGE-01",
  "actor": "operator1"
}
```

---

### POST /api/scrap
Write off material as scrap.

**Auth:** operator or admin

**Request body:**
```json
{
  "material": "ROH-100",
  "quantity": 5,
  "location": "STAGE-01",
  "reason": "Damaged on receipt",
  "actor": "operator1"
}
```

---

### POST /api/reversal
Reverse (cancel) a previous goods movement.

**Auth:** operator or admin

**Request body:**
```json
{
  "movementId": "MV-20260406-042",
  "reason": "Wrong quantity entered",
  "actor": "operator1"
}
```

---

## Inventory

### GET /api/inventory
List all materials with current stock levels and locations.

**Auth:** any role

---

### GET /api/inventory/:id
Get stock levels for a specific material.

**Auth:** any role

---

### POST /api/inventory/receive
Receive materials from a supplier (goods receiving).

**Auth:** operator or admin

**Request body:**
```json
{
  "material": "ROH-100",
  "quantity": 200,
  "unit": "KG",
  "location": "WH-01",
  "supplier": "Supplier A",
  "actor": "operator1"
}
```

---

### GET /api/inventory/:id/movements
Movement history for a specific material.

**Auth:** any role

---

### GET /api/stock
Full stock snapshot grouped by material.

**Auth:** any role

---

### GET /api/locations
List all storage locations.

**Auth:** any role

---

## Quality Management

### GET /api/quality
List all quality inspection records.

**Auth:** any role

---

### POST /api/quality
Create a process inspection record.

**Auth:** operator or admin

**Request body:**
```json
{
  "orderId": "WO-20260406-001",
  "characteristic": "Dimension A",
  "result": "50.2",
  "unit": "mm",
  "status": "Pass",
  "inspector": "qm1"
}
```

---

### POST /api/qm-result
Record the final quality decision for a production order.

**Auth:** operator or admin

**Request body:**
```json
{
  "orderId": "WO-20260406-001",
  "decision": "Accepted",
  "inspector": "qm1",
  "remarks": "All characteristics within tolerance"
}
```

---

## Material Master

### GET /api/materials
List all materials.

**Auth:** any role

---

### GET /api/materials/:id
Get a specific material.

**Auth:** any role

---

### POST /api/materials
Create a material.

**Auth:** operator or admin

**Request body:**
```json
{
  "id": "ROH-300",
  "description": "Aluminium Sheet 1mm",
  "unit": "KG",
  "type": "ROH"
}
```

---

### PUT /api/materials/:id
Update a material.

**Auth:** admin

---

### DELETE /api/materials/:id
Delete a material.

**Auth:** admin

---

### GET /api/materials/:id/where-used
Find which BOMs use this material.

**Auth:** any role

---

## Bill of Materials

### GET /api/bom
List all BOMs.

**Auth:** any role

---

### GET /api/bom/:materialId
Get the BOM for a specific finished product.

**Auth:** any role

---

### POST /api/bom
Create or replace a BOM.

**Auth:** operator or admin

**Request body:**
```json
{
  "materialId": "FG-001",
  "components": [
    { "material": "ROH-100", "quantity": 5, "unit": "KG" },
    { "material": "ROH-200", "quantity": 2, "unit": "EA" }
  ]
}
```

---

### DELETE /api/bom/:materialId
Delete a BOM.

**Auth:** admin

---

## Work Centers

### GET /api/work-centers
List all work centers.

**Auth:** any role

---

### POST /api/work-centers
Create a work center.

**Auth:** operator or admin

**Request body:**
```json
{
  "id": "WELD-02",
  "description": "Welding Station 2",
  "capacity": 480,
  "costRate": 45.00
}
```

---

### PUT /api/work-centers/:id
Update a work center.

**Auth:** admin

---

### DELETE /api/work-centers/:id
Delete a work center.

**Auth:** admin

---

## Routings

### GET /api/routings
List all routings.

**Auth:** any role

---

### GET /api/routings/:id
Get a specific routing.

**Auth:** any role

---

### GET /api/routings/material/:materialId
Get the routing for a specific material.

**Auth:** any role

---

### POST /api/routings
Create a routing.

**Auth:** operator or admin

**Request body:**
```json
{
  "materialId": "FG-001",
  "operations": [
    { "opNo": 10, "description": "Cutting",  "workCenter": "CUT-01",  "stdTime": 30 },
    { "opNo": 20, "description": "Welding",  "workCenter": "WELD-01", "stdTime": 45 },
    { "opNo": 30, "description": "Assembly", "workCenter": "ASSY-01", "stdTime": 60 }
  ]
}
```

---

### DELETE /api/routings/:id
Delete a routing.

**Auth:** admin

---

## Blockchain

### GET /api/chain/status
Chain validity status.

**Auth:** any role

**Response:**
```json
{
  "success": true,
  "data": {
    "valid": true,
    "blockCount": 89,
    "lastHash": "7b2c..."
  }
}
```

---

### GET /api/audit-trail
Filtered audit trail of all blockchain events.

**Auth:** any role

**Query parameters:** `?type=GOODS_ISSUE&orderId=WO-001&from=2026-04-01&to=2026-04-06`

---

### GET /api/chain
Full raw blockchain dump (all blocks with hashes).

**Auth:** admin only

---

## Reports

### GET /api/reports/stock
Current stock levels across all materials and locations.

**Auth:** any role

---

### GET /api/reports/movements
Goods movement history.

**Auth:** any role

**Query parameters:** `?material=ROH-100&from=2026-04-01&to=2026-04-06`

---

### GET /api/reports/materials
Material master report.

**Auth:** any role

---

### GET /api/reports/bom
BOM explosion report.

**Auth:** any role

---

## ERP Bridge *(Admin only)*

### GET /api/erp/order/:id
Production order in ERP-ready JSON format (SAP/Oracle compatible structure).

**Auth:** admin

---

### GET /api/erp/movement/:id
Goods movement in ERP-ready JSON format.

**Auth:** admin

---

### GET /api/orders/:id/erp-export
Alternative ERP export for a specific order.

**Auth:** admin

---

## User Management *(Admin only)*

### GET /api/admin/users
List all users (without password hashes).

**Auth:** admin

**Response:**
```json
{
  "success": true,
  "data": [
    { "username": "admin",    "role": "admin",    "createdAt": "2026-01-01T00:00:00Z" },
    { "username": "operator1","role": "operator", "createdAt": "2026-03-15T09:00:00Z" },
    { "username": "demo",     "role": "visitor",  "createdAt": "2026-04-06T10:07:00Z" }
  ]
}
```

---

### POST /api/admin/users
Create a new user.

**Auth:** admin

**Request body:**
```json
{
  "username": "operator2",
  "password": "SecurePass123",
  "role": "operator"
}
```

Valid roles: `admin`, `operator`, `visitor`

---

### DELETE /api/admin/users/:username
Delete a user. Cannot delete the primary admin.

**Auth:** admin

---

### PUT /api/admin/users/:username/password
Change a user's password.

**Auth:** admin

**Request body:**
```json
{ "newPassword": "NewPass456" }
```

---

## WIP Board

### GET /api/wip
All active WIP orders grouped by work center.

**Auth:** any role

---

### GET /api/wip/:id
WIP details for a specific order.

**Auth:** any role

---

### POST /api/wip/:id/move
Move a WIP order to a different work center or status.

**Auth:** operator or admin

---

## HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Bad request Ñ missing or invalid parameters |
| 401 | Unauthorised Ñ invalid or missing credentials |
| 403 | Forbidden Ñ insufficient role |
| 404 | Not found |
| 500 | Internal server error |
