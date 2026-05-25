# 关键实现片段

> 许可说明：本文件中的代码片段采用 MIT 协议。详见 [../../LICENSE-CODE-MIT](../../LICENSE-CODE-MIT)。

> 说明：以下代码用于课堂复盘和课后参考。实际路径会随 App 名和 DocType 创建方式变化，请以 `pdm_tutorial` App 中生成的模块路径为准。

## 如何阅读这些代码

基础薄弱的学员不要一上来逐字背代码。建议按这个顺序看：

1. 先读标题，知道这段代码解决哪个业务问题。
2. 再读函数名，例如 `validate_items` 表示“校验明细”。
3. 然后找关键判断，例如 `if row.qty <= 0`。
4. 最后再理解 Frappe 提供的方法，例如 `frappe.throw`、`frappe.get_doc`、`frappe.call`。

你可以把代码当成业务规则的翻译：业务人员说“数量不能小于等于 0”，代码写成 `if row.qty <= 0: frappe.throw(...)`。

## 1. P-BOM Controller 校验

**业务目标：** 保存 BOM 时，系统自动检查明细是否合理。

**这段代码做了三件事：**

- 遍历 BOM 明细行。
- 检查是否有重复子项物料。
- 检查数量是否大于 0，并计算总项数。

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
                frappe.throw(f"BOM 明细中存在重复物料：{row.item}")

            if row.qty <= 0:
                frappe.throw(f"物料 {row.item} 的数量必须大于 0")

            seen.add(row.item)

    def calculate_total_qty(self):
        self.total_qty = sum(row.qty or 0 for row in self.items)
```

## 2. P-BOM Client Script

**业务目标：** 用户在页面上修改数量时，表头总项数立即刷新。

**它和后端校验的区别：**

- Client Script 负责页面即时反馈。
- Controller 负责最终保存时的可靠校验。
- 不能只依赖 Client Script，因为用户可能通过 API 或导入工具绕过页面。

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

## 3. P-ECO 发布后升版 BOM

**业务目标：** ECO 被提交后，自动更新关联 BOM 的版本和状态。

**阅读重点：**

- `self.bom` 来自 ECO 表单上的关联 BOM 字段。
- `frappe.get_doc("P-BOM", self.bom)` 表示读取这份 BOM。
- `bom.save()` 表示把修改写回数据库。

```python
import frappe
from frappe.model.document import Document


class PECO(Document):
    def on_submit(self):
        self.apply_bom_revision()

    def apply_bom_revision(self):
        if not self.bom:
            frappe.throw("请先选择需要变更的 BOM")

        bom = frappe.get_doc("P-BOM", self.bom)
        bom.version = self.target_version
        bom.status = "已审核"
        bom.save()
```

## 4. Query Report SQL

**业务目标：** 把 BOM 表头和 BOM 明细连接起来，显示完整物料构成。

**初学者 SQL 提示：**

- `bom` 是 `tabP-BOM` 的别名。
- `child` 是 `tabP-BOM Item` 的别名。
- `INNER JOIN` 用来把主表和子表关联起来。
- `child.parent = bom.name` 表示子表行属于哪一张 BOM。

```sql
SELECT
    bom.name AS "BOM:Link/P-BOM:160",
    bom.item AS "成品物料:Link/P-Item:160",
    bom.version AS "版本:Data:80",
    child.item AS "子项物料:Link/P-Item:160",
    child.qty AS "数量:Float:100"
FROM `tabP-BOM` bom
INNER JOIN `tabP-BOM Item` child ON child.parent = bom.name
WHERE bom.status = '已审核'
    AND (%(item)s IS NULL OR bom.item = %(item)s)
ORDER BY bom.modified DESC
```

## 5. REST API 调用

**业务目标：** 模拟外部系统读取或写入物料数据。

**替换说明：**

- `API_KEY` 替换成用户账号里生成的 API Key。
- `API_SECRET` 替换成同一用户的 API Secret。
- 如果返回 401，优先检查 Token 格式和空格。

```bash
curl -X GET "http://test.localhost:8000/api/resource/P-Item" \
  -H "Authorization: token API_KEY:API_SECRET"
```

```bash
curl -X POST "http://test.localhost:8000/api/resource/P-Item" \
  -H "Authorization: token API_KEY:API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{"item_code":"MAT-001","item_name":"测试物料","specification":"M6"}'
```

## 6. 单元测试样例：后端业务校验

**业务目标：** 自动证明“重复物料不能保存”这条规则没有被破坏。

**初学者理解：**

测试不是为了多写代码，而是为了防止未来改代码时把核心规则改坏。ERP 系统里很多错误不会立刻暴露，但会影响库存、生产和财务，所以测试很重要。

```python
import frappe
import unittest


class TestPBOM(unittest.TestCase):
    def test_duplicate_bom_item_is_blocked(self):
        item = frappe.get_doc({
            "doctype": "P-Item",
            "item_code": "TEST-MAT-001",
            "item_name": "测试物料"
        }).insert()

        bom = frappe.get_doc({
            "doctype": "P-BOM",
            "bom_code": "TEST-BOM-001",
            "item": item.name,
            "version": "A.0",
            "status": "草稿",
            "items": [
                {"item": item.name, "qty": 1},
                {"item": item.name, "qty": 2}
            ]
        })

        with self.assertRaises(frappe.ValidationError):
            bom.insert()
```

## 7. 单元测试样例：REST API 集成

### 7.1 进程内 API 能力测试

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
                "item_name": "API 测试物料",
                "specification": "M8"
            }
        )

        self.assertEqual(response.doctype, "P-Item")
        self.assertEqual(response.item_code, "API-MAT-001")
        self.assertTrue(frappe.db.exists("P-Item", response.name))
```

> 说明：这个测试在 Frappe 测试进程内调用与 REST 资源接口相同的插入能力，用于验证 API 写入路径、权限角色和数据落库。若要做端到端 HTTP 测试，可在独立集成测试中使用 `requests` 携带 `Authorization: token API_KEY:API_SECRET` 调用运行中的站点。

### 7.2 端到端 HTTP API 测试

```python
import requests


BASE_URL = "http://test.localhost:8000"
API_KEY = "替换为实际 API Key"
API_SECRET = "替换为实际 API Secret"


def test_create_item_by_rest_api():
    response = requests.post(
        f"{BASE_URL}/api/resource/P-Item",
        headers={
            "Authorization": f"token {API_KEY}:{API_SECRET}",
            "Content-Type": "application/json"
        },
        json={
            "item_code": "HTTP-MAT-001",
            "item_name": "HTTP 测试物料",
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

> 说明：端到端 HTTP 测试依赖正在运行的站点，适合放在独立的集成测试阶段，不建议混入每次快速单元测试。

## 8. Fixtures 配置示例

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
