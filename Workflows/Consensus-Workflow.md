# Consensus Workflow

Execution Contract: This file is an executable workflow program for agents. All MUST/NEVER rules are hard constraints. No implied exceptions are allowed.

## 1) Purpose

Director-supervised workflow for Design, Implement, and Test on one shared prompt file.

Roles:
- Agent 1 coordinates with Director and may edit spec during Design only.
- Agent 2 executes/reviews based on current tag and never edits spec.
- Director approves transitions and may stop at any time.

Startup contract (asymmetric by design):
- **Agent 2 ALWAYS enters polling mode immediately on startup.** No analysis, no status output, no turn-ending messages. If the file already shows `!NEXT: Agent 2`, polling exits on its first iteration and Agent 2 acts normally.
- **Agent 1 ALWAYS starts in Director-interaction mode:** read state, analyze, present to Director, wait for approval before writing.
- **Startup order is nondeterministic.** Neither agent assumes it started first or second. Both agents must handle either case correctly. All decisions derive from the current file state, never from startup timing.

Control-flow authority:
- Use only: final last `!NEXT:`, last author, pass number, active tag, and role.
- Timestamps are informational only and MUST NOT drive state transitions.

## 2) Parameters

- `POLL_INTERVAL_SECONDS = 5`
- `POLL_COMMAND_TIMEOUT_SECONDS = 600`

## 3) Prompt File Layout

```text
[Spec content]

--- SPEC DIVIDER: Everything above is the spec. Everything below is the log. ---

[Log entries]
```

Rules:
- Divider MUST exist before first log entry.
- Agent 1 MAY edit spec only in `Design`.
- After first Agent 1 `Implement` entry, spec is frozen permanently.
- Agent 2 MUST NEVER edit spec.

## 4) Log Entry Format (Atomic)

```text
--- Agent N | Pass P | Tag | YYYY-MM-DD HH:MM ---
[entry content]
!NEXT: Agent X
```

Rules:
- `N` in `{1,2}`
- `P` continues global sequence; never resets
- `Tag` in `{Design, Implement, Test}`
- `!NEXT:` MUST be last line
- each entry MUST be one atomic write

Recognized handoffs:
- `!NEXT: Agent 1`
- `!NEXT: Agent 2`
- `!NEXT: done` (terminal)
- `!NEXT: collab-done` (non-terminal startup artifact)

## 5) FSM

States:
- `BOOTSTRAP`
- `WAIT_HANDOFF`
- `AGENT1_ANALYZE_PRESENT`
- `AGENT1_WAIT_APPROVAL`
- `AGENT1_WRITE_PASS`
- `AGENT2_EXECUTE_PASS`
- `TERMINATED`

### 5.1 Terminal Conditions (Exclusive)

Only these terminate:
- final last `!NEXT:` is `!NEXT: done`
- explicit Director stop/end/abort

No other condition is terminal.

### 5.2 Non-Terminal Conditions (Mandatory Loop)

These MUST return to `WAIT_HANDOFF`:
- missing `!NEXT:`
- malformed `!NEXT:`
- no handoff change
- polling timeout
- transient read error

Ambiguous handoff is non-terminal. Agent 2 MUST remain inside polling and MUST NOT convert ambiguity into turn-ending user output.

### 5.3 Priority Order

1. Terminal Conditions
2. Hard Prohibitions
3. State Transitions
4. Role Logic

## 6) Tag Progression Rules

Normal progression:
`Design -> Implement -> Test`

Rules:
- Only Agent 1 selects tag, with explicit Director approval.
- Same-tag continuation is allowed.
- Backward transition requires explicit Director override and MUST include:
  - `Director override: reverting to [Tag]`
- Agent 2 inherits tag from Agent 1 most recent entry and MUST NEVER change tag.

## 7) Startup Routing

On activation (`Start Consensus Workflow using [prompt path]`), both agents:
1. Read this file once.
2. Read prompt file once.
3. Parse divider, last author, pass number, final last `!NEXT:`, last Agent 1 tag, and spec-frozen flag.

