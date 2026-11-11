---
name: succession-reporting
description: "Use Reporting to inspect analytics dashboards, evaluate metrics and datasets, and build, run, and review report definitions and their frozen report runs."
---

# Reporting

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/reporting.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `live`
- Category: `surface`
- Risk: `medium`

## Surface Metadata

Routes:
- `/data/reporting`
- `/data/reporting/reports`
- `/data/reporting/reports/builder`

Primary MCP tools:
- `query_metrics`
- `manage_report_definition`
- `manage_dashboard`

## Codex Handoff Rules

- Prefer MCP tool-returned full URLs (`url`, `route`, `library_url`,
  `source_url`, `crm_url`) in final answers so users can open them directly
  in Codex Browser.
- Use Browser automatically for app-visible create, update, and verification
  workflows when Browser is available in the Codex thread: after an MCP write,
  open the returned URL to confirm the change rendered, and use it to walk the
  user through UI-only steps this guide marks human-only.
- Never enter credentials, payment details, or approval decisions in Browser
  on the user's behalf; hand those steps back to the user.
- Ask concise follow-up questions when required context is missing; do not make
  the user know internal trigger, route, or table shapes.

## Workspace Routing

1. On the first Succession interaction, call `ping` without context.
2. If it returns `principal_type=workspace`, use Workspace-scoped tools
   normally. `client_id` is optional and cannot target another Workspace.
3. If it returns `principal_type=agency` (agency operators only), use
   Agency-scoped tools without `client_id`. Before client work, call
   `manage_managed_workspace` with `action="resolve_workspace"` and
   `params.query` set to the client name or UUID, then pass the returned
   `client_id` on every Workspace-scoped call, including
   `get_workspace_context`, `list_skills`, and `run_skill`.
4. Check `succession_context.workspace_name` in every result before relying
   on it. There is no persistent Workspace switch.
5. Tools take `action`, `params`, optional `client_id`, and `confirm`.
   Writes return a dry-run plan until they are called again with
   `confirm=true` after the operator approves. Read-only `read_*` aliases
   need no confirmation.



## Purpose

Reporting is the analytics home under Data: a metric and dataset catalog for
evaluating GTM performance, per-workspace dashboards built from that catalog,
and reusable Report Definitions that produce frozen Report Runs on demand.

## When To Use

Use this when the user asks for KPIs or performance numbers, wants to see or
edit a dashboard, wants to build or save a report, wants to run a report or
check on a past run, or wants data exported as csv/xlsx/pptx.

## Core Concepts

- `query_metrics` is strictly read-only: `catalog` and `list_datasets` describe
  what's available; `evaluate` and `evaluate_dataset` compute values for the
  caller's workspace and an optional period/scope.
- Dataset row reads (`evaluate_dataset`) are capped at 50 rows in the tool
  result even when the underlying result set is larger (up to 1000) — treat
  `truncated` and `total_count` as authoritative, not the row count you see.
- `manage_dashboard` owns per-workspace dashboard widget layouts (`kpi`,
  `table`, `trend`, `bar`, `stacked_bar`, `donut`), each validated against the
  metric/dataset catalog before it saves. `slug='default'` is the primary
  dashboard; other slugs are named saved dashboards.
- `manage_report_definition` owns reusable Report Definitions: a title, scope,
  period, and either V1 scalar `metric_keys` or V2 dataset `sections` (or
  both). Definitions are mutable (draft/active/archived).
- Report Runs, produced by the `run` action, are immutable once queued — never
  try to edit a run; create a new run (or a new definition) instead.
- Recipients on a definition (`recipient_user_ids`) must be current workspace
  members; campaign scope ids are validated against the workspace's campaign
  snapshots before a definition saves.

## Available Actions

- Inspect the metric and dataset catalog.
- Evaluate metrics or dataset rows for a period and scope.
- Read, create, rename, or delete workspace dashboards, and update their
  widgets.
- List, get, create, update, and archive report definitions.
- Run a report definition and list its past runs.

## MCP Tools And Actions

- Use `query_metrics` for read-only catalog and evaluation access: `catalog`,
  `list_datasets`, `evaluate` (scalar metric values), `evaluate_dataset` (row
  data, capped at 50 rows in the tool result), and `get_dashboard` (resolve the
  active dashboard without going through `manage_dashboard`).
- Use `manage_dashboard` to read or edit workspace dashboards: `get`, `list`,
  `update` (supply `expected_updated_at` from a prior `get` — it's an
  optimistic lock, and a mismatch returns a `conflict` error with the current
  timestamp), `create`, `rename`, `reset`, and `delete`.
- Use `manage_report_definition` to manage reusable report definitions and
  trigger runs: `list`, `get`, `create`, `update`, `archive`, `run` (queues an
  async `report.run.requested` job and returns a `run_id`), and `list_runs`
  (poll for `status` and the resulting `artifact_id`).

## Workspace Skills To Discover

Look for the generate-report skill, and any pipeline-review or
performance-summary skills that turn metric evidence into a narrative.

## URLs To Return

Return `/data/reporting` for the dashboard, `/data/reporting/reports` for the
report definition list, and `/data/reporting/reports/builder` for creating or
editing a report definition.

## Approval Rules

Reading metrics, datasets, and dashboards is safe. Creating or updating a
report definition or dashboard is workspace-visible — confirm scope, period,
and recipients before saving, especially when `recipient_user_ids` will
receive the output. Running a report only queues a job and does not require
approval.

## Artifact Rules

A completed Report Run produces a durable Library artifact (`artifact_id` on
the run row) — return and reference that URL rather than re-pasting raw
dataset rows into chat.

## Common Workflows

- Build a report: check `query_metrics` `catalog`/`list_datasets` for the
  right keys, create a definition with `manage_report_definition`
  (`action: "create"`), `run` it, and poll `list_runs` until the artifact is
  ready.
- Check a dashboard: call `manage_dashboard` `get` (or `query_metrics`
  `get_dashboard` for a quicker read) and return `/data/reporting`.

## Failure Modes

- `evaluate_dataset` silently looks complete at 50 rows — always check
  `truncated`/`total_count` before summarizing coverage.
- A `run_id` means the run was queued, not that it finished; poll `list_runs`.
- Report Runs are immutable — a "fix" always means a new run, never an edit.
- Unknown metric/dataset/column/filter/sort keys return structured errors
  listing the valid keys; use them to self-correct instead of guessing.
- A dashboard `update` conflict means someone else edited it since your last
  read — re-fetch with `get` before retrying.

## Human-Only Exceptions

Humans decide who should receive a scheduled report and approve any
distribution of report data outside the workspace.
