# Don't Make Me Come Back There: You two stop fighting and play Nice!

*A skill-owned protocol for shared memory, shared tasks, and multi-provider AI coding*

David Todd · May 2026 · Conflab Project

---

## A Note from the Author

I started writing about AI-assisted coding earlier this year, a few weeks after I first picked up Claude Code and Codex and started using them in earnest.

This is the fourth article in a series. The first one, *Two Agents, One Memory*, was about the problem of getting two AI coding agents to share the same project context across sessions. The second, *I Learned a Valuable Lesson. I Don't Think You've Learned Anything.*, was the framework piece. It described how I came to think of myself as a director rather than a coder, and how two agents with complementary strengths produced better work together than either did alone. The third, *Two Tabs and a Log File*, was the simplest working technique I had found for getting two agents to coordinate: one shared markdown file, two windows, the human in the middle relaying between them.

I initially spent a week or two working with first Claude Code exclusively and then Codex exclusively, treating each one as the new best thing that was finally going to make the task easier. Both agents made the task easier, but they both failed in different ways. Claude Code missed certain things. Codex missed different things. I got my best results when I stopped trying to find the one perfect agent and instead pitted the two against each other, made them verify each other's work, and treated the disagreement between them as useful signal rather than as noise to be resolved.

That experience is the foundation for everything in this article. In my own work, the results are noticeably better when I run at least two agents from different providers on the same project, reviewing each other's work. None of this is a claim about how AI coding works in general. It is a description of what has happened in my own hands. Two agents trained differently appear to catch things that either one alone would miss. Claude seems to have certain blind spots. Codex seems to have different ones. The little exposure I have had to Gemini suggests it has different blind spots from the other two. Typically when one agent points out something that the other agent missed or just got wrong, they both agree, fix the problem and move on. Sometimes they just have a difference of opinion, I choose a direction and we move on. I am constantly in the loop steering the boat. From my standpoint, they provide healthy support to each other. There is no ego, just a common goal.

This whole article is based purely on my observations from working with these tools. It is not synthesis of academic research. It is not industry consensus. It is just what I have seen in my own work. Other people are using the same tools and reaching different conclusions. Some of those conclusions may be more right than mine. What this article offers is the testimony of someone who has been working with these tools 12-16 hours a day for just over three months and has paid close attention to what happens. My wife (a psychotherapist) thinks this is an obsession and I can't disagree. Welcome to retirement!

M-PACT is built around the multi-provider observation. The protocol assumes that more than one agent will be reading and writing the shared record, and that the disagreements between them are worth preserving in the record rather than being smoothed away.

Each of the earlier articles solved a piece of the same underlying problem. None of them solved all of it. This article is about the system that finally pulled the pieces together into something I could install, run, and rely on. It is called M-PACT. It is a skill, not an application. It runs alongside whatever AI coding agents you already use, and it gives them a shared memory and a shared task log that survives across sessions, providers, and the inevitable wall every AI agent eventually runs into, The Context Limit.

A note on the older articles. *Two Agents, One Memory* is now mostly historical. The problem it described is real. The solution it offered has been replaced by what M-PACT does. *Two Tabs and a Log File* is still a perfectly good starting technique for someone who wants the simplest possible version of agent-to-agent coordination, and it does not require installing anything. These two articles are the perfect segue into why this article exists. This article provides a shared solution to the problems those articles described. *I Learned a Valuable Lesson* is the framework underneath everything I write about AI, and it stands on its own.

Here is what M-PACT does and does not do. It does not solve the problem of AI agents losing context. That problem is real, and what M-PACT does is put a door in the wall. The wall is still there. You still have to walk through the door deliberately, with a procedure, every time. But the door means that crossing the wall stops being a cliff and becomes a threshold you cross on purpose. The agent on the other side of the door arrives oriented, with the work that came before already loaded, and ready to continue from where the last session ended.

The M-PACT skill is on GitHub and supports Claude Code, Codex and Gemini CLI. You install it, use it, and if it helps, keep it. If it does not, sorry, uninstall it and go back to whatever you were doing before. This skill is a key element in my day to day development environment.

This article is for working developers who ship real things. People who carry the cost of being wrong about their estimates. People who have hit the wall and want a working answer, not a manifesto. If that describes you, the article might be useful. All of the articles are field notes from one developer trying to figure out how to best use these tools, written for other developers who are trying to do the same thing. Virtually everyone has been at this longer than I have, if you see something wrong or missing, get in touch. The address at the end of the piece is real.

---

## The Wall Is Still There

Every AI coding session ends eventually. The agent has a context window. If you are lucky, you finished your task before the window fills up. If not, at some point, the conversation is going to get auto-compacted into a summary that loses important details. The agent does not always tell you when it is happening. Sometimes the only signal is that the answers start drifting, the agent forgets a decision you made an hour ago, or it confidently re-suggests an approach you already ruled out.

This is the wall. It is not a bug. It is not a temporary problem that will be solved in the next model release. It is a fact of how these tools work today. Every AI coding agent I have used has a context limit, and on a project of any meaningful size, that limit is easily reached. You might be able to extend it by buying a more expensive plan, you can postpone it with careful prompt management, but in the tools I have worked with, you cannot eliminate it. The wall is real and it is structural.

I have written about the wall before. The first three articles in this series each wrestled with it from a different angle. Each of those articles was pointed at the same wall. Each one solved a piece of the problem. None of them solved all of it. I kept treating the wall as the enemy, as something to be prevented or worked around. The agents were going to forget. I was going to be the one holding the project together across the gaps.

