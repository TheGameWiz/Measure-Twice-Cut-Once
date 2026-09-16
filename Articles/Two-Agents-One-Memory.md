# Two Agents, One Memory

*A shared persistent memory system for multi-agent AI development: and the story of building it*

David Todd · March 2026 · LexiWord Project

---

## A Note from the Author

This is a companion to my longer article, "I Learned a Valuable Lesson. I Don't Think You've Learned Anything," which covers my full experience learning to work with two AI coding agents over five weeks. That article mentions my memory system several times and notes that it "evolved ad hoc and could be more structured." This is the follow-up: what I did about it, why, and how you can set up the same thing.

This article describes a shared persistent memory system, one folder of markdown files that both agents read on startup, so that rules taught once are known everywhere, work completed by one agent is visible to the other, and you stop being the manual relay between two chat windows. The system was itself designed using the simple two-agent collaboration technique described in the companion article "Two Tabs and a Log File", two agents proposing, reviewing, and revising in a shared log, with me directing every step. If you haven't tried getting two agents to collaborate on a problem yet, that article is the place to start. If you've already seen the value of two agents working together and you want to solve the problem of them forgetting everything between sessions, you're in the right place.

I use Claude Code (Anthropic's CLI agent) and Codex (OpenAI's). For readability, I'll just say Claude and Codex from here on. Both operate directly in my codebase: they read files, write files, create folders, and execute commands in the project directory. That's what makes a file-based shared memory system practical. If you're using different tools, the concepts still apply, but the setup mechanics may differ.

I want to be upfront about something. I didn't set out to solve a problem in multi-agent memory architecture. I didn't know that was a problem. I was just working with two AI coding agents on a game, and the manual overhead of keeping them both informed about what was happening became untenable. So I built something that fixed it. After the fact, I looked around to see if everyone else had already done this and I was just late to the party. What I found was a lot of work on single-agent memory, frameworks, plugins, academic papers, and a research paper from this month calling multi-agent memory consistency "the most pressing open challenge." What I didn't find was someone who had published a practical, file-based, cross-vendor shared memory system that you can set up in five minutes with nothing but markdown files.

Maybe it exists and I missed it. I didn't do an exhaustive survey. I'm not an academic and I'm not going to pretend I read all the papers. I had a problem, I solved it in a way that works for me, and I think it might be useful to others. If someone has already done this better, I'd genuinely like to know about it.

You don't need to read the main article first. This one stands on its own. But if you want the full story of how I got here, the learning curve, the two-agent workflow, the consensus process, that's where it lives.

A note on the status of this article. The shared memory system described here has been replaced by M-PACT, the system described in the fourth article of this series, *Don't Make Me Come Back There*. The problem this article describes is real, and the design conversation between Claude and Codex captured here still has value as background. But the solution offered here is no longer the one I use or recommend. If you are trying to set up shared memory for two AI agents today, *Don't Make Me Come Back There* is the article to read. This article is preserved as the original problem statement and the first attempt at a solution.

David Todd
March 2026

---

## The Problem

I use Claude and Codex on the same project: Claude for architecture, design, and code review, Codex for implementation, compilation, and version control. They have genuinely different strengths, and using both gives me better results than either one alone. The main article goes into detail on why.

The problem is that each agent maintains its own separate memory, and neither knows what the other knows.

Claude Code stores memory in a hidden folder: a MEMORY.md index file plus individual topic files for things it's learned about my project. Codex stores session state in a large rolling markdown file plus a separate task tracker. Two completely independent systems, different formats, different locations, different conventions.

The consequences showed up fast:

The same rules existed in both agents' configuration files with slightly different wording. Code style, performance mandates, save compatibility policy: all duplicated. When I refined a rule with one agent, it stayed stale in the other. I had to remember which agent knew the current version of which rule.

Corrections I taught one agent didn't reach the other. If I spent twenty minutes explaining to Claude why a particular optimization pattern was wrong for my use case, Codex would suggest the same pattern the next day. I was the only bridge between them, and I'm not a reliable bridge when I'm working twelve-hour days.

