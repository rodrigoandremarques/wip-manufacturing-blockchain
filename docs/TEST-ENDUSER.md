# End-User Test Script

**System:** WIP Manufacturing Blockchain
**URL:** https://wip.rodrigoandremarques.com
**Last updated:** 2026-04-06

This document is a step-by-step execution guide for end users testing the system in a browser.
No technical tools or API knowledge is required. Follow each step exactly as written, verify
the expected result, and mark Pass or Fail in the checklist at the end.

> **Cold start warning:** The server sleeps after 15 minutes of inactivity (Render free tier).
> If the page does not load within 10 seconds, wait 40 seconds and refresh. This is expected.
> Once loaded, the system responds normally for the duration of your session.

---

## Credentials

| Role     | Username | Password       |
|----------|----------|----------------|
| Visitor  | demo     | WIP-Demo-2026! |
| Operator | --       | Provided by Admin (see TEST-ADMIN.md, Section 3) |

---

# PART 1 -- VISITOR ROLE

The Visitor role has read-only access. No data can be created, modified, or deleted.
Use the credentials: **demo / WIP-Demo-2026!**

---

## V-01 -- Sign in

1. Open a browser and navigate to: https://wip.rodrigoandremarques.com
2. The browser displays an authentication dialog box with two fields: Username and Password.
3. Enter Username: `demo`
4. Enter Password: `WIP-Demo-2026!`
5. Click **OK** (or **Sign In**, depending on the browser).

**Expected:** The main dashboard loads. The page title in the browser tab reads
"WIP Manufacturing Blockchain" or similar. A navigation menu appears on the left side
of the screen with module names such as Dashboard, WIP Board, Production Orders,
Inventory, Blockchain, Reports, and others.

**Fail if:** The authentication dialog reappears (wrong credentials or server still starting up).
Wait 40 seconds and retry step 1.

---

## V-02 -- Dashboard

1. After sign-in, you are on the Dashboard by default.
2. Observe the summary panels. They may include: total active orders, total stock movements,
   blockchain block count, and recent events.
3. Verify that all panels load without error messages.

**Expected:** Summary data is visible. No panel shows "Error", "403", or "Unauthorized".

---

## V-03 -- WIP Board

1. In the left navigation menu, click **WIP Board**.
2. The board displays active production orders grouped by Work Center.
3. If orders exist, each card shows: Order Number, Product, Quantity, Status.
4. Click on any order card.
5. A detail panel or page opens showing the full order information.

**Expected:** WIP Board loads. Order cards are visible (or an empty-state message if no
active orders exist). Clicking an order opens its detail view with status, product,
quantity, and work center. No action buttons (Release, Close, Confirm) are present
for the Visitor role.

---

## V-04 -- Production Orders list

1. Click **Production Orders** in the left menu.
2. A table or list of all orders is displayed.
3. Click on any order row to open its detail view.
4. In the detail view, look at all the available buttons and options.

**Expected:** Orders list loads. Detail view shows full order data. Confirm that
no **Create**, **Release**, **Goods Issue**, **Close**, or **Delete** buttons are
active or clickable for the Visitor role.

---

## V-05 -- Blockchain Ledger

1. Click **Blockchain** (or **Ledger**) in the left menu.
2. A list of blocks is displayed, each with: Block Index, Timestamp, Event Type, Hash.
3. Click on any block to expand its details.
4. Verify the following fields are visible: `index`, `timestamp`, `type`, `hash`, `previousHash`.

**Expected:** The ledger loads and shows at minimum 1 block (genesis block at index 0).
Each block's `previousHash` matches the `hash` of the block above it in the list.
If a chain integrity indicator is shown, it reads **Valid** or **OK**.

---

## V-06 -- Inventory

1. Click **Inventory** in the left menu.
2. A list of materials and their stock quantities per location is displayed.
3. Observe column headers: Material Code, Description, Quantity, Unit, Location.
4. Verify no **Transfer**, **Scrap**, **Adjust**, or **Receive** buttons are accessible.

**Expected:** Inventory list loads with material data. All entries are read-only.
No movement actions are available to the Visitor role.

---

## V-07 -- Reports

1. Click **Reports** in the left menu.
2. Locate and run the **WIP Summary** report. Click its button or select it from a list.
3. Observe the report output -- it should show active orders and WIP quantities per work center.
4. Go back and run the **Movement History** report.
5. The report shows a log of all stock movements with date, material, quantity, and type.

**Expected:** Both reports generate successfully and display data (or an empty-state
message if no movements have occurred). No 403 or server error is shown.

---

## V-08 -- Material Master

1. Click **Material Master** (or **Materials**) in the left menu.
2. A list of materials is displayed with Material Code, Description, Unit of Measure, and Type.
3. Click on any material to open its detail view.
4. If a **Bill of Materials (BOM)** section is shown, note the components listed.

