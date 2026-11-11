---
name: succession-profile
description: "Use Profile to inspect and improve workspace positioning, voice, ICP, evidence, messaging, and profile readiness."
---

# Profile

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/profile.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `live`
- Category: `surface`
- Risk: `medium`

## Surface Metadata

Routes:
- `/profile`
- `/profile/improve`
- `/profile/insights`
- `/profile/reply-agent`
- `/profile/motions`
- `/profile/motions/new`

Primary MCP tools:
- `get_workspace_profile`
- `update_profile`
- `run_skill`
- `search_library`
- `manage_artifact`

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



<!-- ADR-0072: Profile has three canonical document children. ADR-0075 adds Commercial Motions as a fourth, distinct Profile child destination (an operational collection over Profile parts, not a new source of truth). Also absorbs former Settings pages that were workspace-knowledge-shaped (workspace profile config, reply agent). -->

## Purpose

Profile is the strategic source for workspace positioning, voice, ICP,
messaging, evidence, and readiness that downstream GTM surfaces should reuse.
It has four children: **Full Profile** (`/profile`, a single-page document
canvas with a section rail — the former 18 section routes like
`/profile/voice` and `/profile/icp` are now anchors on this page and
308-redirect here), **Profile Improvements** (`/profile/improve`, the
recommendation queue), **Messaging Insights** (`/profile/insights`, formerly
Reply Corpus), and **Commercial Motions** (`/profile/motions`, the collection
of recommended, Draft, Active, and Archived Commercial Motions, with
`/profile/motions/new` as the guided Commercial Motion Builder). A Commercial
Motion references exactly one Offering, one or more ICPs, and one or more
Motion Personas from the Full Profile, then owns only combination-specific
context — it is an accelerator, not a Campaign requirement, and Campaign
Builder can still start from a bespoke Profile composition instead.
`/profile/reply-agent` (formerly `/settings/reply-agent`) also lives here as
workspace-knowledge-shaped configuration.

## When To Use

Use this when the user asks to inspect profile readiness, update positioning,
audit voice or evidence, improve messaging, review Messaging Insights, or
work with Commercial Motions (list, create, or edit one).

## Core Concepts

- Profile content affects campaigns, inbox replies, skills, and agent behavior.
- Audits should cite evidence and preserve recommended changes.
- Profile updates should be intentional and traceable.
- Prefer the anchor route on the Full Profile canvas (e.g. `/profile#voice`)
  over the legacy standalone route when linking to a specific section; the
  legacy route still 308-redirects but is not the canonical answer.
- The Commercial Motion Builder (`/profile/motions/new`) is deterministic and
  conversational: one decision at a time, real Profile objects as
  accelerators, and an explicit final review. When a required Offering, ICP,
  or Persona doesn't exist yet, the builder creates it as a canonical Profile
  object first, then selects it — it never creates a private duplicate hidden
  inside the Motion.
- Editing a Profile object never silently rewrites an existing review or live
  Campaign; a Campaign Context Snapshot only adopts newer Profile/Motion state
  after explicit Operator review.

## Available Actions

- Retrieve workspace profile state, including the canonical Commercial Profile
  graph and resolved Commercial Motion context.
- Update profile fields when the user explicitly asks.
- Create, update, archive, or draft a Commercial Motion and its buying-group
  Personas, Sources, and Audience Lists.
- Run audit, bootstrap, voice, positioning, evidence, or messaging skills.
- Save recommendations and source material as artifacts.

## MCP Tools And Actions

- Use `get_workspace_profile` for current profile state. Pass
  `projection: "graph"` for the canonical Commercial Profile graph,
  `projection: "motions"` for the Commercial Motions list, or
  `projection: "resolved_context"` with a `commercial_motion_id` for one
  Motion's fully resolved (deterministic, provenance-preserving) context.
- Use `update_profile` for accepted profile edits, including Commercial
  Motion mutations: `action: "mutate"` (edit an existing Motion or another
  canonical Profile object via a typed command), `action:
  "mutate_motion_relation"` (attach/archive a Motion's sources or audience
  lists), and `action: "create_motion_draft"` (the atomic aggregate create
  used by the Motion Builder — Motion + buying-group Personas + Sources +
  Audience Lists in one idempotency-keyed transaction).
- Use `search_library` for supporting evidence.
- Use `run_skill` for audit and profile-building skills.
- Use `manage_artifact` for briefs and recommendation reports.

## Workspace Skills To Discover

Look for bootstrap profile, audit profile, audit profile voice, audit profile
evidence, audit profile positioning, update profile from input, compile
commercial motions, and write messaging skills.

## URLs To Return

Return `/profile` for the Full Profile canvas (append an anchor such as
`#voice` or `#evidence` when the work is specific to one section),
`/profile/improve` for Profile Improvements, `/profile/insights` for Messaging
Insights, `/profile/motions` for the Commercial Motions collection,
`/profile/motions/new` for the Motion Builder, and `/profile/reply-agent` for
reply-agent configuration.

## Approval Rules

Profile analysis is safe. Profile updates should be confirmed by the user unless
they are explicitly requested and narrow. External claims derived from profile
content still need downstream review.

## Artifact Rules

Save profile audits, recommendation sets, messaging guidance, and evidence packs
as artifacts when they should be reviewed or reused.

## Common Workflows

- Audit profile: fetch profile, run the relevant audit skill, save findings,
  and return profile plus artifact URLs.
- Update profile: clarify target fields, gather evidence, apply the smallest
  accepted edit, and summarize the change.
- Draft a Commercial Motion: fetch the Commercial Profile graph, confirm the
  Offering/ICP/Persona selections (creating any missing canonical object
  first), call `update_profile` `action: "create_motion_draft"`, and return
  the new Motion's `/profile/motions/[motionId]` URL for review.

## Failure Modes

- Profile is incomplete or stale.
- Evidence does not support the proposed claim.
- User asks Codex to rewrite strategic positioning without enough context.
- Editing a `user_locked` profile row without `override_locked` is rejected —
  confirm with the user before overriding a locked field.

## Human-Only Exceptions

Humans approve strategic positioning, sensitive claims, final voice guidance,
and broad profile rewrites.
