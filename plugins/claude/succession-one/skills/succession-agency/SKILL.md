---
name: succession-agency
description: "Use Agency to inspect and operate the agency command center — the managed-workspace roster, cross-workspace work/timeline, service and workspace templates, sending infrastructure, deliverability/health/governance rollups, notifications, approvals, runs, and publishing."
---

# Agency

Generated from `docs/surfaces/guides/agency.md`. Do not edit this generated skill by hand.
Update the source guide and run `pnpm --filter succession-connect gen`.

- Status: `partial`
- Category: `surface`
- Risk: `medium`

## Surface Metadata

Routes:
- `/agency`
- `/agency/roster`
- `/agency/queue`
- `/agency/roll-ups`
- `/agency/infrastructure`
- `/agency/reports`
- `/agency/runs`
- `/agency/settings`
- `/agency/approvals`
- `/agency/audit`
- `/agency/publishing`

Primary MCP tools:
- `get_agency_context`
- `manage_managed_workspace`
- `manage_org_task`
- `manage_org_timeline`
- `manage_service_template`
- `manage_workspace_template`
- `manage_sending_infrastructure`
- `query_deliverability`
- `query_org_health`
- `query_org_governance`
- `manage_notification`
- `list_agent_activity`
- `list_approvals`
- `manage_approval`
- `run_skill`
- `manage_artifact`

## Claude Code Handoff Rules

- Prefer MCP tool-returned full URLs (`url`, `route`, `library_url`,
  `source_url`, `crm_url`) in final answers so users can open them directly.
- When the integrated browser is available in the session, use it for
  app-visible work: after an MCP write, open the returned URL to verify the
  change rendered; open the surface routes above to show results, demo flows,
  or complete UI-only steps this guide marks human-only alongside the user.
- Never enter credentials, payment details, or approval decisions in the
  browser on the user's behalf; hand those steps back to the user.
- Workspace and platform skills surface dynamically as
  `/mcp__succession-one__<slug>` slash commands via MCP prompts — prefer those
  (or `list_skills` -> `run_skill`) over re-implementing a skill inline.
- Ask concise follow-up questions when required context is missing; do not make
  the user know internal trigger, route, or table shapes.

## Workspace Routing

1. On the first Succession interaction, call `ping` without context.
2. If it returns `principal_type=workspace`, use Workspace-scoped tools
   normally. Context is optional and cannot target another Workspace.
3. If it returns `principal_type=agency`, use Agency-scoped tools without
   context. Before client work, call `manage_managed_workspace` with
   `action="resolve_workspace"`, then pass the exact returned `context`
   object on every Workspace-scoped call, including `get_workspace_context`,
   `list_skills`, and `run_skill`.
4. Check `succession_context.workspace_name` in every result before relying
   on it. There is no persistent Workspace switch.



<!-- ADR-0063 (parent-organization hierarchy, org-neutral schema) governs the tool family added here. All agency command-center tools are parent-org-scoped: agency_id resolves from the agencies row for the caller's own workspace, and any managed_workspace_id is verified against the agency_managed_workspaces junction before a write applies. -->

## Purpose

Agency is the operating layer for teams managing work across managed
workspaces: the roster and lifecycle of each managed workspace, cross-workspace
tasks and timeline, reusable service/workspace templates, shared sending
infrastructure, deliverability/health/governance rollups, notifications,
approvals, runs, and publishing.

## When To Use

Use this when the user asks about agency-level workload, a specific managed
workspace's status or lifecycle stage, cross-workspace tasks or timeline
notes, applying a service or provisioning a new workspace from a template,
sending-domain/mailbox inventory, deliverability or health trends across the
book, governance/policy drift, agency notifications, managed-workspace
approvals, or publishing work.

## Core Concepts

- Agency context is not the same as workspace context — `get_agency_context`
  establishes the agency scope before any agency-scoped conclusion.
- Agency-scoped tools stay on the Agency home Workspace and reject a Workspace
  `context`. Workspace-scoped tools require an Agency user to pass
  `context.workspace_id` on every call.
- Resolve a client with `manage_managed_workspace` `resolve_workspace` using an
  exact UUID, slug, or case-insensitive name. It returns the context object to
  reuse. There is no stateful switch, per-client key, configuration edit, or
  restart between managed Workspaces.
