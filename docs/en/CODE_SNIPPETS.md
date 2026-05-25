# Code Snippets: Reference Implementations

> License: code snippets in this file are licensed under the MIT License. See [../../LICENSE-CODE-MIT](../../LICENSE-CODE-MIT).

These snippets are for classroom review and self-study. Actual paths may differ depending on your app name and how DocTypes are generated. The examples assume a custom app such as `pdm_tutorial`.

## How to Read These Snippets

Do not start by memorizing every line. Read in this order:

1. Read the title and identify the business problem.
2. Read function names such as `validate_items`.
3. Find key conditions such as `if row.qty <= 0`.
4. Then learn Frappe helpers such as `frappe.throw`, `frappe.get_doc`, and `frappe.call`.

Code is a translation of business rules. "Quantity must be greater than 0" becomes `if row.qty <= 0: frappe.throw(...)`.

## 1. P-BOM Controller Validation

**Business goal:** When saving a BOM, validate child rows.

**This code does three things:**

- Iterates through BOM child rows.
- Blocks duplicate child items.
- Blocks non-positive quantities and calculates total quantity.

```python
import frappe
from frappe.model.document import Document


class PBOM(Document):
    def validate(self):
        self.validate_items()
        self.calculate_total_qty()

    def validate_items(self):
        seen = set()

        for row in self.items:
            if not row.item:
                continue

            if row.item in seen:
                frappe.throw(f"Duplicate BOM child item: {row.item}")

            if row.qty <= 0:
                frappe.throw(f"Quantity for item {row.item} must be greater than 0")

            seen.add(row.item)

    def calculate_total_qty(self):
        self.total_qty = sum(row.qty or 0 for row in self.items)
```

## 2. P-BOM Client Script

**Business goal:** Update the header total quantity immediately when users edit child quantities.

**Difference from backend validation:**

- Client Script gives instant feedback in the browser.
- Controller validation protects the final save on the server.
- Never rely only on Client Script, because API calls and imports can bypass the browser form.

```javascript
frappe.ui.form.on("P-BOM Item", {
    qty(frm) {
        update_total_qty(frm);
    },
    items_remove(frm) {
        update_total_qty(frm);
    }
});

function update_total_qty(frm) {
    const total = (frm.doc.items || []).reduce((sum, row) => {
        return sum + (flt(row.qty) || 0);
    }, 0);

    frm.set_value("total_qty", total);
}
```

## 3. P-ECO Updates the Linked BOM

**Business goal:** When an ECO is submitted, update the linked BOM version and status.

**Reading notes:**

- `self.bom` comes from the linked BOM field on the ECO.
- `frappe.get_doc("P-BOM", self.bom)` loads that BOM.
- `bom.save()` writes the change back to the database.

```python
import frappe
from frappe.model.document import Document


class PECO(Document):
    def on_submit(self):
        self.apply_bom_revision()

    def apply_bom_revision(self):
        if not self.bom:
            frappe.throw("Please select the BOM to revise")

        bom = frappe.get_doc("P-BOM", self.bom)
        bom.version = self.target_version
        bom.status = "Approved"
        bom.add_comment("Comment", f"Revised by ECO {self.name}")
        bom.save()
```

## 4. Query Report SQL

**Business goal:** Join BOM headers with BOM child rows and show full structure.

**SQL notes for beginners:**

- `bom` is an alias for `tabP-BOM`.
- `child` is an alias for `tabP-BOM Item`.
- `INNER JOIN` connects parent rows with child rows.
- `child.parent = bom.name` means the child row belongs to that BOM.

```sql
SELECT
    bom.name AS "BOM:Link/P-BOM:160",
    bom.item AS "Finished Item:Link/P-Item:160",
    bom.version AS "Version:Data:80",
    child.item AS "Child Item:Link/P-Item:160",
    child.qty AS "Quantity:Float:100"
FROM `tabP-BOM` bom
INNER JOIN `tabP-BOM Item` child ON child.parent = bom.name
WHERE bom.status = 'Approved'
    AND (%(item)s IS NULL OR bom.item = %(item)s)
ORDER BY bom.modified DESC
```

## 5. REST API Calls

**Business goal:** Simulate an external system reading or writing item data.

Replace:

- `API_KEY` with the user's API Key.
- `API_SECRET` with the same user's API Secret.

If you get 401, first check the token format and spaces.