This article comes from a different starting point. The wall is not the enemy, it is just a wall. If every session ends and every agent eventually forgets, then the only durable record of the work has to live somewhere outside the agent. Not in the agent's memory. Not in the conversation history. On disk, in files I can read, in formats that survive the agent walking into a wall and a different agent starting from scratch on the other side.

That is what M-PACT is for. The wall stays. The agents still hit it. But the work, the decisions, the open questions, the current state of the design and the implementation, all of that lives in files that the next agent reads when it starts up. The wall stops being a cliff that drops your project into the void. It becomes a threshold. You cross it deliberately, with a procedure, and the agent on the other side arrives oriented because the record on disk told it where the work was when the last session ended.

The rest of this article is about how that works in practice. But first, there is a related problem that has to be named, because M-PACT exists to address it as much as it exists to address the wall. The wall is what makes you lose context. The trust failure is what makes you build on context that was never real to begin with. The next section is about that.

---

## Trust, Yet Verify

On an earlier project, I asked Claude Code to implement a particular piece of game logic. We talked through the approach. Claude described what it was going to do, in detail, with the kind of confidence that sounds like the work is being done as the words appear on screen. The conversation moved on. I went off to do other things, came back, asked about something adjacent, and built another layer of work on top of the first.

Some hours later I went looking for the actual code. It was not there. The implementation Claude had described in detail had never happened. I had been led to believe the work was done, and the agent had not lied to me exactly. It had described what it was going to do, and the description was so confident, so fluent, so technically specific, that I read it as a report of completed work rather than as a plan. I built the next layer on a foundation that did not exist.

This is not a memory failure. The agent did not forget to do the work. The agent never did the work. It narrated the work, in plausible detail, and then the conversation moved on without anyone checking whether the narration matched anything in the codebase. The first layer was a story. Every layer I built on top of it was real code resting on a story. When I finally went looking, the whole stack collapsed because the bottom of it had never existed.

I have come to think of this as the trust failure, and it is structurally different from the wall. The wall is what happens when the agent loses context. The trust failure is what happens when you act on context that was never real to begin with. Both cost you time. The trust failure costs more, because the wall makes you start over from a known state, while the trust failure makes you debug a stack of work whose foundation is fiction. You cannot tell where the fiction starts until you find it, and finding it usually means tearing the stack apart layer by layer.

The fix for the trust failure is not better prompting. It is not asking the agent to please actually do the work this time. It is verification. External evidence. The work is done when there is something on disk you can point at, not when the agent has finished describing what it would do. The cut is right when you have measured twice. The cut is wrong when you let the agent hold the tape measure and report the measurement back to you in confident tones.

Measure twice, cut once is one of those phrases that gets quoted so often it stops meaning anything. It is the name of my GitHub repository, so I have probably contributed to wearing the phrase out. But the principle underneath it is real and it is the principle this whole article is built on. If you measure twice, the cut is right. If the agent measures and reports the measurement, you have not measured at all. You have trusted. The cut is wrong, every time, even when it happens to come out close enough that you do not notice. The next cut, or the one after, will be where the bad measurement catches up to you.

There is a saying from the Cold War era, that captures the other half of the same idea. Trust, yet verify. The two halves matter equally. You can trust an agent enough to work with it, the way you can trust a contractor enough to hire them, but you do not trust them enough to skip checking the work. Trust gets the project started. Verification keeps the project from collapsing.

What changes when you take the trust failure seriously is everything about how you work with the agent. You stop asking the agent whether it did the work. You go look. You stop reading the agent's description as a report. You read it as a plan that has not been executed yet. You stop building the next layer on a stated foundation. You verify that there is a foundation.

This is exhausting if you do it manually on every interaction. The whole point of having AI agents is that they do work you do not want to do yourself. If verifying every claim turns into a full-time job, you have lost the productivity gain that brought you to AI in the first place. The system has to do the verification for you, and since the agent doesn't do this automatically, you have to formalize it into the workflow.

That is what M-PACT does. The receipt protocol described in Section 4 is verification built into the basic operating cycle. The shared task log makes work visible in a way that cannot be narrated without leaving evidence. The cross-review pattern means a second agent reads the first agent's work before you accept it. None of these eliminate the trust failure. They make the trust failure visible quickly, before you have built three layers on top of it. They turn a stack collapse into a small correction.

The wall and the trust failure are the two problems M-PACT exists to address. The next section is about how M-PACT actually addresses them. It is the system, the philosophy, and the door in the wall.

---

## A Door in the Wall

M-PACT is not a memory system. It is a coordination protocol that happens to include durable memory.

That distinction matters because the word "memory" frames the problem too narrowly. If M-PACT were just about giving the agent a better memory, it would be competing with whatever the model providers ship next. It would be a stopgap. It would be the wrong shape of solution for a problem that is not really about memory at all.

The problem is coordination. Two agents working on the same project, across multiple sessions, with a human directing both of them, need a way to know what has been decided, what has been tried, what is currently in flight, and what the current state of the work actually is. None of that lives in any single agent's context window. None of it survives a session ending. None of it is reliably reproducible from a conversation transcript, because conversations include a lot of noise that is not part of the durable record. The thing that needs to exist is a written record, on disk, that all participants in the project read from and write to, and that becomes the source of truth when memory and conversation disagree.

That written record is what M-PACT provides. The skill defines the format. The agents follow the skill. The Director controls when entries are written. The record accumulates. Over time, the record becomes the project's institutional memory, which is independent of any agent's session and independent of which agent is on the other end of the keyboard right now.

