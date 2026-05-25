# Practical Labs: Build a Mini PDM System with Frappe

These labs guide you through building a small PDM system. The goal is not only to learn Frappe features, but to understand how business objects, rules, approvals, reports, and APIs fit together.

## Before You Start

If this is your first time using Frappe, remember three habits:

- Most configuration pages can be opened from the top search bar. Search for `DocType`, `Role`, `Workflow`, `Report`, or `Client Script`.
- After every configuration change, click Save and refresh the page to verify it.
- If a page does not reflect your changes, try `bench clear-cache` and refresh the browser.

## Shared Sample Data

Use this sample data throughout the labs:

| Type | Code | Name | Notes |
| --- | --- | --- | --- |
| Finished Item | FG-MOTOR-001 | Small Motor Assembly | Final product |
| Child Item | MAT-SCREW-001 | Screw | Quantity 4 |
| Child Item | MAT-COVER-001 | Cover | Quantity 1 |
| Child Item | MAT-MOTOR-001 | Motor | Quantity 1 |
| BOM | BOM-FG-MOTOR-001-A0 | Small Motor Assembly BOM | Initial version A.0 |
| ECO | ECO-FG-MOTOR-001-A1 | Motor Assembly Revision | Target version A.1 |

## Lab 1: Business Entity Modeling

**Goal:** Understand metadata-driven modeling and parent-child document design.

**Tasks:**

1. Create `P-Item` with fields for item code, item name, and specification.
2. Create `P-BOM` with fields for BOM code, finished item Link, version, status, total quantity, and child rows.
3. Create `P-BOM Item` as a child table with child item Link, quantity, and specification.
4. Add a Table field in `P-BOM` that uses `P-BOM Item`.
5. Create `P-ECO` with ECO code, linked BOM, reason, target version, effective date, and approval status.
6. Configure Naming Series or a clear manual naming rule for items, BOMs, and ECOs.

**Menu entry point:**

1. Search `DocType` in the top search bar.
2. Click New and create `P-Item`.
3. Add fields in the Fields table.
4. Save the DocType.
5. Search `P-Item`, click New, and create test records.

**Field examples:**

`P-Item`:

| Label | Fieldname | Type | Required |
| --- | --- | --- | --- |
| Item Code | item_code | Data | Yes |
| Item Name | item_name | Data | Yes |
| Specification | specification | Small Text | No |

`P-BOM Item`:

| Label | Fieldname | Type | Required | Notes |
| --- | --- | --- | --- | --- |
| Child Item | item | Link | Yes | Options: `P-Item` |
| Quantity | qty | Float | Yes | Must be greater than 0 |
| Specification | specification | Small Text | No | Can be fetched from item |

**Acceptance check:** You can create items, one BOM with child rows, and one ECO in the UI.

**Common mistakes:**

- Link field Options is empty, so Frappe does not know which DocType to link.
- `P-BOM Item` is not marked as a child table, so it cannot be used in a Table field.
- Fieldnames use spaces or non-English names, making later code harder. Prefer lowercase English names with underscores.

## Lab 2: Business Logic and Automation

**Goal:** Learn server-side Controller hooks and browser-side Client Script.

**Business rules in plain English:**

- One BOM must not contain the same child item twice.
- Every child quantity must be greater than 0.
- Header total quantity equals the sum of child row quantities.

**Tasks:**

1. In the Python Controller for `P-BOM`, implement a `validate` hook that blocks duplicate child items.
2. In the same hook, block child quantities that are less than or equal to 0.
3. Create a Client Script that updates `total_qty` when child row quantities change.
4. Create a deliberate duplicate child item and observe the `frappe.throw` message.

**Implementation hints:**

1. Server-side code belongs in the generated `P-BOM` Python Controller file.
2. After changing Python code, restart the development server if the change does not apply.
3. For beginners, create Client Script from the UI first. Later you can move JavaScript into the app.

**Acceptance check:**

- Adding `MAT-SCREW-001` twice in the same BOM should fail.
- Setting a child quantity to `0` or `-1` should fail.
- Setting quantities to `4`, `1`, and `1` should make `total_qty` equal `6`.

## Lab 3: Permissions and Workflow

**Goal:** Implement enterprise-style permissions and approval flow.

**Roles:**

| Role | Can Do | Cannot Do |
| --- | --- | --- |
| Engineering User | Create items, edit draft BOMs, create ECOs | Submit BOMs, release ECOs |
| Engineering Manager | Review BOMs, release ECOs | Bypass process and edit released data |
| Quality User | View approved results | Modify BOMs or ECOs |

**Tasks:**

1. Create `Engineering User`, `Engineering Manager`, and `Quality User`.
2. Configure `P-BOM` permissions so engineering users can edit drafts but only managers can submit.
3. Configure `P-ECO` Workflow: Draft -> Under Review -> Released.
4. Make released ECOs read-only.
5. Create at least two test users and verify role differences.

**Menu entry point:**

1. Search `Role` to create roles.
2. Search `User` to create test users and assign roles.
3. Search `Role Permission Manager` to configure DocType permissions.
4. Search `Workflow` to create the ECO workflow.

**Acceptance check:**

- A normal engineering user cannot submit a BOM.
- An engineering manager can complete the approval action.
- After ECO release, changing the reason should fail.
- A quality user can view target data but cannot modify it.