**Expected:** Material Master list loads. Detail view shows material data and BOM
structure (if configured). No create or edit controls are accessible.

---

## V-09 -- Work Centers and Routings

1. Click **Work Centers** in the left menu.
2. A list of work centers is displayed with Code, Description, and Capacity.
3. Click **Routings** in the left menu.
4. A list of routings is displayed. Each routing shows the product and its operation sequence.
5. Click any routing to view the operations: sequence number, operation name, work center,
   and standard time.

**Expected:** Both pages load correctly with read-only data. No edit controls available.

---

## V-10 -- Sign out

1. Locate the **Logout** button in the navigation menu or header area.
2. Click **Logout**.
3. After logout, attempt to navigate to https://wip.rodrigoandremarques.com/api/orders
   in the browser address bar.

**Expected:** After logout, the browser shows the authentication dialog again or
returns a 401 response. The session is fully cleared.

---

# PART 2 -- OPERATOR ROLE

The Operator role can execute all shop floor transactions: goods receiving, staging,
goods issue, operation confirmations, quality, goods receipt, transfers, and scrap.
Operators cannot access User Management or ERP Bridge.

Use credentials provided by the Admin (see TEST-ADMIN.md, Section 3).

The tests below execute a **complete production order lifecycle** from raw material
receipt through finished goods into stock.

---

## O-01 -- Sign in as Operator

1. Navigate to https://wip.rodrigoandremarques.com
2. In the authentication dialog, enter the Operator username and password provided by Admin.
3. Click **OK**.

**Expected:** Dashboard loads. The left menu shows shop floor modules. Confirm that
**User Management** is NOT present in the left menu.

---

## O-02 -- Goods Receiving (inbound raw material from supplier)

This step records the receipt of raw materials into the warehouse.

1. Click **Goods Receiving** in the left menu.
2. The Goods Receiving form opens. Fill in the following fields:

   | Field               | Value to enter          |
   |---------------------|-------------------------|
   | Material Code       | RM-0001                 |
   | Description         | (auto-populated or skip)|
   | Quantity            | 100                     |
   | Unit of Measure     | KG                      |
   | Supplier            | TEST-SUPPLIER-01        |
   | Purchase Order Ref  | PO-TEST-001             |
   | Storage Location    | WH-RAW                  |

3. Click **Post Receipt** (or **Save** / **Confirm**).

**Expected:**
- A success message appears: "Goods Receipt posted" or similar.
- In the Inventory screen (click **Inventory** in the menu), material RM-0001
  now shows 100 KG in location WH-RAW.
- In the Blockchain Ledger, a new block appears with event type `GOODS_RECEIPT`
  or `MATERIAL_RECEIVED`, containing the material code, quantity, supplier, and PO reference.

**Fail if:** Error message is shown, inventory does not update, or no blockchain event appears.

---

## O-03 -- Staging to Production (warehouse to production area)

This step moves raw material from the warehouse to the production area before starting work.

1. Click **Staging to Production** in the left menu.
2. Fill in:

   | Field            | Value          |
   |------------------|----------------|
   | Material Code    | RM-0001        |
   | Quantity         | 50             |
   | From Location    | WH-RAW         |
   | To Location      | PROD-AREA-01   |

3. Click **Transfer** (or **Post**).

**Expected:**
- Success confirmation message appears.
- In **Inventory**: RM-0001 at WH-RAW decreases from 100 to 50 KG.
  RM-0001 at PROD-AREA-01 increases to 50 KG.
- New blockchain event appears in the Ledger: type `STAGING_TO_PRODUCTION` or `TRANSFER`,
  showing from/to locations and quantity.

---

## O-04 -- Create Production Order

1. Click **Production Orders** in the left menu.
2. Click **New Order** or **Create** (button typically in the top right of the list).
3. Fill in the production order form:

   | Field        | Value             |
   |--------------|-------------------|
   | Product      | FG-0001           |
   | Description  | (auto-populated)  |
   | Quantity     | 10                |
   | Unit         | EA                |
   | Priority     | NORMAL            |
   | Due Date     | 2026-12-31        |
   | Cost Center  | CC-TEST-01        |
   | Created By   | (your username)   |

4. Click **Create** or **Save**.

**Expected:**
- Order is created with a system-assigned Order Number (e.g., ORD-0001 or similar).
- Order status shows **Created**.
- A blockchain event `WORK_ORDER_CREATED` appears in the Ledger with the order number,
  product, quantity, and operator name.

---

## O-05 -- Release Production Order

Releasing the order authorizes shop floor execution and makes it visible on the WIP Board.