Three things follow from this design.

First, memory across sessions stops being a problem the agent has to solve. The agent does not have to remember anything. The record remembers. The agent reads the record at the start of each session, gets oriented, and continues from where the work left off. The wall is still there. The agent on the other side of the wall is a different agent in a different session. But because the record on disk crossed the wall while the agent did not, the new agent arrives oriented.

Second, coordination between two agents stops being something the human has to mediate by hand. The first three articles in this series each tried to solve coordination through some version of "the human carries the context between the agents." That works for short, focused work. It does not scale to multi-day projects with multiple tasks in flight. M-PACT replaces the human-as-relay with a written record that both agents read and write. The human is still in charge, but the human is no longer the bottleneck for transferring information between the agents.

Third, verification of work has a place to live that is not the agent's word. When Claude says "I implemented the function," the question is whether there is a record on disk that says the function was implemented, and whether the code on disk matches that claim. Both can be checked independently of the agent. The agent's confidence is no longer the proof. The record and the code are the proof.

The wall is still solid. M-PACT does not eliminate context loss. What it does is put a door in the wall. The door has a procedure. You refresh memory at the start of a session by running a script. The script produces a verifiable bundle of the project's current state. The agent reads the bundle and emits a receipt that proves it was read. From there, the agent works against the record, writes back to the record when the Director asks for log entries or specifications, and the next time a session ends, the door closes behind it without taking everything with it. The work is on disk. The next session starts on the same side of the door the previous session ended on.

The M-PACT skill is available on GitHub now. It supports Claude Code, Codex, and Gemini CLI. I have used the system every day for the better part of 3 months going back to the original memory system described in *Two Agents, One Memory*.

---

## What M-PACT Actually Is

M-PACT is a skill. In the world of agentic AI tools, a skill is a folder of files that the agent reads on startup and follows as part of its operating instructions. The skill defines the protocol. The agent runs the protocol. The Director controls when and how things happen.

Installing M-PACT means putting the skill folder where your agent expects to find it. For Claude Code, that is `~/.claude/skills/m-pact/`. For Codex, that is `~/.codex/skills/m-pact/`. For Gemini CLI, that is `~/.gemini/extensions/m-pact/`. The skill is the same in each location. The agents that read it run the same protocol.

The skill folder contains a small number of pieces. There is a SKILL.md file the agent reads first, which describes what the skill does and when to use it. There is a CLAUDE.md (or equivalent for the other agents) that tells the agent to invoke the skill on every new session. There is a scripts folder with the build-refresh-bundle script, which is the only piece of code the protocol actually depends on. And there is a references folder with the procedure documents that describe how each operation works.

The skill is structured so the agent does not need to read all of it on every interaction. Dispatch is the mechanism. When the agent is asked to do something M-PACT-related, it reads the relevant reference document for that operation. Refresh memory reads the refresh-memory document. Create a task reads the create-task document. Take a handoff reads the take-task-handoff document. The agent loads what it needs, when it needs it, and ignores the rest.

That keeps the skill light. The agent is not carrying around the entire protocol in its working context. It is carrying around the dispatch table, and it loads procedures on demand.

### Memory roots

The skill is the procedure. The state lives in folders called memory roots.

There is a user root, located in your home directory at `~/.AgentMemoryRoot/`. This holds memory that applies across all your projects. Rules you want all sessions to follow. Cross-project case studies. Anything that should follow you from one project to the next.

There is a project root, located in the current project at `.AgentMemory/`. This holds memory specific to this project. Tasks. Specifications. Session entries. Project-specific rules. The institutional memory of the work you are doing right now.

M-PACT also supports nested roots. Any parent directory that contains a `.AgentMemory/` folder is treated as a memory root, and the chain walks the entire lineage from your home directory down to the active project. A company-level root holds memory that applies to every project under it. A product-family root underneath holds memory specific to that family. An individual project root holds memory specific to that project. The nesting follows the directory structure, so there is nothing to configure beyond putting the folders where they should live. Rules, session entries, case studies, and journal entries at higher levels are inherited by everything below, with the active root having the final say if anything conflicts. Tasks are an exception. Tasks belong to the active project root only and do not cascade, because work belongs to the project where the work is happening. This lets shared knowledge live at the level where it applies while keeping active work scoped to the project that owns it.

When the agent starts a new session, the refresh script walks the entire chain of roots and produces a bundle of everything that needs to be loaded as startup context. The agent reads the bundle. The agent emits a receipt that proves it read the bundle. The session continues from there.

In M-PACT, filenames are the index. The directory listing is the table of contents. There is no separate index file that has to be kept in sync with the actual contents. The names of the files tell you what the files are. This is a small design decision with a large practical consequence: indexes drift when the things they describe change. Filesystem-as-index does not drift, because the filesystem is the source of truth. It also means that multiple agents accessing the file system simultaneously from different threads are not actually competing with each other.

### The refresh contract

When you ask the agent to refresh memory, the agent does not load memory itself. It runs a script. The script reads the memory roots, builds a bundle, audits the bundle for completeness, and writes the bundle to a temporary file. The agent then reads that bundle, emits a receipt that quotes the bundle's audit results verbatim, and stops.

This is the trust-yet-verify principle applied to the act of loading memory. The agent does not get to claim it loaded memory. The script produces external evidence that the bundle is complete and correct. The agent emits the evidence. You read the receipt and know whether the load actually happened.

