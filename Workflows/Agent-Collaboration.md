# Agent Collaboration

Execution Contract: This file is an executable workflow program for agents. All MUST/NEVER rules are hard constraints. No implied exceptions are allowed.

## 1) Purpose

Autonomous two-agent spec refinement on one shared prompt file.
No implementation work is allowed.

Roles:
- Agent 1 rewrites spec.
- Agent 2 reviews only and never edits spec.
- Director may interrupt or stop at any time.

Startup contract (asymmetric by design):
- **Agent 2 ALWAYS enters polling mode immediately on startup.** No analysis, no status output, no turn-ending messages. If the file already shows `!NEXT: Agent 2`, polling exits on its first iteration and Agent 2 acts normally.
- **Agent 1 owns first-write authority for fresh runs and odd-pass progression.** Agent 1 executes when routing hands off to Agent 1; otherwise Agent 1 remains in polling wait.
- **Startup order is nondeterministic.** Neither agent assumes it started first or second. Both agents must handle either case correctly. All decisions derive from the current file state, never from startup timing.

Control-flow authority:
- Use only: final last `!NEXT:` line, last log-entry author, pass number, and role.
- Timestamps are informational only and MUST NOT drive state transitions.

## 2) Parameters

- `POLL_INTERVAL_SECONDS = 5`
- `POLL_COMMAND_TIMEOUT_SECONDS = 600`
- `MAX_PASSES = 20`

## 3) Prompt File Layout

```text
[Spec content]

--- SPEC DIVIDER: Everything above is the spec. Everything below is the log. ---

[Log entries]
```

Rules:
- Divider MUST exist before first log entry.
- Agent 1 MAY rewrite spec above divider.
- Agent 2 MUST NEVER edit spec above divider.

## 4) Log Entry Format (Atomic)

```text
--- Agent N | Pass P | Design | YYYY-MM-DD HH:MM ---
[entry content]
!NEXT: Agent X
```

Rules:
- `N` is `1` or `2`.
- `P` starts at `1` and increments by `1`, never resets.
- Tag is always `Design`.
- `!NEXT:` MUST be last line.
- Each entry MUST be a single atomic write.

Recognized handoffs:
- `!NEXT: Agent 1`
- `!NEXT: Agent 2`
- `!NEXT: collab-done` (terminal)
- `!NEXT: done` (non-terminal startup artifact from Consensus Workflow)
- Any other `!NEXT:` token is unrecognized and non-actionable.

## 5) FSM

States:
- `BOOTSTRAP`
- `WAIT_HANDOFF`
- `EXECUTE_PASS`
- `TERMINATED`

### 5.1 Terminal Conditions (Role-Dependent, Exclusive)

Only these terminate:
- Explicit Director stop/end/abort -> both agents terminate.
- Agent 1 terminal predicate: final last `!NEXT:` is `!NEXT: collab-done`.
- Agent 2 terminal predicate: final last `!NEXT:` is `!NEXT: collab-done` AND last author is Agent 2.

When Agent 2 sees `!NEXT: collab-done` authored by Agent 1, Agent 2 is not terminal yet and MUST write acknowledgment first.

### 5.2 Non-Terminal Conditions (Mandatory Loop)

These MUST return to `WAIT_HANDOFF` immediately:
- missing `!NEXT:`
- malformed `!NEXT:`
- no handoff change
- polling timeout
- transient read error

Ambiguous handoff is non-terminal. Both agents MUST remain inside polling; Agent 2 MUST NOT convert ambiguity into turn-ending user output.

### 5.3 Priority Order

1. Terminal Conditions
2. Hard Prohibitions
3. State Transitions
4. Role Logic

## 6) Startup Routing

On activation (`Start Agent Collaboration using [prompt path]`), both agents:
1. Read this file once.
2. Read prompt file once.
3. Parse divider presence, last author, pass number, final last `!NEXT:`.