1. Open the production order created in O-04 (click on it from the list).
2. In the order detail view, click **Release**.
3. A confirmation dialog may appear. Click **Confirm** or **Yes**.

**Expected:**
- Order status changes from **Created** to **Released**.
- The order now appears on the **WIP Board** under the appropriate Work Center.
- A blockchain event `WORK_ORDER_RELEASED` is recorded in the Ledger.

---

## O-06 -- Goods Issue (consume components against the order)

Goods Issue records the material consumed from stock to produce the finished good.

1. Open the released production order (from the Production Orders list or WIP Board).
2. Click **Goods Issue**.
3. Fill in:

   | Field           | Value          |
   |-----------------|----------------|
   | Material Code   | RM-0001        |
   | Quantity        | 10             |
   | From Location   | PROD-AREA-01   |

4. Click **Post Goods Issue** or **Confirm**.

**Expected:**
- Success message appears.
- In **Inventory**: RM-0001 at PROD-AREA-01 decreases by 10 (from 50 to 40 KG).
- The production order detail view shows the consumed component under a "Components" or
  "Goods Issued" section.
- Blockchain event `GOODS_ISSUE` is recorded, linked to the production order number.

---

## O-07 -- Operation Confirmation

Operation Confirmation records work performed at a work center: yield produced, scrap generated,
operator who performed the work, and time of execution.

1. Open the production order.
2. Click the **Operations** tab or section within the order.
3. Locate the first operation in the routing (e.g., **Assembly** or **Operation 0010**).
4. Click **Confirm** next to that operation.
5. Fill in the confirmation form:

   | Field            | Value             |
   |------------------|-------------------|
   | Yield Quantity   | 9                 |
   | Scrap Quantity   | 1                 |
   | Work Center      | WC-ASSEMBLY-01    |
   | Operator         | (your username)   |

6. Click **Confirm Operation** or **Save**.

**Expected:**
- Operation status changes to **Confirmed**.
- The order detail view shows yield=9 and scrap=1 for this operation.
- Blockchain event `OPERATION_CONFIRMED` is recorded with all entered values.
- If the routing has multiple operations, repeat this step for each one before proceeding.

---

## O-08 -- Quality Inspection

Quality Management records the inspection result for the production order.

1. Click **Quality Management** in the left menu.
2. Click **New Inspection** or locate the inspection linked to your order.
3. Fill in:

   | Field           | Value                            |
   |-----------------|----------------------------------|
   | Production Order| (order number from O-04)         |
   | Inspection Type | Final QC                         |
   | Result          | PASS                             |
   | Inspector       | (your username)                  |
   | Observations    | All dimensions within tolerance  |

4. Click **Submit** or **Save**.

**Expected:**
- Inspection record is saved and visible in the Quality Management list.
- The production order detail view shows the linked quality record with result PASS.
- Blockchain event `QM_INSPECTION` is recorded with the result and inspector name.

---

## O-09 -- Goods Receipt -- Finished Goods (FG into stock)

After operations are confirmed and quality is approved, the finished goods are received into
the finished goods warehouse.

1. Open the production order.
2. Click **Goods Receipt** (the finished goods receipt button within the order --
   distinct from the inbound Goods Receiving in the main menu).
3. Fill in:

   | Field           | Value        |
   |-----------------|--------------|
   | Quantity        | 9            |
   | Storage Location| FG-WAREHOUSE |

4. Click **Post** or **Confirm**.

**Expected:**
- In **Inventory**: FG-0001 appears (or increases) at FG-WAREHOUSE with quantity 9.
- Blockchain event `GOODS_RECEIPT` (finished goods variant) is recorded and linked
  to the production order number.
- The production order status may advance to **Goods Received** or **Completed**.

---

## O-10 -- Close Production Order

Closing the order finalizes the production cycle and removes it from the active WIP Board.

1. Open the production order.
2. Click **Close Order**.
3. Confirm when prompted.

**Expected:**
- Order status changes to **Closed**.
- The order is removed from the WIP Board (or moves to a "Closed" filter).
- Blockchain event `WORK_ORDER_CLOSED` is recorded.
- The complete lifecycle is now visible in the Blockchain Ledger:
  CREATED -> RELEASED -> GOODS_ISSUE -> OPERATION_CONFIRMED -> QM_INSPECTION -> GOODS_RECEIPT -> CLOSED

---

## O-11 -- Stock Transfer between locations

1. Click **Inventory** in the left menu.
2. Locate FG-0001 at FG-WAREHOUSE (quantity 9 from O-09).
3. Click **Transfer** next to that entry.
4. Fill in:

   | Field         | Value          |
   |---------------|----------------|
   | Quantity      | 4              |
   | From Location | FG-WAREHOUSE   |
   | To Location   | DISPATCH-AREA  |

5. Click **Post Transfer**.

