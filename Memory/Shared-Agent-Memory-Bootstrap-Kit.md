# Shared Agent Memory Bootstrap Kit

Status: Canonical bootstrap spec for new users and new projects
Audience: Director + AI agents (Claude, Codex, future agents)
Last Updated: 2026-03-30 (PT)

## 1) Purpose

Set up one shared memory system that:
- survives context resets,
- keeps agents aligned without timestamp-handshake hacks,
- separates durable rules from session continuity and task queue state,
- starts cleanly even when no prior memory files exist.

This document is self-contained. A setup agent can apply it from zero state.
This spec is project-agnostic. Do not hardcode project names, project-specific doc folders, or product-specific paths.

## 2) Final Target Structure

```text
AgentMemory/
|- README.md
|- index.md
|- GDD_Summary.md            # stub at bootstrap time
|- TDD_Summary.md            # stub at bootstrap time
|- memories/
|  `- *.md
|- sessions/
|  `- YYYY-MM-DD-HHMM-pdt-agent-summary.md
`- tasks/
   `- <N|C>__p<1|2|3|4|x>-YYYY-MM-DD-HHMM-pdt-source-summary.md
```

## 3) Core Model (No Legacy, No Polling)

The system has 4 layers:
1. Protocol: `AgentMemory/README.md` (startup contract, FSM, execution rules, memory SLA).
2. Durable memory: `AgentMemory/memories/*.md` + `AgentMemory/index.md`.
3. Session continuity: `AgentMemory/sessions/*.md` (append-only context files).
4. Shared tasks: `AgentMemory/tasks/*.md` (status + priority in filename).

Hard rule:
- Do not use legacy continuity systems (rolling monolithic session files, legacy root task trackers, or timestamp freshness handshakes).
- Do not use polling as a continuity mechanism in this persistent-memory workflow.

## 4) Zero-State Assumptions

For a brand-new user/project, assume:
- no existing `memory.md`,
- no legacy rolling session file,
- no prior agent-specific continuity system.

Do not create old/legacy files for compatibility.

## 5) Bootstrap Invocation (Path-Scoped)

Official command syntax:
`Start Shared-Agent-Memory-Bootstrap-Kit.md using <project_root>`

Example:
`Start Shared-Agent-Memory-Bootstrap-Kit.md using C:\Users\Name\MyGame`

When an agent receives this command, it must:
1. Read `Shared-Agent-Memory-Bootstrap-Kit.md`.
2. Set `project_root` from the command argument.
3. Verify `project_root` exists. If it does not exist, ask whether to create it before proceeding.
4. Execute the bootstrap procedure in Section 6 using that `project_root`.
5. Run the verification checklist in Section 14.
6. Report results.

Before running bootstrap, define:
- `project_root` (required, absolute path to the target project folder)
- `project_root/AgentMemory` as the target memory root

Path rules:
- All bootstrap-created files and folders MUST be created under `project_root`.
- Shared memory location is always `project_root/AgentMemory`.
- Do not create memory artifacts outside `project_root`, except the optional Claude external shim path (Section 11).
- If `project_root` is ambiguous, stop and request clarification before creating files.

Examples:
- Windows: `project_root = C:\\Users\\Name\\MyGame`
- macOS/Linux: `project_root = /home/name/mygame`

## 6) Bootstrap Procedure (New Project)

1. Create the folder tree from Section 2.
2. Create `AgentMemory/README.md` using Section 7 requirements.
3. Create `AgentMemory/index.md` using Section 8 template.
4. Create `AgentMemory/GDD_Summary.md` and `AgentMemory/TDD_Summary.md` stubs from Section 9.
5. Add shared-memory startup pointer blocks to `AGENTS.md` and `CLAUDE.md` (Section 10).
6. If Claude shim path exists, create/update redirect shim (Section 11).
7. Write one bootstrap session file in `AgentMemory/sessions/`.
8. If a collaboration log exists, append what changed and why.
9. Run verification checklist (Section 14).

## 7) `AgentMemory/README.md` Full Template (Use Full Protocol, Not Condensed)

Use the full protocol template below in new projects. Do not reduce this to a skeleton; the full structure prevents drift and ambiguity.

