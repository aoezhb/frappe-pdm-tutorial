# Frappe 实训：PDM 系统开发实战

本实训通过一个微型 PDM（产品数据管理）系统的构建，带你走通 Frappe 开发的全流程。

## 实训前说明

如果你是第一次使用 Frappe，请先记住三个操作习惯：

- 大多数配置都可以通过顶部搜索框进入，例如搜索 `DocType`、`Role`、`Workflow`、`Report`。
- 每做完一个配置，都要点击 Save，并刷新页面确认效果。
- 遇到页面没有更新时，先尝试 `bench clear-cache`，再刷新浏览器。

## 本实训统一示例数据

后续实验都可以使用这组数据，避免每一步都重新想名字：

| 类型 | 编号 | 名称 | 说明 |
| --- | --- | --- | --- |
| 成品物料 | FG-MOTOR-001 | 小型电机组件 | 最终产品 |
| 子项物料 | MAT-SCREW-001 | 螺丝 | 数量 4 |
| 子项物料 | MAT-COVER-001 | 外壳 | 数量 1 |
| 子项物料 | MAT-MOTOR-001 | 电机 | 数量 1 |
| BOM | BOM-FG-MOTOR-001-A0 | 小型电机组件 BOM | 初始版本 A.0 |
| ECO | ECO-FG-MOTOR-001-A1 | 电机组件升版 | 目标版本 A.1 |

## 🧪 实验一：业务实体建模 (Item & BOM & ECO)

**目标：** 理解元数据驱动的建模方式，掌握主子表结构与关联设计。

- **任务：**
  1. 创建 `P-Item` (物料)：字段包括物料编号 (ID)、名称、规格。
  2. 创建 `P-BOM` (物料清单)：字段包括 BOM 编号、关联物料 (`Link`)、版本号、状态 (`Select`: 草稿/已审核)、总项数。
  3. 创建 `P-BOM Item` (BOM 明细子表)：字段包括子项物料 (`Link`)、数量、规格。并在 `P-BOM` 中引入该子表。
  4. 创建 `P-ECO` (工程变更单)：字段包括变更编号、关联 BOM (`Link`)、变更原因、目标版本、生效日期、审批状态。
  5. 为物料、BOM、ECO 配置 Naming Series 或清晰的命名规则。
- **验收：** 能在 UI 界面录入物料、BOM（含明细）和 ECO 基础数据。

**操作入口：**

1. 在 Frappe 顶部搜索框输入 `DocType`。
2. 点击 New，新建第一个 DocType：`P-Item`。
3. 勾选是否需要命名规则，根据课堂要求选择手动命名或 Naming Series。
4. 在 Fields 区域逐行添加字段。
5. 保存后，回到搜索框搜索 `P-Item`，点击 New 录入测试数据。

**字段示例：**

`P-Item`：

| Label | Fieldname | Type | 必填 |
| --- | --- | --- | --- |
| 物料编号 | item_code | Data | 是 |
| 物料名称 | item_name | Data | 是 |
| 规格 | specification | Small Text | 否 |

`P-BOM Item`：

| Label | Fieldname | Type | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| 子项物料 | item | Link | 是 | Options 填 `P-Item` |
| 数量 | qty | Float | 是 | 后续校验必须大于 0 |
| 规格 | specification | Small Text | 否 | 可通过 Fetch From 带出 |

**常见错误：**

- Link 字段没有填写 Options，导致系统不知道要链接哪个 DocType。
- 子表 DocType 没有勾选 `Is Child Table`，导致不能作为 Table 字段使用。
- 字段名使用中文或空格，后续写代码时容易出错。建议 Fieldname 使用英文小写和下划线。

## 🧪 实验二：业务逻辑与自动化 (Validate & Client Script)

**目标：** 掌握后端 Controller 钩子与前端 Client Script。

- **任务：**
  1. **后端校验：** 在 `P-BOM` 的 Python 类中实现 `validate` 钩子，禁止 BOM 子项中出现重复的物料，并确保数量大于 0。
  2. **前端联动：** 编写 Client Script，当 `P-BOM Item` 中的数量发生变化时，实时计算并更新表头的“总项数”字段。
  3. **调试练习：** 主动制造一个重复物料错误，观察 `frappe.throw` 的提示效果。
- **验收：** 保存重复物料或负数数量时系统报错；录入子项时总数实时更新。

**业务规则翻译：**

在写代码之前，先把业务规则写成普通话：

- 一份 BOM 里，同一个子项物料只能出现一次。
- 每一行子项物料的数量必须大于 0。
- 表头总项数等于所有明细数量之和。

