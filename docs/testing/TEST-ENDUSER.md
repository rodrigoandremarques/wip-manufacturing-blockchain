# Test End-User Guide

## Overview

This guide walks through the SAP production order workflow with correct status codes, location codes, blockchain event types, and work center configurations.

## Part 1: Visitor Walkthrough

### Viewing Production Order Status

1. Open the Production Orders list
2. Status displays show standardized codes:
   - CRTD = Created
   - REL = Released
   - CNF = Confirmed / In Process
   - DLV = Delivered
   - TECO = Technically Complete
   - CLSD = Closed

3. Each status represents a distinct stage in the order lifecycle
4. Status transitions are logged as blockchain events for full traceability

### Blockchain Events

When viewing order details, you can see the complete audit trail of events:
- WO_CREATED: When the work order is created (status: CRTD)
- WO_RELEASED: When the work order is released (status: REL)
- GOODS_ISSUE_261: Material issued from warehouse
- OP_CONFIRMED: Operation confirmation recorded (status: CNF)
- GOODS_RECEIPT_101: Finished goods receipt posted (status: DLV)
- INSP_LOT_CREATED: Inspection lot created automatically with goods receipt
- INSP_GR_RESULT: Quality result recorded (PASS/FAIL/QI)
- ORDER_TECO: Technically complete (status: TECO)
- ORDER_CLOSED: Order closed (status: CLSD)

### Storage Locations

Materials move through standardized storage locations (sloc codes):
- 1000 = Raw Material warehouse
- 2000 = Semi-Finished
- 3000 = Finished Goods warehouse
- 4000 = Scrap / Reject

### Viewing Material Flow

Open any order to see:
1. Raw materials pulled from sloc 1000
2. Processing through work centers
3. Finished goods received into sloc 3000
4. Final quantities confirmed in finished goods

---

## Part 2: Operator Walkthrough

### Pre-Configured Work Centers

The following work centers are pre-configured and ready to use:
- WC-SMT-01: Surface Mount Technology line
- WC-AOI-01: Automated Optical Inspection
- WC-PTH-01: Through-Hole Technology
- WC-WAVE: Wave Soldering
- WC-TEST: Electrical Testing
- WC-VISUAL-INSP: Visual Inspection Station
- WC-COAT: Coating/Conformal Coating
- WC-ASSY: Assembly Station
- WC-PACK: Packaging Station

For this walkthrough, we will use WC-ASSY (Assembly) as the primary work center.

### Complete Production Order E2E Flow

#### Step O-02: Initial Goods Issue

1. Navigate to Goods Issues
2. Create a goods issue with material type "COMP-001"
3. Source location: 1000 (Raw Material)
4. Quantity: 5 units
5. Submit
6. Blockchain event logged: GOODS_ISSUE_261

#### Step O-03: Create Work Order

1. Go to Create Production Order
2. Select material "PROD-001"
3. Quantity: 5
4. Planned start: Today's date
5. Select work center: WC-ASSY
6. Submit
7. Status displays: CRTD (Created)
8. Blockchain event logged: WO_CREATED

#### Step O-04: Release Work Order

1. Open the created work order
2. Click Release button
3. Confirm release action
4. Status updates to: REL (Released)
5. Blockchain event logged: WO_RELEASED

#### Step O-05: Goods Issue for Production

1. Open the released work order
2. Navigate to Goods Issues section
3. Create goods issue for operation 261
4. Source location: 1000 (Raw Material)
5. Component: "COMP-001"
6. Quantity: 5 units
7. Submit goods issue
8. Blockchain event logged: GOODS_ISSUE_261

#### Step O-06: Confirm Operation

1. Stay in the work order
2. Navigate to Operations section
3. Select operation line for WC-ASSY
4. Enter actual time: 30 minutes
5. Mark operation as confirmed
6. Status automatically updates to: CNF (Confirmed / In Process)
7. Blockchain event logged: OP_CONFIRMED

#### Step O-07: Goods Receipt

1. Navigate to Goods Receipts
2. Create new goods receipt
3. Reference the work order
4. Goods receipt type: 101 (Production completion)
5. Finish location: 3000 (Finished Goods)
6. Quantity: 5 units
7. Submit goods receipt
8. Status automatically updates to: DLV (Delivered)
9. Blockchain events logged:
   - GOODS_RECEIPT_101
   - WO_DELIVERED
   - INSP_LOT_CREATED (automatically created)

#### Step O-08: Quality Management Result

1. Open the created inspection lot
2. Select QM result section
3. Enter result: PASS (unrestricted stock flows to sloc 3000)
4. Alternative options:
   - FAIL (blocks stock, moves to sloc 4000)
   - QI (blocks for quality inspection in sloc 3000)
5. Submit result
6. Blockchain event logged: INSP_GR_RESULT

#### Step O-09: Set Technically Complete (TECO)

1. Open the work order (now at DLV status)
2. Click Set TECO button
3. Confirm action
4. Status updates to: TECO (Technically Complete)
5. Blockchain event logged: ORDER_TECO
6. Note: TECO can only be set after DLV status is reached

#### Step O-10: Close Work Order

1. Open the work order (now at TECO status)
2. Click Close Order button
3. Confirm closure action
4. Status updates to: CLSD (Closed)
5. Blockchain event logged: ORDER_CLOSED
6. Order is now complete and archived

---

## Additional Operations

### Material Transfer Between Locations

Create transfer using sloc codes:
- Transfer from 2000 (Semi-Finished) to 3000 (Finished Goods)
- Blockchain event: TRANSFER_311

### Scrap Posting

If material is rejected during production:
1. Create scrap posting
2. Move from operational location to 4000 (Scrap/Reject)
3. Quantity and reason required
4. Blockchain event: SCRAP_POSTING_551

### Backflush Operations

For automated material consumption:
1. Backflush is triggered automatically during operation confirmation
2. Material consumed from sloc 1000
3. Blockchain event: BACKFLUSH_543

### Reversals

For correcting previous transactions:
- Goods issue reversal: REVERSAL_262
- Goods receipt reversal: REVERSAL_102

---

## Summary

The operator workflow follows a logical progression:
1. O-02: Issue raw materials from warehouse
2. O-03: Create the production order (CRTD)
3. O-04: Release for production (REL)
4. O-05: Issue materials to production
5. O-06: Confirm operation (CNF)
6. O-07: Receive finished goods (DLV)
7. O-08: Confirm quality (PASS/FAIL/QI)
8. O-09: Set technically complete (TECO)
9. O-10: Close the order (CLSD)

Each step is tracked on the blockchain with specific event types for complete auditability and traceability.