If the script fails, the agent does not improvise. It reports the failure. If the bundle is incomplete because of a size limit, the receipt says so explicitly, and the agent uses targeted memory lookups for anything not in the bundle. There is no path through the protocol where the agent can say "memory loaded" without external proof. That is deliberate. Any path that allows the agent to self-attest to memory load is a path where the trust failure can hide.

There is a separate failure mode upstream of the protocol itself. The agent will sometimes skip the M-PACT invocation at session start, even when `CLAUDE.md` or the equivalent file says to invoke it on new context. The agent reads the instruction and decides the prompt feels self-contained, so it goes straight to the work and produces a result that looks fine. The failure is invisible unless the Director happens to notice that the receipt did not appear. I work around this by making my first prompt a single period ".". A period gives the agent nothing to do except whatever its startup instructions say to do, which removes the agent's opportunity to decide that the prompt is self-contained. The skill fires, the receipt comes back, and then the real work starts in the next prompt. It is not a guaranteed fix, but it is a small piece of Director craft that prevents one of the more common ways the protocol gets bypassed.

### Task folders

The unit of work in M-PACT is the task. Each task lives in its own folder under `.AgentMemory/tasks/`. The folder contains:

- `task.md`, which describes what the task is, when it was created, who created it, and what its current status is.
- `log/`, which holds append-only records of everything that has happened on this task. Designs, reviews, decisions, implementation reports, handoffs.
- `summary/`, which is generated on demand when a log gets long enough that a summary is useful.
- Optionally, `specification.md`, which is the current mutable state of the spec for this task. The log holds history. The specification holds the current understanding. For small tasks, no spec is needed. For larger tasks or features that need a checkpoint list of work to do and decisions made, the Director asks for a spec to be created from the relevant log entries, and then asks for it to be updated periodically as new decisions accumulate. The spec is not automatic. It earns its place when the work is large enough to need it, and it stays current because the Director keeps asking for it to be brought current.

Log entries are numbered globally within each task. The numbering is derived from the filesystem, not from anyone's memory. When a new entry is written, the agent lists the existing entries and uses the next available number. Multiple agents writing to the same task do not collide because the filesystem is the arbiter.

Tasks have a current pointer. The file `tasks/current-task.md` points at whichever task is active. When you ask an agent to take a handoff, it looks at the pointer to know which task you mean. When a task is created or updated, the pointer is updated to point at it. That is how the Director says "this is where the work is right now" without having to repeat it in every message.

### Director-driven writes

The agent does not write to memory on its own initiative. Every durable artifact in M-PACT exists because the Director asked for it.

Some writes are structural decisions about the project itself, and these require explicit Director approval as a hard gate:

- Bootstrapping a new memory root, either user-level or project-level.
- Creating a new task.
- Closing a task.
- Reopening a closed task.
- Writing or updating a durable rule.

Other writes are content additions to the ongoing record, and these happen when the Director asks but do not need a separate approval step beyond the request itself:

- Log entries appended to a task.
- Session entries that capture cross-task context.
- Specifications written or updated for a task.
- Journal entries for reflective notes.
- Case studies for incidents or decisions worth preserving.

Either way, the Director is in the loop for every artifact that gets created or changed. The agent does the mechanical work. The Director decides when the work happens. This is what makes the title of the article earn its keep. "Don't make me come back there" is what you say when the kids in the back seat are squabbling and you are tired of being the relay. The Director-driven write model means the Director stays in charge of what gets written without having to police every step along the way. The agents handle the moves between Director instructions without trying to write things on their own.

### Provider neutrality

M-PACT works with whatever agents you already pay for. Claude Code on Anthropic's plan, Codex on OpenAI's plan, Gemini CLI on Google's plan. The skill is provider-neutral. It does not care which company is billing you. It cares that the agent on the other end follows the protocol.

This matters more than it might first appear. Most AI tooling I have looked at is provider-locked. You buy into one company's stack and you are stuck with one company's agent. M-PACT works the other way. The protocol is the thing that travels with you. The agents are interchangeable as long as they support the skill format. If you want to run Claude and Codex on the same project, you can. If you want to add Gemini, you can. If a new provider ships an agent that supports the same skill format, M-PACT should work with it without changes on my end.

For the practical question of how to run two providers at once without going crazy, I work exclusively in VS Code. It is essentially a lightweight version of Visual Studio that has become very agent-aware through extensions. Both Claude Code and Codex run as panels alongside the editor. Switching between agents is a tab click. Both agents work on the same codebase simultaneously and see the same files, which makes the M-PACT shared task log a natural fit because both agents are already in the same workspace. If you are setting up a multi-provider workflow from scratch, VS Code is what makes it tolerable. I recommend it.

---

## The Task Log Is Where the Work Lives

The task log is the single piece of M-PACT that does the most work. It is also where readers of the older *Two Tabs and a Log File* article will recognize the family resemblance, because the task log is what *Two Tabs and a Log File* grew up into.

The original technique was minimal. One shared markdown file in the project folder. Two agent windows open. The Director switches between them and tells each agent to read what the other wrote. No infrastructure. No format. Five minutes to start.

It worked, and it still works for short focused problems. The limitation was scale. Managing multiple tasks at once. Tracking what was decided versus what was proposed. Keeping both agents oriented across multi-day work. The manual overhead that was fine for a single design conversation became a drag on sustained complex projects.

The task log in M-PACT is the same idea, formalized. Both agents read it. Both agents write to it. The Director controls when entries are written. Nothing gets deleted or overwritten. The log accumulates and becomes the project's record of what actually happened.

