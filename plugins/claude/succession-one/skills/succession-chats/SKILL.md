---
name: succession-chats
description: "Use Succession ONE chats to inspect conversations, continue agent work, and connect conversation context to artifacts and actions."
---

# Chat

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/chats.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `live`
- Category: `surface`
- Risk: `low`

## Surface Metadata

Routes:
- `/`
- `/chats`
- `/c/:conversationId`

Primary MCP tools:
- `list_conversations`
- `get_conversation`
- `run_skill`
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



<!-- ADR-0072: the /chats + /maya duplicate conversation-history indexes collapsed into one Chat surface. /maya now 308-redirects to /chats. -->

## Purpose

Chat is where users collaborate with Maya and other agents. For Codex, chats
are context and handoff surfaces, not the only way to act. `/` is the home
canvas (starts a new conversation); `/chats` is the one canonical conversation
history index; `/c/:conversationId` opens a specific conversation. `/maya` was
a second, duplicate history index — it now 308-redirects to `/chats`.

## When To Use

Use this when the user asks to continue a conversation, inspect what an agent
said, recover context, or open a live conversation in the app.

## Core Concepts

- A conversation can link to artifacts, approvals, agent runs, and workbench
  records.
- Codex can often act directly through MCP instead of asking another chat agent.
- Chats are useful when the user needs to see the interaction history.

## Available Actions

- List or retrieve conversations.
- Summarize the relevant context and linked outputs.
- Continue work through tools or workspace skills.
- Save useful conversation outputs as artifacts.

## MCP Tools And Actions

- Use `list_conversations` to find the right thread.
- Use `get_conversation` for message-level context.
- Use `run_skill` when the next step is a bounded workspace skill.
- Use `manage_artifact` when the thread produced reusable output.

## Workspace Skills To Discover

Search for skills named by the conversation outcome, such as research, message
writing, campaign build, inbox reply, profile update, or signal summary.

## URLs To Return

Return `/` for a new chat, `/chats` for the conversation history index, and
`/c/:conversationId` when opening an existing conversation.

## Approval Rules

Reading a conversation is safe. Any action proposed by a conversation still
needs the same approval rules as the underlying surface.

## Artifact Rules

Preserve durable outputs from chats as artifacts when they are drafts, briefs,
plans, or reusable source packs.

## Common Workflows

- Resume a thread: list conversations, fetch the target conversation, summarize
  the latest state, and continue with MCP tools or a workspace skill.
- Turn chat output into an artifact: fetch the conversation, extract the final
  draft or plan, save it, and return the artifact URL.

## Failure Modes

- Conversation not found because token scope is wrong.
- User asks Codex to infer hidden context not present in the conversation.
- Chat agent recommendation conflicts with current platform state.

## Human-Only Exceptions

Humans decide whether chat context is sufficient for high-risk actions,
especially sends, launches, and external writes.
