# Architecture

## Overview

WIP Manufacturing Blockchain is a monolithic Node.js application with three logical layers: a REST API (Express.js), a business logic layer (blockchain engine and manufacturing modules), and a persistence layer (JSON flat files).

    Browser (SPA)
    Vanilla JS / HTML / CSS
           |
           | HTTP Basic Auth over HTTPS
           |
    Express.js REST API  (server.js, ~1,500 lines)
           |
           +-- requireAuth() -> user-manager.js -> users.json
           |
           +-- /api/work-order/*      /api/goods-issue      /api/confirmation
           +-- /api/inventory/*       /api/goods-receipt    /api/quality
           +-- /api/materials/*       /api/bom/*            /api/routings/*
           +-- /api/work-centers/*    /api/reports/*        /api/erp/*
           +-- /api/chain             /api/audit-trail      /api/admin/users
           |
    +------+------------------+----------------------------+
    |                         |                            |
blockchain.js          material-master.js          persistence.js
SHA-256 chain          Materials / BOM             loadState() / saveState()
Block structure        Work Centers                data/orders.json
addTransaction()       Routings                    data/inventory.json
validate()                                         data/movements.json
    |                                              data/blocks.json
    | (optional)                                   data/users.json
    |                                              data/materials.json
blockchain-ethereum.js                             data/work-centers.json
Ethereum / Polygon                                 data/routings.json
Sepolia testnet
Smart contract anchor

---

## Blockchain Design

### Block Structure

Each block contains the following fields:

    {
      index:        42,
      timestamp:    '2026-04-06T10:15:00.000Z',
      type:         'GOODS_ISSUE',
      data: {
        orderId:    'WO-001',
        material:   'ROH-100',
        quantity:   50,
        unit:       'KG',
        location:   'STAGE-01',
        actor:      'operator1'
      },
      previousHash: 'a3f8...',
      hash:         '7b2c...'
    }

### Hash Computation

    hash = SHA-256( previousHash + JSON.stringify(data) + timestamp )

The genesis block uses previousHash = '0'. Each subsequent block's hash includes its predecessor's hash, forming a cryptographic chain. Modifying any block invalidates every block that follows it.

### Chain Validation

blockchain.isValid() iterates every consecutive block pair and verifies:

1. block.hash equals the recomputed hash of the block's own content.
2. block.previousHash equals the hash of the preceding block.

This check runs on every /api/chain/status request and its result is shown live in the Blockchain Ledger screen.

### Event Types

| Event Type            | Trigger                                    |
|-----------------------|--------------------------------------------|
| WORK_ORDER_CREATED    | New production order                       |
| WORK_ORDER_RELEASED   | Order released to shop floor               |
| WORK_ORDER_TECO       | Technical completion                       |
| WORK_ORDER_CLOSED     | Order closed                               |
| GOODS_RECEIVING       | Raw material received from supplier        |
| STAGING_TRANSFER      | Material moved to staging area             |
| GOODS_ISSUE           | Material issued to production order        |
| BACKFLUSH             | Automatic consumption on confirmation      |
| OP_CONFIRMED          | Operation confirmation with yield/scrap    |
| GOODS_RECEIPT         | Finished goods received into stock         |
| SCRAP                 | Material write-off                         |
| TRANSFER              | Internal stock transfer                    |
| QM_RESULT             | Quality management decision                |
| REVERSAL              | Cancellation of a prior movement           |

---

## Authentication and Authorisation

### HTTP Basic Auth Flow

    Client sends: Authorization: Basic base64(username:password)
    Server:       decodes -> looks up user -> hashes password with salt -> compares -> sets req.user

Passwords are stored as SHA-256(password + 'wip-blockchain-2026-salt'). No plaintext passwords are persisted anywhere.

### Role Hierarchy

    admin
      All operator permissions
      User management (create, update, delete)
      ERP export endpoints
      Raw blockchain dump (/api/chain)

    operator
      All visitor permissions
      All write operations (GI, GR, confirmation, transfer, scrap, QM)

    visitor
      All GET endpoints (read-only)
      No write operations of any kind

### Middleware

- requireAuth     -- any authenticated user
- requireAdmin    -- admin role only
- requireOperator -- operator or admin

---

## Data Model

### Production Order

    {
      id:          'WO-20260406-001',
      material:    'FG-001',
      description: 'Finished Product A',
      quantity:    100,
      unit:        'EA',
      workCenter:  'ASSEMBLY-01',
      routing:     'RT-FG-001',
      status:      'Released',
      createdAt:   '2026-04-06T08:00:00Z',
      releasedAt:  '2026-04-06T08:30:00Z',
      operations: [
        {
          id:           'OP-10',
          description:  'Cutting',
          workCenter:   'CUT-01',
          standardTime: 30,
          status:       'Confirmed',
          confirmedQty: 100,
          scrapQty:     2
        }
      ]
    }

### Inventory / Stock

    {
      materialId:   'ROH-100',
      description:  'Steel Sheet 2mm',
      unit:         'KG',
      locations: {
        'WH-01':    500,
        'STAGE-01': 50
      },
      totalStock:   550
    }

### Bill of Materials

    {
      materialId:  'FG-001',
      components: [
        { material: 'ROH-100', quantity: 5, unit: 'KG' },
        { material: 'ROH-200', quantity: 2, unit: 'EA' }
      ]
    }

### Routing

    {
      id:         'RT-FG-001',
      materialId: 'FG-001',
      operations: [
        { opNo: 10, description: 'Cutting',  workCenter: 'CUT-01',      stdTime: 30 },
        { opNo: 20, description: 'Welding',  workCenter: 'WELD-01',     stdTime: 45 },
        { opNo: 30, description: 'Assembly', workCenter: 'ASSEMBLY-01', stdTime: 60 },
        { opNo: 40, description: 'Painting', workCenter: 'PAINT-01',    stdTime: 20 }
      ]
    }

---

## Persistence

All state is stored as JSON files under DATA_DIR (default: ./data/):

| File                | Contents                                  |
|---------------------|-------------------------------------------|
| blocks.json         | Full blockchain (array of blocks)         |
| orders.json         | Production orders                         |
| inventory.json      | Stock levels by material and location     |
| movements.json      | All goods movement history                |
| users.json          | User registry (hashed passwords)          |
| materials.json      | Material master                           |
| work-centers.json   | Work center definitions                   |
| routings.json       | Routing definitions                       |

State is loaded into memory on startup and saved to disk on every write operation. On Render's free tier the filesystem is ephemeral -- data resets on each redeploy. For persistent data, mount a Render Disk (paid tier) or use an external database.

---

## Optional Ethereum Integration

When ETHEREUM_ENABLED=true, each local block addition also submits a transaction to a deployed ManufacturingWIP.sol smart contract on Sepolia testnet or Polygon. The contract stores the local chain's current root hash, providing external verifiability.

    Local SHA-256 chain -> root hash -> Ethereum transaction (Sepolia/Polygon)
                                        Public, immutable, timestamped on-chain

---

## Deployment

    GitHub (private repo)
        |
        | push to main -> auto-deploy trigger
        |
    Render.com (free tier)
        Build:   npm install --omit=dev
        Start:   node server.js
        Port:    $PORT (injected by Render)
        Domain:  wip.rodrigoandremarques.com
        SSL:     Let's Encrypt (Render-managed)
        Storage: Ephemeral (resets on each deploy)

    GoDaddy DNS
        CNAME wip -> wip-blockchain.onrender.com