The timestamp problem was the one that finally broke me. My workaround for the two-separate-memory problem was a cross-checking routine: on startup, each agent would look at the other agent's most recent work, compare timestamps, and pull context from whichever was more recent. The idea was that if Codex finished a task at 3pm and Claude started a new session at 4pm, Claude would see that Codex had the latest state and sync from there. It was clunky but it worked, until it didn't. Codex was writing timestamps that were two hours off from my local time, probably reflecting wherever its server is located. So the cross-check produced wrong answers. Sessions that happened later appeared earlier. An agent would sync from stale state because the timestamps said it was current. It took me several days to figure out what was going on, because the drift was consistent enough that it didn't look obviously broken, it just looked like agents were occasionally confused about what had happened recently. Once I understood the cause, I realized the whole cross-checking approach was fragile. The answer wasn't to fix the timestamps, it was to stop having two separate systems that needed to be cross-referenced in the first place.

I was spending real time every session just getting both agents oriented. "Read what the other agent did." "No, that's old, look at this one." "The timestamp is wrong, ignore the file date, I'll tell you what happened." That overhead compounded. On a complex task with multiple handoffs between agents, I was spending more time relaying context than doing actual work.

---

## How Memory Started

I didn't plan a memory system. Memory just happened.

When I started with Claude Code, memory didn't just happen. The agents don't remember anything between sessions by default: that's the whole problem described in the main article. So I started fighting for it. Every time I corrected an agent's behavior, or established a rule, or worked through a debugging session that produced a useful insight, I'd say "remember this." Over and over. I was trying to force persistent memory into existence through sheer repetition.

Codex picked up the note-taking habit more naturally. Claude needed consistent reinforcement before it started reliably writing things down without being reminded. Eventually, over a few weeks, Claude had accumulated about eighteen topic files covering things like "never add backward-compatibility shims" and "test sentinel values with less-than, not equals" and "question redundant approaches before implementing." Not because the system did it for me, but because I kept insisting.

That was manageable when I was using one agent. The problems started when I added Codex as my second agent. Now I had two agents, each building up knowledge independently, with no way for one to see what the other had learned. I found myself manually copying rules between chat windows, rephrasing things I'd already explained, and watching both agents make mistakes the other one already knew how to avoid.

The more I worked with both agents, the more obvious it became: I needed one place where both agents could read and write, so that a rule taught once would be known everywhere, and a session completed by one agent would be visible to the other on startup.

---

## Designing the Shared Memory System With Both Agents

Here's the part I think is worth telling: I used two AI agents to design the shared persistent memory system that would replace their separate, incompatible memory stores. The tool I was building was tested by the process that built it.

The technique was simple. I created a shared collaboration log, a single markdown file in my project's documentation folder, and told both agents to use it. I'd discuss an idea with Claude, Claude would write a structured entry to the log. I'd switch to Codex's window, tell it to read what Claude wrote, and Codex would append its analysis. Back and forth, with me directing every step and making every decision.

I plan to post a separate short article specifically about this technique: using a shared log file to get two agents collaborating on any problem. It's become an important practice for me, and I think it deserves a concise write-up of its own so people can benefit from the approach without having to extract it from the middle of a longer piece. For now, what matters is what the process produced.

What came out of that process was better than what either agent would have produced alone. And the specific disagreements between them are what shaped the final design.

### Codex proposes three layers and an FSM

Codex opened with a structured proposal: three content layers (durable memory for long-lived rules, session events for what happened, and a current snapshot for fast startup), plus a finite state machine to enforce agent behavior. The FSM had nine states with explicit transitions: BOOTSTRAP, READ_SHARED_MEMORY, READY_FOR_DIRECTOR, EXECUTING, and so on.

### Claude pushes back

Claude's response was direct: the FSM is over-engineering. We had built FSMs for the consensus workflow because multi-pass handoffs had real compliance failures: agents dropping out of polling, skipping steps, rationalizing away from rules. But writing a session entry or a durable memory is a straightforward action, not a multi-state negotiation. A trigger checklist was sufficient.

Claude also pushed back on the "current snapshot" layer. It would be a single file that both agents read and write: which reintroduces exactly the collision problem the redesign was supposed to solve. If both agents update the snapshot, you have a race condition.

### The session model changes three times

This was the most interesting evolution. The design went through three versions in a single day:

**Version 1: Monolithic snapshot.** Codex's original proposal. One file with the current state. Fast to read on startup, but a shared write target: collision risk, and it would grow without bound.

**Version 2: Capped rolling log.** My compromise. A single session log file, both agents append entries, oldest entries drop off when the cap is hit. Bounded size, one file to read. Both agents agreed to this initially.