把这三句话写清楚，再去看代码就容易很多。

**操作提示：**

1. 后端代码写在 `P-BOM` 生成的 Python Controller 文件里。
2. 改完 Python 后，如果页面没有反应，执行 `bench restart` 或重启开发服务。
3. Client Script 可以通过搜索 `Client Script` 创建，也可以写入 App 中对应的 JS 文件，课堂初学阶段推荐先用界面创建。

**检查方法：**

- 在同一份 BOM 中添加两行 `MAT-SCREW-001`，保存时应该报错。
- 将某一行数量改成 `0` 或 `-1`，保存时应该报错。
- 将数量分别设为 `4`、`1`、`1`，表头总项数应显示 `6`。

## 🧪 实验三：安全边界与流程 (Permissions & Workflow)

**目标：** 实现符合企业规范的权限控制与审批流。

- **任务：**
  1. 创建 `Engineering User`、`Engineering Manager`、`Quality User` 三个角色。
  2. 配置 `P-BOM` 权限：工程师可读写草稿，只有工程经理能提交。
  3. 配置 `P-ECO` Workflow：草稿 -> 审核中 -> 已发布。
  4. 设置：ECO 进入“已发布”状态后，整个文档变为只读。
  5. 使用至少两个测试账号验证权限差异。
- **验收：** 非经理用户无法提交 BOM；ECO 发布后无法被修改；质量人员只能查看目标数据。

**先理解三个角色：**

| 角色 | 能做什么 | 不能做什么 |
| --- | --- | --- |
| Engineering User | 创建物料、维护 BOM 草稿、发起 ECO | 提交 BOM、发布 ECO |
| Engineering Manager | 审核 BOM、发布 ECO | 不应该绕过流程直接改已发布数据 |
| Quality User | 查看已审核结果 | 修改 BOM 或 ECO |

**操作入口：**

1. 搜索 `Role` 创建角色。
2. 搜索 `User` 创建测试用户，并分配角色。
3. 搜索 `Role Permission Manager` 配置 DocType 权限。
4. 搜索 `Workflow` 创建 ECO 流程。

**检查方法：**

- 用普通工程师账号登录，确认看不到 Submit 或无法提交。
- 用工程经理账号登录，确认可以完成审批动作。
- ECO 进入已发布后，尝试修改变更原因，应无法保存。

## 🧪 实验四：进阶逻辑——ECO 驱动的 BOM 升版

**目标：** 实现跨单据的复杂业务联动。

- **任务：**
  1. 在 `P-ECO` 的 `on_submit` 中编写 Python 逻辑。
  2. 当 ECO 发布时，自动找到关联的 `P-BOM`。
  3. 将关联 BOM 的状态设为“已审核”。
  4. 将关联 BOM 的版本号更新为 ECO 中定义的“目标版本”。
  5. 记录一条 Comment 或日志，说明哪个 ECO 触发了升版。
- **验收：** 提交并发布 ECO 后，对应 BOM 的版本号和状态自动更新；升版来源可追溯。

**业务故事：**

工程师发现原来的电机供应商停产，需要把产品中的电机换成新型号。为了留下审计记录，不能直接改 BOM，而是先发起 ECO。经理批准后，系统自动把 BOM 从 `A.0` 升级到 `A.1`。

**实现思路：**

1. 用户提交 `P-ECO`。
2. `P-ECO.on_submit` 被触发。
3. 代码读取 ECO 上的 `bom` 字段，找到关联的 `P-BOM`。
4. 将 BOM 的 `version` 改成 ECO 的 `target_version`。
5. 将 BOM 的 `status` 改成 `已审核`。
6. 保存 BOM。

**常见错误：**

- `on_submit` 不触发：检查 `P-ECO` 是否勾选 `Is Submittable`。
- 找不到 BOM：检查 ECO 的 Link 字段 Options 是否填写 `P-BOM`。
- 保存时报权限错误：课堂阶段可以先确认当前用户权限，后续再讨论后台流程是否需要受控使用 `ignore_permissions`。

## 🧪 实验五：数据洞察 (Query Report)

**目标：** 掌握 Frappe 报表开发与业务数据分析。

- **任务：**
  1. 创建一个 `Query Report`。
  2. 编写 SQL，输出已审核 BOM 的成品物料、版本、子项物料和数量。
  3. 配置过滤器：按成品物料筛选。
  4. 验证报表导出 Excel。
  5. 讨论 Query Report 中使用 SQL 时的权限与参数绑定风险。
