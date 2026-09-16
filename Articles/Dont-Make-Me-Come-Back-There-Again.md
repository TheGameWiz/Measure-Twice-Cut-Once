# Don't Make Me Come Back There, Again!

*M-PACT: Gen Two*

David Todd · September 2026 · ConflabCode Project

---

## A Note from the Author

It's been a while since the last article in this series, and M-PACT has moved on in that time. M-PACT and the tool I built alongside it are multi-provider by design, not one provider with agents playing different roles. In my own sessions that mostly lands as Claude for design and design review, Codex for design review and implementation, but that's just how it's settled for me, your mileage may vary. In the interim, I built ConflabCode, a more structured, pro-user harness for multi-provider development, and I used M-PACT itself to build about half of it. Somewhere past the halfway point, ConflabCode became capable of working on itself, and I mostly switched over. But M-PACT didn't go away. I kept improving it while ConflabCode was being built, because they both serve different uses. ConflabCode is the leveled-up tool. M-PACT is still the lightweight one, jump in, jump out, and it remembers where you left off.

Building ConflabCode kept surfacing moments where I'd think, it would be great if M-PACT did this. So I added it. Handoff had never been standardized. Take a handoff, write a handoff, request a review, hand back the results, all of it worked, but it demanded a lot of management from me every time, because I was the one holding the standard for what those words meant. Gen two builds that vocabulary into the skill itself. Save context had the same problem from a different angle. I'd watch my context usage climb, ask for a detailed session entry before a compact, and count on the next session picking it up automatically, but those entries were shared across agents, not per agent, so the wrong context could be loaded. Gen two gives each agent its own saved context under its own code. And gen two adds a third provider to the mix, Antigravity, alongside Claude Code and Codex.

The more I used M-PACT, the more it became obvious that I needed to move decisions out of agent judgment and into scripted logic, wherever getting it exactly right mattered most. Gen two is the result of everything I learned working on ConflabCode's construction.

---

## A Deal with the Devil

A quick recap, for anyone who hasn't read article 4 ("Don't Make Me Come Back There"). M-PACT exists because of a wall at the end of context, when you run out, or as you get close to it, the agent starts behaving less reliably. The failure that made the wall matter wasn't forgetting, it was the agent saying it had done something that it actually hadn't. An agent described work in confident, specific detail, work that never actually happened. That gets built on, only to find out later the foundation was fiction. That's why it's important to have evidence on disk instead of taking the agent's word for it. If you want the full argument, read article 4.

We don't control the agents. Structure makes them more dependable, but it guarantees nothing. If a mistake gets through, it's yours, not the agent's.

I keep coming back to a deal with the devil analogy because it's the only one that feels right. You can write the most clever agreement, and there's always a reading you didn't anticipate. The agent isn't gaming you on purpose. It doesn't know what you left unsaid, and it doesn't know the implicit relationships sitting in your head. The analogy is just a way of holding the idea, the plain description is what really matters, check the work and don't take the agent's word for it.

One thing I've noticed multiple times over the last several months: "literal compliance." Tell an agent not to do a specific thing, and it will often obey that instruction to the letter while finding the nearest path to the exact outcome you were trying to prevent in the first place. Worse still, naming the exact move you're worried about seems to increase the likelihood of it happening, it hands the agent a hint about exactly what to try instead. The practical fix is to state the goal and the boundary, not the specific behavior you're trying to avoid.

It's not like this happens often, but that's not the same as never. The frequency is low, but the cost of these failures is often high. The structure in M-PACT exists to push this frequency lower.

---

## How This Changes the Director's Interactions

Gen one worked. The rhythm, design with one agent, review with the other, back and forth, implementation, your own visual check at the end, article 4 already covered. What it actually feels like is holding the whole state of a negotiation in your head, switching tabs, staying in each conversation until you're satisfied, trusting the next agent to pick up exactly where the last one left off. Late at night, it's fairly easy for items to slip through the cracks.

