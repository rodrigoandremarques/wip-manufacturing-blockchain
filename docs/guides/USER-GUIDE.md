# SAP Production Order User Guide

## Introduction

This guide provides a comprehensive overview of the SAP production order system, including status codes, storage locations, work centers, blockchain event tracking, and quality management.

## Work Order Status Codes

Production orders move through distinct lifecycle stages, each with a standardized SAP status code:

| Code | Status | Description | Set By |
|------|--------|-------------|--------|
| CRTD | Created | Order created, not yet released | System |
| REL | Released | Order released to production | User action |
| CNF | Confirmed / In Process | First operation confirmed | System (automatic) |
| DLV | Delivered | Goods receipt posted | System (automatic) |
| TECO | Technically Complete | Manual completion marker | User action |
| CLSD | Closed | Order finalized | User action |

Status transitions are automated where indicated and logged as blockchain events for auditability.

## Storage Locations (SLOC Codes)

Materials are stored in standardized warehouse locations. Each location serves a specific purpose in the production workflow:

| SLOC Code | Location Name | Purpose | Usage |
|-----------|---------------|---------|-------|
| 1000 | Raw Material warehouse | Initial material storage | Source for goods issues |
| 2000 | Semi-Finished | Intermediate production stages | Transfer between operations |
| 3000 | Finished Goods warehouse | Final product storage | Goods receipt destination |
| 4000 | Scrap / Reject | Defective material | Failed or scrapped items |

## Pre-Configured Work Centers

The system includes nine pre-configured work centers, ready for immediate use:

1. WC-SMT-01 - Surface Mount Technology line
2. WC-AOI-01 - Automated Optical Inspection
3. WC-PTH-01 - Through-Hole Technology
4. WC-WAVE - Wave Soldering
5. WC-TEST - Electrical Testing
6. WC-VISUAL-INSP - Visual Inspection Station
7. WC-COAT - Coating / Conformal Coating
8. WC-ASSY - Assembly Station
9. WC-PACK - Packaging Station

Each work center is configured with standard cycle times and capacity planning data.

## Production Order Lifecycle

### Complete E2E Flow with Status Progression

#### Phase 1: Material Preparation

**Step 1: Initial Goods Issue**
- Navigate to Goods Issues module
- Select material from catalog
- Source location: 1000 (Raw Material warehouse)
- Specify quantity required
- Confirm and submit
- Event: GOODS_ISSUE_261 logged to blockchain

#### Phase 2: Work Order Creation and Release

**Step 2: Create Work Order**
- Access Create Production Order function
- Select finished material to produce
- Set order quantity
- Assign work center (e.g., WC-ASSY)
- Submit to create
- Status: CRTD (Created)
- Event: WO_CREATED logged to blockchain

**Step 3: Release Work Order**
- Open the created order
- Click Release button
- Confirm release decision
- Status changes to: REL (Released)
- Event: WO_RELEASED logged to blockchain
- Order now visible to shop floor

#### Phase 3: Production Execution

**Step 4: Production Goods Issue**
- Open the released work order
- Navigate to Goods Issues for this order
- Create operation-specific goods issue (261)
- Source: sloc 1000 (Raw Material)
- Specify components and quantities
- Submit goods issue
- Event: GOODS_ISSUE_261 logged to blockchain

**Step 5: Operation Confirmation**
- Go to Operations section within work order
- Select operation (e.g., at WC-ASSY)
- Enter actual processing time
- Confirm operation completion
- Status automatically changes to: CNF (Confirmed / In Process)
- Backflush consumption occurs automatically
- Event: OP_CONFIRMED logged to blockchain

#### Phase 4: Goods Receipt and QM

**Step 6: Goods Receipt**
- Navigate to Goods Receipt (GR) module
- Reference the production order
- Goods receipt type: 101 (Production completion)
- Destination location: 3000 (Finished Goods warehouse)
- Enter received quantity
- Submit goods receipt
- Status automatically changes to: DLV (Delivered)
- Inspection lot created automatically
- Events logged:
  - GOODS_RECEIPT_101
  - WO_DELIVERED
  - INSP_LOT_CREATED

**Step 7: Quality Management Result**
- Open the inspection lot (created automatically with GR)
- Navigate to QM results section
- Select usage decision:
  - PASS = Material unrestricted, flows to sloc 3000
  - FAIL = Material blocked, moves to sloc 4000 (Scrap)
  - QI = Quality inspection stock in sloc 3000
- Submit quality result
- Event: INSP_GR_RESULT logged to blockchain

#### Phase 5: Order Completion

**Step 8: Set Technically Complete (TECO)**
- Open work order (now at DLV status)
- Click Set TECO button
- Confirm action
- Status changes to: TECO (Technically Complete)
- Note: TECO can only be set after DLV is reached
- Event: ORDER_TECO logged to blockchain

**Step 9: Close Work Order**
- Open work order (now at TECO status)
- Click Close Order button
- Confirm closure
- Status changes to: CLSD (Closed)
- Order archived and locked from further changes
- Event: ORDER_CLOSED logged to blockchain

---

## Quality Management

### QM Result Options and Effects

Quality management results determine material disposition:

**PASS Result**
- Material is unrestricted-use stock
- Moves to sloc 3000 (Finished Goods)
- Available for sales orders immediately
- Full quantity accepted

**FAIL Result**
- Material is blocked from use
- Moves to sloc 4000 (Scrap / Reject)
- Cannot be sold without special approval
- Requires scrap posting for disposal