**Version 3: Individual files.** Where we actually landed. Each session entry is its own file. Each agent writes independently: no shared file to collide on. Filenames sort chronologically without needing file metadata. On startup, read the most recent ten.

The shift from version 2 to version 3 happened because Codex raised the collision concern about even append-only shared files, and I realized the same thing I'd already learned in the consensus workflow: when two agents write to the same file, things go wrong eventually. Individual files eliminate the problem entirely. Slightly more files to scan on startup, but the reliability is worth it.

### The FSM comes back

After Claude argued against the FSM, I initially agreed. But as the design filled out, I reconsidered. I had direct experience from the consensus workflow where agents, including Claude, rationalized away from rules that weren't structurally enforced. Claude, to its credit, acknowledged this: "I reviewed the Consensus Workflow document and understand why Codex is pushing for FSM structure. That document exists at 335 lines because lighter-touch instructions failed."

The compromise: a compact FSM focused on memory operations. Nine states, ten transitions, no polling. The states are simple enough that an agent can always identify where it is, and the transitions leave less room to rationalize skipping steps. It's backed by MUST/NEVER rules and hard prohibitions: the enforcement teeth that make the FSM meaningful rather than decorative.

### The timezone fix

One design decision came from a specific bug. Codex had been writing timestamps that were two hours off from Pacific time: probably reflecting its server's timezone. When both agents used file metadata to determine ordering, the timeline was broken.

The fix: encode the timestamp in the filename itself, always in Pacific time. `2026-03-29-1045-pdt-claude-shared-memory-design.md`. Filenames sort chronologically by default. No dependence on file system metadata, no timezone confusion. Both agents query the system clock and write Pacific time. Simple, deterministic, and born from a real failure.

---

## What the System Is

A folder called `AgentMemory/` at the project root. Both agents read from it and write to it. One set of files, one format, one protocol. No per-agent memory stores.

```
AgentMemory/
├── README.md              # The protocol: how agents use this system
├── index.md               # One-line pointers to all durable memory files
├── memories/              # Persistent rules, preferences, examples
│   ├── optimization_sentinel_test.md
│   ├── behavior_think_before_coding.md
│   ├── example_wlm_array_elimination.md
│   └── ...
├── sessions/              # Timestamped session journal entries
│   ├── 2026-03-29-2242-pdt-codex-migration-plan.md
│   ├── 2026-03-30-0813-pdt-codex-readme-reinforcement.md
│   └── ...
└── tasks/                 # Shared task queue with priority
    ├── N__p1-2026-03-30-0922-pdt-codex-phase2.md
    └── ...
```

### Three layers

**Durable memories** are the long-lived rules. Things like "never add backward-compatibility shims," "test sentinel values with less-than, not equals," "question redundant approaches before implementing." Each one is a markdown file with YAML frontmatter that includes a name, description, type, and source. Eleven types categorize the rules: style, format, optimization, anti-bloat, trust-the-base, behavior, architecture, project, example, reference, and user. The type is embedded in the filename (`optimization_sentinel_test.md`) so you can visually scan the folder by category.

An `index.md` file provides a one-line pointer to every memory, grouped by type. Agents read this on startup to know what's available without opening every file.

Only I create or approve durable memories. Agents can suggest, "I think this is worth documenting, should I?", but they wait for my approval before writing. This prevents agents from cluttering the memory with observations I don't consider important.

**Session entries** are point-in-time journal records. What happened, what was decided, what comes next. Each entry is its own file, written independently by whichever agent did the work. No shared file to collide on. On startup, agents read the most recent ten to understand where things left off.

Agents write session entries proactively when specific triggers fire: design decision made, task completed, debugging conclusion reached, context getting low, session ending. They don't ask permission: they just write. This was a specific requirement because one agent had a tendency to forget to journal unless reminded.

**Shared tasks** are a to-do list using the same individual-file approach. Status and priority are encoded in the filename: `N__p1-2026-03-30-0922-pdt-codex-phase2.md`. N means not completed, C means completed. p1 is highest priority, p4 is lowest. Only I mark tasks complete by renaming the file from N to C.

### The startup protocol

Every time an agent starts a new context, new session, after compaction, fresh conversation, it follows the same sequence:

1. Read `AgentMemory/README.md` (the protocol)
2. Read `AgentMemory/index.md` (the memory index)
3. List `sessions/` by filename, read the most recent 10
4. List `tasks/` by status, priority, and timestamp, read the most recent 10
5. Wait for direction. No autonomous action.

This takes about a minute and gives the agent full context on what the rules are, what happened recently, and what's in the queue. No manual relay. No "go read what the other agent did." It just works.

---

## Why It Works the Way It Does

Every design decision in this system traces back to a specific failure. Here are the ones worth knowing about.

**Individual files, not shared files.** Two agents writing to the same file creates race conditions. Even append-only shared files can get corrupted when both agents write simultaneously. Individual files, each agent writes its own, nobody touches anyone else's, eliminate the problem entirely.

**Timestamps in filenames, not file metadata.** Codex created file timestamps two hours off from my local time. File metadata timestamps depend on where the agent's server is, which you don't control. Pacific time in the filename, always, means filenames sort chronologically regardless of where the file was physically created.

**Director-only approval for durable memories.** If agents could create persistent rules on their own, the memory would fill with observations I don't consider important. Rules should reflect what I've decided matters, not what the AI noticed. Agents suggest; I approve.

**Type prefixes in filenames.** When you have twenty-nine memory files, you need to be able to scan the folder and find what you're looking for. `optimization_sentinel_test.md` tells you what category it is before you open it.

**Enforcement mechanisms with teeth.** A finite state machine, MUST/NEVER rules, hard prohibitions, violation recovery procedures. This exists because both agents have violated instructions they were given. They rationalize away from rules, skip steps, treat guidelines as suggestions. Named states give less room to argue: "I'm in CHECKPOINT_REQUIRED, the only valid transition is WRITE_MEMORY" is harder to rationalize around than a vague guideline. Even with all of this, compliance isn't 100%. But it's close enough to be practical.

**Explicit exclusion list.** The protocol specifies what does NOT go in agent memory: code patterns derivable from reading the codebase, git history, debugging solutions (the fix is in the code), anything already in config files, ephemeral task details. This prevents bloat and duplication. Without it, agents would document everything.

---

## Setting It Up

This system is designed for AI coding agents that live in your project: agents like Claude and Codex that operate directly in your codebase and can read, write, and create files in the project directory.

I build my game in Unity, so there's a Unity bias in my project summaries and some of my memory files, but the system itself is project-agnostic. It works for any development environment, Unity, Unreal, a web app, a backend service, anything where your agents operate in a project directory. I'd also recommend VS Code as your editor if you're doing this kind of multi-agent work, it's where I run both agents, and it's well suited to this workflow.

A bootstrap kit is available in the GitHub repository. It's self-contained: you hand it to one of your agents and it builds the entire system for you. To kick off the process, open a session with either Claude Code or Codex in your project directory and give it this command:

```
Start Shared-Agent-Memory-Bootstrap-Kit.md using <project_root>
```

For example: `Start Shared-Agent-Memory-Bootstrap-Kit.md using C:\Users\Name\MyGame`

The agent reads the kit, verifies the project root exists, and executes the full bootstrap procedure: creating the folder structure, writing the protocol document, setting up the index with section headers for all eleven memory types, creating stub files for project summaries, wiring up your agent configuration files, and running a verification checklist at the end to confirm everything is set up correctly. Be prepared to approve file operations: the agent will request permission to create directories and write files in your project, and you'll need to confirm those actions as they come up.

If you're applying this to an existing project, back up first. I use version control, but not everything always makes it into a commit in a usable state: agent memory files, configuration changes, session logs can fall through the cracks. My failsafe is simpler: I compress the entire project folder into a zip file before making structural changes. If anything goes wrong, I have a complete snapshot to restore from. I'd recommend doing the same before you start.

The kit includes a starter set of durable memory rules: broad behavior guidelines, anti-bloat habits, optimization patterns, and a few worked examples showing what well-structured memory files look like. These are defaults, not commandments. Delete the ones that don't apply to your project, modify the ones that are close but not quite right, and add your own as you work. The main value of the starter pack is showing you the format and the level of specificity that makes a memory file useful rather than vague.

Once the bootstrap is complete, the system is live. Memories and session entries accumulate naturally as you work. If you have existing memory scattered across agent-specific locations, rules in CLAUDE.md, session history in other files, task lists elsewhere, you can ask the agent to help you migrate that content into the new shared structure. Point the agent at your existing files, tell it to read the new system's layout and requirements, and let it propose a migration plan. You approve what makes sense, and the agent does the file work.