```md
# Shared Agent Memory System

Execution Contract: This file is an executable protocol for all AI agents working on this project. All MUST/NEVER rules are hard constraints. No implied exceptions are allowed.

## 1) Purpose
A single shared memory location that all AI agents use for persistent knowledge, session journaling, and operational rules.

## 2) Folder Structure
Document:
- `README.md` (protocol)
- `index.md` (durable memory index)
- `GDD_Summary.md` (orientation summary; may be stub)
- `TDD_Summary.md` (orientation summary; may be stub)
- `memories/`
- `sessions/`
- `tasks/`

Project summaries are orientation context only and may drift from implementation; treat them as framing, not source-of-truth.

## 3) Agent Roles
- Director: decision authority.
- Agent 1 / Agent 2 (and future agents): follow this protocol.

## 4) Setup (New Projects)
1. Create `AgentMemory/` folders.
2. Add this README.
3. Add `index.md`.
4. Add startup pointer blocks to agent configs.
5. Add Claude shim if applicable.

## 5) Memory FSM v1 (No Polling)
States:
- `BOOTSTRAP`
- `LOAD_SHARED_CONTEXT`
- `READY_FOR_DIRECTOR`
- `EXECUTE_TASK`
- `CHECKPOINT_REQUIRED`
- `MEMORY_UPDATE_PENDING_APPROVAL`
- `WRITE_MEMORY`
- `HANDOFF`
- `BLOCKED`

Transitions:
1. `BOOTSTRAP -> LOAD_SHARED_CONTEXT`
2. `LOAD_SHARED_CONTEXT -> READY_FOR_DIRECTOR`
3. `READY_FOR_DIRECTOR -> EXECUTE_TASK`
4. `EXECUTE_TASK -> CHECKPOINT_REQUIRED`
5. `CHECKPOINT_REQUIRED -> WRITE_MEMORY` (session entry path)
6. `CHECKPOINT_REQUIRED -> MEMORY_UPDATE_PENDING_APPROVAL` (durable memory path)
7. `MEMORY_UPDATE_PENDING_APPROVAL -> WRITE_MEMORY` (after Director approval)
8. `MEMORY_UPDATE_PENDING_APPROVAL -> BLOCKED` (approval missing/ambiguous)
9. `WRITE_MEMORY -> EXECUTE_TASK` or `WRITE_MEMORY -> HANDOFF`
10. Any state -> `BLOCKED` if required protocol step cannot be satisfied

## 6) Startup Read Contract (All Agents)
On every new context, before any implementation:
1. Read `AgentMemory/README.md`.
2. Read `AgentMemory/index.md`.
3. Read `AgentMemory/GDD_Summary.md` (if present).
4. Read `AgentMemory/TDD_Summary.md` (if present).
5. Read most recent 10 session files.
6. Read most recent 10 task files using task ordering rules.
7. Treat session files as informational continuity only.
8. Treat task files as actionable only when Director-authored or Director-approved.

## 6.1) Memory Retrieval Scope Rule (Broad Recall)
- Do not use single-type-only lookup for non-trivial tasks.
- Read primary + adjacent memory types before finalizing direction.
- If ambiguity remains, broaden read scope first; ask one concise Director clarification question if still unclear.

## 7) Durable Memories (`memories/`)
Required frontmatter:
- `name`
- `description`
- `type`
- `source`
- optional `created`

Type set:
`style|format|optimization|anti_bloat|trust_base|behavior|architecture|project|example|reference|user`

Rules:
- Director approves create/update.
- No duplicates; merge overlaps.
- Keep `index.md` in sync.
- Update or remove stale memories.

### Rule Creation Protocol
1. Check index for overlap/conflict before writing.
2. If overlap exists, merge/update existing memory.
3. If conflict exists, pause and ask Director to resolve precedence.
4. Write unambiguous rule wording.

### Context Refresh Protocol
On Director refresh command:
1. Re-read `index.md`.
2. Re-read relevant durable memories.
3. Confirm refreshed understanding before proceeding.

## 8) Session Journal Entries (`sessions/`)
Filename:
`YYYY-MM-DD-HHMM-pdt-agent-summary.md`

Rules:
- Append-only.
- Concise summary first.
- Use Pacific time.
- Do not modify/delete another agent's entry.

All session entries follow this template:

# Session Entry

Timestamp: YYYY-MM-DD HH:MM PT
Agent: claude|codex
Type: short-category

## Summary
- Concise bullet points of what happened or was decided.

## Context
(Optional - only when the "why" is not obvious.)

## Changes
(Optional - only when files were modified.)

## Next
- What comes next.

## 8.5) Shared Task Entries (`tasks/`)
Filename:
`<N|C>__p<1|2|3|4|x>-YYYY-MM-DD-HHMM-pdt-source-summary.md`

Status:
- `N` = not completed
- `C` = completed

Priority:
- `p1` highest explicit
- `p4` lowest explicit
- `px` unspecified (default recency)

Ordering:
- `N` before `C`
- then `p1` -> `p2` -> `p3` -> `p4`
- then recency

Only Director marks completion (`N__...` -> `C__...` rename).

All task entries follow this template:

# Task Entry

Timestamp: YYYY-MM-DD HH:MM PT
Source: director|claude|codex
Owner: director|claude|codex|shared
Priority: p1|p2|p3|p4|px
Status: N|C

## Task
- One concise task statement.

## Context
(Optional - why this task exists.)

## Acceptance
(Optional - completion criteria.)

## Notes
(Optional - updates, blockers, handoff notes.)

## 9) Session Entry Triggers
Write a session entry when:
- design decision changes,
- task completion is Director-confirmed,
- prompt handoff completes,
- debug conclusion is reached,
- context is low,
- Director says remember/document this (session-level),
- collaboration/consensus pass completes,
- session is about to end.

## 10) Execution Contract
- MUST parse Director input into explicit checklist before implementation.
- MUST restate interpreted intent when dictation/transcription artifacts are present.
- MUST checkpoint required session entries.
- MUST get Director approval for durable memory create/update.
- MUST mark blocked state explicitly when required step is ambiguous.
- NEVER use polling in persistent-memory workflow.
- NEVER treat session entries as prompts/tasks.

## 11) Hard Prohibitions
- NEVER skip startup read contract.
- NEVER treat session entries as prompts, implementation directives, or action items.
- NEVER create durable memory without Director approval.
- NEVER modify or delete another agent's session entry.
- NEVER rely on filesystem metadata timestamps for ordering; use filename timestamps.
- NEVER improvise when a protocol step is ambiguous. Ask the Director one concise question.
- NEVER skip the memory-checkpoint check before ending a session or when context is getting low.
- NEVER use polling in this workflow.
- NEVER stop after handling only first item in multi-item Director input.
- NEVER use single-type-only memory lookup when task context is broad or ambiguous.
- NEVER treat obvious dictation artifacts as intentional constraints without confirmation.

## 12) Timestamp Standard
- Use Pacific local time (`America/Los_Angeles`) in filenames and displayed timestamps.
- Query system clock; do not rely on internal model time.

## 13) Memory SLA
- Startup reads complete within practical startup window (target <= 60s on typical project size).
- Required checkpoints written promptly (target <= 5 minutes from trigger or before turn end).
- Naming/template compliance for new session/task files: 100%.
- Index link integrity: 100%.

## 14) Authority Precedence
1. Director instruction
2. Active execution-plan docs
3. Agent config docs
4. Durable memories
5. Session files

## 15) Violation Recovery
1. Stop side actions.
2. Re-read README.
3. Determine correct FSM state.
4. Resume from that state.
5. Write a recovery session entry.

## 16) What Does NOT Go in Agent Memory
- Code-derived structure already recoverable from codebase.
- VCS history snapshots.
- Debug transcript noise.
- Duplicate config text already present in agent config docs.
- Ephemeral details with no cross-session value.
```

