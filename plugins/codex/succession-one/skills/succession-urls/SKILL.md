---
name: succession-urls
description: "Return the right Succession ONE URL after Codex creates, updates, inspects, or prepares work for the user to view live."
---

# URLs And Live Handoffs

Generated from `docs/surfaces/guides/urls.md`. Do not edit this generated skill by hand.
Update the source guide and run `pnpm --filter succession-connect gen`.

- Status: `live`
- Category: `cross_cutting`
- Risk: `low`

## Surface Metadata

Routes:
- `/`
- `/chats`
- `/today`
- `/library/artifacts`
- `/library/:artifactId`
- `/campaigns`
- `/settings/mcp`

Primary MCP tools:
- `get_workspace_context`
- `get_artifact`
- `get_conversation`
- `list_artifacts`
- `search_library`

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

URL handoffs let Codex work through MCP while the user views the live state in
the Codex browser or Succession ONE app.

## When To Use

Use this whenever Codex creates, updates, retrieves, or identifies something the
user should inspect in the product.

## Core Concepts

- Return full absolute URLs, not just surface names or relative routes.
- Prefer URL fields returned by MCP tools (`url`, `route`, `library_url`,
  `source_url`, `crm_url`) over constructing links by hand. `crm_url` is a
  system/field name that survives the "CRM" UI-term retirement (ADR-0072);
  it points at Data → Engaged routes.
- The app is the visual review layer; MCP is the action and data layer.
- Browser access can be limited by signed-in state, so URLs are still useful
  even when Codex cannot drive the page directly.
- Seven top-level surfaces plus Settings (ADR-0072): Chat, Today, Inbox,
  Campaigns, Data, Profile, Library, Settings. Legacy routes below 301/308 to
  their new home; prefer returning the new route directly.

## Available Actions

- Choose the correct route for a surface.
- Return specific artifact or conversation URLs when available.
- Route setup problems to `/settings/mcp`.
- Route queues and approvals to Today or AgentOps.
- Route legacy links to their current home rather than repeating the old path.

## MCP Tools And Actions

- Use `get_artifact` and `list_artifacts` to resolve artifact targets.
- Use `get_conversation` for chat handoffs.
- Use `search_library` when the user names content imprecisely.
- Use `get_workspace_context` before returning workspace-scoped instructions.

## Workspace Skills To Discover

URL handoff usually does not require a skill. Discover a skill only when the URL
is attached to a produced outcome such as a brief, campaign draft, or report.

## URLs To Return

Use full absolute URLs for `/` chat home (new-chat canvas), `/chats` the
conversation history index, `/c/:conversationId` a specific conversation,
`/today` queues, `/library/artifacts` artifact lists, `/library/:artifactId`
artifact details, `/library/agents` agents (no top-level nav entry),
`/campaigns` workbenches, `/inbox` replies, `/data/engaged/companies` Engaged
Companies, `/data/engaged/leads` Engaged Leads, `/data/engaged/deals` Deals,
`/data/companies`, `/data/people`, `/data/tradeshows`, and `/data/signals`
(Discover band), `/data/reporting` dashboards/reports, `/profile` the Full
Profile canvas, `/profile/improve` Profile Improvements, `/profile/insights`
Messaging Insights, `/profile/reply-agent` reply-agent config, `/automations`
automations (a Settings child), `/agentops` runs (no top-level nav entry), and
`/settings/mcp` Codex connection setup. If the app base URL is not known, say
so and return the relative route as a fallback.

**Legacy routes (301/308 redirect):** `/crm/*` → `/data/engaged/*`,
`/intelligence/*` → `/data/*`, `/analytics/*` → `/data/reporting/*`,
`/reply-corpus` → `/profile/insights`, `/settings/reply-agent` →
`/profile/reply-agent`, `/settings/workspace-profile` → `/profile`, `/maya` →
`/chats`, `/agents` → `/library/agents`. The Profile's 18 former section routes
(e.g. `/profile/voice`, `/profile/icp`) are now anchors on the Full Profile
canvas and 308-redirect there. Always return the new route, not the legacy one
— the redirect exists for old links, not as the canonical answer.

## Approval Rules

Returning a URL is safe. Do not imply that a URL means approval or launch
readiness.

## Artifact Rules

When Codex creates an artifact, return the artifact URL and a short summary of
what changed.

## Common Workflows

- Create artifact and open it: save with `manage_artifact`, return the artifact
  URL, and mention the relevant source surface.
- Continue existing work: resolve object or conversation, return URL, and
  summarize what the user will see.

## Failure Modes

- Tool response does not include a direct URL. Use the URLs guide and app base
  URL only when the object route is known; otherwise return the surface URL.
- Browser cannot access signed-in app state.
- User asks for a URL to an object outside token scope.

## Human-Only Exceptions

Humans decide whether to open, review, approve, or publish the linked work.
