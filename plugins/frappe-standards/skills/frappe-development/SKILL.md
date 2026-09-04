---
name: frappe-development
description: Use automatically for all Frappe Framework and ERPNext development tasks — bug fixing, debugging, root-cause analysis, DocTypes, custom fields, workflows, roles, permissions, hooks.py, Client Scripts, Server Scripts, reports, print formats, custom apps, integrations, migrations, database issues, performance issues, security reviews, code reviews, and ERPNext customizations. Enforces standard-before-customization, permission-safe, upgrade-safe development practices and a mandatory plan-first workflow for any repository change.
---

# Frappe & ERPNext Development Standards

## When to use this skill

Apply this skill for any Frappe Framework or ERPNext task: new DocTypes or fields, Client Script or Server Script changes, hooks.py additions, workflow or permission configuration, report or query development, REST API integration, performance tuning, migrations, and bug investigation / root-cause analysis (RCA). It applies whether the work is a new feature, a customization, a refactor, or a debugging session.

It does not need to be invoked for purely informational questions that require no repository change (e.g. "what does `frappe.get_cached_doc` do").

## Standard-before-customization rule

Before writing any custom code, confirm that standard Frappe/ERPNext functionality cannot already achieve the requirement:

- DocType configuration (mandatory fields, fetch fields, link filters, naming series, states) before a script.
- Workflow configuration before a custom state machine in a Server Script.
- Standard permission rules, user permissions, and document sharing before a custom permission query condition.
- Existing standard reports, report filters, and Query Reports before a bespoke report.
- Custom Fields and Property Setters before modifying a core DocType's JSON.
- Fixtures for exporting configuration before hand-written data migrations.

Only recommend customization when the standard mechanism is confirmed insufficient, and state explicitly why.

## Field addition and modification requests

For any request to add a new field or modify an existing field, follow these rules.

### 1. Standard/core DocType

