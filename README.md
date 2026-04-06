# WIP Manufacturing Blockchain

> **Live Demo:** [wip.rodrigoandremarques.com](https://wip.rodrigoandremarques.com)

A full-stack shop floor control system that records every manufacturing movement as an immutable block in a SHA-256 blockchain. Goods receiving, material issue, operation confirmations, quality results, and finished goods receipts are all persisted as tamper-evident ledger entries with a cryptographic chain linking every event.

Designed, developed, and maintained by a SAP S/4HANA PP/WM specialist, the system mirrors standard Production Planning and Warehouse Management workflows from ERP practice, making it suitable as a lightweight MES (Manufacturing Execution System) or an ERP integration companion.

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

## The Problem

Modern manufacturing faces critical challenges that traditional ERP and MES systems cannot adequately solve:

- **Regulatory Compliance Risk**: Auditors require immutable, tamper-proof evidence of every manufacturing step. Current systems rely on centralized databases that can be altered retroactively, creating legal liability.
- **Supply Chain Traceability**: Product recalls demand instant, verifiable traceability from raw material through finished goods. Disconnected systems and manual processes cause delays and incomplete records.
- **Quality Assurance Gaps**: Multiple operators and systems make it difficult to prove that no unauthorized changes occurred during production or testing.
- **Multi-Party Verification**: When products move between suppliers, manufacturers, and distributors, there is no neutral way to verify that the data has not been tampered with.
- **Operational Visibility**: Real-time inventory and WIP tracking across distributed facilities is either absent or unreliable.

---

## Why Blockchain is the Right Solution

Blockchain solves these problems through four key properties that traditional systems cannot match:

1. **Immutability**: Once a transaction is recorded, it cannot be altered without invalidating the entire chain. Auditors can verify this mathematically.
2. **Distributed Trust**: No single entity controls the ledger. Multiple parties can independently verify the same data without needing to trust a centralized authority.
3. **Cryptographic Audit Trail**: Every transaction is timestamped, hashed, and linked to previous transactions. The chain of hashes makes any modification detectable.
4. **Transparent Verification**: Any participant can validate the entire production history by replaying the blockchain -- no special access required.

This system is purpose-built for regulated manufacturing industries where compliance, traceability, and data integrity are non-negotiable.

---

## Industries That Need This

### Pharmaceutical and Life Sciences

- **Regulations**: GMP (Good Manufacturing Practice), 21 CFR Part 11 (FDA), EU GMP Annex 11
- **Use Cases**: Track raw materials, batch serialization, manufacturing steps, and shelf life. Prove that no unauthorized batches were produced.
- **Impact**: Reduce drug recall time from weeks to minutes. Prevent counterfeit medications from entering distribution channels.

### Aerospace and Defense

- **Regulations**: AS9100, NADCAP, ITAR
- **Use Cases**: Track component lot numbers, supplier certifications, critical assembly steps, and NDT results.
- **Impact**: Ensure every part is traceable to its supplier. Prove compliance with export control regulations in real time.

### Automotive

- **Regulations**: IATF 16949, PPAP
- **Use Cases**: WIP tracking across assembly lines, part traceability for recalls, supplier quality metrics, first-article inspection records.
- **Impact**: Accelerate PPAP approval. Reduce recall response time from days to hours.

### Electronics Manufacturing Services (EMS)

- **Regulations**: IPC-A-610, RoHS, WEEE
- **Use Cases**: PCB lot tracking, component authentication, soldering and inspection records, conflict minerals compliance.
- **Impact**: Prevent counterfeit components. Prove environmental compliance.

### Medical Devices

- **Regulations**: ISO 13485, FDA 21 CFR Part 820, UDI
- **Use Cases**: Device serialization, sterilization verification, shelf-life management, adverse event traceability.
- **Impact**: Enable real-time device recalls using UDI. Reduce liability from device failures.

### Food and Beverage

- **Regulations**: FSMA, HACCP
- **Use Cases**: Ingredient lot tracking, production date verification, allergen control, recall response.
- **Impact**: Respond to contamination within hours instead of days.

---

## Why Traditional ERP/MES Falls Short

| Requirement              | Traditional ERP/MES                          | This Blockchain System                              |
|--------------------------|----------------------------------------------|-----------------------------------------------------|
| **Immutability**         | Centralized -- can be edited or deleted       | Cryptographically immutable -- any edit detected    |
| **Audit Trail**          | Log-based, easy to manipulate                | Hash-chain, mathematically impossible to fake       |
| **Verification**         | Trust the system vendor                      | Verify independently using open standards           |
| **Single Point of Fail** | Database goes down = no visibility           | Distributed, continues operating offline            |
| **Regulatory Proof**     | Company claims integrity                     | Blockchain proves integrity to regulators           |
| **Cross-Party Trust**    | Requires VPN, complex agreements             | Trustless -- third parties verify directly          |
| **Recall Speed**         | Manual database queries (hours)              | Instant, complete traceability (seconds)            |

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

## Key Features

**Production Order Management**
Create orders with product, quantity, priority, cost center, and due date. Multi-operation workflow per order with real-time status tracking. SAP integration hooks included (IDOC export ready).

**WIP Tracking**
Automatic WIP item creation when an operation starts. Location-based tracking across work centers, inspection areas, and finished goods storage. Move tracking with reason codes and operator attribution. Full scrap and rework visibility.

**Inventory Management**
Material receipt with supplier and purchase order traceability. Multi-location inventory with transfer history. Physical count adjustments with operator accountability. Issue-to-order for costing and BOM validation.

**Immutable Blockchain**
SHA-256 hash chain with chain integrity verification on every read. Any modification to historical data breaks the chain and is immediately detected. Complete audit trail with timestamps, operators, and event payloads.

**SAP Integration Ready**
IDOC export for orders and inventory movements. Cost center and plant-level data structures. Purchase order linking. Extensible JSON format for direct ERP mapping.

**Audit Trail**
Over 50 transaction types (receive, transfer, move, confirm, close, scrap, and more). Filter by type, operator, or time range. Export to CSV or JSON. Every record is cryptographically bound to the chain.

**Role-Based Access**
Three roles enforced at the API level: admin (full access), operator (all shop floor movements), visitor (read-only). HTTP Basic Auth with SHA-256 hashed passwords. User management via REST API.

---

## Data Flow Example

The following sequence illustrates a complete production order cycle from creation to finished goods receipt:

1. Operator creates a production order -> POST /api/orders -> event queued and mined into a new block
2. System computes SHA-256(previousHash + payload + timestamp) -> block appended to chain
3. Order status transitions to Released -> WORK_ORDER_RELEASED event written to blockchain
4. Goods Issue executed -> raw materials deducted from stock -> GOODS_ISSUE event mined
5. Operation started at work center -> OPERATION_STARTED event with operator and machine recorded
6. Operation confirmed -> yield quantity and scrap entered -> OPERATION_CONFIRMED event mined
7. Quality inspection recorded -> pass/fail decision written as QM_INSPECTION event
8. Goods Receipt posted -> finished goods added to stock -> GOODS_RECEIPT event mined
9. Order closed -> WORK_ORDER_CLOSED event finalizes the chain for this order

Every step is cryptographically linked to the previous one. Any auditor or third party can replay the full sequence and verify that no step was altered, skipped, or inserted after the fact.

---

## Technology Stack

| Layer          | Technology                          | Notes                               |
|----------------|-------------------------------------|-------------------------------------|
| Backend        | Node.js + Express.js                | REST API, approx. 1,500 lines       |
| Blockchain     | Custom SHA-256 chain                | In-process, no external dependency  |
| Public chain   | Ethereum / Polygon (optional)       | Anchors local chain hash on-chain   |
| Authentication | HTTP Basic Auth                     | SHA-256 hashed passwords            |
| Authorisation  | Role-based (admin/operator/visitor) | Per-endpoint enforcement            |
| Persistence    | JSON flat files                     | Portable, zero-dependency storage   |
| Frontend       | Vanilla JS + HTML/CSS               | Single-page application             |
| Hosting        | Render.com                          | Free tier, auto-deploy from GitHub  |
| Domain         | wip.rodrigoandremarques.com         | HTTPS via Let's Encrypt             |

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

**WIP Manufacturing Blockchain** was idealized, designed, developed, and is maintained by **Rodrigo Andre Marques** -- SAP S/4HANA PP/WM specialist and manufacturing systems architect with over a decade of experience implementing production planning and shop floor control in industrial environments.

The system was conceived to demonstrate that a lean, self-contained blockchain stack can deliver the same audit and traceability guarantees as enterprise compliance platforms, at a fraction of the cost and complexity.

- Website: [rodrigoandremarques.com](https://rodrigoandremarques.com)
- Live system: [wip.rodrigoandremarques.com](https://wip.rodrigoandremarques.com)

*Source code is maintained in a private repository. This repository contains documentation and provides access to the public live demo.*

---

## License

**Proprietary** -- Copyright (c) 2026 Rodrigo Andre Marques

All rights reserved. Unauthorized copying, modification, distribution, or use of this software or its documentation, in whole or in part, is prohibited without the express written permission of the copyright holder.

Licensing inquiries: contact via [rodrigoandremarques.com](https://rodrigoandremarques.com)

---

*WIP Manufacturing Blockchain -- Immutable. Auditable. Compliant.*