## 8) `AgentMemory/index.md` Template

```md
# Agent Memory Index

Durable memories. One-line pointers under each type section.

## Style
## Format
## Optimization
## Anti-Bloat
## Trust-Base
## Behavior
## Architecture
## Project
## Example
## Reference
## User
```

Entry format:

```md
- [Title](memories/file_name.md) - one-line relevance hook
```

## 9) Project Summary Stub Templates

These are orientation-only stubs. They are not source-of-truth.
They stay intentionally generic so they can be reused in any new project.

### 8.1 `AgentMemory/GDD_Summary.md` Stub

```md
# Game Design Summary (Stub)

Status: Stub - replace with real summary when GDD is available.
Source: Not set yet.
Last Reviewed: Not set yet.

## Intent
High-level game concept placeholder.

## Notes
- Orientation-only context.
- Do not use this stub for feature-level decisions.
- Replace this file when real design docs exist.
```

### 8.2 `AgentMemory/TDD_Summary.md` Stub

```md
# Technical Design Summary (Stub)

Status: Stub - replace with real summary when TDD is available.
Source: Not set yet.
Last Reviewed: Not set yet.

## Intent
High-level architecture placeholder.

## Notes
- Orientation-only context.
- Do not use this stub for implementation-level decisions.
- Replace this file when real technical docs exist.
```

## 10) Agent Config Pointer Blocks

