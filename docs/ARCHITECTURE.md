# Architecture

## Overview

WIP Manufacturing Blockchain is a monolithic Node.js application with three logical layers: a REST API layer (Express.js), a business logic layer (blockchain + manufacturing modules), and a persistence layer (JSON flat files).

```
???????????????????????????????????????????????????????????????
?                        Browser (SPA)                        ?
?              Vanilla JS / HTML / CSS (index.html)           ?
???????????????????????????????????????????????????????????????
                        ?  HTTP Basic Auth  (HTTPS)
???????????????????????????????????????????????????????????????
?                   Express.js REST API                        ?
?                      server.js (~1,500 lines)                ?
?                                                              ?
?  requireAuth()  ??? user-manager.js  ??? users.json         ?
?  requireAdmin() ??? role check                               ?
?                                                              ?
?  /api/work-order/*   /api/goods-issue    /api/confirmation   ?
?  /api/inventory/*    /api/goods-receipt  /api/quality        ?
?  /api/materials/*    /api/bom/*          /api/routings/*     ?
?  /api/work-centers/* /api/reports/*      /api/erp/*          ?
?  /api/chain          /api/audit-trail    /api/admin/users    ?
????????????????????????????????????????????????????????????????
        ?               ?                  ?
???????????????? ???????????????  ????????????????????????????
? blockchain.js ? ?material-    ?  ? persistence.js            ?
?               ? ?master.js    ?  ?                           ?
? SHA-256 chain ? ?             ?  ? loadState() / saveState() ?
? Block struct  ? ? Materials   ?  ? data/orders.json          ?
? addTx()       ? ? BOM         ?  ? data/inventory.json       ?
? validate()    ? ? Work Centers?  ? data/movements.json       ?
?               ? ? Routings    ?  ? data/blocks.json          ?
????????????????? ???????????????  ? data/users.json          ?
        ?                          ? data/materials.json       ?
        ? optional                 ? data/work-centers.json    ?
????????????????????????????       ? data/routings.json        ?
? blockchain-ethereum.js   ?       ?????????????????????????????
?                          ?
? Ethereum / Polygon       ?
? Sepolia testnet          ?
? Smart contract anchor    ?
? (ETHEREUM_ENABLED=true)  ?
????????????????????????????
```

---

## Blockchain Design

### Block Structure

Each block contains:

```json
{
  "index": 42,
  "timestamp": "2026-04-06T10:15:00.000Z",
  "type": "GOODS_ISSUE",
  "data": {
    "orderId": "WO-001",
    "material": "ROH-100",
    "quantity": 50,
    "unit": "KG",
    "location": "STAGE-01",
    "actor": "operator1"
  },
  "previousHash": "a3f8...",
  "hash": "7b2c..."
}
```

### Hash Computation

```
hash = SHA-256( previousHash + JSON.stringify(data) + timestamp )
```

The genesis block uses `previousHash = "0"`. Every subsequent block's hash includes its predecessor's hash, creating a cryptographic chain. Altering any block invalidates all blocks after it.

### Chain Validation

`blockchain.isValid()` iterates every block pair and verifies:

1. `block.hash === recomputedHash(block)` Ñ block not tampered
2. `block.previousHash === prev.hash` Ñ chain not broken

This runs on every `/api/chain/status` request and is displayed live in the Blockchain Ledger screen.

### Event Types

| Event Type | Trigger |
|-----------|---------|
| `WORK_ORDER_CREATED` | New production order |
| `WORK_ORDER_RELEASED` | Order released to shop floor |
| `WORK_ORDER_TECO` | Technical completion |
| `WORK_ORDER_CLOSED` | Order closed |
| `GOODS_RECEIVING` | Raw material received from supplier |
| `STAGING_TRANSFER` | Material moved to staging area |
| `GOODS_ISSUE` | Material issued to production order |
| `BACKFLUSH` | Automatic material consumption on confirmation |
| `OP_CONFIRMED` | Operation confirmation with yield/scrap |
| `GOODS_RECEIPT` | Finished goods received into stock |
| `SCRAP` | Material written off |
| `TRANSFER` | Internal stock transfer |
| `QM_RESULT` | Quality management result |
| `REVERSAL` | Cancellation of a prior movement |

---

## Authentication & Authorisation

### HTTP Basic Auth Flow

```
Client ? Authorization: Basic base64(username:password)
Server ? decode ? lookup user ? hash(password+SALT) ? compare ? set req.user
```