## Lab 4: ECO-Driven BOM Revision

**Goal:** Implement cross-document business logic.

**Business story:**

An engineer discovers that the current motor supplier has discontinued a model. The product must switch to a new motor. Instead of directly editing the BOM, the engineer creates an ECO. After manager approval, the system updates the BOM from version `A.0` to `A.1`.

**Tasks:**

1. Implement Python logic in `P-ECO.on_submit`.
2. When the ECO is submitted, find the linked `P-BOM`.
3. Set the BOM status to Approved.
4. Update the BOM version to the ECO target version.
5. Add a Comment or log entry showing which ECO triggered the revision.

**Implementation flow:**

1. User submits `P-ECO`.
2. `P-ECO.on_submit` is triggered.
3. Code reads `self.bom`.
4. Code loads the linked `P-BOM`.
5. Code updates `version` and `status`.
6. Code saves the BOM.

**Acceptance check:** After submitting and releasing an ECO, the linked BOM version and status update automatically, and the revision source can be traced.

**Common mistakes:**

- `on_submit` does not trigger because `P-ECO` is not submittable.
- The linked BOM cannot be found because the Link field Options is wrong.
- Save fails due to permissions. First verify user permissions, then discuss whether a controlled backend process should use `ignore_permissions`.

## Lab 5: Query Report

**Goal:** Build a business report for BOM structure analysis.

**Report question:** Which child items are included in each approved BOM?

**Example output:**

| BOM | Finished Item | Version | Child Item | Quantity |
| --- | --- | --- | --- | --- |
| BOM-FG-MOTOR-001-A0 | FG-MOTOR-001 | A.1 | MAT-SCREW-001 | 4 |
| BOM-FG-MOTOR-001-A0 | FG-MOTOR-001 | A.1 | MAT-COVER-001 | 1 |
| BOM-FG-MOTOR-001-A0 | FG-MOTOR-001 | A.1 | MAT-MOTOR-001 | 1 |

**Tasks:**

1. Create a Query Report.
2. Write SQL that joins approved BOMs with child rows.
3. Add a filter by finished item.
4. Verify Excel export.
5. Discuss SQL permissions and parameter binding.

**Menu entry point:**

1. Search `Report`.
2. Create a new Report.
3. Set Report Type to Query Report.
4. Set Reference DocType to `P-BOM`.
5. Paste SQL.
6. Add an `item` filter with type Link and Options `P-Item`.

**Beginner reminder:** Frappe database table names usually start with `tab`, such as `tabP-BOM`. If the DocType name contains spaces or special characters, wrap the table name in backticks.

**Acceptance check:** The report displays correct data, the item filter works, and Excel export works.

## Lab 6: REST API

**Goal:** Learn token-authenticated REST API access.

**Beginner explanation:**

The browser UI is for people. The API is for other systems. Postman, Insomnia, or curl can act like an external system calling Frappe.

**Tasks:**

1. Generate `API Key` and `API Secret` for a user.
2. Use Postman, Insomnia, or curl with `Authorization: token [API Key]:[API Secret]`.
3. Call `/api/resource/P-Item` to read items.
4. Create a test item through the API.
5. Try the same API using a user without permission and observe the response.

**Minimal curl example:**

```bash
curl -X GET "http://test.localhost:8000/api/resource/P-Item" \
  -H "Authorization: token API_KEY:API_SECRET"
```

**HTTP status cheat sheet:**

| Status | Meaning | Common Cause |
| --- | --- | --- |
| 200 | Success | Request and permission are correct |
| 401 | Not authenticated | Token format or key/secret is wrong |
| 403 | Forbidden | User is authenticated but lacks permission |
| 404 | Not found | URL or DocType name is wrong |
| 500 | Server error | Backend code or data error |

**Acceptance check:** The API returns 200 OK, the new item appears in the backend, and you can explain the difference between 401 and 403.

## Lab 7: Tests, Fixtures, and Delivery

**Goal:** Turn classroom work into a portable and verifiable engineering asset.

**Why beginners should care:**

Enterprise systems are not finished just because they worked once on your laptop. You need to prove:

- The system can run in another environment.
- Core business rules are protected.
- Roles, permissions, fields, and workflows are not trapped in one local database.

**Tasks:**

1. Write one unit test for the duplicate BOM child item rule.
2. Run `bench run-tests`.
3. Configure fixtures in `hooks.py` for roles, workflows, custom fields, and property setters.
4. Run `bench export-fixtures` and inspect generated JSON files.
5. Run `bench migrate` and `bench clear-cache`.
6. Complete the final checklist in `FINAL_DELIVERY_CHECKLIST.md`.

**Hints:**

- Start with one test for the most important rule.
- Export the most important fixtures first. Do not try to perfect everything on the first attempt.
- After exporting fixtures, make sure the JSON files are included in Git.

**Acceptance check:** Tests can run, fixtures can export, and the project can be demonstrated using the final delivery checklist.

## Overall Acceptance Checklist

- [ ] DocTypes are clear and include `P-Item`, `P-BOM`, `P-BOM Item`, and `P-ECO`.
- [ ] Permission matrix distinguishes engineering users, engineering managers, and quality users.
- [ ] ECO release triggers linked BOM revision.
- [ ] Report filters work.
- [ ] REST API works through token authentication.
- [ ] Key configuration is exported through fixtures.
- [ ] At least one core unit test passes.