```bash
curl -X GET "http://test.localhost:8000/api/resource/P-Item" \
  -H "Authorization: token API_KEY:API_SECRET"
```

```bash
curl -X POST "http://test.localhost:8000/api/resource/P-Item" \
  -H "Authorization: token API_KEY:API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{"item_code":"MAT-001","item_name":"Test Item","specification":"M6"}'
```

## 6. Unit Test: Backend Business Validation

**Business goal:** Automatically prove that duplicate BOM child items are blocked.

Testing prevents future changes from breaking core ERP rules. Many ERP mistakes do not fail loudly at first, but later affect production, purchasing, inventory, or finance.

```python
import frappe
import unittest


class TestPBOM(unittest.TestCase):
    def test_duplicate_bom_item_is_blocked(self):
        item = frappe.get_doc({
            "doctype": "P-Item",
            "item_code": "TEST-MAT-001",
            "item_name": "Test Item"
        }).insert()

        bom = frappe.get_doc({
            "doctype": "P-BOM",
            "bom_code": "TEST-BOM-001",
            "item": item.name,
            "version": "A.0",
            "status": "Draft",
            "items": [
                {"item": item.name, "qty": 1},
                {"item": item.name, "qty": 2}
            ]
        })

        with self.assertRaises(frappe.ValidationError):
            bom.insert()
```

## 7. API Integration Tests

### 7.1 In-Process API Capability Test

```python
import frappe
import unittest


class TestPItemAPI(unittest.TestCase):
    def setUp(self):
        self.user = "api.test@example.com"

        if not frappe.db.exists("User", self.user):
            user = frappe.get_doc({
                "doctype": "User",
                "email": self.user,
                "first_name": "API",
                "last_name": "Tester",
                "send_welcome_email": 0,
                "roles": [{"role": "Engineering User"}]
            }).insert(ignore_permissions=True)
        else:
            user = frappe.get_doc("User", self.user)

        api_secret = frappe.generate_hash(length=15)
        user.api_key = frappe.generate_hash(length=15)
        user.api_secret = api_secret
        user.save(ignore_permissions=True)

        self.api_key = user.api_key
        self.api_secret = api_secret

    def test_create_item_through_resource_api(self):
        frappe.set_user(self.user)

        response = frappe.call(
            "frappe.client.insert",
            doc={
                "doctype": "P-Item",
                "item_code": "API-MAT-001",
                "item_name": "API Test Item",
                "specification": "M8"
            }
        )

        self.assertEqual(response.doctype, "P-Item")
        self.assertEqual(response.item_code, "API-MAT-001")
        self.assertTrue(frappe.db.exists("P-Item", response.name))
```

This test runs inside the Frappe test process. It validates the same insert capability used by the resource API path. For full HTTP testing, use a separate integration test against a running site.

### 7.2 End-to-End HTTP API Test

```python
import requests


BASE_URL = "http://test.localhost:8000"
API_KEY = "replace-with-real-api-key"
API_SECRET = "replace-with-real-api-secret"


def test_create_item_by_rest_api():
    response = requests.post(
        f"{BASE_URL}/api/resource/P-Item",
        headers={
            "Authorization": f"token {API_KEY}:{API_SECRET}",
            "Content-Type": "application/json"
        },
        json={
            "item_code": "HTTP-MAT-001",
            "item_name": "HTTP Test Item",
            "specification": "M10"
        },
        timeout=10
    )

    assert response.status_code == 200
    payload = response.json()
    assert payload["data"]["item_code"] == "HTTP-MAT-001"


def test_read_item_list_by_rest_api():
    response = requests.get(
        f"{BASE_URL}/api/resource/P-Item",
        headers={"Authorization": f"token {API_KEY}:{API_SECRET}"},
        timeout=10
    )

    assert response.status_code == 200
    assert "data" in response.json()
```

End-to-end HTTP tests depend on a running site. They are useful for integration testing, but should not slow down every quick unit test run.

## 8. Fixtures Example

```python
fixtures = [
    {
        "dt": "Custom Field",
        "filters": [["name", "like", "P-%"]]
    },
    {
        "dt": "Property Setter",
        "filters": [["doc_type", "in", ["P-Item", "P-BOM", "P-ECO"]]]
    },
    {
        "dt": "Workflow",
        "filters": [["document_type", "in", ["P-ECO"]]]
    },
    {
        "dt": "Role",
        "filters": [["role_name", "in", ["Engineering User", "Engineering Manager", "Quality User"]]]
    }
]
```