Passwords are stored as `SHA-256(password + "wip-blockchain-2026-salt")`. No plaintext passwords are ever persisted.

### Role Hierarchy

```
admin
  ?? All operator permissions
  ?? User management (CRUD)
  ?? ERP export endpoints
  ?? Raw blockchain dump (/api/chain)

operator
  ?? All visitor permissions
  ?? All write operations (GI, GR, confirmation, transfer, scrap, QM)

visitor
  ?? All GET endpoints (read-only)
  ?? No write operations
```

### Middleware

```javascript
requireAuth   Ñ any valid user
requireAdmin  Ñ admin role only
requireOperator Ñ operator or admin
```

---

## Data Model

### Production Order

```json
{
  "id": "WO-20260406-001",
  "material": "FG-001",
  "description": "Finished Product A",
  "quantity": 100,
  "unit": "EA",
  "workCenter": "ASSEMBLY-01",
  "routing": "RT-FG-001",
  "status": "Released",
  "createdAt": "2026-04-06T08:00:00Z",
  "releasedAt": "2026-04-06T08:30:00Z",
  "operations": [
    {
      "id": "OP-10",
      "description": "Cutting",
      "workCenter": "CUT-01",
      "standardTime": 30,
      "status": "Confirmed",
      "confirmedQty": 100,
      "scrapQty": 2
    }
  ]
}
```

### Inventory / Stock

```json
{
  "materialId": "ROH-100",
  "description": "Steel Sheet 2mm",
  "unit": "KG",
  "locations": {
    "WH-01": 500,
    "STAGE-01": 50
  },
  "totalStock": 550
}
```

### Material Master

```json
{
  "id": "FG-001",
  "description": "Finished Product A",
  "unit": "EA",
  "type": "FERT",
  "createdAt": "2026-01-01T00:00:00Z"
}
```

### Bill of Materials

```json
{
  "materialId": "FG-001",
  "components": [
    { "material": "ROH-100", "quantity": 5, "unit": "KG" },
    { "material": "ROH-200", "quantity": 2, "unit": "EA" }
  ]
}
```

### Routing

```json
{
  "id": "RT-FG-001",
  "materialId": "FG-001",
  "operations": [
    { "opNo": 10, "description": "Cutting",  "workCenter": "CUT-01",      "stdTime": 30 },
    { "opNo": 20, "description": "Welding",  "workCenter": "WELD-01",     "stdTime": 45 },
    { "opNo": 30, "description": "Assembly", "workCenter": "ASSEMBLY-01", "stdTime": 60 },
    { "opNo": 40, "description": "Painting", "workCenter": "PAINT-01",    "stdTime": 20 }
  ]
}
```

---

## Persistence

All state is stored as JSON files under `DATA_DIR` (default: `./data/`):

| File | Contents |
|------|---------|
| `blocks.json` | Full blockchain (array of blocks) |
| `orders.json` | Production orders |
| `inventory.json` | Stock levels by material and location |
| `movements.json` | All goods movement history |
| `users.json` | User registry (hashed passwords) |
| `materials.json` | Material master |
| `work-centers.json` | Work center definitions |
| `routings.json` | Routing definitions |

State is loaded into memory on startup and saved to disk on every write operation. On Render's free tier, the filesystem is ephemeral Ñ data resets on each redeploy. For persistent data, mount a Render Disk (paid) or use an external database.

---

## Optional Ethereum Integration

When `ETHEREUM_ENABLED=true`, after each local block is added, the chain's current root hash is anchored to an Ethereum smart contract (`ManufacturingWIP.sol`) deployed on Sepolia testnet or Polygon.

This provides **external verifiability**: an independent third party can verify that a given batch of transactions existed at a given time by checking the on-chain hash against the local chain.

```
Local SHA-256 chain  ?  root hash  ?  Ethereum tx (Sepolia/Polygon)
                                        ?? public, immutable, timestamped
```

---

## Deployment Architecture

```
GitHub (private repo)
    ?  push to main
    ?  auto-trigger
    ?
Render.com (free tier)
    ?? Build: npm install --omit=dev
    ?? Start: node server.js
    ?? Port: $PORT (Render injects)
    ?? Custom Domain: wip.rodrigoandremarques.com
    ?? SSL: Let's Encrypt (Render-managed)
    ?? Ephemeral filesystem (data resets on deploy)

GoDaddy DNS
    ?? CNAME wip ? wip-blockchain.onrender.com
```
