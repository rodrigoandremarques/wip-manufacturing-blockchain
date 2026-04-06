# End-to-End Test Script -- End User

**System under test:** WIP Manufacturing Blockchain
**URL:** https://wip.rodrigoandremarques.com
**Last updated:** 2026-04-06

This document provides step-by-step end-to-end test scripts for all three user roles: Visitor, Operator, and Admin. Each test case specifies preconditions, exact steps, expected results, and pass/fail criteria. Tests are designed to be executed in a standard web browser with no technical tools required.

> **Note on cold start:** The server runs on Render's free tier and sleeps after 15 minutes of inactivity. If the first page load shows a loading screen or times out, wait 30-50 seconds and refresh. This is expected behavior.

---

## Credentials

| Role     | Username | Password       | Notes                             |
|----------|----------|----------------|-----------------------------------|
| Visitor  | demo     | WIP-Demo-2026! | Public -- use as-is               |
| Operator | --       | --             | Request from Admin via User Mgmt  |
| Admin    | admin    | (private)      | Check Render dashboard env vars   |

---

## ROLE: VISITOR

Visitor has read-only access to all dashboards, orders, inventory, reports, and the blockchain ledger. No create, edit, or delete actions are available.

---

### V-01 -- Login

**Precondition:** Browser open, no existing session.

**Steps:**
1. Navigate to https://wip.rodrigoandremarques.com
2. A login dialog appears (HTTP Basic Auth prompt from the browser)
3. Enter Username: `demo`
4. Enter Password: `WIP-Demo-2026!`
5. Click OK / Sign In

**Expected result:** Dashboard loads. Page title reads "WIP Manufacturing Blockchain". Navigation sidebar is visible on the left. No error message is shown.

**Fail if:** Login prompt reappears, or a "401 Unauthorized" error is displayed. (If the page takes more than 60 seconds to load, see cold start note above.)

---

### V-02 -- Dashboard overview

**Precondition:** Logged in as Visitor (V-01 passed).

**Steps:**
1. Observe the main dashboard area
2. Confirm the following panels or sections are visible: active production orders summary, WIP status, recent blockchain events or statistics

**Expected result:** At least one summary panel is visible. No "Access Denied" or blank panels caused by auth errors.

**Fail if:** Any panel shows an authorization error or fails to load data.

---

### V-03 -- WIP Board

**Precondition:** Logged in as Visitor.

**Steps:**
1. Click "WIP Board" in the navigation sidebar
2. Observe the board layout -- orders should be grouped by Work Center
3. Click on any visible production order card (if present)
4. Review the order detail view

**Expected result:** WIP Board loads and displays orders grouped by Work Center (or an empty state message if no active orders exist). Order detail view shows order number, product, status, quantity, and work center.

**Fail if:** Page fails to load, or a 403/401 error is shown.

---

### V-04 -- Production Order List (read-only)

**Precondition:** Logged in as Visitor.

**Steps:**
1. Click "Production Orders" in the navigation sidebar
2. Observe the list of orders
3. Click on any order to open its detail view
4. Confirm that no "Create", "Edit", "Release", "Close", or "Delete" buttons are visible or active

**Expected result:** Order list displays correctly. Detail view shows full order information (order number, product, quantity, status, dates, blockchain events). No write-action buttons are available to the Visitor role.

**Fail if:** Create or edit buttons are visible and functional for a Visitor user.

---

### V-05 -- Blockchain Ledger

**Precondition:** Logged in as Visitor.

**Steps:**
1. Click "Blockchain" or "Ledger" in the navigation sidebar
2. Observe the list of blocks or events
3. Click on any block to expand its details
4. Verify that the block shows: block index, timestamp, event type, hash, and previousHash

**Expected result:** Blockchain ledger is fully visible. Each block displays its hash and links to the previous block. Chain integrity indicator (if present) shows "Valid".

**Fail if:** Ledger fails to load, or chain integrity shows "Invalid" or "Broken".

---

### V-06 -- Inventory view

**Precondition:** Logged in as Visitor.

**Steps:**
1. Click "Inventory" in the navigation sidebar
2. Observe the stock list -- material codes, descriptions, quantities, and locations
3. Confirm no "Transfer", "Scrap", "Adjust", or "Receive" buttons are active

