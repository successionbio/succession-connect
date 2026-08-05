---
name: succession-roadmap
description: "Use Roadmap to inspect and update the GTM Roadmap — planning items across campaigns, content, webinars, events, and milestones, tracked through workspace statuses with review comments."
---

# Roadmap

Generated from `docs/surfaces/guides/roadmap.md`. Do not edit this generated skill by hand.
Update the source guide and run `pnpm --filter succession-connect gen`.

- Status: `live`
- Category: `surface`
- Risk: `low`

## Surface Metadata

Routes:
- `/roadmap`
- `/agency/roadmap`

Primary MCP tools:
- `manage_roadmap`

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
   normally. Context is optional and cannot target another Workspace.
3. If it returns `principal_type=agency`, use Agency-scoped tools without
   context. Before client work, call `manage_managed_workspace` with
   `action="resolve_workspace"`, then pass the exact returned `context`
   object on every Workspace-scoped call, including `get_workspace_context`,
   `list_skills`, and `run_skill`.
4. Check `succession_context.workspace_name` in every result before relying
   on it. There is no persistent Workspace switch.



## Purpose

Roadmap is the GTM planning surface: a board of roadmap items (campaign,
content, webinar, event, milestone, or custom) tracked through
workspace-defined statuses, with comments for review discussion. It is
deliberately a planning object only — it does not create campaigns, service
templates, reports, or automations itself, though a planning item can link to
a Campaign for traceability.

## When To Use

Use this when the user asks to see the GTM roadmap, add or move a planning
item, check what's planned for a period, or leave a review comment on a
roadmap item.

## Core Concepts

- Roadmap items live per workspace with a status pulled from that workspace's
  own status definitions (slug, canonical category, order index) — statuses
  are not a fixed global enum.
- Item types: campaign, content, webinar, event, milestone, custom. Priorities:
  low, medium, high, urgent.
- Comments (`comment`, `client_input`, `review_decision`) thread on an item for
  review discussion; `visibility` (`internal`, `shared`, `client_editable`)
  controls whether an item or comment is agency-internal or shared with the
  managed workspace.
- `manage_roadmap` accepts an optional `managed_workspace_id` on every action.
  Omit it to act on the caller's own workspace roadmap. An agency operator can
  pass a different workspace's id to review or act on a managed workspace's
  roadmap at `/agency/roadmap`; reads for another workspace fan out through a
  cross-workspace rollup, while writes are re-verified against the
  agency-manages-that-workspace relationship before they apply.
- A roadmap item can reference a campaign via `linked_workbench_id` for
  traceability, but the campaign itself is managed on the Campaigns surface,
  not here.

## Available Actions

- List roadmap items and the workspace's status definitions.
- Create, update, and archive roadmap items.
- Move an item between statuses.
- Add a comment or review decision to an item.

## MCP Tools And Actions

- Use `manage_roadmap` for everything on this surface: `list_items`,
  `list_statuses`, `create_item`, `update_item`, `archive_item`,
  `move_status`, and `add_comment`. Pass `managed_workspace_id` only when
  acting on behalf of an agency reviewing or updating a managed workspace's
  roadmap; omit it for the caller's own workspace.

## Workspace Skills To Discover

Look for GTM planning, quarterly roadmap review, or campaign-to-roadmap sync
skills if the workspace has them; none are required to use this surface.

## URLs To Return

Return `/roadmap` for the caller's own workspace roadmap board, and
`/agency/roadmap` when the operator is an agency member reviewing the
cross-workspace rollup. Tool results also include a `roadmap_url` pointing
directly at the affected item.

## Approval Rules

Creating, updating, moving, or commenting on roadmap items is a low-risk
planning action and does not require approval — it does not launch or send
anything.

## Artifact Rules

Roadmap items are lightweight planning rows, not Library artifacts. Do not
duplicate roadmap content into an artifact unless the user explicitly asks for
an exportable roadmap summary.

## Common Workflows

- Check what's planned: `list_statuses`, then `list_items`, group by status,
  and return `/roadmap`.
- Add a planning item: `create_item` with title, item type, priority, and
  target dates, then return the item's `roadmap_url`.
- Move an item: `move_status` with the item's `item_id` and either a
  `status_definition_id` or a `status_slug`.

## Failure Modes

- `move_status` requires either `status_definition_id` or `status_slug` —
  supplying neither returns `invalid_input`.
- Dates must be `YYYY-MM-DD`; malformed dates are rejected before any write.
- Acting on a `managed_workspace_id` the caller's agency does not manage is
  rejected — do not assume access just because the id looks valid.

## Human-Only Exceptions

Humans decide roadmap priorities, target dates, and whether a
`review_decision` comment resolves a review thread.