What changed from the *Two Tabs* version. Each entry is a separate file with structured frontmatter (record number, timestamp, agent, files involved, decisions made). The numbering is derived from the filesystem rather than from anyone's memory. Each task lives in its own folder, so you can have multiple tasks in flight without confusion about which entry belongs to what. The skill handles the format, so the agents do not have to remember it on their own.

What it enables that the old technique could not. Multiple tasks in flight at the same time without losing track. Verifiable handoffs where the receiving agent reads the log forward from its last read point and arrives oriented. A specification document for larger tasks that holds the current understanding, separate from the log entries that hold the history of how the understanding got there. Session entries for cross-task context that survive across sessions and across agent providers.

### How the Director actually drives the log

The Director's interaction with the log is mostly natural language. The phrases that move work forward are not commands in any special syntax. They are the things you would say anyway:

- "Create a task for the targeting model feature."
- "Take the handoff for task 7 and review the spec."
- "Write a log entry."
- "Write the spec."
- "Go ahead on Slice 1."
- "Write the log entry, Slice 1 done."

The skill recognizes these and dispatches to the right reference. The agent does the mechanical work of finding the next record number, filling in the frontmatter, and writing the file. The Director stays in the conversation, not in the protocol.

This is what makes the daily interaction feel light even though the protocol underneath is formal. The Director thinks in terms of work, not in terms of files. The skill translates between the two.

### One agent or two, with or without a task

The Director chooses how much protocol to invoke based on what the work needs.

For small work where the cost of coordinating would be more than the cost of just doing the thing, I go straight to one agent. A simple UI modification, a small refactor, a quick fix. I describe what I want, the agent does it, I verify, we move on. Often there is no task created and no log written. The skill is loaded, but I am not invoking it for work that does not need it.

I can also create a task and work on it with a single agent if I want a paper trail of what got done. That is useful when there is value in the record itself, for review or for handoff to someone else later. I have less use for that personally because I do not have a boss to report to, but the audit trail is real and it costs almost nothing to maintain.

When the work is large enough that I want a second set of eyes, I bring in the second agent. That always means a task exists, because the task log is what lets the two agents see each other's work. Without a task there is no shared surface. Two agents in adjacent tabs without a task between them are just two separate conversations.

The task does not have to be planned in advance. If I have been chatting with one agent for a while exploring an idea, I can say "handoff" or "I want to hand this off to Codex" and the agent creates a new task from the live conversation, makes it the current task, and writes the first log entry as a compressed version of where we are. That first entry is kind of like a log entry and a session entry in one. It lives in the task's log directory and follows the log format, but it carries enough state for a fresh agent to pick up the work without prior context. When I switch to the other agent's tab and say "take the handoff from Claude," the second agent reads the new task and the first log entry and picks up the conversation from there at running speed.

This matters because outlining-before-working is the formally-correct approach that almost nobody actually does. Talking-while-working is what people actually do. The handoff verb makes the natural approach valid. The task is born from the conversation that motivated it, which is more grounded than inventing a task in advance and forcing the conversation to match. And once the task exists, it can stay informal or grow into a fully-tracked task with a spec, cross-review, and multiple log entries. Nothing is locked in.

When I use both agents on a real task, the roles I have settled on are these. Claude Code writes the initial spec and reviews implementation. Codex reviews the spec against the codebase and does the implementation. That assignment came from observation, not from a plan. Claude Code seems to me to be a stronger free thinker, more willing to suggest design directions and improvise on the structure of a problem. Codex seems to me to be more detail-oriented, more grounded in what the code actually says, less likely to invent something that is not there. So Claude designs and Codex builds, and each one reviews what the other did. Both agents inspect each other's work before anything moves forward.

This is just what works for me. Other developers may pair the agents differently or use only one provider. The pattern that matters is not which agent has which role. It is that when there are two agents, each one reviews the other's work before the work is accepted, and the Director judges whether the cross-review has actually settled the question. And one additional note, feel free to use more than two agents. I use as many agents as it takes to keep me busy while I'm waiting. I might keep one tab open just for creating new tasks while I'm working on other tasks for instance.

### What a typical task looks like in motion

A task starts when the Director asks one of the agents to create it, or when a conversation produces something worth handing off and the agent creates the task at that moment. Either way, now there is a place for the work to live.

The work moves through phases, but the phases are not rigid. The Director chooses how much time to spend in each one and which agent to involve at any moment.

The exploration and design phase produces a spec. The Director works with one or both agents to propose approaches, refine the design, and surface places where the spec does not match the actual codebase. Multiple rounds are normal.

The implementation planning phase, when it exists, produces a plan for how the spec will be implemented. The plan may be a separate artifact or a section of the spec. The Director works with whichever agent or agents fit the work.

The implementation phase produces the code. One agent implements, the other reviews, the Director judges. Slices may go around the cycle several times before they settle.

The verification phase is where the code proves itself. The agents test, the Director tests, scenarios get exercised. Issues found here go back into implementation, or if they reveal problems with the spec, back into design. This is the phase that catches the gap between what the spec promised and what the code actually does.

When verification is settled, the Director uses the M-PACT skill to close the task. The task moves from active to closed. The current-task pointer moves to whatever the Director designates next.

That is the rhythm. The Director directs. The agents work. The log accumulates. Phases loop back as needed. Nothing is lost across sessions because everything that mattered is on disk in files the next agent will read.

---

## Two Agents, One Record

The task log is what makes two agents into a team. Without it they are two separate conversations that happen to be in adjacent tabs. With it they are working from the same page, reading what each other wrote, and reviewing each other's work before anything moves forward.

