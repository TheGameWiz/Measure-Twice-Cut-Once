# Measure Twice, Cut Once

Articles and companion materials from David Todd's evolving work with AI coding agents, multi-provider collaboration, shared memory, and human-directed software development.

## Published Site

The current landing page is published with GitHub Pages:

**[Measure Twice, Cut Once](https://thegamewiz.github.io/Measure-Twice-Cut-Once/index.html)**

The site is the best starting point. It presents the articles as a numbered reading sequence, explains their historical context, and links to the projects that grew out of the work.

GitHub Pages publishes the contents of [`docs/`](docs/) from the `main` branch. The landing page source is [`docs/index.html`](docs/index.html).

## The Article Series

Read chronologically to follow the development of the workflow:

1. **[Two Tabs and a Log File](Articles/Two-Tabs-and-a-Log-File.md)**
   The simplest practical technique: two agent windows, one shared Markdown record, and a human director coordinating the work.

2. **[Two Agents, One Memory](Articles/Two-Agents-One-Memory.md)**
   The first shared-memory architecture, created to preserve common project context across agents and sessions.

3. **[I Learned a Valuable Lesson. I Don't Think You've Learned Anything.](Articles/I-Learned-a-Valuable-Lesson.md)**
   The longer account of moving from early experiments to a repeatable, director-led development framework.

4. **[Don't Make Me Come Back There](Articles/Dont-Make-Me-Come-Back-There.md)**
   The original case for M-PACT and a durable, provider-neutral record that survives session boundaries.

5. **[Don't Make Me Come Back There, Again!](Articles/Dont-Make-Me-Come-Back-There-Again.md)**
   M-PACT Gen Two, including anchored handoffs, task flow, context recovery, and the later shape of multi-provider collaboration.

The landing page displays the newest article first, but labels every card with its article number so the reading order remains explicit.

## Historical Context

This repository is also a historical record. The articles and their companion files describe the tools, models, agent behavior, and working practices in use when each piece was written.

The ideas and progression remain useful, but specific setup steps, model capabilities, configuration files, and agent behavior may no longer match current systems or the way David works today. Older materials are preserved rather than silently rewritten into current guidance.

For current software and installation information, use the project links below instead of treating the older workflow files as maintained instructions.

## Repository Map

| Path | Purpose |
|------|---------|
| [`docs/index.html`](docs/index.html) | Current GitHub Pages landing page. |
| [`docs/Articles/`](docs/Articles/) | Published HTML versions of the five articles. |
| [`Articles/`](Articles/) | Markdown versions of the article series. |
| [`Workflows/`](Workflows/) | Earlier consensus and agent-collaboration workflow documents. |
| [`Config/`](Config/) | Historical configuration material used with the earlier workflows. |
| [`Examples/`](Examples/) | Worked examples and collaboration records from the original experiments. |
| [`Memory/`](Memory/) | The original shared-agent-memory bootstrap material that preceded M-PACT. |

## Current Projects

### ConflabCode

A VS Code extension for directed multi-agent software work using defined tasks, stages, roles, shared documents, and checkpoints.

[Visit Conflab.ai](https://conflab.ai/conflab-code.html)

### Conflab

A directed multi-agent conversation workspace for running panels across multiple AI providers with persistent summaries, logs, personas, and artifacts.

[Visit Conflab.ai](https://conflab.ai) | [GitHub repository](https://github.com/TheGameWiz/Conflab)

### M-PACT Gen Two

The Multi-Provider Agent Context Toolkit gives compatible local coding agents a shared system for durable rules, task state, handoffs, decisions, and historical context across sessions.

[GitHub repository](https://github.com/TheGameWiz/m-pact) | [User's guide](https://github.com/TheGameWiz/m-pact/blob/main/docs/USER_GUIDE.md) | [Quick reference](https://github.com/TheGameWiz/m-pact/blob/main/docs/USER_GUIDE_QUICK_REFERENCE.md)

## About

David Todd is a longtime game programmer documenting his practical experience directing AI-assisted software work. The material in this repository grew from real development projects and records what worked, what failed, and how the workflow changed over time.

## License

Except where otherwise noted, this repository is licensed under **Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)**.

Copyright (c) 2026 David Todd.

See [LICENSE](LICENSE) and [NOTICE](NOTICE) for details and attribution guidance.