- A managed workspace can be on the agency's roster (a junction row) without
  ever having a record row yet — `manage_managed_workspace` creates the record
  lazily on first write.
- Cross-workspace work (`manage_org_task`, `manage_org_timeline`) lives at the
  agency/parent-org layer, not inside the managed workspace; a task or
  timeline entry can be scoped to one managed workspace or left org-level.
- Service templates (`manage_service_template`) are agency-authored
  configurable sections; applying one to a managed workspace creates concrete
  outputs (GTM Roadmap items, `manage_org_task` rows, automation installs,
  report definitions, benchmark/alert rows) through one engine shared with the
  web UI. Workspace templates (`manage_workspace_template`) are a different
  object: reusable blueprints for provisioning a brand-new managed workspace
  from scratch.
- Sending infrastructure (domains + mailboxes) is inventoried at the agency
  level and only *assigned* to a managed workspace, not owned by it.
- `query_deliverability`, `query_org_health`, and `query_org_governance` are
  strictly read-only rollups across the agency's managed workspaces — no
  telemetry is recorded for reads.
- `manage_notification` is the consumer/operator surface over the
  workspace-generic notifications table; agency attention rules (e.g.
  deliverability or health-drop monitors) are the first producer, but any
  workspace can also have its own non-agency notifications.

## Available Actions

- Inspect agency context, resolve a managed Workspace, and read its roster
  state, lifecycle stage, economics, and custom fields.
- List, create, and update cross-workspace tasks and checklist templates;
  append and read timeline entries and derived activity.
- List, create, and apply service templates; list, create, and provision from
  workspace templates.
- List and update sending domains/mailboxes and their managed-workspace
  assignment.
- Read deliverability, health-score, and governance (fleet/audit/policy
  drift) rollups across the book.
- List and act on agency notifications (ack, resolve, snooze).
- Review approvals and agent activity across permitted scopes.
- Run agency-specific skills when available.
- Save service reports or publishing drafts as artifacts.

## MCP Tools And Actions

- Use `get_agency_context` before any agency-scoped conclusion.
- Use `manage_managed_workspace` for the roster and per-workspace record:
  `list_roster`, `resolve_workspace` (exact UUID, slug, or name to a verified
  identity plus reusable `context`; ambiguity returns candidates), `get`,
  `update_record` (lifecycle stage, owners, package,
  contract/renewal dates, billing, stakeholders — lazy upsert),
  `list_stage_definitions`/`upsert_stage_definition` (the workspace's own
  lifecycle stages), `economics` (month-to-date usage vs. entitlements),
  `get_settings`/`update_settings`, and `list_custom_fields`/
  `upsert_custom_field`.
- Use `manage_org_task` for the agency work layer: `list`/`create`/`update`
  tasks (kinds: task, checklist_item, deliverable, follow_up; can be assigned
  to an agency user or nudged to a managed workspace/its user),
  `instantiate_checklist` to seed tasks from a checklist template, and
  `list_checklist_templates`/`upsert_checklist_template` to manage the
  templates. Task pushes to Linear are best-effort and non-fatal — a sync
  failure is recorded but does not fail the write.
- Use `manage_org_timeline` for manual notes and derived activity: `append` (a
  manual entry — call, meeting, decision, incident, billing, etc.), `list`
  (manual entries only), and `activity` (manual entries plus derived
  inbox/agent-run/approval activity for one managed workspace, fanned out via
  a read-only rollup).
- Use `manage_service_template` to manage and apply service templates:
  `list_templates`, `upsert_template`, `apply_template` (creates concrete
  Roadmap items, tasks, automations, and report definitions in the target
  managed workspace), and `list_applications` (history of what was applied
  where).
- Use `manage_workspace_template` for reusable workspace blueprints and
  provisioning: `list`, `upsert`, and `provision` (creates a brand-new
  managed workspace plus its junction, record, and onboarding checklist tasks
  — a privileged, higher-trust write).
- Use `manage_sending_infrastructure` for the shared sending inventory:
  `list` (domains with nested mailboxes, worst-first), `upsert_domain`,
  `upsert_mailbox`, and `assign` (set or clear a domain's or mailbox's
  managed-workspace assignment).