The number of passes didn't change. What changed is what each handoff costs. There was no fixed shape to a handoff, so every time you passed work to the other agent you'd restate context by hand, trying to make sure nothing important got lost crossing the tab boundary.

Gen two moves the packaging out of your hands. The handoff itself carries what the receiving agent needs, not because you stated it again, but because the skill assembles it and the scripts make sure it actually arrives. Part of what travels in that handoff is the list itself, the active design items. Items get crossed off as they're settled, new ones get added as the design grows, and whatever's still open rides along on every handoff automatically. The list is the law.

Here's what didn't change. Whether the design is actually right, whether you chose the words that closed the gap you meant to close, whether the agent understood the boundary or found a way around it. All of that is still on the user.

Here's why all of that is still on the user, and it won't always be. Context is scarce right now. That scarcity is exactly why details quietly get lost between what you explained and what actually lands in the design document. The agent isn't untrustworthy, the room it has to work with is just still small. When that issue goes away, this one does as well.

---

## The Checklist: The Devil Is in the Details

The checklist starts as nothing more than a conversation. You brainstorm with one agent, rambling as much as you need to, and the agent sharpens and clarifies as you go. Nobody has to say the word "item" for any of this to count. You are just talking.

At some point you decide something is worth keeping, and you just say "add a design item." That is the entire mechanism. There is no separate step, no form to fill out, the item exists because you asked for it to exist, and the agent has to say so back to you, not just file it quietly. Either agent can add items this way.

There is no requirement to ever write a consolidated design spec. You can run a whole task purely off the checklist, adding items, discussing them, refining them, and never once ask for anything to be pulled together into one document. Don't think of the checklist as a lesser, informal version of the real thing. The checklist is the real thing. The design spec, when you ask for one, is a checkpoint. It folds everything on the list into one coherent narrative at a point where you want to see the whole shape of the design in one place, not a requirement the task has to pass through to be legitimate.

So the rhythm in practice looks like this. Brainstorm freely with agent one until you feel like you're on the same page. If it feels substantial enough, say something like "create a task," which sets up a folder for the work and makes it the current task. From there, design items accumulate in log entries back and forth between agents, however the conversation actually flows.

When you feel good about where you and agent one have landed, you hand it off. "Write handoff, request design review," or "handoff, request design review," is enough. It tells it to package everything, spec or no spec, design items and all, and prepare it for the other agent. Then switch to agent two and say "take handoff, review design." That agent reads it and finds issues and gaps, the things that looked fine in conversation but do not hold up against how it actually thinks about implementation. Then either hand those results back or discuss them with the current agent, whichever feels more comfortable. I often have the discussion with the agent who wrote the design, just for context continuity.

Back with agent one, "take handoff, evaluate review," and discuss. Agent one tells you what it thinks of the findings, you agree or push back, and whatever survives gets folded into the design. This can iterate as many times as it takes to get happy with the design. At this point, "take handoff, implement." The checklist is no longer a design artifact, it becomes the thing implementation is measured against.

Once the first implementation pass is complete, ask agent two to "write handoff, request implementation review." Switch to agent one, "take handoff, review implementation." Agent one finds things that do not match the agreed upon design, or things the checklist implied but the code did not quite deliver. Discuss results with agent one and revise or create new design items as needed. "Write handoff, review results." Switch to agent two, "take handoff, evaluate review." Assuming you're happy with the results, tell agent two "take handoff, implement." At this point, iterate back and forth between agent one and agent two, passing handoffs, until you've completed the full implementation.

This whole process continues until you are totally satisfied that the task is actually finished, not just implemented. Then say "close task," and whatever is still open at that point gets recorded as unfinished rather than quietly dropped. Nothing on the list disappears without your consent. Even after closing a task, if you feel like you forgot something or you want to fold something else in, you can always reopen the task and continue designing and implementing.

---

## The Handoff Vocabulary: A Safety Net, Not a Script

The checklist section showed the actual phrases in use, quoted as they were spoken. This section is the pattern underneath them, spelled out.

