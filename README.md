# WIP Manufacturing Blockchain

> **Live Demo ?** [wip.rodrigoandremarques.com](https://wip.rodrigoandremarques.com)

A full-stack **shop floor control system** that records every manufacturing movement Ñ goods receiving, material issue, operation confirmation, quality inspection, and goods receipt Ñ as an immutable block in a SHA-256 blockchain. Every transaction on the shop floor becomes a tamper-evident, auditable ledger entry.

Built by a SAP S/4HANA specialist, it mirrors real PP (Production Planning) and WM (Warehouse Management) workflows, making it suitable as a lightweight ERP companion or standalone MES (Manufacturing Execution System).

---

## Live Demo

| Role | Username | Password | Access |
|------|----------|----------|--------|
| ? Visitor | `demo` | `WIP-Demo-2026!` | Full dashboard Ñ read only |
| ? Operator | *(request access)* | Ñ | All shop floor movements |
| ? Admin | *(private)* | Ñ | Full system + user management + ERP export |

? **[Open Live Demo](https://wip.rodrigoandremarques.com)**

> The system runs on Render's free tier Ñ first request after inactivity may take ~30 seconds to wake up.

---

## What the System Does

### 1. Production Order Lifecycle
Create a Production Order for a finished product, release it to the shop floor, track it through operations, and close it on completion. Each status change (Created ? Released ? In Progress ? Completed ? Closed) is written to the blockchain.

### 2. WIP Board Ñ Real-Time Shop Floor View
A live kanban-style board showing all active Work-in-Progress orders grouped by Work Center. Operators see exactly what is queued, active, and pending confirmation at each machine or assembly station.

### 3. Material Flow (Goods Movements)
The system tracks the full material journey:
- **Goods Receiving** Ñ raw materials arrive from suppliers and are booked into stock
- **Staging to Production** Ñ materials move from warehouse to production staging area
- **Goods Issue** Ñ materials are consumed (issued) against a Production Order
- **Backflush** Ñ automatic material consumption on operation confirmation
- **Goods Receipt** Ñ finished goods are received into stock on order completion
- **Transfer** Ñ internal stock transfers between locations or storage bins
- **Scrap** Ñ material write-offs with reason codes

### 4. Operation Confirmations
Operators confirm each routing operation (e.g., Cutting ? Welding ? Assembly ? Painting) with actual quantities, yield, and scrap. Each confirmation records the operator name, work center, and timestamps.

### 5. Quality Management
Process inspections are logged against Production Orders. QM results (pass/fail, measured values, inspector) are recorded as blockchain entries, providing a permanent quality record.

### 6. Blockchain Audit Trail
Every event generates a block: `previousHash + payload ? SHA-256 ? thisHash`. The chain is validated in real time. Any tamper attempt breaks the chain and is immediately flagged. Optional Ethereum/Polygon anchoring pins the local chain hash to a public blockchain for external verifiability.

### 7. Material Master & Bill of Materials
Maintain a material catalogue and multi-level Bills of Material (BOM). The BOM drives automatic component reservation and backflush calculations.

### 8. Work Centers & Routing
Define Work Centers (machines, assembly lines, cells) with capacity and cost rates. Routings define the sequence of operations for each finished product, including standard times and work center assignments.

### 9. Reports
- WIP Inventory Summary (stock by material and location)
- Movement History (filtered by date, material, order)
- Order Status Report (all orders with quantities and status)
- BOM Explosion Report

### 10. ERP Bridge
Admin-only REST endpoints expose production orders and movements in a structured JSON format ready for consumption by SAP, Oracle, or any ERP system via API integration.

---

## Technology Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| Backend | Node.js + Express.js | REST API, ~1,500 lines |
| Blockchain | Custom SHA-256 chain | In-process, no external dependency |
| Public Blockchain | Ethereum / Polygon (optional) | Anchors local chain hash on-chain |
| Authentication | HTTP Basic Auth | SHA-256 hashed passwords, static salt |
| Authorisation | Role-Based (admin / operator / visitor) | Per-endpoint enforcement |
| Persistence | JSON flat files | Portable, zero-dependency storage |
| Frontend | Vanilla JS + HTML/CSS | Single-page app, no framework |
| Hosting | Render.com | Free tier, auto-deploy from GitHub |
| Domain | wip.rodrigoandremarques.com | HTTPS via Let's Encrypt |

---

## Repository Structure

This is a **documentation-only** repository. Source code is maintained privately.

```
/
??? README.md                  ? This file
??? docs/
    ??? USER-GUIDE.md          ? How to use the system (operators & visitors)
    ??? ARCHITECTURE.md        ? Technical design & data model
    ??? API-REFERENCE.md       ? Full REST API documentation
    ??? CONFIGURATION.md       ? Deployment & configuration guide
```

---

## Documentation

| Document | Description |
|----------|-------------|
| [User Guide](docs/USER-GUIDE.md) | Step-by-step guide for all user roles |
| [Architecture](docs/ARCHITECTURE.md) | System design, blockchain model, data flow |
| [API Reference](docs/API-REFERENCE.md) | Complete REST API with request/response examples |
| [Configuration](docs/CONFIGURATION.md) | Environment variables, deployment, and setup |

---

## About

Built by **Rodrigo Andre Marques** Ñ SAP S/4HANA PP/WM specialist and manufacturing systems architect.

- ? [rodrigoandremarques.com](https://rodrigoandremarques.com)
- ? [Live System](https://wip.rodrigoandremarques.com)

*Source code in private repository. Documentation and live demo are public.*