**Expected result:** Inventory list loads with material data. Read-only view confirmed -- no write actions available.

**Fail if:** Inventory page fails to load, or write buttons are accessible.

---

### V-07 -- Reports

**Precondition:** Logged in as Visitor.

**Steps:**
1. Click "Reports" in the navigation sidebar
2. Run the "WIP Summary" report (click the corresponding button or select from a dropdown)
3. Run the "Movement History" report
4. Run the "Stock Snapshot" report (if available)

**Expected result:** Each report generates and displays data (or an empty-state message if no movements have been recorded). No errors.

**Fail if:** Any report returns a 403 error or fails to render.

---

### V-08 -- Material Master

**Precondition:** Logged in as Visitor.

**Steps:**
1. Click "Material Master" in the navigation sidebar
2. Browse the list of materials
3. Click on any material to view its details, including BOM (Bill of Materials) if available

**Expected result:** Material Master list loads. BOM details are visible. No create or edit actions are accessible.

**Fail if:** Page fails to load, or edit controls are accessible to Visitor.

---

### V-09 -- Work Centers and Routings

**Precondition:** Logged in as Visitor.

**Steps:**
1. Click "Work Centers" in the navigation sidebar
2. Review the list of work centers with their descriptions and capacity rates
3. Click "Routings" in the sidebar
4. Review the routing definitions and their operation sequences

**Expected result:** Both pages load correctly and display read-only data.

**Fail if:** Either page returns an error or is inaccessible to Visitor.

---

### V-10 -- Logout

**Precondition:** Logged in as Visitor.

**Steps:**
1. Click the "Logout" button in the sidebar or header
2. Observe the response

**Expected result:** Session is cleared. Browser returns to the login prompt or a logged-out state. Navigating to any protected page again requires re-authentication.

**Fail if:** Clicking logout has no effect, or the session persists after logout.

---

## ROLE: OPERATOR

Operator has full access to all shop floor movements: goods receiving, staging, goods issue, operation confirmations, goods receipt, quality management, transfers, and scrap. Operator cannot access user management or the ERP Bridge export.

**Precondition for all Operator tests:** Admin must have created an Operator account via User Management. Contact the system administrator for credentials.

---

### O-01 -- Login as Operator

**Precondition:** Operator credentials available.

**Steps:**
1. Navigate to https://wip.rodrigoandremarques.com
2. Enter Operator username and password in the login prompt
3. Click OK

**Expected result:** Dashboard loads. Sidebar shows all shop floor modules including Goods Receiving, Staging to Production, Goods Issue, WIP Board, Quality, and Reports. User Management and ERP Bridge are NOT visible in the sidebar.

**Fail if:** Login fails, or User Management is accessible to the Operator.

---

### O-02 -- Goods Receiving (inbound raw material)

**Precondition:** Logged in as Operator.

**Steps:**
1. Click "Goods Receiving" in the navigation sidebar
2. Fill in the goods receipt form:
   - Material Code: any code present in Material Master (e.g., RM-0001)
   - Quantity: 100
   - Unit: KG (or the unit defined for that material)
   - Supplier: TEST-SUPPLIER-01
   - Purchase Order reference: PO-TEST-001
3. Click "Post Receipt"

**Expected result:** System confirms the receipt. A new blockchain event of type GOODS_RECEIPT or MATERIAL_RECEIVED is added to the ledger. Inventory for the material increases by 100 units. The event appears in the Blockchain Ledger view.

**Fail if:** Receipt fails, inventory does not update, or no blockchain event is generated.

---

### O-03 -- Staging to Production

**Precondition:** Logged in as Operator. At least one material with stock > 0 (O-02 passed).

**Steps:**
1. Click "Staging to Production" in the sidebar
2. Select the material received in O-02
3. Enter quantity: 50
4. Select destination: Production Area or Work Center (e.g., WC-01)
5. Click "Post Transfer"

**Expected result:** Stock in warehouse location decreases by 50. Stock in production area increases by 50. Blockchain event STAGING_TO_PRODUCTION (or equivalent) is recorded.

**Fail if:** Stock totals do not change, or no blockchain event is created.

---

### O-04 -- Create Production Order

