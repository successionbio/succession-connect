---
name: succession-inbox
description: "Use Inbox to analyze Succession lead replies, draft responses, route reminders, and connect threads to Engaged records and approvals."
---

# Inbox

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/inbox.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `live`
- Category: `surface`
- Risk: `high`

## Surface Metadata

Routes:
- `/inbox`

Primary MCP tools:
- `query_inbox`
- `manage_inbox`
- `query_crm`
- `run_skill`
- `manage_approval`
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



## Purpose

Inbox handles Succession-owned reply analysis and response drafting for leads.
It connects Inbox thread context, Engaged-record context, reminders,
approvals, and response artifacts.

## When To Use

Use this when the user asks to analyze replies, draft or improve a response,
summarize inbox status, prioritize follow-ups, identify messages needing
attention, or connect a thread to an Engaged record.

When the user says "my inbox", "our inbox", "replies", "lead replies",
"what needs my attention", or similar inside a Succession ONE task, treat that
as the Succession Inbox at `/inbox`, not the operator's personal email account.
Do not use Gmail, Google Workspace mail, or raw mailbox connectors unless the
user explicitly asks for personal email or says to compare against the connected
mailbox.

## Core Concepts

- Drafting is safe; sending is high risk.
- The Inbox source of truth is `query_inbox`, backed by `inbox_threads`.
- Replies should be grounded in the thread, Engaged account/lead context, and
  workspace messaging guidance.
- Reminders and approvals keep live actions visible to the operator.
- `list_conversations` and `get_conversation` are for Maya/agent chats; they
  are not the Inbox queue and should not be used to decide whether lead replies
  need attention.

## Available Actions

- Inspect Succession Inbox threads and reply context.
- Query Engaged records for the lead and company.
- Discover and run inbox reply or objection-handling skills.
- Save response drafts as artifacts or send them to approvals.

## MCP Tools And Actions

- Use `query_inbox` for Inbox queues, especially `tab="needs_reply"` for
  replies that need attention.
- For questions about what a specific reply says or talks about, call
  `query_inbox` with a narrow `query` and `include_latest_message=true` so the
  answer is grounded in Succession Inbox message content, not personal email.
- Use `manage_inbox` for triage-state writes:
  `action="update_thread"` to mark read/unread, clear needs-reply, or
  archive/unarchive a thread; `action="set_reminder"` /
  `action="clear_reminder"` for follow-up reminders (a matching CRM task is
  mirrored best-effort); `action="get_reply_agent_settings"` /
  `action="update_reply_agent_settings"` for reply-agent configuration
  (workspace-level changes require a workspace owner/admin caller).
  `manage_inbox` never sends, drafts, or unsubscribes — those stay on the
  approval path below.
- Use `query_crm` for Engaged person, company, and deal context.
- Use `run_skill` for inbox reply, objection handling, or qualification skills.
- Use `manage_approval` for any send-ready response.
- Use `manage_artifact` for durable drafts or reply packs.

## Workspace Skills To Discover

Look for inbox reply, lead qualification, objection handling, follow-up,
meeting-booking, and account context skills.

## URLs To Return

Return full absolute Succession ONE app URLs so the operator can open them
directly in Codex Browser. Prefer the top-level `route` returned by
`query_inbox` for the current queue and each thread's returned `url` for
specific replies. Do not shorten these to relative paths like
`/inbox?tab=needs_reply` in the final answer.

## Approval Rules

Never send or schedule a reply without platform approval. Drafting, summarizing,
and recommending next actions are safe if clearly labeled.

## Artifact Rules

Save response drafts, objection analyses, thread summaries, and follow-up plans
as artifacts when the user may reuse or review them later.

## Common Workflows

- Draft reply: get thread, query Engaged context, run the best reply skill,
  save the draft, and request approval for any send.
- Triage inbox: call `query_inbox` with `tab="needs_reply"`, inspect returned
  thread summaries, optionally query Engaged records for the highest-priority
  leads, open Browser to the full `route` returned by `query_inbox` for shared
  visibility, and return the actionable Succession Inbox items with full
  thread URLs when useful.
- Draft from inbox thread: select the thread with `query_inbox`, query Engaged
  records for context, run the best reply skill, save the draft, and request
  approval for any send.

## Failure Modes

- Thread lacks enough context to draft accurately.
- `query_inbox` is unavailable or returns an error. Say the Succession Inbox
  read failed and return the full Succession ONE Inbox URL if known; do not
  substitute Gmail as a silent fallback.
- Engaged lead or company is missing or duplicated.
- Connector state prevents send or sync.
- User asks for a reply that conflicts with workspace voice or policy.

## Human-Only Exceptions

Humans approve sends, sensitive claims, unsubscribe handling, legal responses,
and any Engaged-record write created from reply interpretation.