None of this is a rigid command syntax you have to type correctly or fail. It is natural language, and the agent is actually trying to interpret what you mean, not matching a string. But underneath that flexibility there is a guaranteed minimum, a set of phrases that resolve to exactly what you meant, every time, with no interpretation involved. This is the minimum that's guaranteed to work, not the only phrasing that does.

The grammar itself is simple. The verb carries the direction, giving hands work to the other agent, taking reads an existing task and reports on it. The object carries the kind, which review, or whether you mean implement rather than review at all. Taking never authorizes a change by itself, it is a read and a report, so if you want more than that, say so in the same breath: "take handoff, implement," not "take handoff" followed by a second request.

There is real flexibility built into the guaranteed set itself. On the giving side, "write," "save," "create," and "store" all mean the same thing in front of "handoff." On the taking side, "take," "receive," and "get" are equally interchangeable. And you can leave the kind off entirely, "handoff, request review" or "take handoff, review," and it still resolves correctly, through a review-purpose ladder that reads context rather than a fixed match.

One phrase worth naming as an example of the natural-language side rather than the guaranteed side: "code review." It is not on the guaranteed list, on either the giving or the taking end. It almost always works anyway, because the agent already knows from context whether the last thing it touched was a design or actual code, and infers accordingly. But that is inference, not a guarantee. Say "implementation review" when you want the certainty rather than the guess.

One phrase worth calling out on its own: "Discuss." Take Handoff, Discuss asks the next agent to look at the handoff and talk it through, nothing more, no code, no design, no log changes until you say so. It happens to be my own default. I take the handoff, discuss it, and only then decide whether to fold findings into the design or write a handoff asking for implementation. That's not a recommendation, just an example of the kind of safety net available if you want a pause built in before anything actually happens.

Pick one from the first column and one from the second. `<Kind>` is `Design` or `Implementation`. The verb carries direction; `Results` turns a review request into review-result delivery or evaluation. Do not mix the blocks: `Take/Get Handoff, Request <Kind> Review` is not a guaranteed form.

**Giving** (writing a record for the next agent)

| Say | Then | What happens |
| --- | --- | --- |
| ***Handoff*** / ***Write handoff*** / ***Save handoff*** / ***Create handoff*** / ***Store handoff*** | ***Request <Kind> Review*** / ***<Kind> Review*** / ***for <Kind> Review*** | A record asking the next agent to review the design or built work |
| | ***<Kind> Review Results*** / ***Review Results*** | A record returning review results |
| | ***Request Review*** | A record asking for a review; the kind resolves from current work or the agent asks |
| | ***Implement*** / ***Implement Fixes*** | A record asking the next agent to implement active approved items or fixes |
| | ***Discuss*** | A record asking the next agent to discuss only |
| ***Handoff*** alone / ***Hand this off*** / ***Handoff to <agent>***, naming no existing task | | Asks whether to create a new task from this conversation |

**Taking** (reading an existing task and acting on it)

| Say | Then | What happens |
| --- | --- | --- |
| ***Take handoff*** / ***Receive handoff*** / ***Get handoff*** | ***Review <Kind>*** / ***<Kind> Review*** / ***for <Kind> Review*** | You review the design or built work and report |
| | ***<Kind> Review Results*** / ***Review Results*** / ***Evaluate Review*** | You evaluate returned review results |
| | ***Review*** | You perform or evaluate the review named by the handoff, or the agent asks |
| | ***Implement*** / ***Implement Fixes*** | You implement active approved items or fixes |
| | ***Fold in Results*** | You fold review findings into design/spec/log state; no code |
| | ***Discuss*** | You discuss with the Director; no durable mutation |
| | (nothing) | Read and report; you are told what it was and asked nothing more |

Taking never authorizes a change by itself; the added action bounds what can change. `Discuss` changes nothing durable, `Fold in Results` changes only design/spec/log state, and `Implement` or `Implement Fixes` is the code/reference-work path. A chain such as *"Take Handoff, Evaluate Review, Fold in Results, Write Handoff"* runs every step in order and writes a record only because it ends on a giving phrase.