**QI Result (Quality Inspection)**
- Material held in sloc 3000
- Blocked for 48-hour inspection window
- Inspectors can sample and test further
- Released to sales after inspection approval

### Inspection Lot

An inspection lot is created automatically when goods receipt is posted:
- Links production order to quality results
- Tracks multiple samples and test results
- Maintains full audit trail of QM decisions
- Required for compliance reporting

---

## Material Movements

### Goods Issues (GI)

Goods issues remove material from inventory:
- Type 261: Production issue (work order consumption)
- Source: Typically sloc 1000 (Raw Material)
- Records cost to order
- Blockchain event: GOODS_ISSUE_261

### Goods Receipts (GR)

Goods receipts add material to inventory:
- Type 101: Production receipt (finished goods)
- Destination: Typically sloc 3000 (Finished Goods)
- Updates inventory quantities
- Creates inspection lot automatically
- Blockchain events: GOODS_RECEIPT_101, WO_DELIVERED, INSP_LOT_CREATED

### Transfers Between Locations

Material can be transferred between storage locations:
- From sloc 2000 to sloc 3000 (Semi-Finished to Finished Goods)
- From sloc 3000 to sloc 4000 (Finished Goods to Scrap)
- Maintains full movement history
- Blockchain event: TRANSFER_311

### Backflush Operations

Backflush automatically consumes material during operation confirmation:
- Triggered by OP_CONFIRMED event
- Pulls components from sloc 1000
- No manual goods issue required
- Reduces data entry errors
- Blockchain event: BACKFLUSH_543

### Scrap Postings

Scrap postings record material loss:
- Records rejected or defective parts
- Moves material to sloc 4000 (Scrap / Reject)
- Requires reason code (defect type, etc.)
- Impacts cost and yield reporting
- Blockchain event: SCRAP_POSTING_551

---

## Reversals and Corrections

### Goods Issue Reversal

Reverses a previous goods issue transaction:
- Puts material back in sloc 1000
- Removes cost from order
- Used for shipping errors, over-issues
- Creates offsetting transaction record
- Blockchain event: REVERSAL_262

### Goods Receipt Reversal

Reverses a goods receipt transaction:
- Returns material to production queue
- Removes inventory from sloc 3000
- Cancels associated inspection lot
- Used for receiving errors, returns
- Blockchain event: REVERSAL_102

---

## Blockchain Event Types

All activities are logged as immutable blockchain events for complete auditability:

| Event Type | Trigger | Description |
|------------|---------|-------------|
| WO_CREATED | Create order | Work order initialized (CRTD status) |
| WO_RELEASED | Release action | Order released to production (REL status) |
| GOODS_ISSUE_261 | GI submission | Material issued from warehouse |
| OP_CONFIRMED | Op confirmation | Operation completed (CNF status) |
| GOODS_RECEIPT_101 | GR submission | Finished goods received (DLV status) |
| WO_DELIVERED | GR posted | Automatic when 101 receipt posted |
| INSP_LOT_CREATED | GR posted | Inspection lot auto-created with GR |
| INSP_GR_RESULT | QM result | Quality decision recorded (PASS/FAIL/QI) |
| ORDER_TECO | TECO action | Technically complete marker set |
| ORDER_CLOSED | Close action | Order finalized (CLSD status) |
| TRANSFER_311 | Transfer submission | Material moved between locations |
| SCRAP_POSTING_551 | Scrap posting | Defective material recorded |
| BACKFLUSH_543 | Op confirmation | Automatic material consumption |
| REVERSAL_262 | GI reversal | Goods issue reversed |
| REVERSAL_102 | GR reversal | Goods receipt reversed |

---

## Key Workflow Rules

1. **Status Progression**: CRTD → REL → CNF → DLV → TECO → CLSD
2. **Automatic Status Changes**:
   - CNF: Set automatically on first operation confirmation
   - DLV: Set automatically when goods receipt 101 is posted
3. **Manual Status Changes**:
   - REL: User must click Release
   - TECO: User must click Set TECO (only available after DLV)
   - CLSD: User must click Close Order (only available after TECO)
4. **Location Flow**: 1000 (RM) → Production → 3000 (FG)
5. **QM Decision**: Required before closing; must be PASS, FAIL, or QI
6. **TECO Requirement**: Cannot close order until TECO is set
7. **Blockchain Immutability**: All events are permanent and non-editable

---

## Troubleshooting

### Cannot Release Order
- Verify all required fields are filled
- Check work center capacity for selected date
- Ensure bill of materials is complete

### Cannot Confirm Operation
- Verify operation is assigned to correct work center
- Check work center availability for time period
- Confirm material is available in sloc 1000

### Cannot Post Goods Receipt
- Verify order is in REL status
- Check quantity available matches order quantity
- Ensure destination location (3000) exists in system

### Cannot Set TECO
- Verify order is in DLV status
- Confirm QM result (PASS/FAIL/QI) has been set
- Check for any open batch records or issues

### Cannot Close Order
- Verify order is in TECO status
- Confirm all operations are confirmed
- Check for any outstanding material movements

---

## Summary

The production order system provides a complete, traceable workflow from order creation through completion:

1. Create order in CRTD status
2. Release to production (REL)
3. Execute operations and confirm (CNF)
4. Receive finished goods (DLV)
5. Confirm quality result (PASS/FAIL/QI)
6. Set technically complete (TECO)
7. Close the order (CLSD)

All actions are logged as blockchain events for complete auditability and compliance. The system is configured with pre-built work centers, standard storage locations, and automated material consumption for efficiency.