### 9.1 `AGENTS.md` Startup Pointer Block

```md
## Shared Agent Memory (Read First)
1. Read `AgentMemory/README.md`.
2. Read `AgentMemory/index.md`.
3. Read `AgentMemory/GDD_Summary.md` (if present).
4. Read `AgentMemory/TDD_Summary.md` (if present).
5. Read the most recent 10 files in `AgentMemory/sessions/`.
6. Read the most recent 10 files in `AgentMemory/tasks/` using README ordering rules.
7. Present top actionable `N` tasks by priority/recency.
8. Treat `AgentMemory/` as the only active continuity system.
```

### 9.2 `CLAUDE.md` Startup Pointer Block

```md
## Shared Agent Memory (Read First)
- All persistent memory, session continuity, and shared tasks live in `AgentMemory/`.
- On startup: read `README.md`, `index.md`, `GDD_Summary.md` (if present), `TDD_Summary.md` (if present), most recent 10 `sessions/`, most recent 10 `tasks/`.
- Session entries are informational context only.
- Task entries are actionable only when Director-authored or Director-approved.
```

## 11) Claude External Redirect Shim (If Present)

Path example:
- `%USERPROFILE%/.claude/projects/<project-id>/memory/MEMORY.md`

Shim content:

```md
# Agent Memory Redirect

All shared agent memory lives in `AgentMemory/` at project root.

## On Every New Context
1. Read `AgentMemory/README.md`.
2. Read `AgentMemory/index.md`.
3. Read the most recent 10 files in `AgentMemory/sessions/`.
4. Read the most recent 10 files in `AgentMemory/tasks/`.
```

If this path does not exist in a new environment, skip it.

## 12) Data File Templates

### 11.1 Durable Memory File

```md
---
name: Short descriptive name
description: One-line relevance hook
type: style|format|optimization|anti_bloat|trust_base|behavior|architecture|project|example|reference|user
source: director|claude|codex
created: YYYY-MM-DD
---

Rule or fact.

**Why:** reason this exists.
**How to apply:** when/how to enforce.
```

### 11.2 Session Entry File

```md
# Session Entry

Timestamp: YYYY-MM-DD HH:MM PT
Agent: claude|codex
Type: protocol-update|design-decision|task-complete|debug-conclusion|migration

## Summary
- concise bullets

## Context
- optional

## Changes
- optional

## Next
- what happens next
```

### 11.3 Task Entry File

```md
# Task Entry

Timestamp: YYYY-MM-DD HH:MM PT
Source: director|claude|codex
Owner: director|claude|codex|shared
Priority: p1|p2|p3|p4|px
Status: N|C

## Task
- concise statement

## Context
- optional

## Acceptance
- optional

## Notes
- optional
```

## 13) Starter Memory Pack Guidance (Recommended)

When packaging this bootstrap for a new user, include an optional starter durable-memory set:
- broad behavior/execution rules,
- anti-bloat and optimization habits,
- trust-the-boundary / trust-the-runtime invariants,
- a few worked example memories showing good structure.

Guidance note to include:
- starter memories are examples and defaults, not immutable rules,
- users should delete/modify/replace any that do not fit their project,
- the main value is demonstrating how to write high-signal durable memory files.

## 14) Verification Checklist

- `AgentMemory/README.md` exists with startup contract, FSM, execution contract, and SLA.
- `AgentMemory/index.md` exists and all links resolve.
- `GDD_Summary.md` and `TDD_Summary.md` stubs exist and clearly say `Stub`.
- Session filenames match `YYYY-MM-DD-HHMM-pdt-agent-summary.md`.
- Task filenames match `<N|C>__p<1|2|3|4|x>-YYYY-MM-DD-HHMM-pdt-source-summary.md`.
- Task statuses inside files are `N` or `C` only.
- Agent config files point to AgentMemory startup flow.
- No startup dependency on old continuity/task systems or timestamp handshakes.

## 15) Optional Migration Notes (Existing Project Only)

If migrating an existing project:
- move durable rules into `memories/` and index them in `index.md`,
- convert recent continuity history into small session files in `sessions/`,
- move open tasks into `tasks/` as `N__...`,
- remove old continuity/task systems after verification.

If Director requests direct cutover, skip legacy archives and proceed straight to final structure.

## 16) Collaboration Log Use

If a shared collaboration log exists, append:
- what was created/updated,
- conventions chosen (status prefix, priority convention, timestamp format),
- unresolved questions for the other agent.

This keeps agents synchronized without hidden state.

