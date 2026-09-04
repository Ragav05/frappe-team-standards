# Frappe Team Standards

A reusable Claude Code plugin that enforces professional **Frappe Framework** and **ERPNext** development standards, customization practices, debugging methods, and root-cause analysis (RCA) workflows for any team working in a Frappe/ERPNext codebase.

## Purpose

Frappe and ERPNext offer many extension points (Custom Fields, Client Scripts, Server Scripts, hooks, workflows, permissions), and it's easy to reach for custom code before checking whether standard functionality already solves the problem, or to make a change that isn't upgrade-safe or permission-safe. This plugin packages a `frappe-development` skill and an `erpnext-rca-investigator` agent so that any Claude Code session working in a Frappe/ERPNext repository:

- Prefers standard Frappe/ERPNext functionality before recommending customization.
- Follows a mandatory **plan-first workflow** for any repository change.
- Applies consistent permission, security, and upgrade-safety practices.
- Investigates bugs with a structured root-cause-analysis method instead of guessing at fixes.

## Repository structure

```
.claude-plugin/
  marketplace.json                 # Marketplace manifest (owner + plugin listing)

plugins/
  frappe-standards/
    .claude-plugin/
      plugin.json                  # Plugin metadata (name, version, description, author)
    skills/
      frappe-development/
        SKILL.md                   # Frappe/ERPNext standards, plan-first workflow, RCA method
    agents/
      erpnext-rca-investigator.md  # Read-only investigation agent for bugs/RCA

README.md
```

## Installation

From within a Claude Code session:

```
/plugin marketplace add https://github.com/Ragav05/frappe-team-standards.git
/plugin install frappe-standards@frappe-team-marketplace
```

Or from the CLI:

```
claude plugin marketplace add https://github.com/Ragav05/frappe-team-standards.git
claude plugin install frappe-standards@frappe-team-marketplace
```

For local development, point the marketplace at your working copy instead of the GitHub URL:

```
claude plugin marketplace add /path/to/frappe-team-standards
```

## How to use the plugin

Once installed, no special invocation is needed for the skill — Claude Code loads the `frappe-development` skill automatically for Frappe/ERPNext development, customization, debugging, or RCA work in the session, based on its description.

For bug investigations specifically, ask Claude to use the RCA agent, e.g.:

> "Use the erpnext-rca-investigator agent to find out why Salary Slips aren't generating for department X."

Claude Code will launch it as a subagent; its findings come back as a structured investigation report, not as file changes.

## How the Plan Mode workflow operates

The skill instructs Claude to treat every development, customization, debugging, or refactoring request as **plan-first**:

1. Claude inspects the relevant DocTypes, scripts, hooks, workflows, permissions, and configuration before proposing anything.
2. Claude checks whether standard Frappe/ERPNext functionality already solves the requirement.
3. For bugs, Claude identifies the likely root cause before proposing a fix.
4. Claude presents a plan: requirement, findings, proposed approach, files likely to change, risks/dependencies, and validation approach — clearly separating confirmed findings from assumptions.
5. Claude does **not** modify files, run migrations, install packages, or commit/push code during this stage.

### How to approve a plan

Reply with an explicit approval such as `approved`, `proceed`, `go ahead`, or `implement this plan`. Claude then re-reads the approved plan, re-inspects the relevant files, and implements only the approved scope. If your requirement changes materially mid-task, Claude returns to planning and presents an updated plan before continuing.

## How the RCA agent should be used

The `erpnext-rca-investigator` agent is scoped to investigation only — it has no file-editing tools, so it cannot modify your repository even if asked to. Use it when you have:

- An error/traceback and need the execution path traced through hooks, controllers, and scripts.
- A workflow or permission failure (a status not transitioning, a role that can't see/edit a document it should).
- A report or query producing incorrect or slow results.
- Any "this used to work, now it doesn't" regression where the cause isn't obvious.

It reports using a fixed structure — Issue, Expected behavior, Actual behavior, Evidence, Investigation performed, Root cause, Recommended fix, Files/configuration affected, Risks and side effects, Testing and validation, Rollback approach — and always ends by stating that no files were modified and that the fix needs your approval before implementation. If evidence is insufficient, it will tell you what's missing instead of guessing.

## Development and contribution guidelines

- Keep `SKILL.md` and the agent definition generic and reusable — do not encode any one team's or user's personal environment, MCP servers, or aliases into this plugin.
- Any change to standards, workflow rules, or the RCA method should be made in `SKILL.md` (source of truth) and mirrored in the agent file where relevant, so the two stay consistent.
- Do not modify Frappe/ERPNext core files as part of this plugin's own tooling, and do not have the agent or skill recommend core modification except as an explicitly user-approved last resort.
- Keep `marketplace.json` and `plugin.json` in sync with the actual plugin contents (name, version, description) whenever the plugin's capabilities change.
- Follow this repository's own plan-first workflow for changes to itself: propose a plan, get approval, then implement.

## Validation instructions

After changing plugin content, validate that the marketplace and plugin still load correctly:

```
claude plugin marketplace add /path/to/frappe-team-standards
claude plugin details frappe-standards@frappe-team-marketplace
```

This confirms `marketplace.json`/`plugin.json` parse correctly and that the skill and agent are picked up in the plugin's component inventory. There is no application code in this repository, so validation is limited to manifest/frontmatter correctness and a manual check that `SKILL.md` and the agent file cover the topics described above.

## Official documentation references

- Frappe Framework Guides: https://docs.frappe.io/framework/user/en/guides
- Frappe Developer API: https://docs.frappe.io/framework/user/en/api
- Frappe Users and Permissions: https://docs.frappe.io/framework/user/en/user-management/users-and-permissions
- Frappe REST API: https://docs.frappe.io/framework/user/en/api/rest
- Frappe Site Creation: https://docs.frappe.io/framework/user/en/tutorial/create-a-site
- Frappe Installation: https://docs.frappe.io/framework/user/en/installation
- Frappe What's Next: https://docs.frappe.io/framework/user/en/tutorial/whats-next