**Expected:**
- FG-0001 at FG-WAREHOUSE decreases from 9 to 5.
- FG-0001 at DISPATCH-AREA increases to 4.
- Blockchain event `TRANSFER` recorded with both locations and quantity.

---

## O-12 -- Scrap posting

1. Click **Inventory** in the left menu.
2. Locate any material with quantity > 0 (e.g., RM-0001 at WH-RAW with 50 KG).
3. Click **Scrap** next to that entry.
4. Fill in:

   | Field         | Value     |
   |---------------|-----------|
   | Quantity      | 2         |
   | Location      | WH-RAW    |
   | Reason Code   | DAMAGED   |

5. Click **Post Scrap**.

**Expected:**
- RM-0001 at WH-RAW decreases by 2 (from 50 to 48 KG).
- Blockchain event `SCRAP` recorded with material, quantity, location, and reason code.

---

## O-13 -- Verify full blockchain trail

1. Click **Blockchain** in the left menu.
2. Scroll through all events generated during this test session.
3. Verify the following event types are present (in order):

   | # | Event Type              | From step |
   |---|-------------------------|-----------|
   | 1 | GOODS_RECEIPT / MATERIAL_RECEIVED | O-02 |
   | 2 | STAGING_TO_PRODUCTION / TRANSFER  | O-03 |
   | 3 | WORK_ORDER_CREATED                | O-04 |
   | 4 | WORK_ORDER_RELEASED               | O-05 |
   | 5 | GOODS_ISSUE                       | O-06 |
   | 6 | OPERATION_CONFIRMED               | O-07 |
   | 7 | QM_INSPECTION                     | O-08 |
   | 8 | GOODS_RECEIPT (finished goods)    | O-09 |
   | 9 | WORK_ORDER_CLOSED                 | O-10 |
   | 10| TRANSFER                          | O-11 |
   | 11| SCRAP                             | O-12 |

4. Click on the WORK_ORDER_CLOSED block and verify that its `previousHash` matches
   the `hash` of the block immediately before it in the chain.

**Expected:** All 11+ events are present. Hash linkage is intact across all blocks.
Chain integrity indicator reads Valid.

---

## O-14 -- Verify Admin access is blocked for Operator

1. With the Operator session active, look at the left navigation menu.
2. Confirm that **User Management** is NOT listed.
3. Confirm that **ERP Bridge** is NOT listed.
4. In the browser address bar, manually type:
   `https://wip.rodrigoandremarques.com/api/admin/users`
   and press Enter.

**Expected:** The page returns a 403 Forbidden response or redirects to the main dashboard.
The Operator cannot access or interact with any admin function.

---

## O-15 -- Sign out as Operator

1. Click **Logout** in the navigation menu or header.
2. Verify the browser returns to the authentication dialog.

**Expected:** Session cleared. Re-authentication required for any further action.

---

# TEST COMPLETION CHECKLIST

Mark each test as Pass (P) or Fail (F). Note any observations in the comments column.

| Test | Description                              | Role     | Result | Comments |
|------|------------------------------------------|----------|--------|----------|
| V-01 | Sign in as Visitor                       | Visitor  |        |          |
| V-02 | Dashboard overview                       | Visitor  |        |          |
| V-03 | WIP Board navigation                     | Visitor  |        |          |
| V-04 | Production Orders read-only              | Visitor  |        |          |
| V-05 | Blockchain Ledger                        | Visitor  |        |          |
| V-06 | Inventory read-only                      | Visitor  |        |          |
| V-07 | Reports                                  | Visitor  |        |          |
| V-08 | Material Master read-only                | Visitor  |        |          |
| V-09 | Work Centers and Routings                | Visitor  |        |          |
| V-10 | Sign out                                 | Visitor  |        |          |
| O-01 | Sign in as Operator                      | Operator |        |          |
| O-02 | Goods Receiving (inbound RM)             | Operator |        |          |
| O-03 | Staging to Production                    | Operator |        |          |
| O-04 | Create Production Order                  | Operator |        |          |
| O-05 | Release Production Order                 | Operator |        |          |
| O-06 | Goods Issue (component consumption)      | Operator |        |          |
| O-07 | Operation Confirmation (yield + scrap)   | Operator |        |          |
| O-08 | Quality Inspection                       | Operator |        |          |
| O-09 | Goods Receipt -- Finished Goods          | Operator |        |          |
| O-10 | Close Production Order                   | Operator |        |          |
| O-11 | Stock Transfer between locations         | Operator |        |          |
| O-12 | Scrap posting                            | Operator |        |          |
| O-13 | Full blockchain trail verification       | Operator |        |          |
| O-14 | Admin access blocked for Operator        | Operator |        |          |
| O-15 | Sign out as Operator                     | Operator |        |          |