Routing:
- final `!NEXT: collab-done` and last author Agent 2 -> both `TERMINATED`
- final `!NEXT: collab-done` and last author Agent 1 -> Agent 1 `TERMINATED`, Agent 2 `EXECUTE_PASS` (ack)
- final `!NEXT: Agent 1` -> Agent 1 `EXECUTE_PASS`, Agent 2 `WAIT_HANDOFF`
- final `!NEXT: Agent 2` -> Agent 2 `EXECUTE_PASS`, Agent 1 `WAIT_HANDOFF`
- final `!NEXT: done` -> Agent 1 `EXECUTE_PASS` (treat as fresh start), Agent 2 `WAIT_HANDOFF`
- no valid final `!NEXT:`:
  - Agent 1 inserts divider if missing, then `EXECUTE_PASS` (Pass 1)
  - Agent 2 `WAIT_HANDOFF`

Agent 2 startup behavior:
- If final `!NEXT:` is `!NEXT: Agent 2`: proceed directly to `EXECUTE_PASS`.
- If final `!NEXT:` is `!NEXT: collab-done` and last author is Agent 1: proceed to `EXECUTE_PASS` (ack).
- **ALL other cases** (including `!NEXT: Agent 1`, `!NEXT: done`, missing, malformed, or unrecognized tokens): enter polling **immediately**. No analysis, no status messages, no turn-ending output. Polling is Agent 2's first and only startup action when it is not Agent 2's turn.

## 7) Pass Rules

### 7.1 Common Pre-Write Checks

Before any write:
1. Last entry MUST be from the other agent (or empty-log startup).
2. Parity MUST hold (Agent 1 odd pass numbers, Agent 2 even pass numbers).
3. If checks fail, do not write; transition to `WAIT_HANDOFF`.

### 7.2 Agent 1 Execute Pass

1. Read Agent 2 latest feedback (or initial spec on Pass 1).
2. Rewrite full spec if needed.
3. Write exactly one atomic entry.
4. End with:
   - `!NEXT: Agent 2`, or
   - `!NEXT: collab-done` (final signoff, only after Agent 2 verdict is `ready`).
5. If non-terminal, transition to `WAIT_HANDOFF`.

Pass 1 rule:
- Insert divider first if missing.

Max pass rule:
- If next pass would exceed `MAX_PASSES`, Agent 1 writes a checkpoint entry and waits for Director input.

### 7.3 Agent 2 Execute Pass

Normal pass:
1. Read full prompt file (spec and full log).
2. Analyze issues/ambiguities/edge cases/conflicts/risks.
3. Write one atomic entry with verdict (`needs revision` or `ready`) ending `!NEXT: Agent 1`.
4. Transition to `WAIT_HANDOFF`.

Completion acknowledgment pass:
- If final last `!NEXT:` is `!NEXT: collab-done` and last author is Agent 1, Agent 2 MUST write one acknowledgment entry ending `!NEXT: collab-done`, then transition `TERMINATED`.
- If final last `!NEXT:` is `!NEXT: collab-done` and last author is Agent 2, Agent 2 MUST NOT write another acknowledgment and MUST transition `TERMINATED`.

## 8) Single Authoritative Polling Block

Use this command and no alternate polling logic:

```bash
PROMPT_FILE="{PROMPT_FILE}"
TARGET_NEXT="!NEXT: Agent N"
POLL_INTERVAL="{POLL_INTERVAL_SECONDS}"

# Set to the last known !NEXT: value BEFORE the change you're waiting for:
# - After writing a pass: the !NEXT: you just wrote (e.g., "!NEXT: Agent 2").
# - At startup (no pass written): the value observed during startup routing.
# NEVER grep the file here — the other agent may respond before polling
# starts, making last_seen equal TARGET_NEXT and preventing detection.
last_seen="!NEXT: Agent M"  # The known prior !NEXT: state

while true; do
  current="$(grep -F '!NEXT:' "$PROMPT_FILE" | tail -1 || true)"

  if [ "$current" = "!NEXT: collab-done" ]; then
    echo "terminal:collab-done"
    break
  fi

  if [ -z "$current" ] || ! echo "$current" | grep -qE '^!NEXT: '; then
    echo "waiting (ambiguous handoff)"
    sleep "$POLL_INTERVAL"
    continue
  fi

  if [ "$current" != "$last_seen" ] && [ "$current" = "$TARGET_NEXT" ]; then
    echo "handoff:self"
    break
  fi

  sleep "$POLL_INTERVAL"
done
```