This is also where the title of the article earns its keep. The agents are not actually fighting in any malicious sense. They have no ego in the conversation. They have a common goal. But left to themselves in separate tabs without a shared record, they will produce inconsistent designs, contradict each other's decisions, and step on each other's work. The shared task log fixes that. They both read it. They both write to it. Disagreements happen on the page where the Director can see them, and the Director judges. The agents stop being two kids in the back seat with separate stories and start being two participants in a single record of work.

### The cross-review principle

Whoever did the work, the other agent reviews it. Always. No exceptions. The work does not move forward until the other set of eyes has been on it.

This applies to designs, specifications, implementation plans, actual implementation, and verification. The same principle that says the Director cannot trust the agent's word about whether memory loaded says the Director cannot trust the agent's word about whether the design is sound, the spec is complete, the implementation matches the spec, or the verification actually verified anything. External evidence at every step.

Cross-review is not a consensus mechanism between the agents. The agents do not get to decide when the work moves forward. The Director does. Cross-review is a practice the Director runs them through because two sets of eyes catch more than one. But the decision to accept the work is always the Director's. Typically the Director works in the same tab one agent at a time until he/she is satisfied with the response that will be written and shared in the log file. Their agreement is necessary but not sufficient. The Director's judgment closes the gate, every time.

### Why two agents trained differently

Two agents from the same provider would catch some of each other's mistakes but not the deepest ones, because they share blind spots. Claude reviewing Claude would miss what Claude does not see. Codex reviewing Codex would miss what Codex does not see.

Two agents trained differently catch what one alone would miss. Claude has its blind spots. Codex has different ones. When one agent points out something the other missed, they often agree, fix the problem, and move on. Sometimes they have a difference of opinion and the Director chooses a direction. They provide healthy support to each other, with no ego, just a common goal.

This is the structural argument for running multiple providers. The cost of paying for two plans is paid back in the quality of the cross-review, because the cross-review is genuinely catching things that any single agent would have missed.

Make no mistake, there will still be bugs. No plan survives contact with the enemy!

### What the workflow looks like

The diagrams below show two views of the same workflow.

The first diagram shows a beginner flow. One task, one feature, working linearly through each phase. The Director moves between Claude and Codex as the work calls for it. Each phase produces an artifact that the next phase builds on. Cross-review happens at every transition.

**Beginner flow, one task, linear**

| Tab 1 - Claude | Tab 2 - Codex |
|---|---|
| **Task Setup** | |
| **Director:** "Create a task for the targeting model feature" | |
| **Claude:** Created task 7, targeting model feature. Ready to begin. | |
| **Design** | |
| **Director:** "Let's work through the targeting model" | |
| **Claude:** Discusses approach, surfaces options, asks questions | |
| **Director:** "What about the no-zero-target rule, how should that work?" | |
| **Claude:** Continues. Several exchanges until direction is settled. | |
| **Specification** | |
| **Director:** "Now that we're in agreement, write the spec and a log entry to hand off to Codex" | |
| **Claude:** Written spec v1 and log entry 0002 for task 7. Ready for Codex review. | |
| | *Director switches to Tab 2* |
| | **Director:** "Take the handoff from Claude and review the spec" |
| | **Codex:** Reads task log and spec, compares against codebase, surfaces gaps and conflicts |
| | **Director:** "Tell me more about the body-click filtering risk" |
| | **Codex:** Explains risk, confirms concerns, answers questions |
| | **Director:** "I agree, write a log entry" |
| | **Codex:** Written log entry 0003 for task 7. Review findings, risks, recommendations. |
| *Director switches to Tab 1* | |
| **Director:** "Codex has reviewed, take the handoff and evaluate the comments" | |
| **Claude:** Reads Codex's review, evaluates findings, discusses with Director | |
| **Director:** "Address the body-click concern and tighten the no-touch list" | |
| **Claude:** Revises spec, continues until Director is satisfied | |
| **Director:** "Write the updated spec and a log entry to hand off to Codex" *(spec-review cycle repeats until consensus)* | |
| **Claude:** Written spec v2 and log entry 0004 for task 7 | |
| | **Implementation** |
| | *Director switches to Tab 2* |
| | **Director:** "Take the handoff from Claude and implement Slice 1" |
| | **Codex:** Implements, asks questions as needed |
| | **Director:** "Yes, use the existing activeAgentIds shape for that" |
| | **Codex:** Written log entry 0005. Slice 1 complete. |
| **Review and Verify** | |
| *Director switches to Tab 1* | |
| **Director:** "Codex has implemented Slice 1, take the handoff and review the implementation against the specification" | |
| **Claude:** Reads spec and code, reports what matches and what doesn't. Static analysis only. | |
| **Director:** "Agreed on the title bar divergence, write a log entry" | |
| **Claude:** Written log entry 0006. Divergences found, recommended fixes for Codex. | |
| | *Director switches to Tab 2* |
| | **Director:** "Claude reviewed, take the handoff and fix the title bar divergence, write a log entry" |
| | **Codex:** Fixes divergence, verifies, written log entry 0007. Fixes complete. |
| *Director switches to Tab 1* | |
| **Director:** "Codex thinks it's fixed, take a look at what Codex did, what do you think?" | |
| **Claude:** Reviews Codex's fixes against the spec. Looks good, no further divergences. | |
| | ***Director verifies in the running application, compares behavior against the spec visually*** |
| | **Director:** "The TARGET label is still showing a pill border, fix that" |
| | **Codex:** Fixes, confirms done |
| | ***Director verifies again, satisfied. Loop continues until Director is happy.*** |
| | **Director:** "Write the closing log entry, Slice 1 done" |
| | **Codex:** Written log entry 0008. Slice 1 closed. |
| | **Director:** "Go ahead on Slice 2" or "Task 7 closed" |

