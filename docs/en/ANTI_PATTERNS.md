# Frappe Engineering Anti-Patterns

## 1. Modifying Standard Source Code Directly

Do not directly edit standard Frappe or ERPNext source code. It may look fast in the short term, but it destroys the upgrade path. Prefer Custom Apps, hooks, fixtures, Property Setters, and custom DocTypes.

## 2. Putting All Logic into Controllers

Controllers are good for document lifecycle events, but not for all business complexity. Logic that crosses documents, domains, or systems should be moved into service, domain, or helper modules.

## 3. Overusing `ignore_permissions=True`

`ignore_permissions=True` should only appear in controlled backend flows. When using it, clearly document who triggers it, what data scope it affects, and how it is audited.

## 4. Bypassing Permissions with `frappe.db.sql`

Raw SQL does not automatically behave like standard UI permission paths. Reports, APIs, and background jobs that use SQL must handle data visibility, parameter binding, and SQL injection risks.

## 5. Using Query Reports as Business APIs

Query Reports are for analysis and export. They should not replace stable integration APIs. External systems should use REST API, whitelisted methods, or Webhooks.

## 6. Creating Overly Wide DocTypes

A DocType with more than 100 fields often indicates that the model should be split. Very wide tables affect query performance, form loading, permissions, and long-term maintenance.

## 7. Editing Production Configuration Manually

Changing fields, permissions, or workflows directly in production causes environment drift. Important configuration should live in a Custom App and move through fixtures, migrations, and code review.

## 8. Ignoring Naming and Versioning Rules

ERP document numbers are not decoration. They support audit, traceability, and organizational policy. Items, BOMs, and ECOs should have clear naming and versioning rules.

## 9. Ignoring Tests and Migration

Clicking through the UI once does not mean the system is deliverable. At minimum, cover core validation, permission boundaries, ECO revision logic, and report filters.

## 10. Thinking Low-Code Means No Technical Foundation

Frappe reduces repetitive development, but it does not remove the need to understand databases, permissions, transactions, cache, queues, and deployment. Real productivity comes from knowing when to use the framework and when deeper engineering is required.