**Precondition:** Logged in as Operator.

**Steps:**
1. Click "Production Orders" in the sidebar
2. Click "New Order" or "Create"
3. Fill in the form:
   - Product: FG-0001 (or any finished goods material in master data)
   - Quantity: 10
   - Priority: NORMAL
   - Due Date: any future date
   - Cost Center: CC-TEST-01
4. Click "Create" / "Save"

**Expected result:** New production order is created with status "Created". Order number is assigned (e.g., ORD-0001). Blockchain event WORK_ORDER_CREATED is recorded with timestamp and operator name.

**Fail if:** Order creation fails, or the blockchain event is not generated.

---

### O-05 -- Release Production Order

**Precondition:** Production order in "Created" status (O-04 passed).

**Steps:**
1. Open the production order created in O-04
2. Click "Release"
3. Confirm the action

**Expected result:** Order status changes to "Released". Blockchain event WORK_ORDER_RELEASED is recorded. The order becomes visible on the WIP Board.

**Fail if:** Status does not change, or the release blockchain event is missing.

---

### O-06 -- Goods Issue (material consumption)

**Precondition:** Order in "Released" status (O-05 passed). Staged material available.

**Steps:**
1. Open the released production order
2. Click "Goods Issue"
3. Select the component material from the staged stock
4. Enter quantity to consume: 10
5. Confirm the goods issue

**Expected result:** Component stock decreases by 10. Blockchain event GOODS_ISSUE is recorded, linked to the production order number. Order details show the consumed components.

**Fail if:** Stock does not decrease, or the GOODS_ISSUE event is missing from the ledger.

---

### O-07 -- Operation Confirmation

**Precondition:** Order released and components issued (O-06 passed).

**Steps:**
1. Open the production order
2. Navigate to "Operations" or "Confirmations"
3. Select the first operation in the routing (e.g., Cutting)
4. Fill in the confirmation form:
   - Yield Quantity: 9
   - Scrap Quantity: 1
   - Work Center: WC-CUTTING-01
   - Operator name: (your name or test value)
5. Click "Confirm Operation"

**Expected result:** Operation status updates to "Confirmed". Yield and scrap quantities are recorded. Blockchain event OPERATION_CONFIRMED is added with yield, scrap, work center, and operator. If backflush is enabled, component stock updates automatically.

**Fail if:** Confirmation fails, or the OPERATION_CONFIRMED event is missing.

---

### O-08 -- Quality Management

**Precondition:** At least one operation confirmed (O-07 passed).

**Steps:**
1. Click "Quality Management" in the sidebar
2. Click "New Inspection" or open the inspection linked to the production order
3. Fill in:
   - Production Order: (order from O-04)
   - Inspection Type: Final QC
   - Result: Pass
   - Inspector: (test name)
   - Observations: "All dimensions within tolerance"
4. Click "Submit" or "Save"

**Expected result:** Quality inspection record is saved and linked to the production order. Blockchain event QM_INSPECTION is recorded with result and inspector name. Inspection appears in the order's detail view.

**Fail if:** Inspection record is not saved, or no blockchain event is generated.

---

### O-09 -- Goods Receipt (finished goods into stock)

**Precondition:** Order operations confirmed and quality passed (O-07 and O-08 passed).

**Steps:**
1. Open the production order
2. Click "Goods Receipt" (finished goods receipt, distinct from inbound goods receiving)
3. Enter:
   - Quantity: 9 (yield from O-07)
   - Storage Location: FG-WAREHOUSE
4. Click "Post"

**Expected result:** Finished goods inventory increases by 9 units. Blockchain event GOODS_RECEIPT (finished goods) is recorded and linked to the order. Order status may transition to "Goods Received" or "In Progress" depending on remaining operations.

**Fail if:** FG stock does not increase, or the blockchain event is missing.

---

### O-10 -- Transfer between locations

**Precondition:** Logged in as Operator. Stock exists in at least one location.

**Steps:**
1. Click "Inventory" in the sidebar
2. Select a material with quantity > 0
3. Click "Transfer"
4. Set:
   - From location: FG-WAREHOUSE
   - To location: DISPATCH-AREA (or any defined location)
   - Quantity: 5
5. Confirm the transfer

