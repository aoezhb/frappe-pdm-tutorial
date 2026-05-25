# Frappe 工程化反模式清单

## 1. 直接修改标准源码

不要直接改 Frappe 或 ERPNext 的标准代码。短期看最快，长期会破坏升级路径。优先使用 Custom App、hooks、fixtures、Property Setter 和自定义 DocType。

## 2. 所有逻辑都塞进 Controller

Controller 适合承接生命周期事件，但不适合承载所有业务复杂度。跨单据、跨领域、可复用的逻辑应拆到 domain/service/helper 模块中。

## 3. 滥用 `ignore_permissions=True`

`ignore_permissions=True` 只应该出现在明确受控的后台流程中。使用它时必须说明调用来源、数据范围和审计方式。

## 4. 用 `frappe.db.sql` 绕过权限体系

原生 SQL 不会自动等价于标准 UI 权限路径。报表、接口和后台任务中使用 SQL 时，需要主动处理用户可见范围、参数绑定和 SQL 注入风险。

## 5. 把 Query Report 当业务接口

Query Report 是面向分析和导出的工具，不应替代正式 API。外部系统集成应优先使用 REST API、白名单方法或 Webhook。

## 6. 单个 DocType 堆积过多字段

超过 100 个字段的 DocType 通常说明模型需要拆分。过宽的表会影响查询、表单加载、权限管理和后续维护。

## 7. 生产环境直接手工改配置

生产环境中临时改字段、权限和工作流，会造成环境漂移。关键配置应进入 Custom App，并通过 fixtures、migrate 和代码审查进入目标环境。

## 8. 不做命名与版本规则

ERP 系统中的编号不是装饰，它承载审计、追溯和组织规则。物料、BOM、ECO 都应有清晰的命名系列与版本策略。

## 9. 忽视测试和迁移

只在 UI 中点通一次，不代表系统可交付。至少要覆盖核心校验、权限边界、ECO 升版逻辑和报表过滤条件。

## 10. 把低代码理解成不用懂底层

Frappe 可以减少重复开发，但不能替代对数据库、权限、事务、缓存、队列和部署结构的理解。真正的效率来自“知道什么时候用框架，什么时候越过框架”。