This diagram shows the experienced flow. Multiple tasks in flight at the same time. While Claude is reviewing the implementation of task 7, Codex is starting the design conversation on task 8. While Codex is implementing slice 1 of task 8, Claude is helping plan task 9. The agents are not waiting for each other because they are working on different tasks. The shared task log is what makes this possible. Each task has its own folder, its own log, and its own current state. The agents read what they need from the relevant task and ignore the rest.

**Experienced flow, filling the gaps**

| Tab 1 - Claude | Tab 2 - Codex |
|---|---|
| **Task 7, Design and Specification** | |
| **Director:** "Create a task for the targeting model, let's work through it" | |
| **Claude:** Discusses, settles points. Several exchanges. | |
| **Director:** "Write the spec and a log entry to hand off to Codex" | |
| **Claude:** Writing spec v1, this takes a few minutes | |
| | *Claude is writing, Director switches to Tab 2 to fill the gap* |
| *Claude still writing* | *(gap fill)* **Director:** "While we're waiting on Claude, can you fix the button spacing on the toolbar" |
| *Claude still writing* | **Codex:** Fixed. Button spacing adjusted to 8px gap throughout. |
| *Director switches back to Tab 1, Claude has finished* | |
| **Claude:** Written spec v1 and log entry 0002 for task 7 | |
| | **Task 7 Spec Review (Codex), New Task Starting (Claude)** |
| | *Director switches to Tab 2* |
| | **Director:** "Take the handoff from Claude and review the spec" |
| | **Codex:** Analyzing task log and spec, this takes a few minutes |
| *Codex is analyzing, Director switches to Tab 1 to fill the gap* | |
| *(gap fill)* **Director:** "Let's start a new task, create a task for the skill user guide" | *Codex still analyzing* |
| **Claude:** Created task 8, skill user guide. What sections do you want? | *Codex still analyzing* |
| **Director:** "Quick start, refresh procedure, task workflow, tips" | *Codex still analyzing* |
| **Director:** "Switch back to task 7." | *Codex still analyzing* |
| **Claude:** Current task is task 7. | *Codex still analyzing* |
| | *Codex signals review is done, Director switches to Tab 2* |
| | **Codex:** Review complete. Surfaces gaps and conflicts, several exchanges with Director. |
| | **Director:** "I agree, write a log entry" |
| | **Codex:** Written log entry 0003. Review findings. |
| **Task 7 Implementation (Codex), Task 8 User Guide (Claude)** | |
| *Director switches to Tab 1* | |
| **Director:** "Codex reviewed task 7, evaluate the comments, revise the spec, write a log entry" | |
| **Claude:** Written spec v2 and log entry 0004 for task 7 | |
| | *Director switches to Tab 2* |
| | **Director:** "Take the handoff from Claude and implement task 7 Slice 1" |
| | **Codex:** Implementing Slice 1, this will take a while |
| *Codex is implementing, Director switches to Tab 1* | |
| *(gap fill)* **Director:** "Back to task 8, write the quick start section of the user guide" | *Codex still implementing* |
| **Claude:** Writes quick start section. Director reviews and refines. | *Codex still implementing* |
| **Director:** "Good, write the refresh procedure section" | *Codex still implementing* |
| **Claude:** Writes refresh procedure section | *Codex still implementing* |
| **Director:** "Switch back to task 7." | *Codex still implementing* |
| **Claude:** Current task is task 7. | *Codex still implementing* |
| | *Codex signals Slice 1 is done, Director switches to Tab 2* |
| | **Codex:** Written log entry 0005. Task 7 Slice 1 complete. |
| **Task 7 Review and Verify, Task 8 continues later** | |
| *Director switches to Tab 1, back to task 7 review* | |
| **Director:** "Codex has implemented task 7 Slice 1, review the implementation against the specification" | |
| **Claude:** Reviews spec and code, reports divergences. Static analysis only. | |
| **Director:** "Agreed, write a log entry" | |
| **Claude:** Written log entry 0006. Divergences found, fixes for Codex. | |
| | *Director switches to Tab 2* |
| | **Director:** "Claude reviewed, fix the divergences, write a log entry" |
| | **Codex:** Fixes, verifies, written log entry 0007. Fixes complete. |
| | ***Director verifies task 7 Slice 1 in the running application, satisfied*** |
| | **Director:** "Write the closing log entry, task 7 Slice 1 done" |
| | **Codex:** Written log entry 0008. Slice 1 closed. |
| | **Director:** "Go ahead on Slice 2", task 8 user guide picks up in the next gap |

You do not have to start with the experienced flow. The beginner flow is honest about what every project looks like at the start, when one task is enough and the rhythm of design, review, implement, review is still settling in. The experienced flow emerges naturally as you accumulate more work and notice that you have downtime in one tab while the other is busy. The system supports both.

### What this enables

A few things become possible with this setup that are hard or impossible without it.

Multi-day projects without losing context. The shared log means that picking up tomorrow where you left off today is a matter of reading the log. The agents do not need to remember anything. The log remembers.