Routing:
- final `!NEXT: done` -> `TERMINATED`
- final `!NEXT: Agent 1` -> Agent 1 `AGENT1_ANALYZE_PRESENT`, Agent 2 `WAIT_HANDOFF`
- final `!NEXT: Agent 2` -> Agent 2 `AGENT2_EXECUTE_PASS`, Agent 1 `WAIT_HANDOFF`
- final `!NEXT: collab-done` -> Agent 1 `AGENT1_ANALYZE_PRESENT`, Agent 2 `WAIT_HANDOFF`
- no valid final `!NEXT:`:
  - Agent 1 inserts divider if missing, then `AGENT1_ANALYZE_PRESENT`
  - Agent 2 `WAIT_HANDOFF`

Agent 2 startup behavior:
- If final `!NEXT:` is `!NEXT: Agent 2`: proceed directly to `AGENT2_EXECUTE_PASS`.
- **ALL other cases** (including `!NEXT: Agent 1`, `!NEXT: collab-done`, missing, malformed, or unrecognized tokens): enter polling **immediately**. No analysis, no status messages, no turn-ending output. Polling is Agent 2's first and only startup action when it is not Agent 2's turn.

## 8) Agent 1 Flow

### 8.1 AGENT1_ANALYZE_PRESENT

Agent 1 MUST:
1. Read latest Agent 2 entry (or startup spec/log context).
2. Present findings, resolutions, and recommended next tag/completion to Director.
3. Transition to `AGENT1_WAIT_APPROVAL`.

### 8.2 AGENT1_WAIT_APPROVAL

This is a conversation wait, not file polling.

Rules:
- Agent 1 MUST NOT write pass entries without explicit Director approval in current interaction.
- Agent 1 MAY end a turn in this state to wait for Director reply.

Transitions:
- explicit stop -> `TERMINATED`
- explicit approval -> `AGENT1_WRITE_PASS`
- else remain in `AGENT1_WAIT_APPROVAL`

### 8.3 AGENT1_WRITE_PASS

Pre-write checks:
1. Last entry author is Agent 2 (or empty-log startup).
2. Alternation/parity valid.
3. Selected tag follows Section 6.

Write by tag:
- `Design`: rewrite spec if needed, append atomic entry ending `!NEXT: Agent 2`.
- `Implement`: append atomic entry ending `!NEXT: Agent 2`, set spec frozen.
- `Test`: append atomic entry ending `!NEXT: Agent 2`.

Completion entry rule:
- On Director-approved completion in Test phase, Agent 1 MUST write final Test entry whose first non-empty content line is exactly:
  - `COMPLETION:`
- Final completion entry MUST end `!NEXT: Agent 2`.
- This is still non-terminal until Agent 2 writes `!NEXT: done`.
- If final last `!NEXT:` is already `!NEXT: done`, Agent 1 MUST NOT write a completion entry.

Then transition to `WAIT_HANDOFF`.

## 9) Agent 2 Flow

### 9.1 AGENT2_EXECUTE_PASS

Preconditions:
1. final last `!NEXT:` is `!NEXT: Agent 2`
2. last entry author is Agent 1
3. active tag is Agent 1 most recent tag

Tag actions:
- `Design`: analyze spec/log; write findings and verdict (`needs revision` or `ready`); end `!NEXT: Agent 1`
- `Implement`: execute Agent 1 directive; write changes/decisions/open items; end `!NEXT: Agent 1`
- `Test`: address test findings; write changes; end `!NEXT: Agent 1`

Completion acknowledgment:
- If Agent 1 latest entry starts with `COMPLETION:` as defined in Section 8.3, Agent 2 MUST write one acknowledgment ending `!NEXT: done`, then transition `TERMINATED`.
- Otherwise transition `WAIT_HANDOFF`.
- If final last `!NEXT:` is already `!NEXT: done`, Agent 2 MUST transition `TERMINATED` and MUST NOT write another acknowledgment.

## 10) Single Authoritative Polling Block

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

  if [ "$current" = "!NEXT: done" ]; then
    echo "terminal:done"
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

Polling command output is an internal control signal. Agent 2 MUST NOT relay polling heartbeat/status lines as turn-ending user output while non-terminal.

