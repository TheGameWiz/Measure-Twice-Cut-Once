# Measure Twice, Cut Once

**A practitioner's framework for two-agent AI collaboration**

This repository contains workflow documents, configuration files, examples, and articles from a five-week experiment in using two AI coding agents — Claude Code (Anthropic) and Codex (OpenAI) — to collaboratively develop, implement, and verify software.

The core idea: two agents with different strengths review the same specification and catch different problems. A human director makes all decisions. Only when all three agree does implementation begin.

---

## Read the Article

**[`Articles/I-Learned-a-Valuable-Lesson.md`](Articles/I-Learned-a-Valuable-Lesson.md)**

The full account — from zero AI experience to a working multi-agent consensus workflow. Covers the learning curve, what AI gets wrong about code, context management, workflow documents as programs, the two-agent approach, and practical setup instructions.

---

## What's Here

### Workflows

| File | Description |
|------|-------------|
| [Consensus-Workflow.md](Workflows/Consensus-Workflow.md) | Director-supervised workflow with Design, Implement, and Test phases. Both agents communicate through a shared prompt file with structured handoffs. |
| [Agent-Collaboration.md](Workflows/Agent-Collaboration.md) | Autonomous mode — two agents iterate on a spec without Director involvement until they reach consensus. |

### Configuration

| File | Description |
|------|-------------|
| [Config-Insert.md](Config/Config-Insert.md) | Configuration block to paste into both CLAUDE.md and AGENTS.md. Defines startup behavior, role assignments, and polling rules. |

### Examples

| File | Description |
|------|-------------|
| [WordLabelManager-Refactor.md](Examples/WordLabelManager-Refactor.md) | Complete consensus workflow example — eliminated 15 arrays and 5 legacy collections from a broker class. Includes full iteration log. |
| [RenderSortSystem-DirtyArrayElim.md](Examples/RenderSortSystem-DirtyArrayElim.md) | Agent collaboration into consensus workflow — identified ~605 KB of waste in oversized cache arrays. |
| [Shared-Memory-Collaboration-Log.md](Examples/Shared-Memory-Collaboration-Log.md) | The design conversation between Claude and Codex that produced the shared memory system. A real-world example of the two-tabs technique described in the companion article. |

### Memory System

| File | Description |
|------|-------------|
| [Shared-Agent-Memory-Bootstrap-Kit.md](Memory/Shared-Agent-Memory-Bootstrap-Kit.md) | Hand it to an agent and it builds the entire shared memory system. Project-agnostic. |

### Companion Articles

| File | Description |
|------|-------------|
| [Two-Agents-One-Memory.md](Articles/Two-Agents-One-Memory.md) | Unifying two agents' separate memory systems into a single shared architecture. |
| [Two-Tabs-and-a-Log-File.md](Articles/Two-Tabs-and-a-Log-File.md) | The simplest way to start — one prompt, one shared file, no infrastructure needed. |

---

## Quick Start

1. Place `Consensus-Workflow.md` and `Agent-Collaboration.md` in your project root.
2. Add the contents of `Config-Insert.md` to both your `CLAUDE.md` and `AGENTS.md` files.
3. Create a prompt file describing what you want to build.
4. In both agent windows, type: `Start Consensus Workflow using: [path to your prompt file]`
5. Or, for autonomous spec refinement: `Start Agent Collaboration using: [path to your prompt file]`

See [Getting Started](Articles/I-Learned-a-Valuable-Lesson.md#section-10-getting-started) in the main article for full setup instructions.

---

## About

Built by David Todd — 50+ years of game programming, 5 weeks of AI. These workflows were developed and tested on a Unity 6.2 project but are not Unity-specific. They should work with any development environment and language.

The workflow documents are provided as-is. They instruct AI agents to execute bash commands within your project directory. Review them before use. See the [Important Notice](Articles/I-Learned-a-Valuable-Lesson.md#important-notice) for full details.

---

## License

Except where otherwise noted, this repository is licensed under
**Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)**.

Copyright (c) 2026 David Todd.

See [LICENSE](LICENSE) and [NOTICE](NOTICE) for details and attribution guidance.

