---
name: succession-crm
description: "Use Data → Engaged to inspect and update companies, leads, deals, and native CRM configuration for workspace-owned records."
---

# Data — Engaged

Generated from `docs/surfaces/guides/crm.md`. Do not edit this generated skill by hand.
Update the source guide and run `pnpm --filter succession-connect gen`.

- Status: `live`
- Category: `surface`
- Risk: `high`

## Surface Metadata

Routes:
- `/data/engaged`
- `/data/engaged/companies`
- `/data/engaged/leads`
- `/data/engaged/deals`
- `/data/engaged/settings`

Primary MCP tools:
- `query_crm`
- `manage_native_crm`
- `query_nucleus`
- `manage_artifact`
- `run_skill`
- `search_library`

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



<!-- formerly "CRM" (own top-level nav item at /crm/*), renamed and merged into Data → Engaged in ADR-0072 -->

## Purpose

Data → Engaged is the system of record for companies, leads, deals, and native
CRM configuration used by campaigns, inbox, and research workflows. It is the
workspace-owned, RLS-scoped band of the Data surface — distinct from Data →
Discover, which is Nucleus-backed and read-mostly. "CRM" is retired as a UI/
section term; use "Engaged Companies," "Engaged Leads," or "Deals."

## When To Use

Use this when the user asks about accounts, contacts, leads, deals, pipeline,
field configuration, CRM hygiene, or CRM-linked GTM decisions.

## Core Concepts

- Engaged writes affect the workspace record and may sync externally later.
- Engaged data should be reconciled with Data → Discover and library evidence.
- Campaign and inbox actions should resolve to clear Engaged entities.
- Engaged action requests usually span Data plus Automations. If the operator
  asks for an action on a record, infer a CRM action automation instead of
  treating it as a generic configuration change. The `crm_action` automation
  field name is a system name and survives the UI rename.
- Record-action workflows must be verified in Browser because the operator
  needs to see the action on the Engaged record and confirm the destination
  opens with selected-record context.
- Engaged Leads tables show Stage and Category pills and an "Open conversation"
  deep link into Inbox.

## Available Actions

- Query companies, leads, and deals.
- Inspect or update native CRM configuration.
- Configure record actions through the Automations surface when the action
  should appear in a record Actions menu.
- Enrich Engaged context from Data → Discover and library artifacts.
- Run qualification, scoring, or account research skills.
- Save Engaged analyses as artifacts.

## MCP Tools And Actions

- Use `query_crm` for record lookup and reporting.
- Use `manage_native_crm` for native CRM mutations and configuration.
- Use `query_nucleus` for enrichment evidence.
- Use `run_skill` for account research, qualification, or scoring.
- Use `manage_artifact` for briefs, account plans, and exports.

## Workspace Skills To Discover

Look for qualify company, score prospect list, account research, CRM hygiene,
and campaign targeting skills.

## URLs To Return

Return `/data/engaged/companies`, `/data/engaged/leads`, `/data/engaged/deals`,
or `/data/engaged/settings` based on the object the user should review.

## Approval Rules

Read-only analysis is safe. Creating, updating, merging, deleting, or syncing
records is high risk and must be confirmed by the operator and governed by
workspace policy.

## Artifact Rules

Save account briefs, pipeline analyses, prospect lists, and cleanup plans as
artifacts when the output is more than a simple lookup.

## Common Workflows

- Account review: query Engaged company and lead records, enrich with Data →
  Discover, run account research if available, save a brief, and return Data
  plus artifact URLs.
- Pipeline summary: query deals, segment by stage or owner, flag risks, and save
  a shareable report if requested.
- Company action: default to a single-company CRM action when the operator says
  "company action" or names a company deep-dive workflow. Use Automations to
  create a `crm.action.clicked` trigger for `object_key = companies`, destination
  `maya_new_chat` unless the operator specifies otherwise, then open Browser to
  `/data/engaged/companies` and verify the action appears and passes selected
  company context.

## Failure Modes

- Duplicate or stale Engaged records.
- Missing mapping between inbox/campaign entities and Engaged entities.
- Tool scope prevents write or field visibility.
- User asks for a write that should be staged as an approval.
- CRM action automation is created but selected-record context does not reach
  Maya or the destination output.
- The action is configured but not visible in the Engaged record Actions menu.

## Human-Only Exceptions

Humans approve record writes, merges, deletions, external syncs, ownership
changes, and any Engaged update inferred from ambiguous text.
