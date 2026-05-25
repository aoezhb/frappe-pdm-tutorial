# Security Policy

This repository is primarily educational documentation, but it includes examples involving permissions, API tokens, REST calls, and ERP-style business data. Please treat security-related feedback seriously.

## Supported Versions

The latest version on the default branch is supported for documentation corrections and security-related improvements.

## Reporting a Security Issue

Please do not open a public issue for sensitive security concerns.

If you find a problem such as:

- unsafe API token handling in examples,
- misleading permission guidance,
- insecure use of `ignore_permissions=True`,
- SQL examples that could encourage injection risks,
- instructions that may expose local secrets,

please report it privately to the repository maintainer. If no private contact is available yet, open a public issue with a minimal description such as "Security concern in API example" and avoid posting exploit details or real credentials.

## Security Expectations for Examples

- Never commit real API keys, API secrets, passwords, database dumps, or customer data.
- Replace secrets with placeholders such as `API_KEY` and `API_SECRET`.
- Use parameterized SQL patterns where possible.
- Explain permission bypasses clearly when an example uses elevated backend behavior.
- Treat ERP data as sensitive by default.

## Scope

This policy covers documentation and code snippets in this repository. If a runnable sample Frappe app is added later, this policy should be expanded to include dependency updates, vulnerability reporting, and release patching.
