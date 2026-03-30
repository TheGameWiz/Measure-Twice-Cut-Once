The following should be inserted by the AI into both "Claude.md" and "Agents.md".

### Consensus Workflow & Agent Collaboration Mode (Make this heading appropriate for "Claude.md" and "Agents.md" files)

When the Director says "Start Consensus Workflow" or "Start Agent Collaboration" with a prompt file path:
- **Do not implement.** This is never an instruction to write code.
- Claude Code is **Agent 1**. Codex is **Agent 2**.
- For Consensus Workflow, read `"Consensus-Workflow.md"`. For Agent Collaboration, read `"Agent-Collaboration.md"`.
- Then read the prompt file once.
- Follow those directions explicitly.

### Startup Behavior (Asymmetric by Design)

- **Agent 2 (Codex):** Enter polling mode **immediately**. No analysis, no status output, no turn-ending messages. If the file already shows `!NEXT: Agent 2`, polling exits on its first iteration and Agent 2 acts normally.
- **Agent 1 (Claude Code):** Start in the role-appropriate mode — Director-interaction for CW, or execute first pass for AC. Agent 1 acts when routing hands off to Agent 1; otherwise Agent 1 polls.
- **Startup order is nondeterministic.** Neither agent assumes it started first. All decisions derive from current file state.

### While Polling — Hard Prohibitions

- Do not send completion/status messages that end the turn.
- Do not run implementation steps.
- Do not edit code or docs.
- Do not perform non-essential reads beyond the polling grep.
- After every complete log entry, immediately re-enter continuous polling mode.
- Do not stop polling unless the workflow reaches terminal completion state or the Director explicitly ends it.
- If handoff state is ambiguous, malformed, or missing, remain in polling mode — do not convert ambiguity into turn-ending output.
- Polling output is an internal control signal, not user-facing status.