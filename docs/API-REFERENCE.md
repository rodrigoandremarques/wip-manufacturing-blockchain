# API Reference

All endpoints require HTTP Basic Auth. Visitor accounts may only call GET endpoints.

Base URL: https://wip.rodrigoandremarques.com

Authentication header: Authorization: Basic base64(username:password)

All responses follow the format:
- Success: { "success": true, "data": ... }
- Failure: { "success": false, "error": "..." }

---

## Authentication

### GET /api/auth/me
Verify credentials and return the authenticated user profile.

Auth required: any role

    curl -u 'demo:WIP-Demo-2026!' https://wip.rodrigoandremarques.com/api/auth/me

Response:

    { "success": true, "data": { "username": "demo", "role": "visitor" } }

---

### GET /api/health
Health check. No authentication required.

    curl https://wip.rodrigoandremarques.com/api/health

Response:

    { "status": "ok", "uptime": 3600.5 }

---

### GET /api/stats
Dashboard summary statistics.

Auth required: any role

Response:

    {
      "success": true,
      "data": {
        "openOrders": 5,
        "totalMovements": 142,
        "blockCount": 89,
        "chainValid": true
      }
    }

---

## Production Orders

### GET /api/work-orders
List all production orders.

Auth required: any role

### GET /api/work-order/:id
Full detail for a specific production order, including operations.

Auth required: any role

### POST /api/work-order
Create a new production order.

Auth required: operator or admin

Request body:

    {
      "material":   "FG-001",
      "quantity":   100,
      "unit":       "EA",
      "workCenter": "ASSEMBLY-01",
      "routing":    "RT-FG-001"
    }

Response includes the created order and the corresponding blockchain block reference.

### POST /api/work-order/:id/release
Release the order to the shop floor. Status changes to Released.

Auth required: operator or admin

### POST /api/work-order/:id/teco
Mark the order as technically complete.

Auth required: operator or admin

### POST /api/work-order/:id/close
Close the order. No further movements allowed after this point.

Auth required: operator or admin

---

## Goods Movements

### POST /api/goods-issue
Issue materials from stock against a production order.

Auth required: operator or admin

Request body:

    {
      "orderId": "WO-20260406-001",
      "movements": [
        { "material": "ROH-100", "quantity": 50, "unit": "KG", "location": "STAGE-01" }
      ],
      "actor": "operator1"
    }

### POST /api/goods-receipt
Receive finished goods into stock on order completion.

Auth required: operator or admin

Request body:

    {
      "orderId":  "WO-20260406-001",
      "quantity": 98,
      "unit":     "EA",
      "location": "FG-STORE",
      "actor":    "operator1"
    }

### POST /api/confirmation
Confirm completion of a routing operation.

Auth required: operator or admin

Request body:

    {
      "orderId":     "WO-20260406-001",
      "operationId": "OP-10",
      "yieldQty":    100,
      "scrapQty":    2,
      "operator":    "operator1",
      "workCenter":  "CUT-01"
    }

### POST /api/backflush
Automatic material consumption triggered on operation confirmation.

Auth required: operator or admin

Request body:

    {
      "orderId":      "WO-20260406-001",
      "operationId":  "OP-10",
      "confirmedQty": 100,
      "actor":        "operator1"
    }

### POST /api/transfer
Internal stock transfer between locations.

Auth required: operator or admin

Request body:

    {
      "material":     "ROH-100",
      "quantity":     50,
      "fromLocation": "WH-01",
      "toLocation":   "STAGE-01",
      "actor":        "operator1"
    }

### POST /api/scrap
Write off material as scrap.

Auth required: operator or admin

Request body:

    {
      "material": "ROH-100",
      "quantity": 5,
      "location": "STAGE-01",
      "reason":   "Damaged on receipt",
      "actor":    "operator1"
    }

### POST /api/reversal
Cancel a previous goods movement.

Auth required: operator or admin

Request body:

    {
      "movementId": "MV-20260406-042",
      "reason":     "Wrong quantity entered",
      "actor":      "operator1"
    }

---

## Inventory

### GET /api/inventory
All materials with current stock levels by location.

Auth required: any role

### GET /api/inventory/:id
Stock detail for a specific material.

Auth required: any role

### POST /api/inventory/receive
Receive materials from a supplier (goods receiving).

Auth required: operator or admin

Request body:

    {
      "material": "ROH-100",
      "quantity": 200,
      "unit":     "KG",
      "location": "WH-01",
      "supplier": "Supplier A",
      "actor":    "operator1"
    }

### GET /api/inventory/:id/movements
Movement history for a specific material.

Auth required: any role

### GET /api/stock
Full stock snapshot grouped by material.

Auth required: any role

### GET /api/locations
List all storage locations.

Auth required: any role

---

## Quality Management

### GET /api/quality
List all quality inspection records.

Auth required: any role

### POST /api/quality
Record a process inspection result.

Auth required: operator or admin

Request body:

    {
      "orderId":        "WO-20260406-001",
      "characteristic": "Dimension A",
      "result":         "50.2",
      "unit":           "mm",
      "status":         "Pass",
      "inspector":      "qm1"
    }

### POST /api/qm-result
Record the final quality decision for a production order.

Auth required: operator or admin

