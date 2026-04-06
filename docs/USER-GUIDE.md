# User Guide

## Getting Started

### Accessing the System

Open **[wip.rodrigoandremarques.com](https://wip.rodrigoandremarques.com)** in any browser.

> **Note Ñ Free Tier Warm-Up:** The server sleeps after 15 minutes of inactivity. The first request after inactivity may take 30Ð50 seconds. Wait for the page to load fully before logging in.

### Logging In

Enter your username and password on the login screen and click **Sign In**.

| Role | What you can do |
|------|----------------|
| **Admin** | Everything: user management, all movements, ERP export, raw blockchain |
| **Operator** | All shop floor movements (GI, GR, confirmation, transfer, scrap, QM) |
| **Visitor** | Read-only: view all dashboards, reports, ledger Ñ no write operations |

**Demo account (read-only):** `demo` / `WIP-Demo-2026!`

### Logging Out

After logging in, a **Logout** button (?) appears in the bottom-left corner of the sidebar next to your username. Click it to end your session.

---

## Navigation

The left sidebar organises the system into sections:

| Section | Contents |
|---------|---------|
| **Dashboard** | KPI summary Ñ active orders, WIP value, movements today |
| **Maintain Production Orders** | Create and manage production orders |
| **WIP Board** | Live shop floor view by work center |
| **Stock & Movements** | Goods Receiving, Staging, Issue, Confirmation, Goods Receipt |
| **Process Inspection** | Quality management and QM results |
| **Material** | Material Master catalogue |
| **Bill of Material** | BOM maintenance |
| **Work Centers** | Work center definitions |
| **Routing** | Operation sequences per material |
| **Audit Trail** | Filtered event history |
| **Blockchain Ledger** | Full SHA-256 chain view with validation |
| **ERP Bridge** | REST API output for SAP/Oracle integration *(admin only)* |
| **All Reports** | WIP summary, stock, movements, BOM |

---

## Production Order Lifecycle

### Step 1 Ñ Create a Production Order

1. Go to **Maintain Production Orders**
2. Click **New Order**
3. Fill in:
   - **Material** Ñ the finished product to produce
   - **Quantity** Ñ planned output quantity
   - **Work Center** Ñ primary work center
   - **Routing** Ñ operation sequence (auto-populated from material routing)
4. Click **Create** ? order is created with status **Created** and recorded on the blockchain

### Step 2 Ñ Release the Order

From the order detail view, click **Release**. Status changes to **Released**, making it visible on the WIP Board and allowing material issue.

### Step 3 Ñ Issue Materials (Goods Issue)

1. Go to **Stock & Movements ? Goods Issue**
2. Select the Production Order
3. System shows the BOM components with required quantities
4. Adjust quantities if needed and confirm ? stock is decremented, blockchain entry created

### Step 4 Ñ Confirm Operations

For each routing step:

1. Go to **Stock & Movements ? Confirmation**
2. Select the order and operation
3. Enter:
   - **Yield quantity** Ñ good units produced
   - **Scrap quantity** Ñ rejected units
   - **Operator name**
4. Click **Confirm** ? operation is recorded, status updated

### Step 5 Ñ Goods Receipt (Finished Goods)

When production is complete:

1. Go to **Stock & Movements ? Goods Receipt**
2. Select the completed Production Order
3. Confirm the received quantity ? finished goods enter stock, blockchain entry created
4. Order status moves to **Completed**

### Step 6 Ñ Close the Order

From the order detail view, click **TECO** (Technically Complete) then **Close**. No further movements are allowed.

---

## Goods Receiving (Raw Materials from Suppliers)

1. Go to **Stock & Movements ? Goods Receiving**
2. Select the material being received
3. Enter quantity and storage location
4. Click **Receive** ? stock increases, blockchain entry created

---

## Staging to Production

Move materials from the warehouse to the production staging area before issuing them:

1. Go to **Stock & Movements ? Staging to Production**
2. Select material, source location, and destination location
3. Enter quantity and click **Transfer** ? internal stock transfer recorded on chain

---

## Quality Management

### Process Inspection

Record in-process quality results during production:

1. Go to **Process Inspection**
2. Select the Production Order
3. Enter inspection results:
   - **Characteristic** Ñ what was measured (e.g., dimension, weight)
   - **Result** Ñ measured value
   - **Status** Ñ Pass / Fail
   - **Inspector** Ñ name of the inspector
4. Click **Submit** ? QM result recorded on blockchain

### QM Result

Final quality decision for the order Ñ accepts or rejects the batch.

---

## Blockchain Ledger

Every transaction generates an immutable block:

- **Block number** Ñ sequential index
- **Timestamp** Ñ ISO 8601 UTC timestamp
- **Event type** Ñ e.g., `GOODS_ISSUE`, `OP_CONFIRMED`, `GOODS_RECEIPT`
- **Payload** Ñ order number, material, quantity, actor
- **Hash** Ñ SHA-256 of `previousHash + JSON payload`
- **Previous Hash** Ñ links to the prior block, forming the chain
- **Valid** Ñ green ? if chain is intact; red ? if tampered

The **Chain Status** indicator at the top of the Blockchain Ledger screen shows overall chain validity in real time.

---

## Reports

| Report | Description |
|--------|-------------|
| **WIP Summary** | Current stock by material and location |
| **Movement History** | All goods movements, filterable by date / material / order |
| **Order Status** | All production orders with quantities and status |
| **BOM Report** | Bill of materials explosion for a selected material |
| **Stock Report** | Full inventory snapshot |

---

## Material Master

### Create a Material

1. Go to **Material ? Material Master**
2. Click **New Material**
3. Enter:
   - **Material ID** Ñ unique code (e.g., `FG-001`)
   - **Description** Ñ human-readable name
   - **Unit of Measure** Ñ EA, KG, L, M, etc.
   - **Material Type** Ñ FERT (finished good), ROH (raw material), HALB (semi-finished)
4. Click **Save**

### Bill of Materials

1. Go to **Bill of Material**
2. Select or create a BOM for a material
3. Add components with quantities and units
4. Save Ñ the BOM will be used for automatic component explosion on Goods Issue

---

## Work Centers & Routings

### Work Centers

Define the machines, cells, or assembly stations:

1. Go to **Work Centers**
2. Click **New Work Center**
3. Enter ID, description, and optionally capacity and cost rate
4. Save

### Routings

Define the sequence of operations for a finished product:

1. Go to **Routing**
2. Click **New Routing** and select the material
3. Add operations in sequence:
   - **Operation number** (e.g., 10, 20, 30)
   - **Work Center**
   - **Description** (e.g., Cutting, Welding, Assembly)
   - **Standard time** (minutes)
4. Save Ñ this routing will be available when creating Production Orders

---

## ERP Bridge *(Admin only)*

The ERP Bridge exposes production orders and goods movements in a structured JSON format for integration with SAP, Oracle, or any external system.

Endpoints:
- `GET /api/erp/order/{id}` Ñ full order data in ERP-ready format
- `GET /api/erp/movement/{id}` Ñ movement data in ERP-ready format

These endpoints use the same HTTP Basic Auth as the rest of the API. Admin role required.

---

## User Management *(Admin only)*

### Create a User

Send a POST request to `/api/admin/users`:

```bash
curl -X POST https://wip.rodrigoandremarques.com/api/admin/users \
  -u 'admin:YOUR_PASSWORD' \
  -H 'Content-Type: application/json' \
  -d '{"username": "operator1", "password": "SecurePass123", "role": "operator"}'
```

Valid roles: `admin`, `operator`, `visitor`

### Change a Password

```bash
curl -X PUT https://wip.rodrigoandremarques.com/api/admin/users/operator1/password \
  -u 'admin:YOUR_PASSWORD' \
  -H 'Content-Type: application/json' \
  -d '{"newPassword": "NewSecurePass456"}'
```

### Delete a User

```bash
curl -X DELETE https://wip.rodrigoandremarques.com/api/admin/users/operator1 \
  -u 'admin:YOUR_PASSWORD'
```

> The primary admin account (set via `ADMIN_USER` env var) cannot be deleted.

---

## Tips & Troubleshooting

| Issue | Solution |
|-------|---------|
| Login page shows error on first load | Wait 30Ð50 seconds for the free-tier server to wake up, then try again |
| Logout button not visible | It only appears after a successful login, in the sidebar footer |
| Chain shows as invalid | Do not manually edit `data/*.json` files Ñ all changes must go through the API |
| Goods Issue fails | Check that materials are in stock and the order is in Released status |
| Confirmation fails | Ensure the order has been released and materials issued |
