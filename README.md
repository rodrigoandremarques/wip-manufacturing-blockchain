# WIP Manufacturing Blockchain

> **Live Demo:** [wip.rodrigoandremarques.com](https://wip.rodrigoandremarques.com)

A full-stack shop floor control system that records every manufacturing movement as an
immutable block in a SHA-256 blockchain. Goods receiving, material issue, operation
confirmations, quality results, and finished goods receipts are all persisted as
tamper-evident ledger entries with a cryptographic chain linking every event.

Designed, developed, and maintained by a SAP S/4HANA PP/WM specialist, the system
mirrors standard Production Planning and Warehouse Management workflows from ERP
practice -- including SAP-style movement type codes (261, 101, 311, 551, 543) and
work order status flow (CRTD -> REL -> CNF -> DLV -> TECO -> CLSD). It is suitable
as a lightweight MES (Manufacturing Execution System) or an ERP integration companion.

---

## Live Demo

| Role     | Username         | Password        | Access                                    |
|----------|------------------|-----------------|-------------------------------------------|
| Visitor  | demo             | WIP-Demo-2026!  | Full dashboard, read-only                 |
| Operator | (request access) | --              | All shop floor movements                  |
| Admin    | (private)        | --              | Full system, user management, ERP export  |

[Open Live Demo](https://wip.rodrigoandremarques.com)

> The server runs on Render's free tier and sleeps after 15 minutes of inactivity.
> The first request after a period of inactivity may take 30-50 seconds to respond.

---

## The Problem

Modern manufacturing faces critical challenges that traditional ERP and MES systems
cannot adequately solve:

- **Regulatory Compliance Risk**: Auditors require immutable, tamper-proof evidence of
  every manufacturing step. Current systems rely on centralized databases that can be
  altered retroactively, creating legal liability.
- **Supply Chain Traceability**: Product recalls demand instant, verifiable traceability
  from raw material through finished goods. Disconnected systems and manual processes
  cause delays and incomplete records.
- **Quality Assurance Gaps**: Multiple operators and systems make it difficult to prove
  that no unauthorized changes occurred during production or testing.
- **Multi-Party Verification**: When products move between suppliers, manufacturers, and
  distributors, there is no neutral way to verify that the data has not been tampered with.
- **Operational Visibility**: Real-time inventory and WIP tracking across distributed
  facilities is either absent or unreliable.

---

## Why Blockchain is the Right Solution

Blockchain solves these problems through four key properties that traditional systems
cannot match:

1. **Immutability**: Once a transaction is recorded, it cannot be altered without
   invalidating the entire chain. Auditors can verify this mathematically.
2. **Distributed Trust**: No single entity controls the ledger. Multiple parties can
   independently verify the same data without needing to trust a centralized authority.
3. **Cryptographic Audit Trail**: Every transaction is timestamped, hashed, and linked
   to previous transactions. The chain of hashes makes any modification detectable.
4. **Transparent Verification**: Any participant can validate the entire production
   history by replaying the blockchain -- no special access required.

This system is purpose-built for regulated manufacturing industries where compliance,
traceability, and data integrity are non-negotiable.

---

## Industries That Need This

### Pharmaceutical and Life Sciences
- **Regulations**: GMP, 21 CFR Part 11 (FDA), EU GMP Annex 11
- **Use Cases**: Batch serialization, raw material tracking, shelf-life management.
- **Impact**: Reduce drug recall time from weeks to minutes.

### Aerospace and Defense
- **Regulations**: AS9100, NADCAP, ITAR
- **Use Cases**: Component lot traceability, supplier certifications, NDT results.
- **Impact**: Prove compliance with export control regulations in real time.

### Automotive
- **Regulations**: IATF 16949, PPAP
- **Use Cases**: WIP tracking, part traceability for recalls, first-article inspection.
- **Impact**: Accelerate PPAP approval. Reduce recall response time from days to hours.

### Electronics Manufacturing Services (EMS)
- **Regulations**: IPC-A-610, RoHS, WEEE
- **Use Cases**: PCB lot tracking, component authentication, soldering records.
- **Impact**: Prevent counterfeit components. Prove environmental compliance.

### Medical Devices
- **Regulations**: ISO 13485, FDA 21 CFR Part 820, UDI
- **Use Cases**: Device serialization, sterilization records, adverse event traceability.
- **Impact**: Enable real-time device recalls using UDI.

### Food and Beverage
- **Regulations**: FSMA, HACCP
- **Use Cases**: Ingredient lot tracking, allergen control, contamination response.
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
Full SAP-style lifecycle: CRTD -> REL -> CNF -> DLV -> TECO -> CLSD. Every status
transition is written to the blockchain with timestamp and actor.

**2. WIP Board**
Real-time shop floor view grouping active orders by Work Center. Operators see queue,
active, and pending-confirmation states at each work center.

**3. Material Movements**
Complete goods movement coverage using SAP movement type codes:
- Goods Issue (mvt 261) -- component consumption against a work order
- Backflush (mvt 543) -- automatic consumption on operation confirmation
- Goods Receipt (mvt 101) -- finished goods into stock
- Transfer (mvt 311) -- stock movement between storage locations
- Scrap (mvt 551) -- material write-offs with reason codes
- Reversal (mvt 262/102/552) -- correction of posted documents

**4. Operation Confirmations (CO11N style)**
Per-operation confirmations with yield quantity, scrap, work center, operator name,
actual labor time, and machine time. Supports multi-operation routings.

**5. Quality Management**
Inspection lot created automatically on goods receipt (mvt 101). QM result records
usage decision: PASS (unrestricted stock in sloc 3000), FAIL (blocked stock in 4000),
or QI (quality inspection stock). All results are permanent blockchain events.

**6. Blockchain Ledger**
Every event: previousHash + payload -> SHA-256 -> thisHash. Chain integrity is
validated in real time. Optional Ethereum/Polygon anchoring available for external
verifiability at key events (WO_CREATED, GOODS_RECEIPT_101).

**7. Material Master and Bill of Materials**
Material catalogue with BOM maintenance. BOMs drive automatic component reservation
and backflush quantity calculation.

**8. Work Centers and Routing**
Nine work centers pre-configured (SMT, AOI, PTH, Wave Soldering, ICT Test, Visual
Inspection, Conformal Coating, Assembly, Packaging). Routings define operation
sequences, work center assignments, and standard times.

**9. Reports and Stock View**
Stock by storage location (sloc 1000/2000/3000/4000), movement history, BOM explosion,
material report, and order status report.

**10. ERP Bridge**
Admin-only endpoints that export work orders as SAP IDOC type LOIPRO01 (PP module)
and material movements as MBGMCR02 (MM module) -- ready for direct ERP integration.

---

## Key Features

**SAP-Aligned Workflow**
Work order status codes (CRTD/REL/CNF/DLV/TECO/CLSD), movement type codes
(261/101/311/551/543), and IDOC export formats (LOIPRO01/MBGMCR02) match SAP PP/MM
conventions, enabling straightforward integration with existing ERP systems.

**Immutable Blockchain**
SHA-256 hash chain. Any modification to historical data breaks the chain and is
immediately detected via the /api/chain/status endpoint or the dashboard integrity
indicator.

**Quality Traceability**
Inspection lot auto-created on every goods receipt. QM result (PASS/FAIL/QI)
permanently recorded with inspector name, usage decision, and stock effect.

**Document Registry**
Every posted material movement generates a document number (e.g., 261a0001, 101a0002).
Documents can be reversed individually using their document number. Full document
history accessible via /api/documents.

**Ethereum Anchoring (optional)**
Key events (WO_CREATED, GOODS_RECEIPT_101) can be anchored to Ethereum/Polygon for
external, public verifiability. Anchoring is async and non-blocking -- the local chain
operates independently even when Ethereum is not configured.

**Role-Based Access**
Three roles enforced at the API level: admin (full access including ERP bridge and user
management), operator (all shop floor movements), visitor (read-only GET endpoints only).

---

## Data Flow Example

Complete production order cycle from creation to closure:

1. POST /api/work-order -- WO_CREATED event mined (status: CRTD)
2. POST /api/work-order/:id/release -- WO_RELEASED event (status: REL)
3. POST /api/goods-issue -- GOODS_ISSUE_261 event, component deducted from sloc 1000
4. POST /api/confirmation -- OP_CONFIRMED event with yield, scrap, work center (status: CNF)
5. POST /api/goods-receipt -- GOODS_RECEIPT_101 + WO_DELIVERED + INSP_LOT_CREATED events,
   finished goods added to sloc 3000 (status: DLV)
6. POST /api/qm-result -- INSP_GR_RESULT event, usageDecision PASS releases stock
7. POST /api/work-order/:id/teco -- ORDER_TECO event (status: TECO)
8. POST /api/work-order/:id/close -- ORDER_CLOSED event (status: CLSD)

Every step is cryptographically linked. Any auditor can replay the full sequence and
verify that no step was altered, skipped, or inserted after the fact.

---

## Technology Stack

| Layer          | Technology                          | Notes                               |
|----------------|-------------------------------------|-------------------------------------|
| Backend        | Node.js + Express.js                | REST API, approx. 1,500 lines       |
| Blockchain     | Custom SHA-256 chain                | In-process, no external dependency  |
| Public chain   | Ethereum / Polygon (optional)       | Anchors key events on-chain         |
| Authentication | HTTP Basic Auth                     | SHA-256 hashed passwords + salt     |
| Authorisation  | Role-based (admin/operator/visitor) | Per-endpoint enforcement            |
| Persistence    | JSON flat files (./data/)           | Portable, zero-dependency storage   |
| Frontend       | Vanilla JS + HTML/CSS               | Single-page application             |
| Hosting        | Render.com                          | Free tier, auto-deploy from GitHub  |
| Domain         | wip.rodrigoandremarques.com         | HTTPS via Let's Encrypt             |

---

## Repository Structure

This is a documentation-only repository. Source code is maintained in a private
repository.

    /
    +-- README.md
    +-- LICENSE
    +-- docs/
        +-- guides/
        |   +-- USER-GUIDE.md        <- Step-by-step guide for all roles
        |   +-- CONFIGURATION.md     <- Deployment and environment setup
        +-- technical/
        |   +-- ARCHITECTURE.md      <- System design, blockchain model, data flow
        |   +-- API-REFERENCE.md     <- Full REST API reference with curl examples
        +-- testing/
        |   +-- TEST-ENDUSER.md      <- Browser-based E2E test for Visitor and Operator
        |   +-- TEST-ADMIN.md        <- Full system E2E: infra, config, API, sign-off
        +-- logs/
            +-- CHANGELOG.md         <- Change log and change request history

---

## Documentation

| Document                                              | Description                                   |
|-------------------------------------------------------|-----------------------------------------------|
| [User Guide](docs/guides/USER-GUIDE.md)               | Step-by-step guide for all user roles         |
| [Configuration](docs/guides/CONFIGURATION.md)         | Environment variables, deployment, setup      |
| [Architecture](docs/technical/ARCHITECTURE.md)        | System design, blockchain model, data flow    |
| [API Reference](docs/technical/API-REFERENCE.md)      | Complete REST API with curl examples          |
| [End-User Tests](docs/testing/TEST-ENDUSER.md)        | Browser E2E test for Visitor and Operator     |
| [Admin Tests](docs/testing/TEST-ADMIN.md)             | Full system E2E: infra to sign-off            |
| [Changelog](docs/logs/CHANGELOG.md)                   | Change log and change request history         |

---

## About

**WIP Manufacturing Blockchain** was idealized, designed, developed, and is maintained
by **Rodrigo Andre Marques** -- SAP S/4HANA PP/WM specialist and manufacturing systems
architect with over a decade of experience implementing production planning and shop
floor control in industrial environments.

The system was conceived to demonstrate that a lean, self-contained blockchain stack can
deliver the same audit and traceability guarantees as enterprise compliance platforms,
at a fraction of the cost and complexity.

- Website: [rodrigoandremarques.com](https://rodrigoandremarques.com)
- Live system: [wip.rodrigoandremarques.com](https://wip.rodrigoandremarques.com)

*Source code is maintained in a private repository. This repository contains
documentation and provides access to the public live demo.*

---

## License

**Proprietary** -- Copyright (c) 2026 Rodrigo Andre Marques

All rights reserved. Unauthorized copying, modification, distribution, or use of this
software or its documentation, in whole or in part, is prohibited without the express
written permission of the copyright holder.

Licensing inquiries: contact via [rodrigoandremarques.com](https://rodrigoandremarques.com)

---

*WIP Manufacturing Blockchain -- Immutable. Auditable. Compliant.*