- Use `query_deliverability` for read-only bounce/suppression rollups:
  `rollup` (per managed workspace, worst-first) or `workspace` (one
  workspace's daily series). `days` defaults to 7, max 90.
- Use `query_org_health` for read-only health-score rollups: `rollup` (latest
  score + 7-day delta per managed workspace, worst-first) or `workspace`
  (daily series). `days` defaults to 30, max 90.
- Use `query_org_governance` for read-only governance: `fleet` (connector
  connections across the book), `audit` (audit-event feed, filterable by
  workspace/window/limit), and `policies` (policy templates plus per-workspace
  applied-version drift).
- Use `manage_notification` for agency and workspace notifications: `list`
  (status/kind/category/`managed_workspace_id` filters, defaulting to the open
  set — unread, read, snoozed), `ack` (mark read), `resolve` (mark resolved),
  and `snooze` (defer until a future timestamp).
- Use `list_agent_activity` for run history.
- Use `list_approvals` and `manage_approval` for review flows.
- Use `run_skill` for agency-specific repeatable workflows.
- Use `manage_artifact` for reports and deliverables.

## Workspace Skills To Discover

Look for agency-specific reporting, publishing, managed-workspace review,
approval triage, onboarding/offboarding checklist, and service delivery
skills.

## URLs To Return

Return `/agency` for the command center home, `/agency/roster` for the
managed-workspace roster, `/agency/queue` for cross-workspace tasks,
`/agency/roll-ups` for deliverability/health rollups, `/agency/infrastructure`
for sending domains/mailboxes, `/agency/reports` for agency reporting,
`/agency/runs` for run history, `/agency/settings` for agency settings and
templates, `/agency/approvals` for pending approvals, `/agency/audit` for the
governance audit feed, or `/agency/publishing` for publishing work — pick the
one matching the user's next review surface. Tool results also return a
`roadmap_url` or similar direct link where applicable.

## Approval Rules

Agency actions often affect managed workspaces. Confirm target workspace and
approval authority before approving, publishing, or making changes. Higher-
trust writes — provisioning a new managed workspace
(`manage_workspace_template` `provision`) and applying a service template
(`manage_service_template` `apply_template`) — should always be confirmed
with the operator before running, since they create multiple concrete rows
(and, for provisioning, an entirely new workspace) in one call.

## Artifact Rules

Save managed-workspace reports, publishing drafts, run summaries, and approval
packets as artifacts with clear workspace scope.

## Common Workflows

- Agency review: get agency context, list approvals and runs, group by
  workspace, and return the highest-risk next actions.
- Managed-client work: call `resolve_workspace`, repeat the returned `context`
  on every Workspace-scoped call, and confirm the echoed
  `succession_context.workspace_name`. Use `get` without context only for the
  Agency-scoped roster record itself.
- Roster check: `manage_managed_workspace` `list_roster`, resolve the intended
  row, then `get` it for its record, stage definitions, and junction state.
- Book-wide health check: `query_org_health` `rollup` and
  `query_deliverability` `rollup`, surface the worst-first entries, and
  cross-reference `manage_org_task`/`manage_org_timeline` for known context.
- Apply a service: confirm the target managed workspace and template with the
  operator, call `manage_service_template` `apply_template`, and report the
  generated Roadmap/task/report counts.
- Publishing prep: inspect draft artifacts, approval state, and policy
  readiness (`query_org_governance` `policies`) before returning the
  publishing URL.

## Failure Modes

- Operator lacks access to a managed workspace — `managed_workspace_id` is
  re-verified against the agency_managed_workspaces junction on every write.
- `workspace_resolution_ambiguous`: present the returned candidates and ask the
  operator to choose; never select the first same-name Workspace silently.
- Agency and Workspace scopes are mixed: remove context from Agency-scoped
  tools, or add the resolved context to Workspace-scoped tools.
- A valid but unintended managed UUID is still authorized. Treat the echoed
  Workspace name as the final tenant check before follow-up actions.
- Publishing status is assumed from draft state alone.
- A Linear push failure on `manage_org_task` is non-fatal — the task write
  still succeeds; check `metadata.linear_sync_error` rather than assuming the
  whole call failed.
- `manage_workspace_template` `provision` can partially fail; the error
  envelope's `created` field lists which ids (workspace, junction, record)
  were actually created so cleanup or retry can be scoped correctly.

## Human-Only Exceptions

Humans approve managed-workspace-facing publishing, cross-workspace policy
changes, approval outcomes, service-template applications, new-workspace
provisioning, and other service commitments.
