---
name: succession-skills
description: "Use Skills to discover, run, and manage server-side Succession ONE workspace, agency, and platform skills."
---

# Skills

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/skills.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `live`
- Category: `surface`
- Risk: `medium`

## Surface Metadata

Routes:
- `/library/skills`
- `/settings/admin/skills`

Primary MCP tools:
- `list_skills`
- `run_skill`
- `manage_skill`
- `manage_artifact`
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

Skills are server-side, workspace-scoped task packages. They should remain the
canonical place for repeatable Succession ONE capabilities.

## When To Use

Use this when the user asks what skills exist, wants to run a skill, wants a new
repeatable workflow, or asks why Codex should use a skill instead of raw tools.

## Core Concepts

- Workspace skills live on the platform and are discovered at runtime.
- Local Codex skills are guide wrappers, not duplicates of workspace skills.
- Skills execute with audit, usage metering, permissions, and approval gates.

## Available Actions

- List available skills.
- Run a selected skill with structured payload.
- Manage skill records when the user explicitly asks.
- Save skill outputs as artifacts.

## MCP Tools And Actions

- Use `list_skills` before assuming a skill exists.
- Use `run_skill` for execution.
- Use `manage_skill` for skill CRUD or administrative changes.
- Use `manage_artifact` for durable outputs from skill runs.
- Use `search_library` for source material a skill may need.

## Workspace Skills To Discover

Discover by outcome and surface: profile audit, account research, messaging,
campaign build, prospect scoring, signal summary, inbox reply, Engaged-data
cleanup, and agency-specific workflows.

## URLs To Return

Return `/library/skills` for workspace skill browsing and
`/settings/admin/skills` for admin skill management when available.

## Approval Rules

Running a read-only skill is usually safe. Skills that trigger external systems,
live sends, launches, or broad workspace changes must respect platform approval
requirements.

## Artifact Rules

Persist meaningful skill outputs as artifacts unless the skill already created
one. Return artifact URLs for review.

## Common Workflows

- Run skill: list skills, select the best match, gather missing inputs, run it,
  save or return the result, and expose approvals if created.
- Build skill: clarify repeatable outcome, create or update the server-side
  skill, and preserve docs or examples in the Library.

## Failure Modes

- Skill absent or permission-filtered.
- Payload lacks required structured inputs.
- User asks Codex to reimplement a skill inline.
- Skill creates a pending approval that the user expects to be complete.

## Human-Only Exceptions

Humans approve new high-impact workspace skills, external-action skills, and
changes to shared skill behavior.