Request body:

    {
      "orderId":   "WO-20260406-001",
      "decision":  "Accepted",
      "inspector": "qm1",
      "remarks":   "All characteristics within tolerance"
    }

---

## Material Master

### GET /api/materials
List all materials.

Auth required: any role

### GET /api/materials/:id
Get a specific material.

Auth required: any role

### POST /api/materials
Create a material.

Auth required: operator or admin

Request body:

    {
      "id":          "ROH-300",
      "description": "Aluminium Sheet 1mm",
      "unit":        "KG",
      "type":        "ROH"
    }

### PUT /api/materials/:id
Update a material record.

Auth required: admin

### DELETE /api/materials/:id
Delete a material.

Auth required: admin

### GET /api/materials/:id/where-used
Returns which BOMs reference this material.

Auth required: any role

---

## Bill of Materials

### GET /api/bom
List all BOMs.

Auth required: any role

### GET /api/bom/:materialId
BOM for a specific finished product.

Auth required: any role

### POST /api/bom
Create or replace a BOM.

Auth required: operator or admin

Request body:

    {
      "materialId": "FG-001",
      "components": [
        { "material": "ROH-100", "quantity": 5, "unit": "KG" },
        { "material": "ROH-200", "quantity": 2, "unit": "EA" }
      ]
    }

### DELETE /api/bom/:materialId
Delete a BOM.

Auth required: admin

---

## Work Centers

### GET /api/work-centers
List all work centers.

Auth required: any role

### POST /api/work-centers
Create a work center.

Auth required: operator or admin

Request body:

    {
      "id":          "WELD-02",
      "description": "Welding Station 2",
      "capacity":    480,
      "costRate":    45.00
    }

### PUT /api/work-centers/:id
Update a work center.

Auth required: admin

### DELETE /api/work-centers/:id
Delete a work center.

Auth required: admin

---

## Routings

### GET /api/routings
List all routings.

Auth required: any role

### GET /api/routings/:id
Get a specific routing.

Auth required: any role

### GET /api/routings/material/:materialId
Get the routing for a specific material.

Auth required: any role

### POST /api/routings
Create a routing.

Auth required: operator or admin

Request body:

    {
      "materialId": "FG-001",
      "operations": [
        { "opNo": 10, "description": "Cutting",  "workCenter": "CUT-01",  "stdTime": 30 },
        { "opNo": 20, "description": "Welding",  "workCenter": "WELD-01", "stdTime": 45 },
        { "opNo": 30, "description": "Assembly", "workCenter": "ASSY-01", "stdTime": 60 }
      ]
    }

### DELETE /api/routings/:id
Delete a routing.

Auth required: admin

---

## Blockchain

### GET /api/chain/status
Chain integrity status.

Auth required: any role

Response:

    {
      "success": true,
      "data": {
        "valid":      true,
        "blockCount": 89,
        "lastHash":   "7b2c..."
      }
    }

### GET /api/audit-trail
Filtered event log from the blockchain.

Auth required: any role

Query parameters: type, orderId, from (date), to (date)

### GET /api/chain
Full raw blockchain dump with all blocks and hashes.

Auth required: admin

---

## Reports

### GET /api/reports/stock
Current stock levels across all materials and locations.

Auth required: any role

### GET /api/reports/movements
Goods movement history. Supports query parameters: material, from, to.

Auth required: any role

### GET /api/reports/materials
Material master report.

Auth required: any role

### GET /api/reports/bom
BOM explosion report.

Auth required: any role

---

## ERP Bridge (Admin Only)

### GET /api/erp/order/:id
Production order data in ERP-compatible JSON format.

Auth required: admin

### GET /api/erp/movement/:id
Goods movement in ERP-compatible JSON format.

Auth required: admin

---

## User Management (Admin Only)

### GET /api/admin/users
List all users (without password hashes).

Auth required: admin

Response:

    {
      "success": true,
      "data": [
        { "username": "admin",     "role": "admin",    "createdAt": "2026-01-01T00:00:00Z" },
        { "username": "operator1", "role": "operator", "createdAt": "2026-03-15T09:00:00Z" },
        { "username": "demo",      "role": "visitor",  "createdAt": "2026-04-06T10:07:00Z" }
      ]
    }

### POST /api/admin/users
Create a new user.

Auth required: admin

Request body:

    {
      "username": "operator2",
      "password": "SecurePass123",
      "role":     "operator"
    }

Valid roles: admin, operator, visitor.

### DELETE /api/admin/users/:username
Delete a user. The primary admin account cannot be deleted.

Auth required: admin

### PUT /api/admin/users/:username/password
Change a user's password.

Auth required: admin

Request body:

    { "newPassword": "NewPass456" }

---

## WIP Board

### GET /api/wip
All active WIP orders grouped by work center.

Auth required: any role

### GET /api/wip/:id
WIP detail for a specific order.

Auth required: any role

### POST /api/wip/:id/move
Move a WIP order to a different work center or status.

Auth required: operator or admin

---

## HTTP Status Codes

| Code | Meaning                                       |
|------|-----------------------------------------------|
| 200  | Success                                       |
| 400  | Bad request -- missing or invalid parameters  |
| 401  | Unauthorised -- invalid or missing credentials |
| 403  | Forbidden -- insufficient role                |
| 404  | Not found                                     |
| 500  | Internal server error                         |