### 10.1 Post-Polling Routing

When polling exits:
- `handoff:self`:
  - re-read final last `!NEXT:` before acting
  - Agent 1:
    - if final last `!NEXT:` is still `!NEXT: Agent 1` -> `AGENT1_ANALYZE_PRESENT`
    - else -> `WAIT_HANDOFF`
  - Agent 2:
    - if final last `!NEXT:` is still `!NEXT: Agent 2` -> `AGENT2_EXECUTE_PASS`
    - else -> `WAIT_HANDOFF`
- `terminal:done` -> `TERMINATED`

### 10.2 Timeout Rule

Run polling with timeout `POLL_COMMAND_TIMEOUT_SECONDS`.
Timeout is non-terminal and expected; immediately rerun the same polling block.

## 11) Pre-Exit Guard (Mandatory)

Before turn-ending output:
1. Re-read final last `!NEXT:`.
2. If terminal predicate (Section 5.1) is false, output MUST NOT end turn.
3. Return to `WAIT_HANDOFF` and continue polling.

Exception:
- Agent 1 MAY end turn in `AGENT1_ANALYZE_PRESENT` or `AGENT1_WAIT_APPROVAL` for Director interaction.
- Agent 2 has no non-terminal pre-exit exception.

## 12) Output Contract By State

`WAIT_HANDOFF`:
- Agent 2 allowed output: none.
- Agent 1 allowed output: none, or exact heartbeat `waiting (ambiguous handoff)` when explicitly needed for ambiguity signaling.
- Forbidden: completion/final summaries, implementation summaries, unrelated status.

`AGENT1_ANALYZE_PRESENT`:
- Output MUST be Director-facing analysis/recommendations only.

`AGENT1_WAIT_APPROVAL`:
- Output MAY answer Director questions.
- Forbidden: writing passes, polling actions, unrelated edits.

`AGENT1_WRITE_PASS` and `AGENT2_EXECUTE_PASS`:
- Output MUST match active state actions only.

## 13) Violation Recovery

If protocol is violated:
1. Stop side actions.
2. Re-read final last `!NEXT:`.
3. If terminal predicate true -> `TERMINATED`.
4. Else transition to `WAIT_HANDOFF` and resume authoritative polling.

No Director relay required for recovery.

## 14) Director Interrupts

Director input can occur in any state:
- explicit stop/end/abort -> immediate `TERMINATED`
- non-stop guidance:
  - Agent 1 applies in next approved pass and logs `Director input: [summary]` when applicable
  - Agent 2 applies on next valid handoff pass under active tag

## 15) Completion Contract

Only valid completion sequence:
1. Director approves completion in Test phase.
2. Agent 1 writes final Test entry starting with `COMPLETION:` and ending `!NEXT: Agent 2`.
3. Agent 2 detects marker and writes one acknowledgment ending `!NEXT: done`.
4. Agent 2 terminates.
5. Agent 1 detects `!NEXT: done` and terminates.

No shortcut is allowed.

## 16) Hard Prohibitions

Global:
- NEVER invent terminal states beyond Section 5.1.
- NEVER split log writes.
- NEVER violate alternation/parity.
- NEVER let Agent 2 act without final `!NEXT: Agent 2`.
- NEVER stop polling due to timeout, ambiguity, or inactivity.
- NEVER emit turn-ending output while non-terminal, except Agent 1 Director-interaction states in Section 11.

Agent 1:
- NEVER write without explicit Director approval.
- NEVER edit spec outside Design.
- NEVER edit spec after first Implement entry.

Agent 2:
- NEVER edit spec.
- NEVER change tag.
- NEVER produce output or end turn before entering polling at startup (unless final `!NEXT:` is `!NEXT: Agent 2`).
- NEVER emit user-facing output while in `WAIT_HANDOFF` (including heartbeat/status lines), unless responding to an explicit Director interrupt.
- NEVER end a turn from `WAIT_HANDOFF` while non-terminal.
- NEVER skip acknowledgment when completion marker is present.
- NEVER write `!NEXT: done` unless acknowledging a valid `COMPLETION:` entry.