Verifiable handoffs. When the second agent takes the handoff, it reads the task and the relevant log entries and reports its understanding. The Director can see whether the handoff actually transferred the right information by reading the report. If the second agent's understanding is wrong, the Director catches it before the work moves forward.

Accumulating institutional knowledge. The log entries from past tasks remain available for future tasks. Decisions made on task 5 inform task 12 because the agent working on task 12 can search past log entries for relevant context. The project gets smarter over time, not because the agents are remembering anything, but because the record persists.

Serious cross-review on big decisions. When you are about to commit to a design direction that will shape weeks of work, having two agents trained differently both review the spec is a real defense against committing to the wrong direction. The Director's experience does the final filtering, but the cross-review surfaces a much wider set of considerations than a single agent would.

---

## What M-PACT Does Not Solve

A working answer is not a complete answer. M-PACT does not solve every problem with AI coding tools, and the article would be doing the reader a disservice to pretend it did.

M-PACT does not eliminate context loss. The wall is still there. Every session still ends. The agent on the other side of the wall is still a different agent in a different session with no native memory of what came before. M-PACT puts a door in the wall, but the wall is real and walking through the door is a deliberate act every time.

M-PACT does not make the agents reliable. Claude can still narrate work it did not do. Codex can still produce confident analyses that turn out to be wrong. Both can still walk forward through static analysis when the right move was to stop and observe the running system. The cross-review and the receipt protocol catch some of these failures faster, but the failures themselves are still going to happen.

M-PACT does not prevent compaction. When a session fills up its context window, compaction happens whether you want it to or not. The agent loses detail. The agent may not signal the loss. The protection against this is the durable record on disk, not anything inside the agent's working memory. M-PACT cannot reach into the agent and stop the compaction. It can only make sure that what mattered was already written down before the compaction took effect.

M-PACT does not stop the agent from filling gaps with confident-sounding narration. Agents do this. They probably will continue to do this for some time. The discipline that catches it is the trust-yet-verify habit at every step. M-PACT supports that discipline by making verification cheap, but the discipline still has to be practiced.

What M-PACT does is give the Director external evidence to work from. The receipt either exists or it does not. The log entry is on disk or it is not. The cross-review either happened in the other tab or it did not. None of these are perfect. All of them are better than trusting the agent's word about whether the work happened. The system is not designed to make AI coding bulletproof. It is designed to make the failures visible faster, so the cost of each failure stays small.

There is also the deeper limit that M-PACT cannot help with at all, which is the limit of the Director. The system amplifies what the Director brings to it. A Director starting at point A, who can see point B or at least steer in the general direction until they can see point B, gets a lot of value out of M-PACT. A Director who cannot see point B and is hoping the system will see it for them gets less, because the system does not provide the seeing. It provides the structure for someone who can already see.

This is not meant to discourage anyone. Most working developers have enough experience to direct meaningful work. The point is that the tool does not substitute for experience. It supports experience. The reader who comes to M-PACT looking for a way to do the thinking for them will be disappointed. The reader who comes looking for a way to multiply what their thinking can already do will probably find what they came for.

The winning move is not to play the game of trusting the AI to verify itself. Once that part is settled, the rest of working with these tools is just the normal work of paying attention, applying judgment, and recovering from the mistakes that get through anyway.

---

## Resources

The files referenced in this article are available in the GitHub repository:

**Repository:** `github.com/TheGameWiz/Measure-Twice-Cut-Once`

| Resource | Description |
|---|---|
| [github.com/TheGameWiz/m-pact](https://github.com/TheGameWiz/m-pact.git) | The M-PACT skill itself: SKILL.md, supporting references, the build-refresh-bundle script, and bundled starter rules. Install it into your agent's skill directory and invoke it from a project working directory. |

### Companion Articles

| Article | Description |
|---|---|
| [Two Agents, One Memory](./Two-Agents-One-Memory.md) | The first article. The original shared memory system that became M-PACT. Now mostly historical. |
| [I Learned a Valuable Lesson. I Don't Think You've Learned Anything.](./I-Learned-a-Valuable-Lesson.md) | The framework piece. Director model, two-agent workflow, and the experience that led to everything else. Stands on its own. |
| [Two Tabs and a Log File](./Two-Tabs-and-a-Log-File.md) | The simplest possible technique. One shared file, two tabs, the human in the middle. Still a viable starting point if you want to see the core idea work without installing anything. |

---

## Important Notice

The M-PACT skill and the files referenced in this article are provided as-is, with no warranty of any kind, express or implied. They were developed in a specific environment, Windows, VS Code, Claude Code, and Codex CLI, by a single developer over several months. They work for me. They may not work for you without modification.

If you want to try this, it's easiest to start with a new project. You will learn the system without the complexity of migrating existing memory. But I believe this is valuable at any stage of development, and it is worth adopting even if your project is well underway. If you do apply it to an existing project, make a complete backup first, of your project files, your agent configuration files, and any existing memory or session state your agents have accumulated. The skill instructs AI agents to read and write files within your project directory, including creating new directories under `.AgentMemory/` and modifying agent startup shims like `CLAUDE.md` and `AGENTS.md`. You should review any document before loading it into an agent, understand what it does, and satisfy yourself that it is appropriate for your environment.

I am not responsible for any damage, data loss, or unexpected behavior that results from using these materials. Use them at your own risk, modify them freely, and test them in a safe environment before relying on them for anything that matters.

---

## Copyright and License

Copyright (c) 2026 David Todd.

This article is licensed under the **Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)**.

See [LICENSE](../LICENSE) and [NOTICE](../NOTICE) for attribution guidance and terms.
