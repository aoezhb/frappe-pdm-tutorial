# Teacher Guide

## 0. Teaching Principles for This Version

This beginner-friendly version does not assume learners already understand ERP, databases, permissions, APIs, or enterprise architecture.

For each new concept, teach in this order:

1. Start with the business scenario.
2. Introduce the Frappe term.
3. Show the UI entry point.
4. Then show configuration or code.

The goal is not to cover the maximum number of concepts. The goal is to help learners complete and explain one full business flow by themselves.

## 1. Suggested Duration

Recommended format: 3-day workshop or 6 half-day sessions. If learners already know ERP or Python, it can be compressed into 2 days.

| Module | Suggested Time | Goal |
| --- | --- | --- |
| Beginner guide and project specification | 90 min | Explain terms, roles, business objects, and sample data |
| Lecture notes | 120 min | Explain DocType, permissions, workflow, report, API |
| Lab 1 | 150 min | Slowly complete DocType modeling and data entry |
| Lab 2 | 120 min | Complete backend validation and client-side interaction |
| Labs 3-4 | 180 min | Complete permissions, workflow, and ECO revision |
| Labs 5-6 | 150 min | Complete reports and API integration |
| Lab 7 and final delivery | 120 min | Complete tests, fixtures, and final checklist |
| Final demo and review | 60 min | Assess using the rubric |

## 2. Teaching Focus

- For beginners, do not start with source code structure. Start with DocTypes and fields.
- Explain business objects before explaining DocTypes.
- Emphasize that Link and Table are the core modeling tools for ERP data.
- In the permission section, always demonstrate the difference between a normal user and a manager.
- In the ECO revision lab, make learners explain why direct BOM editing is risky.
- In reports and APIs, remind learners about permissions, security, and parameter binding.
- End with tests and fixtures to bring the course back to engineering delivery.

## 3. Common Learner Blockers

| Problem | Teaching Response |
| --- | --- |
| Confuses DocType and Document | Use "form template" vs "one actual form" |
| Confuses Link and Table | Link selects one existing record; Table stores multiple child rows |
| Does not know where to configure things | Train the top search bar: DocType, Role, Workflow, Report |
| `bench start` fails | Check Redis, MariaDB, port conflicts, and virtual environment |
| `test.localhost` cannot open | Check hosts, port, and site creation |
| DocType field does not appear | Save DocType, clear cache, refresh browser |
| Controller does not run | Check Python class name, module path, and whether the DocType is submittable |
| Workflow state does not change | Check workflow state field and transition permissions |
| API returns 401 | Check `Authorization: token key:secret` format |
| Query Report SQL fails | Check table names, backticks, and filter parameters |

## 4. Classroom Questions

- `P-Item` is a DocType. What is `MAT-SCREW-001`?
- Why do we model BOM header and BOM lines as two DocTypes?
- Why is a BOM child table better than entering all child items as free text?
- Why should approved BOMs not be directly edited by normal engineering users?
- What is the difference between `frappe.get_list` and `frappe.db.sql` from a permission perspective?
- If an ECO is released with the wrong target version, should users edit it directly or create another change?
- Which configuration must be exported through fixtures?

## 5. Demo Recommendations

- Introduce only one new concept at a time. For example, teach Link before mixing in Fetch From, permissions, and scripts.
- At the end of each lab, ask learners to explain the business object they just created.
- For every lab, show the business scenario before showing the Frappe implementation.
- After writing code, create one failure case in the UI: duplicate item, negative quantity, or unauthorized submit.
- Let learners complete one full flow using a normal user account, then release the ECO using a manager account.

## 6. Fallback Plan

If many learners have environment problems, prepare:

- A backup site completed through Labs 1-3.
- Screenshots of DocType, Workflow, Role Permission, and Query Report configuration.
- A prepared API Key and Secret for API demonstration.
- A sample broken code file for debugging practice.