Polling command output is an internal control signal. Agents MUST NOT relay polling heartbeat/status lines as turn-ending user output while non-terminal.

### 8.1 Post-Polling Routing

When polling exits:
- `handoff:self`:
  - re-read final last `!NEXT:` and last author before acting
  - if final last `!NEXT:` is still `!NEXT: Agent N` for self -> `EXECUTE_PASS`
  - otherwise -> `WAIT_HANDOFF`
- `terminal:collab-done`:
  - if role is Agent 1 -> `TERMINATED`
  - if role is Agent 2:
    - re-read last author
    - last author Agent 1 -> `EXECUTE_PASS` (ack), then `TERMINATED`
    - last author Agent 2 -> `TERMINATED` (already acknowledged)

### 8.2 Timeout Rule

Run polling with timeout `POLL_COMMAND_TIMEOUT_SECONDS`.
Timeout is non-terminal and expected; immediately rerun the same polling block.

## 9) Pre-Exit Guard (Mandatory)

Before any turn-ending output:
1. Re-read final last `!NEXT:` and last author.
2. Evaluate role-specific terminal predicate (Section 5.1).
3. If predicate is false, output MUST NOT end turn.
4. Transition immediately to `WAIT_HANDOFF` and continue polling.

Neither agent has a non-terminal pre-exit exception in this workflow.

## 10) Output Contract

While in `WAIT_HANDOFF`:
- Agent 1 allowed output: none.
- Agent 2 allowed output: none.
- Forbidden: heartbeat text, completion/final statements, implementation summaries, status reports, unrelated output.

While in `EXECUTE_PASS`:
- Output MUST be pass-specific work only.

## 11) Violation Recovery

If protocol is violated:
1. Stop new side actions.
2. Re-read final last `!NEXT:` and last author.
3. If role-specific terminal predicate is true -> `TERMINATED`.
4. Else transition to `WAIT_HANDOFF` and resume authoritative polling.

No Director relay is required for recovery.

## 12) Director Interrupts

Director input may occur any time:
- explicit stop/end/abort -> immediate `TERMINATED`
- non-stop guidance -> incorporate in next pass and include `Director input: [summary]` when applicable

## 13) Completion Contract

Only valid autonomous completion sequence:
1. Agent 2 writes verdict `ready` ending `!NEXT: Agent 1`.
2. Agent 1 writes final signoff ending `!NEXT: collab-done`.
3. Agent 1 terminates.
4. Agent 2 detects `collab-done` from Agent 1 and writes one acknowledgment ending `!NEXT: collab-done`.
5. Agent 2 terminates.

No shortcut is allowed.

## 14) Hard Prohibitions

Global:
- NEVER invent terminal states beyond Section 5.1.
- NEVER split log writes.
- NEVER violate alternation/parity.
- NEVER stop polling due to timeout, ambiguity, or inactivity.
- NEVER emit turn-ending output while non-terminal.

Agent 1:
- On empty-log startup, NEVER allow Agent 2 to write Pass 1. Agent 1 owns Pass 1.
- NEVER write `!NEXT: collab-done` unless Agent 2's last verdict was `ready`.

Agent 2:
- NEVER edit spec.
- NEVER produce output or end turn before entering polling at startup (unless final `!NEXT:` is `!NEXT: Agent 2` or ack is required).
- NEVER emit user-facing output while in `WAIT_HANDOFF` (including heartbeat/status lines), unless responding to an explicit Director interrupt.
- NEVER end a turn from `WAIT_HANDOFF` while non-terminal.
- NEVER skip acknowledgment when `collab-done` is authored by Agent 1.
- NEVER write duplicate acknowledgment when `collab-done` is already authored by Agent 2.
