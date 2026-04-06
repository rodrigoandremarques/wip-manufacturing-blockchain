# WIP Manufacturing Blockchain

> **Live Demo →** [wip.rodrigoandremarques.com](https://wip.rodrigoandremarques.com)

A full-stack manufacturing control system that uses a SHA-256 blockchain to create an immutable audit trail of all WIP (Work-in-Progress) movements on the shop floor.

## Live Demo

| Role | Username | Password | Access |
|------|----------|----------|--------|
| 👁 Visitor | demo | WIP-Demo-2026! | Read-only — full dashboard visible |
| 👤 Operator | *(contact owner)* | — | Production movements |
| 🔑 Admin | *(private)* | — | Full system + user management |

🔗 **[Open Live Demo](https://wip.rodrigoandremarques.com)**

## System Modules

- **Production Orders** — full lifecycle (Created → Released → Completed)
- **WIP Board** — real-time shop floor view by work center
- **Stock & Movements** — Goods Receiving, Staging, Issue, Confirmation, Receipt
- **Quality** — Process Inspection
- **Blockchain Ledger** — SHA-256 chain, tamper-evident, real-time chain validation
- **Reports** — WIP inventory, movement history, order status
- **ERP Bridge** — REST API for SAP/Oracle integration

## Technology

| Layer | Technology |
|-------|-----------|
| Backend | Node.js + Express.js |
| Blockchain | Custom SHA-256 chain |
| Auth | HTTP Basic Auth + SHA-256 hashed passwords, multi-role |
| Hosting | Render.com |
| Domain | wip.rodrigoandremarques.com |

## API Overview

All endpoints require HTTP Basic Auth. Visitor accounts are GET-only.

```
GET  /api/blocks              — Full blockchain ledger
GET  /api/work-order          — List production orders
POST /api/work-order          — Create production order
GET  /api/inventory           — Current stock levels
POST /api/goods-receiving     — Register goods receipt
POST /api/goods-issue         — Issue material to production
POST /api/confirmation        — Confirm operation completion
GET  /api/reports/wip-summary — WIP summary report
```

## Architecture

```
Browser → HTTP Basic Auth → Express.js → blockchain.js (SHA-256)
                                       → user-manager.js (roles)
                                       → persistence.js (JSON store)
```

## About

Built by **Rodrigo Andre Marques** — SAP S/4HANA & Manufacturing Systems specialist.

- 🌐 [rodrigoandremarques.com](https://rodrigoandremarques.com)
- 🔗 [Live System](https://wip.rodrigoandremarques.com)

*This repository contains documentation only. Source code is in a private repository.*
