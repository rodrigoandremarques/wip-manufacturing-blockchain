# WIP Blockchain Architecture

## System Overview

WIP is a blockchain-integrated manufacturing execution and inventory management system that combines a SAP-style work order system with a permissioned blockchain for immutable event logging. The system provides real-time shop floor visibility, quality management integration, and ERP export capabilities.

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Web Browser Client                        │
│                    (Dashboard, Work Order UI)                    │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTPS
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                Express.js REST API (Port 3000)                   │
│                                                                  │
│  /api/work-order, /api/release, /api/goods-issue,              │
│  /api/confirmation, /api/goods-receipt, /api/qm-result,        │
│  /api/teco, /api/close, /api/erp/*, /api/material-master       │
└──────────┬──────────────────┬──────────────────┬─────────────────┘
           │                  │                  │
           ▼                  ▼                  ▼
    ┌─────────────┐   ┌──────────────┐   ┌─────────────────┐
    │blockchain.js│   │material-     │   │persistence.js   │
    │             │   │master.js     │   │                 │
    │SHA-256      │   │              │   │JSON flat file   │
    │chain        │   │BOM, routing, │   │./data/          │
    │(in-process) │   │work centers  │   │                 │
    └─────────────┘   └──────────────┘   └─────────────────┘
           │
           ▼
    ┌─────────────────────────────┐
    │blockchain-ethereum.js       │
    │                             │
    │Ethereum/Polygon Anchoring   │
    │(Optional, async, non-block) │
    └─────────────────────────────┘
```

## Work Order Status Lifecycle

```
CRTD              REL              CNF              DLV            TECO          CLSD
(Created)  ->  (Released)  ->  (Confirmed/  ->  (Delivered/  ->  (Technical)  -> (Closed)
              POST /release    In Process)      GR Posted)      Complete
              WO_RELEASED      OP_CONFIRMED    GOODS_RECEIPT    ORDER_TECO   ORDER_CLOSED
                               status=CNF      _101
                                              WO_DELIVERED
                                              status=DLV
```

Work orders flow through six distinct statuses, triggered by specific API operations and corresponding blockchain events. Reversals (REVERSAL_262, REVERSAL_102, REVERSAL_552) can occur at any stage, rolling back the chain without deletion.

## Blockchain Event Types

The blockchain records all manufacturing activities as immutable events. All events include timestamp, actor, previousHash, and computed hash.

| Event Type | Code | Trigger | Description |
|---|---|---|---|
| GENESIS | - | System init | Chain initialization with genesis block |
| WO_CREATED | - | POST /api/work-order | Work order creation (status: CRTD) |
| WO_RELEASED | - | POST /api/work-order/:id/release | Release to shop floor (status: REL) |
| GOODS_ISSUE_261 | 261 | POST /api/goods-issue | Raw material issue from warehouse (mvt 261, slocFrom: 1000) |
| BACKFLUSH_543 | 543 | System auto | Component backflush on GR (mvt 543) |
| OP_CONFIRMED | - | POST /api/confirmation | Operation confirmation, CO11N (status: CNF) |
| GOODS_RECEIPT_101 | 101 | POST /api/goods-receipt | Finished goods receipt (mvt 101, slocTo: 3000) |
| WO_DELIVERED | - | Auto on GR 101 | Work order marked delivered (status: DLV) |
| INSP_LOT_CREATED | - | Auto on GR 101 | Quality management inspection lot auto-created |
| SCRAP_POSTING_551 | 551 | POST /api/scrap | Scrap/reject posting (mvt 551, slocTo: 4000) |
| INSP_GR_RESULT | - | POST /api/qm-result | QM inspection result (usageDecision: PASS/FAIL/QI) |
| ORDER_TECO | - | POST /api/work-order/:id/teco | Technically complete (status: TECO) |
| ORDER_CLOSED | - | POST /api/work-order/:id/close | Final closure (status: CLSD) |
| TRANSFER_311 | 311 | POST /api/stock-transfer | Stock transfer between storage locations (mvt 311) |
| REVERSAL_262 | 262 | POST /api/reversal/goods-issue | Reversal of GI 261 (mvt 262) |
| REVERSAL_102 | 102 | POST /api/reversal/goods-receipt | Reversal of GR 101 (mvt 102) |
| REVERSAL_552 | 552 | POST /api/reversal/backflush | Reversal of backflush 543 (mvt 552) |
| QUALITY_PASS | - | POST /api/qm-result | Quality pass result event |
| QUALITY_FAIL | - | POST /api/qm-result | Quality fail result event |
| WIP_* | - | Shop floor | WIP board events (dynamic) |

## Storage Locations (SLOC Codes)

Storage locations represent physical warehouse areas. Goods issue (261) draws from source; goods receipt (101) populates destination.

| SLOC Code | Description | Purpose |
|---|---|---|
| 1000 | Raw Material Warehouse | Input materials, issued via GI 261 |
| 2000 | Semi-Finished Goods | In-process inventory, used for staging |
| 3000 | Finished Goods Warehouse | Completed products post GR 101 (unrestricted after QM PASS) |
| 4000 | Scrap / Reject | Defective or excess material (mvt 551) |

## Block Structure (Example)

Each block is a cryptographically linked JSON record with SHA-256 hashing.

```json
{
  "index": 5,
  "timestamp": "2026-04-06T14:08:21.700Z",
  "type": "GOODS_ISSUE_261",
  "docNumber": "261a0001",
  "woNumber": "WO-0001",
  "material": "MAT-001",
  "quantity": 10,
  "slocFrom": "1000",
  "actor": "operator01",
  "previousHash": "abc123def456...",
  "hash": "def456abc789...",
  "data": {
    "batch": "BATCH-2026-001",
    "cost": 500.00
  }
}
```

Hash computation: `SHA-256(previousHash + JSON.stringify(data) + timestamp)`

## Typical Data Flow (Complete Work Order Lifecycle)

The following sequence represents a standard manufacturing cycle using actual API endpoints:

```
Step 1: Create Work Order
  POST /api/work-order
  { "woNumber": "WO-0001", "material": "MAT-001", "quantity": 10, ... }
  Event: WO_CREATED
  Status: CRTD

Step 2: Release to Shop Floor
  POST /api/work-order/WO-0001/release
  Event: WO_RELEASED
  Status: REL

Step 3: Issue Raw Materials
  POST /api/goods-issue
  { "woNumber": "WO-0001", "material": "MAT-001", "quantity": 10,
    "slocFrom": "1000" }
  Event: GOODS_ISSUE_261 (mvt 261)
  Storage: Remove from 1000 (Raw Material)

Step 4: Confirm Operations
  POST /api/confirmation
  { "woNumber": "WO-0001", "operationNo": "0010", ... }
  Event: OP_CONFIRMED
  Status: CNF (In Process)

Step 5: Post Goods Receipt
  POST /api/goods-receipt
  { "woNumber": "WO-0001", "material": "MAT-001", "quantity": 10,
    "slocTo": "3000" }
  Events: GOODS_RECEIPT_101 (mvt 101), WO_DELIVERED, INSP_LOT_CREATED
  Status: DLV (Delivered)
  Storage: Add to 3000 (Finished Goods, restricted)

Step 6: Quality Inspection Result
  POST /api/qm-result
  { "inspectionLot": "IL-0001", "usageDecision": "PASS" }
  Events: INSP_GR_RESULT, QUALITY_PASS
  Stock unrestricted in 3000

Step 7: Technical Complete
  POST /api/work-order/WO-0001/teco
  Event: ORDER_TECO
  Status: TECO

Step 8: Close Work Order
  POST /api/work-order/WO-0001/close
  Event: ORDER_CLOSED
  Status: CLSD
```

## Storage Locations and Material Flows

Material movement between storage locations is tracked via inventory movements (SAP-style):

- **Goods Issue (GI 261)**: Raw material withdrawal from SLOC 1000 to production
- **Backflush (543)**: Component consumption, auto-triggered on GR
- **Goods Receipt (GR 101)**: Finished product receipt to SLOC 3000 (initially quality-restricted)
- **Scrap Posting (551)**: Defective units to SLOC 4000
- **Stock Transfer (311)**: Inter-warehouse movement between any SLOCs
- **Reversals (262, 102, 552)**: Roll back prior movements without deletion, maintaining full audit trail

## ERP Integration (IDOC Export)

WIP exports work orders and materials movements as SAP IDOC (Intermediate Document) files for ERP systems.

### IDOC Types

| IDOC Type | Module | Content | Endpoint |
|---|---|---|---|
| LOIPRO01 | PP (Production Planning) | Work order header, operations, components | GET /api/erp/order/:woNumber |
| MBGMCR02 | MM (Materials Management) | Stock movement records, SLOC transitions | GET /api/erp/movement/:docNumber |

### Example Export Workflow

```
GET /api/erp/order/WO-0001
  Returns LOIPRO01 IDOC:
    - Work order details (number, material, quantity, due date)
    - Component list (BOM)
    - Operations (work center, duration, confirmation status)
    - Status history (CRTD -> REL -> CNF -> DLV -> TECO -> CLSD)

GET /api/erp/movement/261a0001
  Returns MBGMCR02 IDOC:
    - Movement type (261 GI, 101 GR, 551 scrap, 311 transfer)
    - Material, quantity, SLOC from/to
    - Posting date, batch, cost
    - Block reference (immutable audit trail)
```

ERP systems can consume these IDOCs via standard middleware (SAP PI/PO) to synchronize work orders, inventory, and cost accounting.

## Material Master and Routings

The material-master.js module maintains:

- **Material Master Records**: Material code, description, unit of measure, cost, status (active/obsolete)
- **Bill of Materials (BOM)**: Exploded component lists per material and work order
- **Work Centers**: Production facilities, labor rates, capacity
- **Routings**: Standard operations per material, sequencing, duration, work center assignment

These define the manufacturing structure and drive operation confirmation, backflush logic, and capacity planning.

## Authentication and Authorization

WIP uses HTTP Basic Auth with SHA-256 password hashing for all API access.

### Auth Configuration

- **Password Hash**: SHA-256(password + "wip-blockchain-2026-salt")
- **Trigger**: When ADMIN_PASSWORD environment variable is set, AUTH_ENABLED = true
- **Delivery**: Base64-encoded Authorization header

### Role Hierarchy

| Role | Permissions |
|---|---|
| admin | All API operations, user management, system config, export |
| operator | Create/release work orders, post goods movements, confirm operations, view inventory |
| visitor | Read-only access (view work orders, blockchain, inventory) |

Role-based access control restricts dangerous operations (close order, reverse movement, export) to appropriate personnel.

## Persistence

WIP persists all data to a single JSON flat-file store for simplicity and auditability.

### Data File Location

- **Path**: `./data/blockchain-state.json`
- **Contents**:
  - Full blockchain (all blocks in sequence)
  - Work order registry (all WO records with status)
  - Inventory state (SLOC balances, batch tracking)
  - User registry (SHA-256 hashed credentials)
  - Material master (materials, BOMs, routings)

### Backup and Recovery

The flat-file design allows easy backup (copy blockchain-state.json) and recovery (restore file). No database migration or schema management required. Full audit trail preserved in blockchain.

## Deployment

WIP is deployed as a containerized Node.js application on Render.com with automatic CI/CD.

### Deployment Pipeline

- **Repository**: Private GitHub repository (source control)
- **CI/CD**: GitHub Actions (test, build)
- **Hosting**: Render.com (auto-deploy on push)
- **Domain**: wip.rodrigoandremarques.com
- **DNS**: GoDaddy CNAME record pointing to Render service URL
- **SSL/TLS**: Let's Encrypt certificate via Render custom domain integration

### Environment Configuration

Critical settings via environment variables:

- `ADMIN_PASSWORD`: Master user credential; AUTH_ENABLED when set
- `ETH_RPC_URL`: Ethereum/Polygon node RPC for optional blockchain anchoring
- `ETH_PRIVATE_KEY`: Signer key for Ethereum contract interactions
- `ETH_CONTRACT_ADDRESS`: Smart contract address for anchoring events
- `PORT`: API server port (default 3000)

## Concurrency and Performance

### In-Process Blockchain

The blockchain is maintained in memory and serialized to disk on each event. This design ensures:

- Fast SHA-256 hashing (no external service)
- Immediate consistency (single-threaded Node.js event loop)
- Simple recovery (reload from blockchain-state.json)
- No distributed consensus overhead

### Ethereum Anchoring (Optional, Async)

Blockchain anchoring to Ethereum or Polygon is optional and non-blocking:

- Events are first recorded in the local chain
- Async batch writes to smart contract (every N events or on timer)
- Anchor hash = SHA-256(blockIndex + chainHash)
- If Ethereum unavailable, system continues normally; anchoring retried later

### Scalability Considerations

For high-volume manufacturing:

- Consider event batching (multiple movements per block)
- Partition blockchain by time period or work order range
- Implement inventory sharding across SLOCs
- Add caching layer (Redis) for frequently queried work orders/inventory

## API Design Principles

- RESTful endpoints following HTTP semantics
- Idempotent POST/PUT for safety (duplicate submission returns same result)
- Complete request validation and error responses (400, 403, 409, 500)
- Comprehensive audit logging (all operations recorded to blockchain)
- Rate limiting per authenticated user or IP (future enhancement)

## Security Considerations

- SHA-256 hashing for passwords and blockchain; no plaintext secrets stored
- HTTP Basic Auth over HTTPS; credentials validated on every request
- Role-based access control for sensitive operations
- Immutable blockchain prevents tampering with historical records
- Reversal operations (262, 102, 552) maintain audit trail; no deletion
- Environment variables for sensitive keys; never hardcoded credentials

## Summary

WIP provides a transparent, auditable manufacturing system combining SAP-style work order management, real-time inventory tracking, and permissioned blockchain immutability. The architecture scales from small production environments to complex multi-site operations while maintaining full traceability for regulatory compliance and process improvement.