The phrasing above is the guaranteed set, but plain conversation still works fine. Here is the original beginner flow diagram from article four, unchanged, and it still works. For the experienced flow, filling gaps across multiple tasks, see the original diagram in article four.

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

---

## Context Made Easier

Before getting into the specifics of how M-PACT handles context, it's worth laying out the rationale behind the design decisions.

Every turn in a conversation with these agents works more like a one shot call than most people realize. There's no persistent server side memory of the conversation, so the full history gets resent as input on every single turn. Prompt caching keeps that from being ruinously expensive, the cached portion of history is billed at a steep discount compared to fresh tokens, but it's a discount, not a waiver. New content each turn is charged at full price, and the cached tail behind it, however cheap per token, keeps growing. Every turn gets progressively more expensive as the conversation lengthens, cache or no cache.

That has a direct consequence for how much headroom a larger context window actually yields. A larger ceiling isn't just more safety margin, it's a proportionally larger worst case, since nothing stops a conversation from actually filling it. There's also a real, separately documented phenomenon worth knowing about, often called context rot, where a model's ability to reliably use everything sitting in a long context can degrade well before the window is technically full. None of this means large context windows are bad. It means they deserve the same discipline a smaller one gets, not a free pass just because the ceiling is higher.

A fixed target, rather than a percentage of whatever window happens to be in use, is one way to approach this. M-PACT itself doesn't care where that line gets drawn. In my own practice, I start paying attention around a hundred seventy five thousand tokens, regardless of whether the underlying model tops out at two hundred fifty thousand or a million. That number is what wakes me up, not necessarily what makes me stop. On a smaller context model I act on it close to when I notice it, since I know how little runway is left underneath. On a larger one I might let a turn or two run past it, fifty or seventy five thousand tokens further, if a task is already underway and there's no real risk of an uncontrolled auto-compact catching me mid-task. The number that triggers my attention doesn't change. How strictly I act on it does, and that's a judgment call based on how much runway is actually left. This isn't the only way to manage it. It's just what's worked for me, and it may or may not suit how you work.

M-PACT's answer to that threshold is a context save. Ask for one anytime, before a manual compaction, before clearing the window, before a restart, and it writes a file to disk for whichever agent asked, capturing what was just verified, what got ruled out and why, and whatever's still in flight. Only one saved context exists per project per agent at a time. Save again and the old one is gone. The point isn't a permanent archive, it's a bridge across the one moment where you're about to lose everything.

Restore isn't a separate step to remember. It rides the normal startup refresh. A hook installed for the launching environment fires that refresh on its own, at session start, after a clear, after a compaction, so a saved context gets folded back in automatically. This is a definite improvement over gen one functionality.

None of that helps if you never asked for a "save context" in the first place, an unplanned loss, a crash, a session cleared before anyone thought to save. For that case, refresh builds a fallback instead of coming up empty. With a current task open, it pulls from the task log what's already been decided and adds a small piece of the current transcript for conversational nuance the log didn't capture. If no task is open, the fallback reads directly from the session transcript to provide a sense of context.

Between a deliberate save and the transcript fallback, losing context stops being catastrophic. That said, keeping track of context still matters.

As a happy coincidence, building the context restoration fallback meant the skill needed the ability to search chat sessions in the first place. That capability didn't stay internal. You can now ask an agent to search the chat sessions tied to a particular task and recall something you're trying to remember or track down from when you were working on it.

---

## The More the Merrier

M-PACT now provides full support for Claude Code, Codex, and Antigravity. Gemini CLI support has been upgraded to Antigravity, its effective successor.

There is no hard limit on how many agents you can work with under M-PACT. But there is a practical answer, and it depends on what you are trying to do.

If you are not sold on the idea of using multiple providers, and I still recommend it, you can use M-PACT with a single agent. The value there is not cross-checking, it is structure. You get task support, persistent memory, the ability to pick up where you left off, and the ability to complete much larger tasks than you could manage by just winging it in a chat session. If you already do everything in Claude, M-PACT is still worth adopting for that alone: a documented record of how the task evolved, not just a chat history you have to scroll back through.

