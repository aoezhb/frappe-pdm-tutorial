# Lecture Notes: Frappe Concepts and Architecture

These notes are written for beginners. Each concept starts from a concrete business example before moving into Frappe terminology. Keep one question in mind while reading: what business problem does this feature help us solve?

## Phase 1: Mindset Shift

### 1. From SQL-Centric to Metadata-Driven

In many traditional ERP projects, the database schema is the starting point. Developers create tables first, then backend models, APIs, frontend pages, and permission logic.

- **Frappe approach:** Frappe uses `DocType` as a metadata layer. A DocType describes fields, field types, naming rules, permissions, UI behavior, and database structure.
- **Technical boundary:** Frappe automates a lot of database and UI work, but it is not magic. For large data migration, complex indexing, cross-database operations, or performance tuning, developers still need to understand MariaDB and sometimes use `frappe.db` or SQL directly.

**Example:**

If you want to add a Specification field to Item master data, traditional systems may require database migration, backend model changes, API updates, frontend form changes, and permission review. In Frappe, you add a `specification` field to the `P-Item` DocType, and the form, list, API, and database column all work around that metadata.

**Beginner mental model:**

Think of a DocType as a system blueprint. You tell Frappe what the business form looks like, and Frappe builds much of the standard system behavior from that blueprint.

### 2. Batteries Included

Frappe includes many capabilities that enterprise systems usually need:

- standard create/read/update/delete forms
- lists and filters
- role permissions
- file attachments
- import and export
- reports
- REST API
- background jobs

**Low-code CRUD:** For standard business documents, Frappe can provide most CRUD UI and API behavior without custom frontend and backend code.

**Important boundary:** Low-code does not mean no-code. It means you write less repetitive code. Your most valuable code should express business rules.

**Example:**

After creating `P-BOM`, you do not need to first write a BOM creation page, edit page, list page, and basic API. Frappe gives you those foundations. You should focus on rules such as "BOM child items cannot repeat" and "when an ECO is released, the BOM version must update."

## Phase 2: Core Building Blocks

### 1. DocType: More Than a Table

A DocType is not just a database table. It is a business object definition.

- Use `Link` fields to reference other DocTypes.
- Use `Table` fields to model parent-child details.
- Avoid very wide DocTypes with too many fields. More than 100 fields can hurt MariaDB query performance and form loading.
- For fields frequently used in filters, sorting, or joins on large datasets, evaluate `db_index`. Do not blindly index low-selectivity fields.

**Example:**

`P-BOM` is the parent document. It represents one BOM. `P-BOM Item` is a child table. It represents the rows inside that BOM.

If a paper BOM has a header and a detail table, Frappe models that as one parent DocType plus one child table DocType.

**Field type cheat sheet:**

| Requirement | Recommended Field Type |
| --- | --- |
| Short text | Data |
| Longer description | Small Text |
| Fixed options | Select |
| Select another business record | Link |
| Upload a file | Attach |
| Multiple child rows inside a parent document | Table |
| Date input | Date |
| Quantity or amount | Float or Currency |

### 2. Backend Logic: Controller and Domain Split

A Controller is the Python class behind a DocType. It can handle events such as validation before saving or actions after submitting.

- Simple validation can live in Controller hooks such as `validate`.
- Complex logic involving multiple documents or systems should be moved into service or domain modules.

**Example:**

Checking that BOM child quantities are greater than 0 can live in `P-BOM.validate`.

But a full ECO release process may involve updating BOM versions, notifying quality users, syncing with MES, and writing audit logs. That should be split into clearer service functions.

**Beginner rule of thumb:**

If the rule affects only the current document and can be explained in one sentence, start with the Controller. If it crosses documents or systems, consider a service module.

## Phase 3: Complex Business Patterns and Safety Boundaries

### 1. Permission Model

Frappe permissions are layered:

- Role Permission controls broad access to a DocType.
- User Permission can restrict records by user-specific scope.
- Perm Level can restrict fields.

**Security boundary:** Standard UI and functions such as `frappe.get_list` participate in permission checks. If you use `frappe.db.sql` or `frappe.db.get_all(..., ignore_permissions=True)`, you must handle authorization yourself.

**Example:**

Engineering users can create draft BOMs but cannot submit them. Engineering managers can submit BOMs. Quality users can only view approved BOMs. This is role-based permission.

If a future system requires Shanghai users to see only Shanghai BOMs and Beijing users to see only Beijing BOMs, that is row-level data access.

**Beginner reminder:**

Permission is not just hiding buttons. A secure system must also protect API, reports, and backend access paths.

### 2. Workflow and BPMN Boundary

Frappe Workflow is well suited for document state transitions, approval steps, and permission locks.

It is not a full BPMN engine. If your process needs complex compensation transactions, high-concurrency simulation, or strict BPMN 2.0 compliance, Frappe Workflow should be treated as a lightweight approval workflow.

**Example ECO flow:**

1. Engineering user creates a draft ECO.
2. Engineering user submits it for review.
3. Engineering manager approves and releases it.
4. After release, the ECO becomes read-only and the linked BOM is revised.

This is a good fit for Frappe Workflow.

## Phase 4: Integration and DevOps

### 1. REST API and Webhooks

Every DocType can expose standard REST API capabilities. This makes Frappe useful as a headless backend for mobile apps, external systems, or integration services.

Webhooks support event-driven integration. For example, when an ECO is released, Frappe can send a JSON payload to an external URL.

**Example:**

An MES system can call `/api/resource/P-Item` to read item data. A chat robot can receive a webhook when an ECO is released.

**Beginner reminder:**

Reports are for humans to read. APIs are for systems to call. Do not use reports as long-term integration contracts.

### 2. Bench Governance

Bench is the command-line tool used to manage Frappe apps, sites, development servers, migrations, and production setup.

- `bench start` is for development.
- `bench setup production` configures a production stack with Nginx, Supervisor, Gunicorn, Redis, and workers.

**Queue isolation awareness:** In multi-site or high-concurrency setups, background jobs run through RQ and Redis. Architects must monitor site-level workload, queue types such as short/default/long, worker counts, and job backlog. Otherwise background tasks may become the next throughput bottleneck.

**Example:**

If one site imports a huge BOM dataset, long-running background jobs may occupy workers and delay email, notification, or report tasks for another site. This is why advanced teams care about queue isolation.

## Phase 5: AI-Era Value Shift

AI can generate Frappe-style code snippets, but it does not understand your factory, approval policy, supplier constraints, or version rules unless you explain them.

- AI can help write validation code.
- You must decide what the validation should mean.
- AI can help write a hook.
- You must decide where the hook belongs and what business risk it controls.

**Custom App isolation principle:** To keep the system upgradeable, never directly modify standard Frappe or ERPNext source code. Use a Custom App, Property Setters, fixtures, and hooks.

**Example:**

You can ask AI to write code that prevents duplicate BOM child items. But you must define what "duplicate" means, whether substitutes count, which field is authoritative, and what message the user should see.

**Final goal of this course:**

You should be able to look at a real business problem and decide which DocTypes to create, how fields should be designed, which rules belong in Controllers, which process should use Workflow, and what data should be exposed through Reports or APIs.
