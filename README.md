# WIP Manufacturing Blockchain

> **Live Demo:** [wip.rodrigoandremarques.com](https://wip.rodrigoandremarques.com)

A full-stack shop floor control system that records every manufacturing movement as an immutable block in a SHA-256 blockchain. Goods receiving, material issue, operation confirmations, quality results, and finished goods receipts are all persisted as tamper-evident ledger entries with a cryptographic chain linking every event.

Built by a SAP S/4HANA specialist, the system mirrors standard PP (Production Planning) and WM (Warehouse Management) workflows from ERP practice, making it suitable as a lightweight MES (Manufacturing Execution System) or an ERP integration companion.

---

## Live Demo

| Role     | Username         | Password        | Access                                    |
|----------|------------------|-----------------|-------------------------------------------|
| Visitor  | demo             | WIP-Demo-2026!  | Full dashboard, read-only                 |
| Operator | (request access) | --              | All shop floor movements                  |
| Admin    | (private)        | --              | Full system, user management, ERP export  |

[Open Live Demo](https://wip.rodrigoandremarques.com)

> The server runs on Render's free tier and sleeps after 15 minutes of inactivity. The first request after a period of inactivity may take 30-50 seconds to respond.

---

## System Modules

**1. Production Order Lifecycle**
Full lifecycle management from creation to closure. Each status transition (Created -> Released -> In Progress -> Completed -> Closed) is written to the blockchain with timestamp and actor.

**2. WIP Board**
Real-time shop floor view grouping active orders by Work Center. Operators see queue, active, and pending-confirmation states at each machine or assembly station.

**3. Material Flow**
Complete goods movement coverage:
- Goods Receiving -- inbound raw materials from suppliers
- Staging to Production -- warehouse to production area transfers
- Goods Issue -- material consumption against a production order
- Backflush -- automatic consumption on operation confirmation
- Goods Receipt -- finished goods into stock
- Transfer -- internal stock movements between locations
- Scrap -- material write-offs with reason codes

**4. Operation Confirmations**
Per-operation confirmations with yield quantity, scrap, operator name, work center, and timestamp. Supports multi-operation routings (e.g., Cutting -> Welding -> Assembly -> Painting).

**5. Quality Management**
Process inspection records and final QM decisions logged against production orders. Permanent quality audit trail on the blockchain.

**6. Blockchain Ledger**
Every event: previousHash + payload -> SHA-256 -> thisHash. Chain integrity is validated in real time. Any modification to historical data breaks the chain and is immediately detected. Optional Ethereum/Polygon anchoring is available for external verifiability.

**7. Material Master and Bill of Materials**
Material catalogue with BOM maintenance. BOMs drive automatic component reservation and backflush quantity calculation.

**8. Work Centers and Routing**
Work center definitions with capacity and cost rates. Routings define the operation sequence, work center assignments, and standard times for each finished product.

**9. Reports**
WIP inventory summary, movement history, order status, BOM explosion, and stock snapshot reports.

**10. ERP Bridge**
Admin-only REST endpoints that expose production orders and movements in a structured JSON format for SAP, Oracle, or any external ERP system.

---

## Technology Stack

| Layer          | Technology                      | Notes                               |
|----------------|---------------------------------|-------------------------------------|
| Backend        | Node.js + Express.js            | REST API, approx. 1,500 lines       |
| Blockchain     | Custom SHA-256 chain            | In-process, no external dependency  |
| Public chain   | Ethereum / Polygon (optional)   | Anchors local chain hash on-chain   |
| Authentication | HTTP Basic Auth                 | SHA-256 hashed passwords            |
| Authorisation  | Role-based (admin/operator/visitor) | Per-endpoint enforcement        |
| Persistence    | JSON flat files                 | Portable, zero-dependency storage   |
| Frontend       | Vanilla JS + HTML/CSS           | Single-page application             |
| Hosting        | Render.com                      | Free tier, auto-deploy from GitHub  |
| Domain         | wip.rodrigoandremarques.com     | HTTPS via Let's Encrypt             |

---

## Repository Structure

This is a documentation-only repository. Source code is maintained in a private repository.

    /
    +-- README.md                <- This file
    +-- docs/
        +-- USER-GUIDE.md        <- Step-by-step guide for operators and visitors
        +-- ARCHITECTURE.md      <- System design, blockchain model, data flow
        +-- API-REFERENCE.md     <- Full REST API reference
        +-- CONFIGURATION.md     <- Deployment and configuration guide

---

## Documentation

| Document                                  | Description                                        |
|-------------------------------------------|----------------------------------------------------|
| [User Guide](docs/USER-GUIDE.md)          | Step-by-step guide for all user roles              |
| [Architecture](docs/ARCHITECTURE.md)      | System design, blockchain model, data model        |
| [API Reference](docs/API-REFERENCE.md)    | Complete REST API with request/response examples   |
| [Configuration](docs/CONFIGURATION.md)    | Environment variables, deployment, setup           |

---

## About

Built by **Rodrigo Andre Marques** -- SAP S/4HANA PP/WM specialist and manufacturing systems architect.

- Website: [rodrigoandremarques.com](https://rodrigoandremarques.com)
- Live system: [wip.rodrigoandremarques.com](https://wip.rodrigoandremarques.com)

*Source code is in a private repository. This repository contains documentation and a public live demo.*