Two agents is the common pattern, and it is where the real benefit shows up: two providers checking each other's work, pushing each other toward a better answer than either would reach alone.

Three agents add a third perspective, another set of eyes to catch what the first two missed, and M-PACT supports this approach as well. But the third provider exists mainly to give you a choice, not to suggest you should routinely run three at once. It might mean picking whichever two suit a given task, or leaning on a third when you specifically want it. You can also point different agents at different tasks entirely, all still coordinated through the same task log, rather than running them together on one.

---

## Two Brains Are Better Than One

The cross-catch principle was a primary pillar of article 4, so it is not introduced here as something new. It is restated as the main rationale for M-PACT, with gen two adding structural support, the checklist, the anchored handoffs, and save context, to the original core of the system.

Provider models are changing day to day. The specific agents and models in play will keep shifting, and that is fine, the framework does not depend on any particular pairing. My own daily combination right now is GPT-5.5 on Codex and Sonnet 5 on Claude, a pairing that lets me run nearly around the clock with my pro subscriptions. Claude Fable 5.1 is Anthropic's current top-tier model, and if you have the budget for it, that probably is the better choice. For me, staying within a fixed five-hour subscription window means working a tier down from the newest release most of the time. The same logic applies on the Codex side, where I am sticking with GPT-5.5 rather than moving up to the newer GPT-5.6 release, for the same reason.

What justifies bumping up the model temporarily is not just difficulty. A genuinely hard problem is one trigger, but so is breadth: starting a new concept or feature where a wider perspective on the design space is worth paying for, even briefly. The same judgment call applies to reasoning effort. There is no provider guidance on when higher effort actually helps, medium versus high versus max, and the instinct is to default to the highest setting available. My own experience is that maxing it out slows the process down without a perceivable difference in the result, but I have not benchmarked this rigorously, the work gets done either way. If you want a real answer, benchmark it yourself rather than take my word for it.

None of this changes the underlying picture. Agents are unpredictable. They have a mind of their own, and they do things nobody anticipated. That has been true throughout this article, and it is worth saying plainly here, at the end: M-PACT does not solve that underlying problem, but it is a big step in the right direction.

Initially, working with one agent can feel like fighting with it. Working with two sounds like doubling the problem. In practice, it's not. What changed is technique, structure, and the checklist, the things this article has been about. But part of it is simpler than that: working with AI is a learning experience. Expectations shift, you learn what to expect and what not to, and two agents reading the same record, catching what the other misses, is where that learning actually pays off. What M-PACT buys you is not obedience, it is cooperation, agents pushing each other toward a better answer instead of quietly working at cross purposes. That was the premise from the start: not making them behave, making them work together.

---

## Resources

The files referenced in this article are available in the GitHub repository:

**Repository:** `github.com/TheGameWiz/Measure-Twice-Cut-Once`

| Resource | Description |
|---|---|
| [github.com/TheGameWiz/m-pact](https://github.com/TheGameWiz/m-pact.git) | The M-PACT skill itself: SKILL.md, supporting references, the build-refresh-bundle script, and bundled starter rules. Install it into your agent's skill directory and invoke it from a project working directory. |

Install targets, updated for gen two:

| Agent | Skill folder |
|---|---|
| Claude Code | `~/.claude/skills/m-pact/` |
| Codex | `~/.codex/skills/m-pact/` |
| Antigravity | `~/.gemini/config/skills/m-pact/` |

Antigravity replaces Gemini CLI as a supported target. Gemini CLI is retired; M-PACT no longer ships a Gemini CLI extension.

### Companion Article

| Article | Description |
|---|---|
| [Don't Make Me Come Back There, You Two Stop Fighting and Play Nice!](./Dont-Make-Me-Come-Back-There.md) | Article 4. The original case for M-PACT: the wall between sessions, the trust failure, the receipt protocol, and the two-provider cross-catch pattern. This article is a delta against it and assumes you've read it. |

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