**Expected result:** Stock decreases in source location by 5. Stock increases in destination location by 5. Blockchain event TRANSFER is recorded with from/to locations, quantity, and operator.

**Fail if:** Stock totals do not balance, or no blockchain event is generated.

---

### O-11 -- Scrap posting

**Precondition:** Logged in as Operator. Stock exists in any location.

**Steps:**
1. Click "Inventory" in the sidebar
2. Select a material with quantity > 0
3. Click "Scrap"
4. Enter:
   - Quantity: 1
   - Reason Code: DAMAGED (or any available code)
   - Location: source location of the material
5. Confirm the scrap

**Expected result:** Stock decreases by 1 in the selected location. Blockchain event SCRAP is recorded with reason code, quantity, and operator. Total inventory reflects the write-off.

**Fail if:** Stock does not decrease, or no blockchain event is recorded.

---

### O-12 -- Close Production Order

**Precondition:** Order goods receipt posted (O-09 passed).

**Steps:**
1. Open the production order
2. Click "Close Order"
3. Confirm the action

**Expected result:** Order status changes to "Closed". Blockchain event WORK_ORDER_CLOSED is recorded. Order is no longer visible on the WIP Board (or moves to a "Closed" section).

**Fail if:** Status does not update to Closed, or the WORK_ORDER_CLOSED blockchain event is missing.

---

### O-13 -- Verify Admin access is blocked

**Precondition:** Logged in as Operator.

**Steps:**
1. Observe the navigation sidebar -- "User Management" and "ERP Bridge" must not be present
2. Attempt to navigate directly to /admin/prototype.html or /api/admin/users
3. Note the response

**Expected result:** Admin routes are not present in the Operator's sidebar. Direct URL navigation to admin pages results in a 403 Forbidden response or a redirect to the main dashboard.

**Fail if:** Operator can access or interact with any user management or ERP Bridge function.

---

## ROLE: ADMIN

Admin has full system access: all Operator actions plus User Management, ERP Bridge export, and the Admin Panel.

**Precondition for all Admin tests:** Admin credentials from the Render dashboard environment variable `ADMIN_PASSWORD`.

---

### A-01 -- Login as Admin

**Precondition:** Admin credentials available.

**Steps:**
1. Navigate to https://wip.rodrigoandremarques.com
2. Enter Username: `admin` and the admin password
3. Click OK

**Expected result:** Dashboard loads with all modules visible in the sidebar, including User Management and ERP Bridge. Admin Panel link is accessible at /admin/prototype.html.

**Fail if:** Login fails, or the sidebar does not show admin-specific modules.

---

### A-02 -- Create Operator user

**Precondition:** Logged in as Admin.

**Steps:**
1. Click "User Management" in the sidebar (or navigate to the Admin Panel)
2. Click "New User" or "Create User"
3. Fill in:
   - Username: test-operator
   - Password: TestOp-2026!
   - Role: operator
4. Click "Create"

**Expected result:** User "test-operator" is created with role "operator". The user appears in the user list. The creation is acknowledged by the system.

**Fail if:** User creation fails, or the new user does not appear in the list.

---

### A-03 -- Verify Operator user can log in

**Precondition:** test-operator created in A-02.

**Steps:**
1. Log out from Admin (click Logout)
2. Log in with Username: `test-operator`, Password: `TestOp-2026!`
3. Verify that the Operator dashboard is shown (no User Management in sidebar)
4. Log out and log back in as Admin

**Expected result:** test-operator can authenticate successfully and has the Operator role view.

**Fail if:** test-operator login fails, or the user can access admin-only sections.

---

### A-04 -- Delete Operator user

**Precondition:** Logged in as Admin. test-operator exists.

**Steps:**
1. Click "User Management" in the sidebar
2. Find "test-operator" in the user list
3. Click "Delete" next to the user
4. Confirm the deletion

**Expected result:** test-operator is removed from the user list. Attempting to log in with test-operator credentials now returns a 401 Unauthorized response.

**Fail if:** User is not removed, or deleted user can still authenticate.

---

### A-05 -- ERP Bridge export

**Precondition:** Logged in as Admin. At least one production order exists.