If the requirement is for a standard/core Frappe or ERPNext DocType (shipped by Frappe, ERPNext, or another installed app — not owned by the team's own custom app):

- Do not modify the core DocType directly.
- Use **Customize Form** to add or modify fields.
- Prefer **Custom Field** records or other supported customization mechanisms, backed by a Property Setter for any property change.
- Do not edit the original DocType JSON inside the Frappe or ERPNext core application.
- Ensure the customization is upgrade-safe.
- Consider whether the field should be added through fixtures when it must be deployed across environments.
- Validate field permissions, mandatory status, read-only behavior, default values, options, and dependencies.

### 2. Custom/own DocType

If the requirement is for a custom DocType owned and versioned by the team's own app (`custom: 0` in its own `doctype.json`):

- Use the DocType editor / **Edit DocType** to add or modify fields.
- Define the field in the custom DocType's own field configuration.
- Follow proper field naming, field type, label, options, and permission conventions.
- Do not create unnecessary Custom Field records for fields that belong directly to the custom DocType.
- Ensure the field is included correctly in the custom application's source or migration process where applicable.

### 3. Before adding a field, determine

- Whether the DocType is a core DocType or a custom DocType.
- Whether the field already exists.
- Whether an existing standard field can be reused instead.
- Whether the field should be a Custom Field or a native field on the custom DocType.
- Whether the field affects workflows, permissions, reports, scripts, print formats, integrations, or existing business logic.

### 4. Deployment and upgrade considerations

- Core DocType customizations must be exportable and deployable.
- Use fixtures or another supported deployment mechanism when required.
- Avoid manual database-only changes.
- Validate the customization on a test site before production deployment.
- Document the field's purpose, type, options, and affected functionality.

### 5. Required Plan Mode disclosure

Before implementing any field addition or modification, the plan must explicitly state:

- The DocType type: core or custom.
- The recommended method: Customize Form, Custom Field, or Edit DocType.
- The affected field properties.
- The deployment and rollback approach.

Do not implement the field change until the user has approved the proposed method.

## Frappe / ERPNext development standards

- Use standard Frappe APIs and framework conventions (`frappe.get_doc`, `frappe.get_cached_doc`, `frappe.db.get_value`, `frappe.db.get_list`/`get_all`, `frappe.new_doc`) rather than ad-hoc data access.
- Use Client Scripts only for client-side behavior (form UX, field visibility, client-side validation feedback); do not put business logic that affects data integrity there.
- Use Server Scripts only where permitted by the site's configuration and appropriate for the change; prefer a proper custom app for anything non-trivial or that needs version control and code review.
- Use `hooks.py` for supported extension points (`doc_events`, `scheduler_events`, `permission_query_conditions`, `override_whitelisted_methods`, `fixtures`, etc.) instead of monkey-patching core modules.
- Expose server-side entry points via `@frappe.whitelist()` methods with explicit intent (`allow_guest` only when truly required); never expose an internal helper directly to the client without input validation.
- Validate all user-provided input server-side, even if it is also validated client-side.
- Respect role permissions, user permissions, sharing, and document-level permissions. Never bypass permission checks (`ignore_permissions=True`, `frappe.db.sql` writes that skip controller logic) without an explicit, approved reason recorded in the plan.
- Prefer the document API (`doc.save()`, `doc.submit()`, `doc.db_set()` where appropriate) over direct SQL writes so controller hooks, validation, and permissions still run.
- Avoid raw SQL; when it is genuinely required (e.g. a reporting query that the ORM cannot express efficiently), use parameterized queries (`frappe.db.sql(query, values)`) — never string-formatted/interpolated SQL.
- Avoid unnecessary `frappe.db.commit()` calls; let the request/job lifecycle manage transactions.
- Do not log or return sensitive data (passwords, tokens, personal data beyond what the caller is permitted to see) in error messages, reports, or API responses.
- Avoid hardcoding company-, site-, or tenant-specific values; use Custom Fields, Singles/Settings DocTypes, or site config instead.
- Keep business logic in the appropriate layer (controller/server-side for integrity-critical logic; client-side only for UX).
- Follow existing naming conventions in the target app for DocTypes, fields, methods, and files.
- Add validation for mandatory fields, valid state transitions, and business rules in `validate`/`before_submit`/`on_submit` as appropriate.
- Consider performance for list views, reports, background jobs, and bulk operations: avoid N+1 queries (batch with `frappe.get_all`/`get_list` instead of looping `frappe.get_doc` per row), add indexes where justified, and paginate large result sets.
- Use background jobs (`frappe.enqueue`) for long-running or resource-intensive operations instead of blocking the request cycle.

## Code quality standards

- Keep changes small and focused; don't bundle unrelated improvements into a fix or feature.
- Reuse existing Frappe utilities and app-local helpers instead of duplicating business logic.
- Use clear, descriptive names consistent with the surrounding code.
- Add comments only where the logic is genuinely non-obvious — don't narrate what the code already says.
- Don't introduce unnecessary dependencies.
- Preserve backward compatibility and the project's existing coding style.
- Don't change files unrelated to the approved scope.
- Add or update tests for important business rules when you touch them.

## Permission and security requirements

- Every new DocType needs an explicit permission matrix (Role Permissions Manager / `permissions` in the DocType JSON) — do not leave a DocType permission-less by omission.
- Use `permission_query_conditions` and `has_permission` hooks for row-level access rules instead of filtering only in the UI.
- Never trust client-supplied role/user context; re-derive it server-side (`frappe.session.user`, `frappe.has_permission`).
- Treat any use of `ignore_permissions=True`, `frappe.db.sql` direct writes, or `flags.ignore_validate` as a flagged risk requiring explicit justification and user approval in the plan.
- Do not commit credentials, API keys, or site-specific secrets into the repository; use `site_config.json` / environment configuration.

### Security review checklist

When reviewing or investigating a change for security impact, check:

- Role permissions and User Permissions.
- Document sharing.
- Workflow transition permissions.
- Server-side validation (never rely on client-side validation alone for security or authorization).
- API access and whitelisted-method exposure.
- Direct database updates that bypass controller/permission logic.
- Sensitive data exposure in responses, logs, reports, or error messages.
- Permission bypasses (`ignore_permissions`, `flags.ignore_validate`, and similar).
- Self-approval conditions — confirm whether allowing a user to approve their own document is intentional.
- Unauthorized document state changes (submit/cancel/workflow transitions reachable by the wrong role).

### Workflow and permission investigation checklist

When investigating a workflow or permission issue specifically:

- Inspect the Workflow configuration, its States, and the roles allowed at each state.
- Inspect transition conditions in detail.
- Check whether required fields can be empty or `None`, and whether that lets a transition through unintentionally.
- Check whether the current user is correctly linked to an Employee record where that link drives approval logic.
- Check whether Department Head or other approver-resolution fields are configured for the record in question.
- Make sure comparisons can't accidentally pass because both sides are `None` (e.g. `approver == current_user` silently succeeding when both are unset).
- Confirm whether self-approval is intentionally allowed or is an oversight.
- Check DocType permissions and User Permissions together — a role can pass DocType-level permission and still be blocked (or wrongly allowed) by a User Permission.
- Validate the actual behavior using test users with the different roles involved, not just Administrator/System Manager.
- Confirm server-side validation can't be bypassed by calling the API directly, skipping whatever the client-side/workflow UI enforces.

## Supported extension points (in order of preference)

1. Standard configuration: DocType settings, Workflow, Role Permissions, Print Formats, Report Builder.
2. Custom Fields, Property Setters, Fixtures.
3. Client Script (client-side only).
4. Server Script (where enabled and appropriate for the change size).
5. Custom DocType controller (Python controller class on a DocType the team's own app owns).
6. `hooks.py` in a custom app (`doc_events`, `scheduler_events`, `override_doctype_class`, `override_whitelisted_methods`, `jinja`, `fixtures`).
7. Whitelisted methods in a custom app for controlled server-side access.
8. A new custom app, for functionality substantial enough to warrant its own package.
9. Modifying Frappe/ERPNext core files directly — never do this unless the user has explicitly approved it as a last resort; it breaks upgrade safety.

### When core modification is genuinely unavoidable

If every option above has been ruled out and core modification is the only path:

- Explain specifically why hooks or a custom app cannot achieve the requirement.
- Identify the exact core files that would need to change.
- Explain the upgrade and merge-conflict risk this creates for future `bench update`s.
- Provide a rollback approach before making the change.

## Debugging and root-cause analysis (RCA) practices

For bugs, unexpected behavior, workflow issues, permission issues, report problems, or system errors, follow this structure and do not skip steps:

1. **Issue** — restate the observed problem.
2. **Expected behavior** — what should happen.
3. **Actual behavior** — what actually happens, with exact error text/traceback if available.
4. **Evidence** — error messages, tracebacks, the affected document, user, role, and reproduction steps.
5. **Investigation performed** — inspect the relevant DocType, workflow, permissions, Client Scripts, Server Scripts, hooks, and controller logic; trace execution flow end to end.
6. **Root cause** — the first point where actual behavior diverges from expected behavior; separate root cause from downstream symptoms.
7. **Recommended fix** — the smallest safe change that addresses the root cause, not just the symptom.
8. **Files or configuration affected**.
9. **Risks and side effects** — including permission, performance, and backward-compatibility impact.
10. **Testing and validation** — how the fix will be verified.
11. **Rollback approach** — how to revert if the fix causes a regression.

Do not propose a fix based only on assumption. If evidence is insufficient (missing traceback, unclear reproduction steps, unknown app version), say so explicitly and ask for what's missing rather than guessing. For non-trivial or ERPNext-specific investigations, delegate to the `erpnext-rca-investigator` agent bundled with this plugin.

### Bug investigation checklist

While gathering evidence and tracing execution flow (step 4-5 above), work through:

- Reproduce the issue where possible.
- Read the complete traceback or error message — don't treat the visible error message as the root cause without inspecting the code it points to.
- Identify the exact function, method, DocType, event, or workflow state involved.
- Trace the execution flow end to end, both client-side and server-side.
- Check permissions and role restrictions along that path.
- Check whether empty or missing values can bypass validation.
- Check whether the issue affects existing records only, new records only, or both.
- Check for side effects on other DocTypes or workflows.
- Consider caching as a possible cause (site cache, cached DocType metadata, browser cache) before assuming the code itself is wrong.
- Consider incorrect deployment or a missed/failed migration as a possible cause, especially if the issue is environment-specific.

## Database and migration standards

When changing DocTypes, fields, fixtures, or other schema-related code:

- Consider the impact on existing production data, not just new records.
- Determine whether a patch or migration is required, and whether it needs to run automatically (`bench migrate`) or as a manual data patch.
- Avoid destructive changes (dropping fields/tables, data-losing migrations) without explicit user approval.
- Avoid direct database table updates unless necessary and clearly justified — prefer Frappe ORM/document APIs so controller and permission logic still runs.
- If raw SQL is required, use parameterized queries, never string-formatted/interpolated SQL.
- Consider indexes and query performance for large datasets.
- Explicitly state whether the change requires `bench migrate` and whether it's safe to run on a live production site or needs a maintenance window.

## Testing and validation requirements

- After any change, run the relevant checks available in the project: `bench run-tests` / `bench --site <site> run-tests --app <app>` for Python tests, linting (`ruff`/`flake8` if configured), and `bench build` if client assets changed.
- For DocType or schema changes, verify with `bench migrate` on a non-production site before recommending it for production, and confirm the migration is reversible or has a documented rollback.
- Manually verify permission changes with the actual target role, not just the Administrator/System Manager account.
- As applicable to the change, validate: Python syntax, JavaScript syntax, unit tests, DocType validation, workflow behavior, permission behavior, API behavior, and browser/client-side behavior. Run `bench clear-cache` when configuration caching could be masking or affecting the result.
- Report failed tests, warnings, and unresolved issues explicitly — never claim a test passed, or a fix is validated, without having actually run or described the validation performed.

## Upgrade-safe customization practices

- Prefer custom apps, hooks, Custom Fields, Property Setters, and fixtures over any change to Frappe/ERPNext core files, since core changes are overwritten or conflict on the next `bench update`.
- When overriding standard behavior, use supported override hooks (`override_whitelisted_methods`, `override_doctype_class`, `doc_events`) rather than patching core source in place.
- Keep customizations exportable via fixtures/migrations so they survive a fresh site setup or a new environment.
- Document any deviation from standard behavior so a future upgrade doesn't silently reintroduce the original behavior without anyone noticing.

## Plan-first workflow (mandatory for repository changes)

For every development, debugging, customization, implementation, or refactoring request:

1. Understand the requirement and the expected outcome.
2. Inspect the relevant repository files, DocTypes, scripts, hooks, workflows, permissions, reports, and configuration before proposing anything.
3. Identify whether standard Frappe/ERPNext functionality already solves it.
4. For any field addition or modification, classify the target DocType as standard/core or custom and explicitly state the DocType type, recommended method (Customize Form, Custom Field, or Edit DocType), affected field properties, and deployment/rollback approach (see "Field addition and modification requests" above) before anything else.
5. For bugs, identify the likely root cause using the RCA structure above.
6. List the files, modules, DocTypes, hooks, or configuration likely to be affected.
7. Explain the proposed approach, its risks, dependencies, backward-compatibility and security considerations, and the validation plan.
8. Clearly separate confirmed findings from assumptions.
9. Present this as a plan and wait for explicit approval (e.g. "approved", "proceed", "go ahead") before modifying any file, running migrations, installing packages, or committing/pushing code.

If the requirement changes materially mid-task, return to planning and present an updated plan before continuing.

## Implementation workflow (after approval)

1. Re-read the approved plan and re-inspect the relevant files before editing.
2. Implement only the approved scope — no unrelated refactoring or scope expansion.
3. Follow official Frappe/ERPNext conventions and reuse existing utilities/patterns.
4. Preserve backward compatibility unless breaking changes were explicitly approved.
5. Apply the security, permission, validation, and performance considerations described above.
6. Update documentation when behavior, configuration, or usage changes.
7. Run available validation: tests, linting, formatting, migration dry-runs.
8. Report failed tests, warnings, limitations, or unresolved issues honestly.
9. Summarize all changed files clearly.
10. Do not commit or push unless explicitly requested.

## Documentation references

Use official Frappe documentation as the primary source of truth for framework behavior and supported APIs. If something can't be verified against these (or the installed app's own code), say so explicitly rather than inventing behavior:

- Frappe Framework Guides: https://docs.frappe.io/framework/user/en/guides
- Frappe Developer API: https://docs.frappe.io/framework/user/en/api
- Frappe Users and Permissions: https://docs.frappe.io/framework/user/en/user-management/users-and-permissions
- Frappe REST API: https://docs.frappe.io/framework/user/en/api/rest
- Frappe Site Creation: https://docs.frappe.io/framework/user/en/tutorial/create-a-site
- Frappe Installation: https://docs.frappe.io/framework/user/en/installation
- Frappe What's Next: https://docs.frappe.io/framework/user/en/tutorial/whats-next