- **验收：** 报表正确显示数据；过滤器生效；支持导出 Excel。

**报表目标：**

让业务人员回答这个问题：每个已审核 BOM 由哪些子项组成？

示例输出：

| BOM | 成品物料 | 版本 | 子项物料 | 数量 |
| --- | --- | --- | --- | --- |
| BOM-FG-MOTOR-001-A0 | FG-MOTOR-001 | A.1 | MAT-SCREW-001 | 4 |
| BOM-FG-MOTOR-001-A0 | FG-MOTOR-001 | A.1 | MAT-COVER-001 | 1 |
| BOM-FG-MOTOR-001-A0 | FG-MOTOR-001 | A.1 | MAT-MOTOR-001 | 1 |

**操作入口：**

1. 搜索 `Report`。
2. 新建 Report，类型选择 Query Report。
3. Reference DocType 选择 `P-BOM`。
4. 粘贴 SQL。
5. 添加过滤器 `item`，类型选择 Link，Options 填 `P-Item`。

**初学者提醒：**

SQL 中表名通常是 `tab` 加 DocType 名，例如 `tabP-BOM`。如果 DocType 名中有空格或短横线，要用反引号包起来。

## 🧪 实验六：开放能力 (REST API)

**目标：** 掌握 Frappe REST API 与 Token 认证。

- **任务：**
  1. 在用户账号中生成 `API Key` 和 `API Secret`。
  2. 使用 Postman、Insomnia 或 curl 设置 `Authorization: token [API Key]:[API Secret]`。
  3. 调用 `/api/resource/P-Item` 获取物料列表。
  4. 通过 API 新建一条测试物料。
  5. 尝试使用无权限用户 Token 调用同一接口，观察返回结果。
- **验收：** API 调用返回 200 OK；后台能看到 API 写入的新物料；能解释 401/403 的区别。

**先理解 API 调用：**

浏览器页面是人操作系统，API 是系统操作系统。Postman 或 curl 就像一个“外部系统”，它带着身份凭证请求 Frappe。

**最小 curl 示例：**

```bash
curl -X GET "http://test.localhost:8000/api/resource/P-Item" \
  -H "Authorization: token API_KEY:API_SECRET"
```

**状态码小抄：**

| 状态码 | 含义 | 常见原因 |
| --- | --- | --- |
| 200 | 成功 | 请求和权限都正确 |
| 401 | 未认证 | Token 格式错误或 Key/Secret 错误 |
| 403 | 无权限 | 用户已认证，但角色权限不足 |
| 404 | 找不到 | URL 或 DocType 名写错 |
| 500 | 服务端错误 | 后端代码或数据异常 |

## 🧪 实验七：测试、Fixtures 与交付

**目标：** 将课堂成果整理成可迁移、可验证、可交付的工程资产。

- **任务：**
  1. 为 `P-BOM` 重复物料校验编写一个单元测试。
  2. 运行 `bench run-tests` 验证核心逻辑。
  3. 在 `hooks.py` 中配置 fixtures，导出角色、工作流、Custom Field 和 Property Setter。
  4. 执行 `bench export-fixtures`，检查生成的 JSON 文件。
  5. 执行 `bench migrate` 和 `bench clear-cache`，验证迁移与刷新流程。
  6. 对照 `FINAL_DELIVERY_CHECKLIST.md` 完成最终验收。
- **验收：** 测试可运行；fixtures 可导出；项目能按交付清单完整演示。

**为什么初学者也要知道测试和 fixtures？**

因为企业系统不是“我电脑上能点通一次”就算完成。你需要证明：

- 换一个环境还能跑。
- 关键规则不会被改坏。
- 角色、权限、字段、工作流不会只存在于某个人的本地数据库里。

**操作提示：**

- 单元测试先覆盖最重要的一条规则：BOM 明细不能重复物料。
- fixtures 先导出角色、工作流、字段配置，不要一开始追求完美。
- 每次导出后都检查 JSON 文件是否进入 Git。

---

## 🏆 综合验收清单
- [ ] DocType 结构清晰，包含 P-Item, P-BOM, P-BOM Item, P-ECO。
- [ ] 权限矩阵配置完整，区分普通工程师与经理角色。
- [ ] ECO 发布能触发关联 BOM 的升版逻辑。
- [ ] 报表过滤器生效。
- [ ] API 能够通过 Token 认证成功访问。
- [ ] 关键配置已进入 fixtures。
- [ ] 至少一个核心单元测试通过。