**Steps:**
1. Click "ERP Bridge" in the sidebar
2. Click "Export" or navigate to the export endpoint
3. Download or view the JSON export of production orders and movements

**Expected result:** A structured JSON payload is returned containing production orders and their movements, formatted for ERP integration (SAP IDOC-compatible structure). The response includes order numbers, materials, quantities, cost centers, and timestamps.

**Fail if:** Export returns an empty payload when orders exist, or returns a 403 for the Admin role.

---

### A-06 -- Admin Panel access

**Precondition:** Logged in as Admin.

**Steps:**
1. Navigate to https://wip.rodrigoandremarques.com/admin/prototype.html
2. Verify the admin panel loads
3. Review available controls

**Expected result:** Admin Panel loads correctly and provides access to advanced controls such as chain inspection, user management, and system status.

**Fail if:** Page returns 403, 404, or does not load.

---

### A-07 -- Full blockchain chain inspection

**Precondition:** Logged in as Admin. Multiple events have been recorded during prior tests.

**Steps:**
1. Click "Blockchain" or "Ledger" in the sidebar
2. Scroll through the full list of blocks from genesis to the latest
3. Verify that each block shows its hash and previousHash
4. Verify that block N's previousHash matches block N-1's hash
5. Check the chain integrity indicator if present

**Expected result:** Chain is fully visible. Hash linkage is consistent across all blocks. Chain integrity indicator reads "Valid". Total block count matches the number of events generated during the test session.

**Fail if:** Any block has a mismatched hash reference, or the integrity indicator shows "Invalid".

---

### A-08 -- Admin cannot be locked out by Operator actions

**Precondition:** Logged in as Admin.

**Steps:**
1. Attempt to delete the "admin" user via User Management
2. Note the system response

**Expected result:** System refuses to delete the admin account, returning an error message such as "Cannot delete the primary admin user".

**Fail if:** Admin account can be deleted through the UI.

---

### A-09 -- Logout as Admin

**Precondition:** Logged in as Admin.

**Steps:**
1. Click "Logout"
2. Attempt to access /api/admin/users or any protected page

**Expected result:** Session is cleared. All protected routes require re-authentication.

**Fail if:** Any route is accessible after logout.

---

## Test Completion Summary

| Test ID | Description                              | Role     | Result |
|---------|------------------------------------------|----------|--------|
| V-01    | Login as Visitor                         | Visitor  |        |
| V-02    | Dashboard overview                       | Visitor  |        |
| V-03    | WIP Board                                | Visitor  |        |
| V-04    | Production Order List (read-only)        | Visitor  |        |
| V-05    | Blockchain Ledger                        | Visitor  |        |
| V-06    | Inventory view                           | Visitor  |        |
| V-07    | Reports                                  | Visitor  |        |
| V-08    | Material Master                          | Visitor  |        |
| V-09    | Work Centers and Routings                | Visitor  |        |
| V-10    | Logout                                   | Visitor  |        |
| O-01    | Login as Operator                        | Operator |        |
| O-02    | Goods Receiving                          | Operator |        |
| O-03    | Staging to Production                    | Operator |        |
| O-04    | Create Production Order                  | Operator |        |
| O-05    | Release Production Order                 | Operator |        |
| O-06    | Goods Issue                              | Operator |        |
| O-07    | Operation Confirmation                   | Operator |        |
| O-08    | Quality Management                       | Operator |        |
| O-09    | Goods Receipt (finished goods)           | Operator |        |
| O-10    | Transfer between locations               | Operator |        |
| O-11    | Scrap posting                            | Operator |        |
| O-12    | Close Production Order                   | Operator |        |
| O-13    | Verify Admin access is blocked           | Operator |        |
| A-01    | Login as Admin                           | Admin    |        |
| A-02    | Create Operator user                     | Admin    |        |
| A-03    | Verify Operator user can log in          | Admin    |        |
| A-04    | Delete Operator user                     | Admin    |        |
| A-05    | ERP Bridge export                        | Admin    |        |
| A-06    | Admin Panel access                       | Admin    |        |
| A-07    | Full blockchain chain inspection         | Admin    |        |
| A-08    | Admin cannot be locked out               | Admin    |        |
| A-09    | Logout as Admin                          | Admin    |        |
