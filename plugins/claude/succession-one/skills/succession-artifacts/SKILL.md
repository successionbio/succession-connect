---
name: succession-artifacts
description: "Use Artifacts to create, retrieve, update, and open durable Succession ONE outputs in the Library."
---

# Artifacts

Generated from `docs/surfaces/guides/artifacts.md`. Do not edit this generated skill by hand.
Update the source guide and run `pnpm --filter succession-connect gen`.

- Status: `live`
- Category: `cross_cutting`
- Risk: `low`

## Surface Metadata

Routes:
- `/library/artifacts`
- `/library/:artifactId`

Primary MCP tools:
- `list_artifacts`
- `get_artifact`
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
   normally. Context is optional and cannot target another Workspace.
3. If it returns `principal_type=agency`, use Agency-scoped tools without
   context. Before client work, call `manage_managed_workspace` with
   `action="resolve_workspace"`, then pass the exact returned `context`
   object on every Workspace-scoped call, including `get_workspace_context`,
   `list_skills`, and `run_skill`.
4. Check `succession_context.workspace_name` in every result before relying
   on it. There is no persistent Workspace switch.



## Purpose

Artifacts are durable outputs that users and agents can revisit, open in the app,
attach to workflows, or reuse as source material.

## When To Use

Use this when the user asks Codex to create a document, draft, analysis, source
pack, report, plan, or anything that should appear in Succession ONE.

## Core Concepts

- Important output should not live only in chat.
- Artifacts need useful titles, types, metadata, and source context.
- Artifact URLs are the best handoff when the user should view work live.

## Available Actions

- List and search artifacts.
- Retrieve artifact content and metadata.
- Create or update an artifact.
- Return artifact URLs for review.

## MCP Tools And Actions

- Use `search_library` before creating duplicate work.
- Use `list_artifacts` for browsing.
- Use `get_artifact` for content retrieval.
- Use `manage_artifact` for create or update operations.

## Workspace Skills To Discover

Look for skills that generate the artifact's content when the user wants a
quality-controlled output, such as messaging, research, profile audit, or
campaign readiness.

## URLs To Return

Return the full absolute artifact URL from the tool response when available.
Artifact detail pages use `/library/:artifactId`; `/library/artifacts` is the
artifact list. Do not invent `/library/artifacts/:artifactId`.

## Approval Rules

Creating drafts is safe. Publishing, external sharing, deleting, or using an
artifact to trigger a live action follows the underlying surface approval rules.

## Artifact Rules

Use artifacts for durable output. Include source notes and avoid secrets,
credentials, or private tokens.

## Common Workflows

- Create and open: generate content, save it with `manage_artifact`, then return
  the artifact URL for live review.
- Update existing: search or retrieve the artifact, apply the requested change,
  save a new version, and summarize what changed.

## Failure Modes

- Duplicate artifact exists.
- Artifact body is missing or stored externally.
- User asks to overwrite content without confirming the target.

## Human-Only Exceptions

Humans approve destructive changes, external sharing, and using artifacts as
send-ready or launch-ready content.