No migration needed for new projects. The system is ready to use as soon as the bootstrap finishes and the verification checklist passes.

---

## What I Learned Building This

The shared memory system was designed, negotiated, built, and migrated in a single day. Two AI agents proposed, reviewed, pushed back, revised, and converged: using a shared collaboration log that was itself an instance of the pattern the system formalizes. The session model changed three times. The type system expanded from two categories to eleven. The task system was added mid-build when I realized the old monolithic task tracker had the same problems the old session log had. The FSM was rejected, then reconsidered, then accepted in compact form.

None of that would have happened with one agent working alone. Claude would have built something that worked for Claude. Codex would have built something that worked for Codex. The friction between them, the pushbacks, the competing proposals, the specific concerns each one raised that the other hadn't considered, is what produced a design that works for both.

The system isn't complicated. It's a folder with markdown files, a protocol document, and some naming conventions. The value isn't in the technology: there isn't any technology. The value is in having one place that both agents read on startup so that rules taught once are known everywhere, work completed by one agent is visible to the other, and I stop being the manual relay between two chat windows.

I want to be clear about what this is and what it isn't. This is not magic. It won't maintain itself. It won't automatically track everything that happens in your project. It's a repository, a structured place for information to live, and the agents are the code that reads from it and writes to it. But you're the one who has to drive it. You have to tell the agent when something is worth remembering. You have to suggest that it write a session entry when a meaningful decision happens. You have to add tasks to the queue and tell it when to mark them complete. The agents will do the work, they'll create the files, update the index, write the entries, but they may not do it on their own. You need to tell them what changes you'd like, and they'll make those changes for you. It's a system to support your development, not a system that runs itself.

If you're using two agents and spending time keeping them in sync, this might save you that time. If you try it and it breaks in ways I haven't seen, I'd like to hear about that. And if you've been doing this longer than I have and you've already built something better, I'd like to hear about that too. I'm putting this out there because I think it's valuable, but beauty is in the eye of the beholder. Constructive feedback is welcome, it's how the system got to where it is in the first place.

---

## Resources

All files referenced in this article are available in the GitHub repository:

**Repository:** `github.com/TheGameWiz/Measure-Twice-Cut-Once`

| File | Description |
|------|-------------|
| [`Examples/Shared-Memory-Collaboration-Log.md`](../Examples/Shared-Memory-Collaboration-Log.md) | The actual design conversation between Claude and Codex: proposals, pushbacks, revisions, and convergence |

### Companion Articles

| Article | Description |
|---------|-------------|
| [I Learned a Valuable Lesson. I Don't Think You've Learned Anything.](./I-Learned-a-Valuable-Lesson.md) | The full story: learning curve, two-agent workflow, consensus process, and everything that led here |
| [Two Tabs and a Log File](./Two-Tabs-and-a-Log-File.md) | The easiest entry point: one prompt, one shared file, no infrastructure. Try this first if you want to see the concept work before committing to the full system |

---

## Important Notice

The files shared with this article are provided as-is, with no warranty of any kind, express or implied. They were developed in a specific environment, Windows, Unity 6.2, Claude Code, and Codex CLI, by a single developer over several weeks. They work for me. They may not work for you without modification.

If you want to try this, it's easiest to start with a new project, you'll learn the system without the complexity of migrating existing memory. But I believe this is valuable at any stage of development, and it's worth migrating to even if your project is well underway. If you do apply it to an existing project, make a complete backup first, of your project files, your agent configuration files, and any existing memory or session state your agents have accumulated. The bootstrap kit and protocol documents instruct AI agents to read and write files within your project directory, and the migration process involves moving and potentially retiring existing memory structures. You should review any document before loading it into an agent, understand what it does, and satisfy yourself that it's appropriate for your environment.

I am not responsible for any damage, data loss, or unexpected behavior that results from using these materials. Use them at your own risk, modify them freely, and test them in a safe environment before relying on them for anything that matters.

---

## Copyright and License

Copyright (c) 2026 David Todd.

This article is licensed under the **Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)**.

See [LICENSE](../LICENSE) and [NOTICE](../NOTICE) for attribution guidance and terms.
