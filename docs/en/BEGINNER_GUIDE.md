# Beginner Guide: Terms and Learning Path

This guide is for learners who are new to Frappe, ERP systems, or enterprise application development. You do not need to understand every internal detail at the beginning, but you should become comfortable with the core vocabulary before starting the labs.

## 1. What Are We Building?

We are building a small PDM system.

PDM means Product Data Management. In a manufacturing company, a PDM system helps manage:

- Items, such as screws, motors, covers, and circuit boards.
- BOMs, which define what materials are needed to build a product.
- ECOs, which record engineering changes, such as replacing one motor model with another.

In one sentence: this course is not just about building screens. It is about learning how to turn business rules into a working enterprise system.

## 2. What Is Frappe?

Frappe is a web framework for building business applications. You can think of it as a framework that helps you:

- Generate admin-style forms and lists.
- Store data in a database.
- Manage permissions, workflows, reports, file attachments, imports, exports, and APIs.
- Focus more on business objects and business rules.

In traditional development, you often create database tables first, then backend APIs, then frontend pages. In Frappe, you usually start by defining a DocType. Frappe then generates a lot of standard capability around that definition.

## 3. Ten Essential Terms

### DocType

A DocType is the most important concept in Frappe. It is a business document type or form type.

Examples:

- `P-Item` is the DocType for item master data.
- `P-BOM` is the DocType for bills of materials.
- `P-ECO` is the DocType for engineering change orders.

If you come from databases, a DocType is somewhat like a table. If you come from business operations, a DocType is a kind of form, master data record, or transaction document.

### Document

A Document is one actual record under a DocType.

Examples:

- `P-Item` is a DocType.
- `MAT-001 Screw` is one Document.
- `BOM-001 Motor Assembly BOM` is one Document.

### Field

A Field is one input or display item on a form.

Examples:

- Item Code
- Item Name
- Specification
- Quantity
- Effective Date

### Link

A Link field lets the user select a record from another DocType.

Example: a BOM has a finished item field. It should select from `P-Item`, not allow free text. This avoids messy data such as "screw", "Screw", and "bolt" all meaning different or overlapping things.

### Table

A Table field is used for child rows inside a parent document.

Example: one BOM contains many child lines:

| Child Item | Quantity |
| --- | --- |
| Screw | 4 |
| Cover | 1 |
| Motor | 1 |

Those child rows belong in a Table field.

### Controller

A Controller is a Python class behind a DocType. It is used for server-side business logic such as validation before saving or actions after submitting.

Example: before saving a BOM, check that the same child item does not appear twice.

### Client Script

A Client Script is JavaScript that runs in the browser form.

Example: when the user changes child item quantities, immediately update the total quantity on the form.

### Workflow

A Workflow controls document states and transitions.

Example: an ECO moves from Draft to Under Review to Released. Different roles can perform different actions at each state.

### Report

A Report presents data to users for analysis and export.

Example: list all released BOMs and show each finished item, child item, and quantity.

### API

An API lets other systems call your Frappe system.

Example: an MES system can read item and BOM data from Frappe through REST API endpoints.

## 4. Common Beginner Confusions

| Confusing Pair | Correct Understanding |
| --- | --- |
| DocType vs Document | DocType is the type. Document is one record. |
| Link vs Table | Link selects one record from another DocType. Table stores multiple child rows inside the current document. |
| Controller vs Client Script | Controller runs on the server. Client Script runs in the browser. |
| Role Permission vs Workflow | Role Permission controls who can do what. Workflow controls how document states move. |
| Report vs API | Reports are for people to view. APIs are for systems to call. |

## 5. Minimal Pre-Study List

If the course feels difficult, review these basics:

- Python: variables, functions, classes, lists, dictionaries, exceptions.
- JavaScript: functions, objects, arrays, events.
- SQL: `SELECT`, `WHERE`, `JOIN`.
- Web basics: browser, server, HTTP, JSON.
- ERP basics: master data, transaction documents, approval, permission, report.

## 6. One Example Used Throughout the Course

Suppose we manufacture a small motor assembly:

- Finished item: `FG-MOTOR-001 Small Motor Assembly`
- Child items:
  - `MAT-SCREW-001 Screw`, quantity 4
  - `MAT-COVER-001 Cover`, quantity 1
  - `MAT-MOTOR-001 Motor`, quantity 1
- BOM: `BOM-FG-MOTOR-001-A0`
- ECO: replace the motor model and upgrade the target version from `A.0` to `A.1`

Use this same data throughout the labs. This keeps the course concrete: you are not just learning fields and code, you are implementing a real business change.
