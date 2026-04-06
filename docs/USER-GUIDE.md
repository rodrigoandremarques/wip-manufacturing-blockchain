# User Guide

## Getting Started

### Accessing the System

Open [wip.rodrigoandremarques.com](https://wip.rodrigoandremarques.com) in any browser.

**Note on server warm-up:** The server runs on Render's free tier and sleeps after 15 minutes of inactivity. The first request after a dormant period can take 30-50 seconds. Wait for the page to finish loading before attempting to sign in.

### Signing In

Enter your username and password on the login screen and click Sign In.

| Role     | Permissions                                                                 |
|----------|-----------------------------------------------------------------------------|
| Admin    | Full access: user management, all movements, ERP export, raw blockchain dump |
| Operator | All shop floor write operations: GI, GR, confirmation, transfer, scrap, QM  |
| Visitor  | Read-only: all dashboards, reports, and blockchain ledger                    |

Demo account (read-only): username `demo`, password `WIP-Demo-2026!`

### Signing Out

After signing in, a Logout button appears in the bottom-left corner of the sidebar next to your username. Click it to clear the session.

---

## Navigation

The left sidebar is divided into sections:

| Section                     | Contents                                                        |
|-----------------------------|-----------------------------------------------------------------|
| Dashboard                   | KPI summary: active orders, WIP value, movements today          |
| Maintain Production Orders  | Create and manage production orders                             |
| WIP Board                   | Live shop floor view grouped by work center                     |
| Stock & Movements           | Goods Receiving, Staging, Issue, Confirmation, Goods Receipt    |
| Process Inspection          | Quality management and QM results                               |
| Material                    | Material master catalogue                                        |
| Bill of Material            | BOM maintenance                                                 |
| Work Centers                | Work center definitions                                         |
| Routing                     | Operation sequences per finished product                        |
| Audit Trail                 | Filtered event history                                          |
| Blockchain Ledger           | Full SHA-256 chain with real-time integrity validation          |
| ERP Bridge                  | Structured API output for SAP/Oracle integration (admin only)   |
| All Reports                 | WIP summary, stock, movements, BOM explosion                    |

---

## Production Order Lifecycle

### Step 1 -- Create a Production Order

1. Go to Maintain Production Orders.
2. Click New Order.
3. Fill in the material, planned quantity, work center, and routing.
4. Click Create. The order is created with status Created and recorded on the blockchain.

### Step 2 -- Release the Order

From the order detail view, click Release. Status changes to Released, making the order visible on the WIP Board and allowing material issue.

### Step 3 -- Issue Materials

1. Go to Stock & Movements -> Goods Issue.
2. Select the production order.
3. The system shows BOM components with required quantities. Adjust if needed.
4. Confirm. Stock is decremented and a blockchain entry is created.

### Step 4 -- Confirm Operations

For each routing operation:

1. Go to Stock & Movements -> Confirmation.
2. Select the order and operation.
3. Enter yield quantity, scrap quantity, and operator name.
4. Click Confirm. The operation is recorded and the order status is updated.

### Step 5 -- Goods Receipt (Finished Goods)

When production is complete:

1. Go to Stock & Movements -> Goods Receipt.
2. Select the completed production order and confirm the received quantity.
3. Finished goods enter stock and a blockchain entry is created.
4. Order status moves to Completed.

### Step 6 -- Close the Order

From the order detail view, click TECO (Technically Complete), then Close. No further movements are permitted after closing.

---

## Goods Receiving (Raw Materials)

1. Go to Stock & Movements -> Goods Receiving.
2. Select the material being received from the supplier.
3. Enter quantity and storage location.
4. Click Receive. Stock increases and a blockchain entry is created.

---

## Staging to Production

Moves materials from the warehouse to the production staging area before issue:

1. Go to Stock & Movements -> Staging to Production.
2. Select material, source location, and destination.
3. Enter quantity and confirm. A transfer entry is recorded on the blockchain.

---

## Quality Management

### Process Inspection

Records in-process quality results during production:

1. Go to Process Inspection.
2. Select the production order.
3. Enter the characteristic, measured value, unit, result status (Pass/Fail), and inspector name.
4. Click Submit. The result is recorded on the blockchain.

### QM Result

The final quality decision for the order (Accept or Reject) is recorded as a separate blockchain entry.

---

## Blockchain Ledger

Every transaction creates an immutable block:

- **Block number** -- sequential index from genesis
- **Timestamp** -- ISO 8601 UTC
- **Event type** -- e.g. GOODS_ISSUE, OP_CONFIRMED, GOODS_RECEIPT
- **Payload** -- order, material, quantity, actor
- **Hash** -- SHA-256 of previousHash + JSON payload
- **Previous hash** -- cryptographic link to the prior block
- **Valid** -- green check if the chain is intact, red warning if tampered

The Chain Status indicator at the top of the Blockchain Ledger screen shows overall chain validity in real time.

---

## Reports

| Report            | Description                                           |
|-------------------|-------------------------------------------------------|
| WIP Summary       | Current stock by material and location                |
| Movement History  | All goods movements, filterable by date/material/order |
| Order Status      | All production orders with quantities and status       |
| BOM Report        | Bill of materials explosion for a selected material   |
| Stock Report      | Full inventory snapshot                               |

---

## Material Master

1. Go to Material -> Material Master -> New Material.
2. Enter the material ID, description, unit of measure, and material type (FERT, ROH, HALB).
3. Save.

### Bill of Materials

1. Go to Bill of Material and create or select a BOM for a finished product.
2. Add components with quantities and units.
3. Save. The BOM is used for automatic component explosion on Goods Issue.

---

## Work Centers and Routings

### Work Centers

1. Go to Work Centers -> New Work Center.
2. Enter ID, description, and optionally capacity and cost rate.

### Routings

1. Go to Routing -> New Routing and select the material.
2. Add operations in sequence: operation number, work center, description, and standard time.
3. Save. This routing is available when creating production orders for that material.

---

## ERP Bridge (Admin Only)

Exposes production orders and goods movements in a structured JSON format for integration with SAP or other ERP systems.

Endpoints:
- GET /api/erp/order/{id}
- GET /api/erp/movement/{id}

Authentication uses the same HTTP Basic Auth as the rest of the API. Admin role required.

---

## User Management (Admin Only)

### Create a User

```
curl -X POST https://wip.rodrigoandremarques.com/api/admin/users 
  -u 'admin:PASSWORD' 
  -H 'Content-Type: application/json' 
  -d '{"username": "operator1", "password": "SecurePass123", "role": "operator"}'
```

Valid roles: admin, operator, visitor.

### Change a Password

```
curl -X PUT https://wip.rodrigoandremarques.com/api/admin/users/operator1/password 
  -u 'admin:PASSWORD' 
  -H 'Content-Type: application/json' 
  -d '{"newPassword": "NewPass456"}'
```

### Delete a User

```
curl -X DELETE https://wip.rodrigoandremarques.com/api/admin/users/operator1 
  -u 'admin:PASSWORD'
```

The primary admin account (set via ADMIN_USER env var) cannot be deleted.

---

## Troubleshooting

| Issue                          | Resolution                                                                  |
|--------------------------------|-----------------------------------------------------------------------------|
| Login fails on first attempt   | Wait 30-50 seconds for the free-tier server to wake up, then try again      |
| Logout button not visible      | The button appears only after a successful login, in the sidebar footer     |
| Chain shown as invalid         | Do not edit data/*.json files manually -- all changes must go through the API |
| Goods Issue fails              | Verify the order is in Released status and the materials are in stock       |
| Confirmation fails             | Ensure the order is released and materials have been issued                 |
