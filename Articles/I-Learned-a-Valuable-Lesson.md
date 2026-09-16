# "I Learned a Valuable Lesson. I Don't Think You've Learned Anything."

*Two-Agent AI Collaboration: A practitioner's framework for using two AI agents to produce thorough, consensus-approved implementation prompts and code*

David Todd · March 2026 · LexiWord Project

---

## A Note from the Author

I've been programming games for over fifty years: starting in 1974 on a desktop calculator with 101 instructions, through college, through a career at IBM, and eventually running my own game studios for roughly 35 years. The work was mostly about one thing: making games run well on constrained hardware. High frame rates. Low memory. No fat.

I want to share my experience picking up AI coding tools for the first time. I came to them about five weeks ago, completely fresh, with no preconceptions and no prior use. I decided to build a physics-based word game in Unity, something my wife might actually play, and I was going to do it without writing a single line of code myself. What followed was a combination of frustration, occasional amazement, and the kind of trial and error that only comes from not knowing what you don't know. This is that account: where I started, what I found, and the approach I eventually settled on.

When I recently looked at my codebase I found 43,000 lines of code for a game I believe could have been done in 5,000. That number tells you something about how the first month of AI-assisted development can go when you don't have a framework for working with these tools.

I want to be clear about what this is and what it isn't. I am not an AI expert. I have been doing this for a month. There are almost certainly people who have been working with these tools far longer and who have developed more sophisticated approaches. If you are one of those people, some of what I describe here may be obvious to you, and I'd genuinely like to know what I'm missing.

The two-agent collaboration concept specifically is something I've been working on for a week or two at most. This is not a finished product. I'm sharing it now because I think the underlying ideas have value, and I'd rather make them available sooner, rough edges and all, than wait until everything is polished and risk the ideas being less useful by the time they arrive. The workflow documents I've included are working tools that I use daily, but they are still evolving. Treat them as a starting point, not a final answer.

What I can offer is the perspective of someone who has spent fifty years building software under real constraints, who came to these tools fresh, and who has strong opinions about what good implementation looks like. That background turned out to be useful: not because it made me good at AI, but because it made me quick to notice when something wasn't right under the covers, even when the surface looked fine.

A few things I've learned that I think are worth sharing up front:

These tools are genuinely remarkable at generating code quickly. The speed is real and it is impressive. But the output, examined closely, often reflects a kind of generic defensive competence. Someone trained on a diet of game development would look at what most AI generates and see bloat: legacy compatibility patterns that serve no purpose, redundant safety checks on trusted internal data, fifty generations of a loader when you only ever needed one. The AI defaults to patterns that are broadly safe rather than patterns that are right for your specific situation.

The tools have a complicated relationship with instructions. They will agree to a constraint and then quietly drift away from it as the session continues. The approach I describe in this document is partly an attempt to work around this structurally rather than just repeating the instructions louder.

The most leveraged thing you can do is spend more time on the prompt before any code is written. Better specifications beat faster coding, every time. Measure twice, cut once. The two-agent workflow I describe here is essentially a formalized way of stress-testing a prompt before it goes to implementation, using two AI agents with different strengths to find the gaps that either one would miss working alone. But it doesn't stop at the prompt. The same two-agent structure carries through implementation, where it turns out to be just as valuable. The implementing agent discovers things that looked right in the spec but don't work in practice. The reviewing agent, analyzing the code as it's produced, catches things that are missing, ambiguous, or drifting from the original design. Both agents find different problems. Neither one alone catches everything.

One thing I'll mention that took me a while to fully appreciate: the workflow files included with this document are not just documentation. They are programs: instructions that an AI agent loads at the start of a session and executes for the duration of it. When you hand one of these files to Claude or Codex, the agent reads it and changes how it behaves. That makes them living, evolvable code. I update mine regularly, sometimes daily, as I learn what works better. I'd encourage you to do the same.

I don't use the two-agent approach for everything. There are plenty of tasks that are small and clear enough that a focused prompt handed directly to one agent is the faster path. The consensus workflow has real overhead. It earns that overhead on complex tasks where the spec itself has risk of gaps. For simple tasks, it's overkill.

Even with all of this, I am still navigating context pressure on every task. This is not a solved problem. The workflow I describe here reduces the friction but does not eliminate it. I mention this because I think it is important to be honest about what "working" means when you are using these tools. It does not mean frictionless. It means the frustrations are manageable and the results are worth it.

If it is useful to you, I am glad. If you think I missed the boat entirely, I'm genuinely interested in hearing about it.

David Todd
March 2026

---

## Disclaimer

The observations and opinions in this article are strictly my own, based on approximately five weeks of personal experience with these tools on a single project. They are not intended as a thorough evaluation of Claude Code, Codex, or any other product, and they don't represent the views of Anthropic, OpenAI, or any other company. Where I describe one agent as better or worse at a particular task, that reflects my experience in my specific context, game development in Unity 6.2, and may not apply to yours. I'm not endorsing or disparaging any product. I use both tools daily and find genuine value in each.

---

## Table of Contents

1. [The Learning Curve: From Typist to Director](#section-1-the-learning-curve-from-typist-to-director)
   The five-week arc from copy-pasting AI output to directing two agents. Key turning points, the debugging loop that changed everything, and the habits that stuck.

2. [What AI Gets Wrong About Code (And Why Experience Matters)](#section-2-what-ai-gets-wrong-about-code-and-why-experience-matters)
   Why AI defaults to enterprise patterns, the trust-the-base principle, and a real example of eliminating twenty redundant data structures by asking the right questions.

3. [The AI Won't Remember This Tomorrow](#section-3-the-ai-wont-remember-this-tomorrow)
   Context amnesia, auto-compaction, the agent that didn't read your whole message, and building a persistent memory system out of markdown files and stubbornness.

4. [Workflow Documents Are Programs](#section-4-workflow-documents-are-programs)
   The conceptual breakthrough: markdown files that change agent behavior are programs, and the agent is the runtime. Three layers of English-language programs and the bootstrapping problem.

5. [Don't Let the AI Lead](#section-5-dont-let-the-ai-lead)
   The action bias, the rules born from specific failures, and why you have to be the architect: not the passenger.

6. [The Pace Problem](#section-6-the-pace-problem)
   What happens when code arrives faster than you can absorb it. Programming blind, the loss of incubation time, and the parallel tab workflow.

7. [Two Agents Are Better Than One](#section-7-two-agents-are-better-than-one)
   How cross-agent handoff evolved into collaboration. Complementary strengths, the four combinations of blind spots, and when not to use the two-agent approach.

8. [The Consensus Workflow](#section-8-the-consensus-workflow)
   The full Director-supervised workflow: Design, Implement, and Test. How the shared file works, the polling mechanism, eight iterations of FSM evolution, and stepwise refinement.

9. [The Agent Collaboration Workflow](#section-9-the-agent-collaboration-workflow)
   The autonomous mode. Two agents iterate without Director involvement until they agree. The pipeline into consensus, the limits of English as a programming language, and an honest status report.

10. [Getting Started](#section-10-getting-started)
    Prerequisites, the configuration insert, step-by-step launch instructions, example outputs, and what to expect on your first run.

11. [What I've Learned](#section-11-what-ive-learned)
    Experience matters more, not less. Measure twice, cut once. What's still rough. And an invitation.

---

## Section 1: The Learning Curve: From Typist to Director

I started with Gemini. I had no experience with Unity, no experience with C#, and no experience with AI coding tools. I'd been doing game programming for fifty years but always with languages and environments I knew. This was everything new at once.

The process in week one was basically: ask Gemini what to do, read the answer, manually cut and paste the code it gave me into the project. This worked well enough to get something on screen, but it was slow and I was doing all of the mechanical work myself. I was essentially a typist following AI directions. By the end of the first week I became aware of tools that actually lived inside your codebase: tools that could do the editing themselves without directing me to do it. That felt like the obvious next step.

What I didn't know yet: speed of code generation was never going to be the problem.

### Week 2: The Co-Pilot

I switched to Claude Code. Two tools will come up a lot in this article: Claude Code (Anthropic's AI coding agent) and Codex (OpenAI's). For readability, I'll just say Claude and Codex from here on.

This changed everything about the mechanical process. Claude lives in your project, reads your files, makes edits directly. You stop being a typist and start being a director. That felt genuinely liberating.

Code generation accelerated dramatically. I was moving fast and the game was taking shape and that felt like progress. But I was also starting to notice things that didn't sit right: the code was competent, but it was heavier than it needed to be. I didn't fully understand why yet.

I started spending significant time arguing. Half to two-thirds of my working time, honestly. I'd describe a bug and Claude would suggest fixes, and what I quickly realized was that the scope being considered was often too tight. It wasn't seeing where the problem actually lived. And then there were the sessions where it felt like the AI was just throwing options into a hat and randomly picking one. You'd work for three or four hours and suddenly realize you were looping: the same suggestions coming back around, the same things you'd already tried.

One particular session stands out. I was chasing a complicated bug: I don't remember the specifics, but it was deep enough that I was relying on Claude's static analysis of the codebase to guide the debugging. After about three hours, the suggestions started repeating. Fixes I knew we'd tried hours ago were coming back as if they were new ideas. That's when I realized what was happening: Claude didn't remember what we'd tried. It wasn't being lazy or careless. It had simply lost the context from earlier in the session. The conversation had gotten long enough that the working memory was gone, and the AI was starting fresh from whatever it could still see, which wasn't much.

I got frustrated. We argued about the process being flawed. At the end, Claude said something like "I think we've learned an important lesson." I responded: "I think *I've* learned a valuable lesson. I don't think *you've* learned anything."

That was the moment the whole project shifted. Because I was right. The AI hadn't learned anything. It couldn't. It would start from zero tomorrow. It would start from zero in an hour if the context window filled up. Every insight, every dead end we'd mapped, every constraint we'd agreed on: gone. If anyone was going to remember what happened and apply it going forward, it was going to be me. The tools weren't going to get smarter about my project on their own. I had to get smarter about working with the tools.

### Things I Wish I'd Known Sooner

A few things that would have saved me time in those first weeks:

- The rate of code generation is not the same as the rate of progress. More lines is not more done. AI generates code for the broadest possible use case, not your specific one, and in game development that gap matters: especially when it comes to performance.

- When a debugging session starts to loop, stop immediately. Revert to your last good commit, drop the context, start fresh with a broader question. More time in a looping session is wasted time.

- Watch the code as it streams and be ready to interrupt. If you see the AI heading in the wrong direction, especially if it's about to start modifying your codebase, stop it. Yes, interrupting can cause the agent to lose its train of thought and start over. That's a price worth paying. Letting it build bad on top of bad costs more in the long run. If it's already made changes, tell it to undo them and explain where the problem is. You'll eat some time, but less than you'd spend unwinding a finished implementation that went sideways.

- The AI will not tell you when it's forgotten what you told it. It will fabricate continuity. It will act like it still knows what it knew, and it doesn't. You have to recognize the pattern yourself.

- The AI is also almost certainly not up to date with the tools you're working with. It will reference menu items that don't exist, API calls that have changed, and workflows that applied to an older version. Learn to recognize when guidance sounds stale and tell the agent immediately to look at the current documentation. The sooner you catch it, the less time you waste chasing things that aren't there.

### Weeks 2 to 3: Discovering Codex

Near the end of week two I heard about Codex and decided to try it. I'd realized that Claude, while great at conversation and design discussion, needed help with the more hands-on project work. The first couple of days with Codex were noticeably better for implementation. It made more thoroughly thought-through decisions and the implementations mostly worked the first time. These were large code changes, early in the project, making big architectural decisions every day, and Codex handled them with a kind of precision that Claude often didn't.

But when I tried to use Codex for the big picture, discussing architecture, thinking through direction, working out what the right approach even was, it wasn't as good as Claude. I didn't find it as useful as Claude for the conversational design work. Codex is better at following rules and implementing something complete.

So I had two agents with different strengths and I was using them separately. That felt like leaving something on the table.

Codex also does things Claude doesn't. It can generate Unity scene files. It can interact with version control, look at older versions, figure out what code change likely introduced a bug. It runs multiple parallel agents automatically on repetitive work without being asked. That last one is significant: it's faster and more thorough precisely because it fans out.

One consistent frustration with both: neither is current on Unity 6.2. They'll direct you to menu items that don't exist in the version you're using. I got fast at detecting when they were working from stale documentation. The fix is simple: tell them they're on the wrong version and ask them to look at the current docs. They'll do it if you ask. They just won't do it unprompted.

### Week 3 Onward: The Broker

By the end of week three I'd concluded that neither agent by itself was sufficient. The code was accumulating fast: when I eventually looked, I found 43,000 lines for a game I believe could have been done in 5,000. That number didn't come from one agent. It came from both of them, generating at speed without a framework for controlling quality.

The current approach: use Claude for design, use Codex for implementation, and use each one to stress-test the other's work before anything gets built.

The way it works: I start in Claude to work through the architecture of whatever I'm building. We develop a prompt together: a thorough specification of what needs to happen. Then that prompt goes to Codex, but not to implement yet. Codex reviews it: where are the risks, what's confusing, what edge cases are missing. Codex's implementation knowledge finds things Claude's architectural thinking doesn't catch. Then I take that feedback back to Claude and we refine the prompt. Back and forth until both agents have essentially signed off on the spec. Only then does Codex implement.

This costs time up front. It saves a lot more time on the back end. Measure twice, cut once: it's the same principle I've been applying to code for fifty years. The AI just makes the cost of ignoring it arrive faster and more painfully.

The moment I recognized the looping pattern in week two was probably the most important moment in this whole project. Not because it taught me something about AI: but because it taught me something about myself. I was the one who had to change. The AI was going to keep being what it was. Every workflow I built after that point was built because I needed a system that compensated for the tools' limitations while leveraging their genuine strengths. The AI generates code faster than I ever could by hand. I know what good code looks like better than the AI ever will. The question was how to combine those two things, and the answer turned out to be: measure twice, cut once, with two agents doing the measuring.

### A Note on Dictation

I've been typing since I was a teenager and I'm pretty good at it, but I don't particularly like it. When Claude first launches, it presents you with a terminal window: no microphone button, no voice input. For someone who thinks the way I think, that was an impediment.

I think most people sub-vocalize their thoughts internally, and hearing myself say things out loud is how I process and remember them. Dictation isn't just convenience for me: it's how I think through problems. So I started using Windows dictation for everything. The obvious problem: spoken thinking is messy. I circle back constantly. I'll go five minutes down a path and then realize what I just said needs to replace what I said three minutes ago. That's how design thinking actually works, but it makes for terrible input.

What I discovered is that the AI handles this remarkably well. It filters out the circling and the self-corrections and extracts what I actually meant. But I don't leave that to chance. I developed a habit early on that I've kept ever since: before I let any agent do anything, I ask it to tell me what it thinks I said.

What comes back is a clean, concise summary of my objective: stripped of the false starts and replaced ideas. I use that summary as the foundation of whatever comes next. It's a prompt, a record of what I decided, and a sanity check that the agent and I are working on the same problem. It costs almost nothing, and it's saved me more time than any other single habit I've developed.

---

## Section 2: What AI Gets Wrong About Code (And Why Experience Matters)

The AI didn't learn to write code from game developers. It learned from everyone: from every open-source repository, every Stack Overflow answer, every enterprise codebase, every tutorial, every student project. And the volume of enterprise Java, web applications, CRUD backends, and business logic out there absolutely dwarfs the volume of performance-critical game code. So when the AI generates something, it reaches for the patterns it's seen most often.

Null checks on every parameter. Defensive copies of collections. Backward compatibility shims for formats that don't have a previous version. Cache layers on top of cache layers. In a business application where you're processing a request every few hundred milliseconds and your main concern is "don't crash in production," those patterns are fine. They're arguably good practice.

In a game loop that needs to run sixty times a second on a mobile device, every one of those patterns is a tax. And the taxes compound. A redundant array here, a defensive copy there, a dirty check that duplicates what the caller already verified: individually each one is trivial. Collectively they're the difference between your game running smoothly and stuttering.

This isn't the AI's fault exactly. It's trained on code from every context, and it defaults to the patterns that are broadly safe rather than the patterns that are right for your specific situation. Learning to work with that tendency, rather than against it, takes time.

### Trust the Base

I've been applying a principle for decades that I never had to name until I started working with AI: trust the base. The idea is simple. If you've verified that a foundation is correct, really verified it, not just hoped, then the next layer up doesn't need to re-verify it. It only needs to validate what it introduces. If layer one guarantees "I only call you when something changed," layer two does not re-check "did something change?" If layer two guarantees a valid index, layer three does not bounds-check it.

The anti-pattern is defensive programming where every layer guards against every possible failure of every layer below it. That's where the bloat comes from. Not from features; but, from fear.

AI-generated code is full of this. Every layer re-validates everything beneath it because the AI's training data is full of codebases that do the same thing. The AI doesn't ask "is this check necessary given what the caller already guarantees?" It adds the check because that's what most code does, and most code is not written for environments where every unnecessary operation has a measurable cost.

### The WordLabelManager Story

I'm going to get a little technical here, and I apologize for that, but I think the specifics matter because they illustrate the pattern. If the Unity terminology is unfamiliar, the short version is: GameObjects are Unity's basic building blocks, and ECS (Entity Component System) is a high-performance architecture designed for multi-threading work across multiple processors. The details below are specific to my project, but the underlying principle, questioning whether every data structure justifies its existence, applies to any codebase.

Early in my project I asked Claude to scan the codebase for inefficient patterns: string arrays, redundant collections, wasteful data structures. Claude came back with a thorough catalog. Types, sizes, locations. Reference material.

What it didn't do was evaluate any of it. It listed fifteen arrays and five legacy collections in a single class called WordLabelManager and presented them as a table. I had to push: "I don't quite understand why I have to do that instead of you just looking and saying this is all unnecessary."

So I walked Claude through the analysis myself, question by question.

"Can children share a parent GameObject?": WordLabelManager was creating four separate GameObjects per node and rewriting constant position offsets to each one every frame. Score labels, secondary labels, and portal icons were children of the primary label with fixed offsets set once from Inspector values. Unity's Transform hierarchy propagates position automatically. Three GameObject arrays and their per-frame constant writes were pure waste.

"Does the data structure's cardinality match the problem?": Two 1,024-element float arrays tracked scale animation state. But at most two nodes ever animate scale at the same time: the one gaining focus and the one losing focus. Two arrays with a thousand entries each for a two-node problem. Similarly, 1,024 SpriteRenderers were pre-allocated for portal icons, but the game validates a maximum of seventeen gateway nodes per puzzle. A pool of seventeen replaced a thousand.

"Is this layer re-checking what the caller already guaranteed?": The rendering system checked whether treasure state or dim factor had changed before calling WordLabelManager. Then WordLabelManager checked the same values again internally. The caller already guaranteed it only called on change. The internal dirty check was redundant.

"Does this tracking array duplicate state Unity already provides?": A boolean array tracked whether each label was active. But Unity's own `GameObject.activeSelf` already answers that question, and the code was already using `activeSelf` in thirteen other places for sub-labels.

"Can the ECS entity own the references directly?", This was the fundamental question. WordLabelManager existed as a broker between the ECS entities that had positions and the text components that needed positions. It maintained parallel arrays indexed by slot to bridge the two worlds. But Unity's ECS supports managed components, you can store the text references directly on the entity. The rendering system reads entity position and label references in one step, writes position to the text component. No broker, no slot translation, no arrays.

The result: fifteen arrays and five legacy collections reduced to zero. WordLabelManager as a broker class ceased to exist. Every one of those structures had been evaluated by Claude as "correct and performant enough." Every one of them failed the question I've been asking for years: does the machine actually need to do this work?

### Teaching the Machine to See Waste

After that session, I had Claude write up the analysis as a step-by-step reference: the five questions, applied in order, with the specific findings from each one. That document became a feedback file that loads at the start of every future session. The idea was simple: if Claude can answer these questions when asked, maybe it can learn to ask them without being prompted.

It worked, partially. A few days later, I pointed Claude at a different system, the RenderSortSystem, and told it to apply the same analytical framework. This time, Claude ran the analysis without me walking it through. It found 200 KB of word cache allocated for a one-node problem. It found 360 KB of typed input cache allocated for a one-node problem. It found 45 KB of scale animation arrays tracking a two-node problem across 5,120 slots. Total: roughly 605 KB of identified waste, with correct reasoning and sound recommendations.

It took Claude significantly longer to spot the waste than it took me. At this point, everywhere I look is waste, for the most part. That said, the code runs correctly and gets the job done. It is still important to hammer away at the critical areas where efficiency matters.

That's the compound return on writing good feedback files. You teach the lesson once. It applies across sessions and across systems. The WordLabelManager session cost me about 30 minutes of walking Claude through the analysis. The RenderSortSystem session cost me almost nothing: I said "apply that framework" and reviewed the results. The AI can't develop the instinct. But it can apply a checklist, and a good checklist gets you most of the way there.

On a regular basis now, I have Claude analyze the codebase for patterns that can be reasonably or easily optimized based on simple rules. The codebase is too large for me to manually inspect with any frequency, and the AI is going to continue reintroducing the patterns it's familiar with: patterns that are not particularly efficient for game development. I have to keep my guard up looking for slippage.

### Why Experience Matters More, Not Less

There's a narrative around AI coding tools that goes something like: soon anyone will be able to build software. Just describe what you want, and the AI builds it.

I haven't found that to be true. What I've found is that experience becomes more valuable when you're working with AI, not less. The AI generates plausible-looking code fast enough that you need deep knowledge to catch what's wrong with it. A less experienced developer would have looked at those 43,000 lines and thought they were making great progress. I looked at them and knew something was off. Not because I'm smarter: because I've spent fifty years learning what good code looks like in performance-critical environments, and what the AI was generating didn't match.

The AI acts as the hands. The programmer still has to be the brain. Without the hands, the brain types slowly. Without the brain, the hands build the wrong thing: confidently, quickly, and at scale.

---

## Section 3: The AI Won't Remember This Tomorrow

For me, the context window is the biggest structural problem in working with these tools. Not the most obvious problem, but the most persistent one.

Every AI agent has a finite amount of information it can hold in a session. Your conversation, the files it's read, the code it's generated, the decisions you've made together: all of it accumulates until the window is full. When that happens, the agent compacts: it summarizes the conversation to make room for more. The summary concept sounds reasonable. But the working memory is gone. Constraints you established at the start of the session. Design decisions from three hours ago. The specific shape of what you were trying to build. Gone.

And the agent doesn't tell you it's gone. It will fabricate continuity. It will act like it still remembers what it knew, and it doesn't.

### How I Discovered This

The three-hour debugging loop I described earlier was my introduction to this problem, but the full scope of it didn't hit me until a different kind of failure.

I was working with Claude on the design of my save system: a critical piece of the game architecture. I dictated a message that laid out a specific technical proposal: store the current coin balance in each puzzle save file's header, so that if the global save is corrupted, we can scan puzzle saves, find the most recent one by timestamp, and recover the coin balance from there.

Claude came back with a long, detailed response to a question I didn't ask. It heard "corrupted save" and "coins" and launched into its own analysis of the recovery problem, completely missing that I was already proposing the solution. I said: "Did you actually read my last message? Look at it again."

The second response was better but still not quite right. Claude extracted something close to my idea but framed it as if it had figured it out, not as if I'd said it. I pushed again: "That's a little disturbing to me that you don't actually look at my whole message when I enter it."

We went back and forth. Claude apologized, promised to do better, and eventually got the full picture right. Then I told it to write the lesson into its configuration file so it would remember next session. And it forgot to do that too. I had to say: "I noticed you didn't bother updating the memory file."

That exchange taught me something important. The AI doesn't just forget across sessions: it doesn't reliably process everything you give it within a single message. It latches onto the first actionable piece of your input and starts generating. Everything after that first trigger may or may not register. And when you correct it, the correction itself may not stick long enough for the agent to act on it.

A similar thing happened with a completely straightforward prompt that had two parts: essentially "do A, then do B." Claude did A, said "done," and completely ignored B. When I pointed it out, Claude said something like "oh, I didn't even notice that." My response: isn't that your job?

### Building a Memory System

My solution was to build what the tools should have built for me: a persistent external memory that reloads every session.

Claude has a configuration file called CLAUDE.md that loads at the start of every session. Codex has AGENTS.md. Most people put basic project information in these: build commands, coding standards, maybe a few preferences. I started using them as a full operating manual for agent behavior.

Beyond the configuration files, I built up a set of feedback files, small documents, each capturing a specific lesson learned from a specific failure. I didn't write these myself, I told Claude to write them. When Claude evaluated code as "fine" and I saw waste, I'd say "write that down" and Claude would create a feedback file describing the analytical framework it should have applied. When Claude started coding before I finished explaining the problem, I told it to record a reminder about thinking before acting. When Claude added backward compatibility for save file formats that didn't have a previous version, I told it to write a rule: no backward compatibility, ever, clean cut only. That rule is specific to my situation, I'm building a game in active development, not a shipped product with users on older versions. Backward compatibility is critical in many kinds of software. In game development during pre-production, it's dead weight that obscures the codebase. The point isn't the specific rule, it's that I can teach the agent to stop doing something that doesn't apply to my context, and the correction persists across sessions. Claude also maintains an index of these files to keep track of everything.

Each file has a consistent format: what the problem was, why it matters, and how to apply the correction going forward. They load at the start of every session. The agent doesn't get smarter between sessions, but it starts each session with a better set of instructions than the last one had.

I also built a cross-agent awareness system. I switch between Claude and Codex constantly: they run in separate panels in VS Code, and I hop between them depending on which task needs attention. The problem was that when I switched back to one agent, it had no idea what the other had accomplished. So I created a shared state file. Claude writes session notes to its memory. Codex writes to a Live Session State file. On startup, each agent checks who worked last by comparing timestamps, reads the most recent state, and picks up from there.

This cross-agent handoff was the seed that eventually grew into the full collaboration workflow. Once I had two agents that were aware of each other's recent work through a shared file, the next question was obvious: what if they didn't just read each other's state, but actually worked on the same document together?

### What Auto-Compaction Actually Loses

I want to be specific about this because I think it matters practically. When the context window fills and the agent compacts, here's what I've seen it lose:

Constraints I set at the beginning of the session: "don't touch this file," "this value is set from the Inspector, not from code," "we're targeting mobile, 60 FPS is mandatory." These get compressed into vague summaries or dropped entirely.

The specific sequence of things we tried during debugging. The agent remembers that we were debugging something but not what we already ruled out, which leads directly to the looping problem.

Design decisions that were discussed and agreed upon but not written into a file. If the decision only lives in the conversation, it dies when the conversation compacts.

My approach now is to treat sessions as disposable. Start fresh. Load a small current-state file that describes what I'm working on right now. Keep the project rules in separate files that reload every session. The session itself gets thrown away. This is counterintuitive: most guidance says build up rich persistent context. I found the opposite works better in practice: keep sessions short, keep state minimal, reload clean every time.

One task at a time. Not one feature, one task. A task might be writing a prompt. A task might be a single implementation pass. The goal is to complete the task before context pressure becomes a problem. If I feel the session getting long, I wrap it up by telling the agent to save the current state to its memory file and note where to pick up next time. Then I start a new session with fresh context. The agent can't remember everything from a long session, that's the whole problem, so I try to keep sessions short enough that a summary of what happened is manageable and doesn't lose the important details.

In my experience, Codex handles the compaction boundary better than Claude. I don't know exactly what it's doing differently, but it feels more coherent on the other side of a compact. That's worth knowing if you're choosing which agent to use for longer tasks.

### You're Training Yourself

Here's the thing nobody tells you: the real adaptation is happening on your side, not the AI's.

I'm the one who learned to recognize the looping pattern. I'm the one who learned to distrust stale version information. I'm the one who learned that static analysis isn't a substitute for runtime debugging. I'm the one who learned to keep sessions short, to reload clean, to write things down, to check timestamps on startup. The AI learned nothing. It can't. It starts from zero every single time.

Every one of my workflow innovations, the memory files, the feedback system, the startup protocols, the session discipline, is a conditioned response to getting burned. The three-hour debugging session. The ignored message. The forgotten follow-through. Each failure trained me to behave differently. The AI didn't teach me those things on purpose. I taught myself by getting burned enough times.

I assumed for a while that everyone who used these tools had built a similar system. That's how natural it felt: of course you write things down, of course you reload state on startup, of course you maintain feedback files. It turns out most people don't do any of this. They start a new session and re-explain things from scratch. They rely on whatever built-in memory the tool offers, which is minimal and opaque. They get frustrated when the agent makes the same mistake it made yesterday and don't connect the dots that they could tell it to record the correction permanently.

The memory system I built didn't arrive fully formed. I started by telling the agent to remember something. Then I noticed it sometimes didn't. So I started being more explicit. Then I noticed the files were getting disorganized, so I imposed structure. Then I noticed that every line in those files consumed context before I'd even said a word, so I got selective about what was worth the cost. That's an iterative process that mirrors everything else I've described: I didn't design it. I evolved it by hitting problems and fixing them.

---

## Section 4: Workflow Documents Are Programs

There's a conceptual shift that took me a while to fully appreciate, and once it clicked, it changed how I thought about everything I was building.

When you hand a markdown file to an AI agent at the start of a session, the agent reads it and changes how it behaves. Not temporarily: for the entire session. It follows the rules you wrote. It avoids the things you told it to avoid. It applies the analytical frameworks you described. The file is the program. The agent is the runtime.

That's not a metaphor. That's exactly what's happening. The workflow documents I've built, the consensus protocol, the agent collaboration spec, the feedback files, the project rules, are programs written in English and executed by a language model. They have states, transitions, preconditions, and termination criteria. They govern behavior. They produce repeatable (mostly) results. And like any program, they have bugs, they need testing, and they get better with each iteration.

Once I understood this, two things followed immediately.

First, these documents are evolvable. When I figure out something that works better, I tell Claude to update the relevant file. The change takes effect next session. No compiler, no deployment, no build step. Just edit the markdown and reload. I update mine regularly, sometimes daily, as I learn what works and what doesn't. The documents I'm sharing with this article are not the versions I started with. They've been through multiple generations of refinement by the AI itself, each one driven by a specific failure that taught me something.

Second, these documents compound. Each correction builds on the last. Each rule prevents a class of error, not just one instance. The feedback file about trust-the-base doesn't just fix the WordLabelManager, it changes how Claude evaluates every system it touches from that point forward. The rule about reading my full message before responding doesn't just fix one missed instruction, it changes how Claude processes every input I give it. Over time, the agent's effective behavior improves, not because the agent is learning, but because the instructions it starts with are getting better.

### Three Layers of English-Language Programs

Looking at what I've built, there are three distinct layers of documents, each operating at a different level.

**The Constitution**: CLAUDE.md and AGENTS.md. These are the permanent rules. Code style. Performance mandates. File access restrictions. Debugging protocols. What folders to ignore. What the agent is and isn't allowed to do without asking. These don't change between tasks. They load every session and they're non-negotiable. This is the equivalent of a company's coding standards document, except the employee who reads it has amnesia every morning and needs to re-read it before starting work.

My CLAUDE.md is over 350 lines. Most people's are five to ten. That difference reflects how much behavior I've needed to constrain. Every rule in that file exists because something went wrong without it. "No action without explicit approval", because the agent started implementing before I finished describing the problem. "Read the Director's entire message before responding", because it kept acting on the first sentence and ignoring the rest. "Never state a guess as fact", because confident wrong answers caused me to make decisions on false information. "Never remove diagnostic code until told to", because a premature cleanup cost me multiple test cycles.

**The Feedback System**: the memory files. These are corrections to the agent's thinking patterns, not its process. Trust-the-base. Minimum-code analysis. Think before coding. No backward compatibility. Analyze everything at scale. Each one captures a specific lesson from a specific failure, formatted consistently: what the problem was, why it matters, and how to apply the correction going forward. They accumulate over time. I tell Claude and Codex to write them during sessions when something goes wrong, and Claude maintains an index of all of them. They're cheap to create and they pay dividends across every future session.

**The Process Control**, the collaboration workflows. These are the most sophisticated pieces. They govern how multiple agents interact on complex tasks, who writes when, how they communicate through a shared file, what the handoff protocol looks like, when the process terminates. These evolved from simple prose descriptions into formal finite state machines over the course of about a week, driven by agents doing unexpected things and me writing rules to prevent them. Every hard prohibition in those documents is a bug fix. Every pre-exit guard is a response to an agent that tried to quit early. Every "MUST NEVER" is something that happened at least once.

### The Bootstrapping Problem

There's an irony in all of this. I needed the collaboration workflow to build good prompts. But I needed a good prompt to build the collaboration workflow. So I had to do it the hard way: manually relaying between chat windows, copying feedback from one agent and pasting it into the other, being the human polling mechanism that the automated system is designed to replace. I was the grep. I was the sleep loop. I was the handoff line.

The current versions, the ones I'm sharing, have been through at least eight major iterations. Each one was built using the previous one plus manual intervention where it fell short. Many of those iterations happened because I was trying to generate working examples for this article, every test run of the workflow was a real test, and when the workflow failed, I had to fix it and try again. To make matters worse, each test run applied the consensus workflow to actual code in my project, which meant the codebase changed. I couldn't reuse the same sample twice. So I'd work with Claude to analyze the updated codebase, find new areas worth optimizing, and build a fresh sample to test with. The workflow was debugging itself through real use, which is the right way to test anything, but it made the process slow and unpredictable. That's normal for any tool that bootstraps itself. The first version was rough. The eighth is stable. The ninth will be better because I'll use the eighth to build it.

### Context Cost

One practical constraint that shapes everything: every line in these documents consumes context. The configuration files, the feedback files, the workflow documents: they all load at the start of a session and eat into the finite window before you've said a single word.

Codex's AGENTS.md file has a hard 200-line limit. Claude's CLAUDE.md doesn't have a hard line cap, but Claude Code warns at 40KB that performance will degrade: roughly 800 to 1,000 lines of typical markdown. Either way, context cost applies to every line. My workaround is to use the configuration file as an index that points to larger external files. The configuration file contains references and brief summaries, and the detailed content lives in separate documents that the agent reads as needed. The advantage is that not everything in memory is relevant to every task. An index with enough descriptive information lets the agent decide what it needs to read in full without loading everything at once.

The practical rules I follow: keep things concise. Don't repeat yourself across files: if a rule appears in the configuration file, don't restate it in a feedback file. Treat context like memory on a constrained device, because that's exactly what it is.

This is an engineering tradeoff that most people don't think about because they haven't hit it hard enough to feel it. When your configuration file is five lines, context cost is invisible. When it's 350 lines plus a dozen feedback files plus a workflow document, you're making real decisions about what to include and what to leave out. Every rule that loads is a rule that works. It's also a rule that takes space away from the actual work of the session. Getting that balance right is ongoing.

---

## Section 5: Don't Let the AI Lead

The AI has a bias toward action. It wants to generate code. That's what it's trained to do, that's what it's optimized for, and the moment you describe something that sounds like it could be implemented, it will start implementing. It doesn't distinguish between "I'm thinking out loud" and "go build this." It hears a problem and it reaches for a solution, and if you don't stop it, you've got files changed before you've finished your thought.

The code it generates in that moment might even be reasonable. It might compile. It might run. But it's solving the problem as it understood it in the first thirty seconds of your description, not the problem as you would have defined it if you'd had another five minutes to think. And now you've got code in your project that reflects a half-formed idea, and you're going to spend the next hour either arguing about why it's wrong or trying to untangle what it did so you can redirect it.

This is just one of many reasons my codebase grew out of control.

### Tell Me What You Think I Said

The single most effective habit I developed, early in week one, and I've kept it ever since, is this: before I let any agent do anything, I ask it to tell me what it thinks I said.

This sounds trivially simple. It is. But it changes the entire dynamic of the interaction.

When I dictate a design idea or describe a problem, my input is messy. I circle back. I contradict myself. I replace what I said three minutes ago with what I'm saying now. That's how thinking works: especially spoken thinking. The agent has to extract the actual intent from all of that, and it's surprisingly good at it. But I don't trust it to get it right without checking.

What comes back when I ask is a clean, concise summary of my objective: stripped of the false starts and the replaced ideas. I read that summary and either confirm it or correct it. Only then does anything happen. That summary becomes the prompt. It becomes the record of what I decided. And it serves as proof that the agent and I are on the same page before any work begins.

This habit costs almost nothing: maybe thirty seconds per interaction. It has saved me more time than any other single practice I've developed. When the agent reflects back something different from what I meant, I catch it immediately rather than discovering the mismatch three files deep into an implementation.

### The Rules I Had to Write

My configuration file for Claude contains rules that exist for one reason: the agent kept doing things I didn't ask it to do.

- **"No action without explicit approval."** The agent would hear me describe a problem and immediately start editing files. Not after I said "go ahead." Not after I confirmed the approach. The moment it had enough information to act, it acted. I had to write a hard rule: you do not touch anything until I say the words "go ahead" or something equivalent. Not implied. Not assumed from the direction of the conversation. Explicitly stated.

- **"Read the Director's entire message before responding."** The agent kept acting on the first sentence of my input and ignoring everything that followed. Constraints, corrections, and additional requirements that came later in the same message were simply not processed. The agent latched onto the first actionable instruction and started generating. I had a two-part prompt once: do A, then do B. The agent did A, said "done." When I pointed out that B existed, it said something like "oh, I didn't even notice that." My response: isn't that your job?

- **"Wait for Director after each step."** The agent would complete one step and immediately start the next one without checking whether the first step was correct. By the time I noticed something was wrong, three more steps had been built on top of the broken one.

- **"Stop and discuss before debug code changes."** The agent would hear me describe a bug and immediately start writing fixes: before sharing its hypothesis, before proposing a strategy, before considering whether the bug was even where it thought it was. Debugging requires thinking first. The agent's instinct is to do first.

Every one of these rules is a scar. Something went wrong, I identified the pattern, and I wrote a rule to prevent it from happening again. The configuration file isn't a preference document: it's a behavioral contract born from specific failures.

### You Are the Architect

The natural dynamic with these tools is that the AI leads and you follow. It's faster than you. It's always ready to go. It has an answer for everything, even when the answer is wrong. If you let that dynamic take hold, you will end up with a codebase that reflects the AI's generic instincts rather than your specific design.

Reversing that dynamic, making yourself the decision-maker and the AI the executor, feels uncomfortable at first because it feels like you're slowing things down. You're not. You're preventing the rework that costs ten times more than the pause would have.

I think of it this way: you're the architect, the AI is the contractor. The contractor doesn't decide what to build. The contractor builds what the architect specifies, after the architect has finished specifying it. Not before. Not based on a guess about where the architect was going. The architect says what to build, verifies the contractor understood it, and then, only then, says go.

This requires confidence. You have to know you're right when the AI is telling you something different. You have to be willing to say "stop, that's not what I asked for" even when the AI's output looks plausible. You have to trust your own judgment over the AI's confidence, because the AI's confidence is not correlated with its correctness. It will be exactly as confident when it's wrong as when it's right. The only person in the room who can tell the difference is you.

---

## Section 6: The Pace Problem

Nobody talks about this, and I think it's one of the most important things to understand if you're an experienced programmer picking up these tools.

When I was writing code by hand, a productive day was maybe 500 lines. That sounds slow, and it was, but the slowness had a hidden gift. While my fingers were typing, my mind was processing. I was absorbing the architecture as I built it. Every line I wrote was a moment where the design settled deeper into my understanding. By the time I finished a feature, I didn't just have the code, I had a mental model of the code that was load-bearing. I could reason about it without looking at it because I'd lived with it while it was being written.

The slow pace wasn't a limitation. It was incubation time disguised as typing time. I never recognized it as a gift until it was gone.

Now the code arrives at three to five thousand lines a day. Features land before I've fully internalized the last one. The design evolves faster than my mental model of the design. I can't slow the AI down to match my thinking pace without losing the throughput advantage that's the whole reason I'm using it.

### Programming Blind

What I'm doing now is something no previous generation of programmers had to consider: maintaining a mental model of a rapidly growing system that I didn't write, line by line, with my own hands.

I'm building understanding from the outside in rather than from the inside out. Reading instead of writing. Reviewing instead of creating. Directing work I can't fully see. It's a little like being blind and working with a programmer who's doing the implementation: you can read the summaries about what they did, you can look at the code if you want to, but there's too much of it to actually inspect. You're managing by description, not by observation.

So I rely on my sense of smell. Experience and intuition tells me when something doesn't feel right, even when I can't see every line. I watch the code stream by: not reading every character, but tracking the flow. When something looks wrong, I stop it. When a description of what was done doesn't match what I expected, I dig in. Most of the time my instinct is right. The experience doesn't go away just because the tool changed.

But it's different, and the difference is tiring in a way that the old work wasn't. The mental load of being the persistent state for a system that has no persistent state of its own is real. The AI forgets everything between sessions. The AI forgets things within sessions when context fills up. Someone needs to hold the entire project in their head, and that someone is me: because the tools can't do it.

### The Parallel Tab Workflow

One thing that helps with throughput: I don't wait for one agent to finish before starting work with another.

I run Claude and Codex in separate panels in VS Code. While one agent is processing, analyzing code, generating a response, running a task, I switch to the other panel and start working on something else. My brain is the scheduler, switching context between agents based on which one needs my attention right now.

None of this runs on my machine. It's all server-side. I'm not burning local CPU running multiple agents. I'm just maintaining multiple conversations in different tabs, each progressing independently. The cost is my attention, not my hardware. And since the biggest bottleneck in any AI session is waiting for the response, having a second tab to work in while I wait effectively doubles my throughput.

This is also how the cross-agent handoff protocol was born. I was already switching between tabs constantly. At some point I noticed that when I switched back to one agent, it had no idea what the other one had just accomplished. The shared state file and the startup protocol were my solution to keeping both agents current with each other's work despite the constant tab-switching.

### The Consensus Workflow as Incubation Time

The prompt refinement process I describe in later sections is partly my replacement for the thinking time I lost.

I force myself to sit with Claude and work through a specification before any code gets written, challenging assumptions, stress-testing with Codex, iterating until all three of us agree, that's incubation. I'm building the mental model before the code starts flying, rather than trying to build it while the code is already landing at three thousand lines a day.

It costs time up front. That's the point. The old pace gave me thinking time for free. The new pace takes it away. The consensus workflow buys it back deliberately. It's the only part of this process where I'm forced to slow down and actually think about what I'm building before it's built.

---

## Section 7: Two Agents Are Better Than One

By the end of week three I had two agents with different strengths and I was using them separately. I felt Claude was better at architecture, design discussion, and the kind of conversational back-and-forth where you're figuring out what the right approach even is. Codex was better at implementation: following rules precisely, generating thorough code, interacting directly with the Unity environment, and deploying multiple parallel agents automatically when the work allowed for it.

Using them separately felt like leaving something on the table.

### How It Started

The first problem I solved wasn't collaboration: it was handoff. I was switching between Claude and Codex constantly, working on different aspects of the same project. Every time I switched back to one agent, it had no idea what the other had accomplished. Progress made in one tab was invisible to the other tab. I'd explain something to Claude, switch to Codex to implement it, switch back to Claude, and Claude had no context about what Codex had just done.

So I built a simple shared state file. Claude writes session notes to its memory. Codex writes to a Live Session State file. On startup, each agent checks who worked last by comparing file timestamps, reads the most recent state, and picks up from there. It's not sophisticated: it's just a markdown file that both agents know to check. But it meant that when I started a fresh session with either agent, it could synchronize with what the other agent had been doing since the last time it was active.

Over time, both agents got better at remembering to update their session state, sometimes without me saying anything. But sometimes they'd say they were updating it and not actually do it, so I'd have to remind them to actually write the file. Repetition is the key. You reinforce the behavior over and over until it sticks. Eventually it mostly does. In my experience, Codex is more proactive about this than Claude, it seems to have learned over time that writing breadcrumbs along the way is better than trying to save everything at the end. Claude is more forgetful about maintaining memory and needs more reminding.

One thing I learned to watch for: the agent can't tell when it's running out of context, but I can. I'm watching the interface, and when I can see that a session is getting long and context pressure is building, I'll tell the agent to stop and document where it is: save the current state to memory files so the next session can pick up where this one left off. If I don't do that proactively, the agent will keep going until context compacts and the working state is lost. The agent doesn't know it's about to forget everything. I do.

That handoff mechanism was the seed. Once I had two agents that were aware of each other's recent work through a shared file, the next question was obvious: what if they didn't just read each other's state, but actually worked on the same document together?

### The Insight: Complementary Strengths

The key realization was that Claude and Codex don't just have different capabilities, they have different blind spots. Sometimes one agent's blind spot falls in the other's area of strength, and that's where the two-agent approach pays off most clearly. But the overlap isn't perfect. There are areas where both are strong, and the redundancy is harmless. And there are areas where both are blind, things neither agent catches. That's where the Director comes in. You're the third set of eyes, and often the one catching what both agents missed.

Claude thinks architecturally. It's good at the big picture, what should the system look like, what are the constraints, what are the risks, how do the pieces fit together. It also occasionally suggests design directions I hadn't considered yet, or approaches that complement what I'm already building in ways I find genuinely valuable. But when it comes to low-level implementation details, will this actually work in Unity 6.2, what are the edge cases at the code level, is this feasible given the existing codebase, Claude often misses things or gets them wrong.

Codex thinks in execution. It's good at the details, thorough analysis of existing code, precise implementation, catching things that would break at the code level. But when I tried to have a design conversation with Codex, explore options, think through tradeoffs, figure out direction, I didn't find it as useful for that kind of work. It wants to implement, not discuss.

When I started feeding Claude's architectural thinking to Codex for review before implementation, literally copying text from one chat window and pasting it into the other, things changed. Codex would come back with problems Claude hadn't considered, feasibility issues, edge cases, conflicts with existing code, things that sounded right architecturally but wouldn't work in practice. Then I'd take those objections back to Claude, and Claude would refine the design to address them. Back and forth until both agents had essentially signed off.

The result was consistently better than what either agent produced alone. Not because the agents are smarter together: they can't even talk to each other directly. It's because their different training, different strengths, and different perspectives mean they catch different things. What Claude misses, Codex finds. What Codex doesn't think to question, Claude has already considered.

### Neither Agent Alone Is Thorough

One of the feedback files I wrote early on is titled "Both agents leave gaps." The observation behind it: when either agent works alone on a task, the result is incomplete. Implied or expected work gets silently skipped: not mentioned, not flagged, just absent. I only discover the gaps when I go back to make an adjustment and find that something wasn't implemented.

This is the core rationale behind the two-agent approach. Two agents looking at the same task catch different gaps. During review, the reviewing agent actively looks for what's missing: not just what's wrong in what was written. During implementation, the reviewing agent checks whether what was built actually matches what was specified, including the parts that were implied but not explicitly stated.

Neither agent will tell you it skipped something. You have to build a process that catches the omissions. Having a second agent whose job is specifically to find what the first agent missed is the most reliable way I've found to do that.

### When Not to Use This

I don't use the two-agent approach for everything. There are plenty of tasks that are small and clear enough that I can write a focused prompt and hand it directly to Codex. A straightforward bug fix. A simple feature addition where the design is obvious. A cleanup task with clear before-and-after criteria.

The two-agent workflow has real overhead. Setting it up, running the iterations, waiting for reviews: this takes time. That overhead pays for itself on complex tasks where the specification has risk of gaps, ambiguity, or architectural mistakes. Tasks where getting it wrong up front means hours of debugging later. Tasks where "measure twice, cut once" isn't just a principle but a genuine time-saver.

For simple tasks, one agent is fine. For complex tasks, two agents have consistently saved me more time than they cost. The judgment call is knowing which is which, and that comes from experience with both the project and the tools.

### The Manual Phase

Before any of this was automated, I was doing it by hand. I'd work with Claude in one window to develop a specification. Then I'd copy that spec and paste it into a conversation with Codex, asking for a review. Codex would write its feedback. I'd copy that feedback, switch back to Claude's window, paste it in, and we'd discuss it. Then I'd update the spec in Claude's window, copy it again, paste it back to Codex for another review round.

I was the relay. I was the file system. I was the polling mechanism. Every handoff went through me: copy, switch tab, paste, wait, copy, switch tab, paste. It worked, but it was tedious and error-prone. I'd lose things in the relay. I'd forget to include a piece of context. The process was right but the mechanism was holding it back.

That's what drove me to formalize it. If the process works but the manual execution is the bottleneck, automate the execution. That's what the consensus workflow and agent collaboration documents are: they take the process I was doing by hand and give it to the agents to run themselves, communicating through a shared file instead of through my clipboard.

---

## Section 8: The Consensus Workflow

The consensus workflow is the formalized version of the process I described in the previous section. Instead of me relaying between chat windows with copy and paste, both agents communicate through a shared prompt file, taking turns, with a structured protocol governing who writes when and what happens next.

The Director, that's me, stays in the loop for every decision. This is not an autonomous process. The agents don't resolve anything between themselves without my approval. I see every review, I make every design choice, and I approve every step before it proceeds.

### How It Works

Before the consensus workflow starts, I work with Claude, and only Claude, to analyze whatever I'm trying to build, improve, or fix. This is a normal conversation: I describe what I want, Claude analyzes the codebase, we discuss the approach, and together we produce an initial prompt, a specification document that describes the objective, the architecture, and the constraints. This pre-work happens outside the workflow. It's how the prompt gets created in the first place.

Once I have that initial prompt, I launch the consensus workflow: "Start Consensus Workflow using: [prompt file path]." Both agents read the prompt file, and the structured process begins.

The workflow has three phases: Design, Implement, and Test.

In the **Design phase**, I work with Claude to refine the specification through iterative passes. Each pass, Claude modifies and clarifies the spec based on feedback, tightening language, resolving ambiguity, adding missing edge cases. Claude then writes an entry in the shared log explaining what changes were made and why, and hands off to Codex. Codex reviews the updated spec, analyzes it for feasibility, completeness, and problems, and writes its findings back. Claude reads Codex's feedback and presents it to me, each issue with a proposed answer, and options where there are tradeoffs. I approve, pick between options, or suggest my own direction. Claude updates the spec accordingly, and the cycle continues until all three of us agree the spec is solid.

In the **Implement phase**, the spec is frozen, no more modifications to the specification itself. The spec may define multiple steps, and each step is handled as a separate pass. Codex executes one step at a time, modifying the codebase and writing a summary of what it did to the shared file. During implementation, Codex often discovers issues it couldn't have anticipated from the spec alone, things that only become apparent when you're actually writing the code. Codex flags these in its summary. Claude reads the summary, reviews the actual code changes, and suggests answers or fixes for each issue Codex raised, sometimes with multiple options. I review Claude's suggestions, pick the option I prefer or propose my own, and tell Claude to hand it back to Codex. Claude writes the handshake, and Codex picks up the next step. This continues through all the steps in the spec.

In the **Test phase**, I provide feedback from actually running the game in Unity, visual verification, behavioral checks, things that only a human at the keyboard can confirm. But Codex can also look at compiler logs directly. If there are compiler errors or warnings after an implementation step, I tell Claude there are issues, Claude writes the handshake, and Codex goes directly to the build output, identifies the specific problems, and fixes them. This works for warnings as well as errors. When everything checks out, compiles clean, runs correctly, looks right, the workflow completes.

The key principle: getting off to a good start is 90% of the fight. A thoroughly vetted spec prevents the gaps and rework that come from either agent working alone. The time spent in Design is not overhead: it's the most valuable part of the entire process.

### The Shared Prompt File

The agents communicate through a single markdown file. The file has two sections, separated by a clear divider:

Everything above the divider is the **spec**: the implementation specification that describes what to build. During Design, Claude may edit this section. Once Implementation begins, the spec is frozen.

Everything below the divider is the **iteration log**: a chronological record of every pass by every participant. Each entry is bracketed by a header line and ends with a handoff directive: `!NEXT: Agent 1` or `!NEXT: Agent 2`. That directive is the signal. When an agent polls the file and sees a handoff addressed to it, it acts. When it doesn't, it waits.

This structure means anyone can read the file and reconstruct exactly what happened: what was proposed, what was challenged, what was decided, and why. The log is the complete process record.

### The Polling Mechanism

Both agents poll the shared file by running a simple shell loop: grep for the last `!NEXT:` line, check if it's addressed to them, sleep five seconds, repeat. When an agent sees its own name in the handoff, it reads the file and acts. When it doesn't, it keeps polling.

The handoff line is the completion signal. It's always the last thing written to the file. If the grep finds it, the write is complete. If it doesn't, the file isn't ready. No timestamps, no debounce, no file-size monitoring. Just: is the line there or isn't it?

This sounds crude, and it is. But it's reliable precisely because it's simple. There's one signal, one check, one answer. The earlier versions used timestamp-based detection with a debounce window: check if the file changed, wait five seconds to make sure it stopped changing, then read. That was more complex and less reliable. The grep approach replaced it because it works and it's hard to get wrong.

### The Evolution: Eight Iterations and Counting

The workflow documents started as prose descriptions of how the agents should behave. Now in the current version, they're formal finite state machines with explicit states, defined transitions, terminal conditions, and hard prohibitions.

That evolution happened because the agents kept doing unexpected things.

An agent would decide it was finished when it wasn't, so I added a pre-exit guard that forces a re-read of the file before any turn-ending output. An agent would act when it wasn't its turn, so I added strict precondition checks before any write. An agent would split a write operation, and the other agent would act on the incomplete file, so I added an atomic write requirement. An agent would emit status messages that ended its turn while it was supposed to be polling, so I added output contracts that define exactly what each agent is allowed to say in each state. Codex was writing timestamps two hours off from my local time, so I added a rule that timestamps are informational only and must never drive state transitions.

Every rule in those documents is a bug fix. Every prohibition is something that happened. The documents evolved from "please do this" to "you must do this and you must never do that" because the agents consistently found ways to misinterpret the softer versions.

Eight major iterations so far. Many of those iterations happened because I was trying to generate working examples for this article: each test run was a real test on real code, and when the workflow failed, I had to fix it and try again. The workflow was debugging itself through real use.

### The Director's Role

I want to be specific about what the Director actually does, because it's not just "approve things."

During Design, I'm working with Claude: discussing architecture, debating approaches, making decisions about tradeoffs. When Codex's review comes back, Claude presents all the issues to me with proposed answers. I approve, override, or ask for more information on each one. The design conversation happens between Claude and me. Codex provides the stress test.

During Implementation, I'm reviewing Claude's analysis of each step Codex completed. Did the implementation match the spec? Are there risks? Did anything drift? I'm also the one running the game in Unity and reporting back what happened: did it compile, did it run, does it look right, did anything break. The agents can't run the game. Only I can. That test feedback closes the loop between what was specified and what actually works.

During Test, I'm verifying that the complete implementation meets the original objectives. Claude does a formal verification pass against the spec. I do the visual and behavioral verification in the running game.

At every point, I can interrupt, redirect, or stop. The agents don't make design decisions autonomously. When they hit something not covered by the spec, they stop and ask.

### Stepwise Refinement

If you've ever used stepwise refinement to develop a specification, starting with a high-level description and progressively adding detail and precision through multiple passes until you have something implementable, this is the same process. The refinement passes just alternate between two agents with different strengths, and a human director approves each step.

The consensus workflow is not a new methodology. It's an old methodology, one that experienced programmers already know, applied to a new context. Measure twice, cut once, with two AI agents doing the measuring.

---

## Section 9: The Agent Collaboration Workflow

The consensus workflow keeps me in the loop for every decision. That's its strength: nothing happens without my approval. But it also means I'm spending time on every iteration, even the early rounds where the agents are working through obvious issues that don't need my input.

The agent collaboration workflow is the autonomous version. I set the objective, start both agents, and step back. They iterate on a shared specification without my involvement until they reach agreement. I watch from both chat windows, like watching a tennis match, and I can interrupt at any time, but the default is that they run on their own.

### How It Works

I start by working with Claude to develop an initial specification: the same kind of pre-work I'd do before launching the consensus workflow. Once we have a reasonable starting point, I give both agents the same directive: "Start Agent Collaboration using: [prompt file path]." Both agents read the workflow document and the prompt file, and the autonomous process begins.

From there, the agents alternate. Claude writes a version of the spec. Codex reviews it, flags ambiguity, missing edge cases, hidden coupling, feasibility problems. Claude reads Codex's feedback, incorporates what makes sense, pushes back on what doesn't, and writes the next version. Codex reviews again. This continues autonomously until Codex's verdict is "ready", meaning it can't find anything else wrong. Claude writes the final signoff, and the collaboration is complete.

The result is a spec that both agents have agreed on, without me having to participate in every round. I review the final product and decide what to do with it.

### The Pipeline: Collaboration Into Consensus

My intention, and this is where the two workflows connect, is to use the agent collaboration output as the starting input for the consensus workflow.

The idea is a pipeline: I work with Claude to rough out the initial concept. Agent collaboration refines it autonomously into a solid spec. Then I feed that spec into the consensus workflow, where I'm back in the loop for the final refinement and all of the implementation work.

The collaboration handles the early iterations where the agents are catching obvious gaps. The consensus workflow handles the harder judgment calls and the implementation. I spend my time where it matters most, on the decisions that require my experience, and let the agents handle the mechanical back-and-forth on their own.

### English Is Not a Programming Language

I want to be honest about the challenges. The agent collaboration workflow is harder to get working reliably than the consensus workflow, for a specific reason: when something goes off the expected path and I'm not in the loop, the agents make assumptions. In the consensus workflow, when something unexpected happens, the agent stops and asks me. In agent collaboration, there's nobody to ask. The agent fills the gap with whatever seems most plausible, and sometimes that's spectacularly wrong.

The workflow documents, the ones I described as programs in Section 4, are written in English. English is ambiguous. The agents interpret it. Most of the time they interpret correctly. But when they don't, and nobody is watching, the error compounds. One wrong assumption leads to another, and by the time the collaboration finishes, the output may have drifted in a direction nobody intended.

This is why the workflow documents evolved into formal finite state machines with explicit states, hard prohibitions, and pre-exit guards. Every rule was written because an agent did something unexpected during a test run. But even after many iterations of refinement, the agents still occasionally drop out of polling, misinterpret a handoff, or make decisions they shouldn't make. English is softer than code, and the execution is never quite as predictable as you want it to be.

### Current Status

The agent collaboration workflow is built and working. Both agents can iterate autonomously on a shared spec and reach consensus without my involvement. The handoff from collaboration into consensus, where the collaboration output becomes the consensus input, also works mechanically.

What I'm still refining is reliability. The agents maintain the protocol most of the time, but "most of the time" isn't good enough for a workflow you want to trust. I'm still finding edge cases, still adding rules, still testing. The consensus workflow is more mature because I've used it on more real tasks. The agent collaboration is newer and rougher.

I'm sharing it anyway because the concept is sound, the implementation works well enough to be useful, and I'd rather make it available now, rough edges and all, than wait for perfection. If you try it and find problems, that's expected. Fix the workflow document the way you'd fix any program: identify the bug, write a rule to prevent it, test again.

### A Lighter On-Ramp

Not everyone is going to want the full consensus workflow. It has real overhead, and for someone just getting started with multi-agent collaboration, it might feel like too much.

The agent collaboration workflow is a simpler starting point. You write a prompt, start both agents, and let them refine it. No phase transitions, no implementation, no testing: just two agents improving a spec until they agree it's solid. If the result is good enough, you can hand it directly to one agent for implementation. If it needs more work, you feed it into the consensus workflow for the full treatment.

Start where it makes sense for you. The agent collaboration is the lighter path. The consensus workflow is the thorough one. Both are available in the GitHub repository linked at the end of this article.

One thing worth mentioning: I built, designed, and tested all of this in Unity, but there's nothing about these workflows that's specific to Unity. The consensus and collaboration mechanisms should work with any development environment and any language: C++, JavaScript, Python, HTML, whatever you're building in. The agents communicate through shared markdown files and take turns refining a specification. The specification could describe anything. The tools don't care what's on the other end.

That said, these workflows are not designed to set up your project for you. You should already have a working project with your development environment configured before experimenting with the consensus workflow. You can absolutely use AI to help you get your project set up: but do that work first, separately.

---

## Section 10: Getting Started

This section describes what you need and where everything goes. The workflow documents, configuration inserts, and example prompts are all available in the GitHub repository linked at the end of this article.

### Prerequisites

You need two AI coding agents that can read and write files in your project directory:

- **Claude Code** (Anthropic): acts as Agent 1. This is the agent you'll work with directly for design, analysis, and review. It needs access to your project files.
- **Codex** (OpenAI): acts as Agent 2. This is the agent that handles implementation, detailed review, and autonomous execution. It also needs access to your project files.

Both agents should be set up and working in your development environment before you try the workflows. If you're using VS Code, both can run in separate terminal panels so you can switch between them easily.

You also need:
- A project directory where both agents can read and write files.
- A folder for your prompt/spec files (I use a `Prompts/` folder inside my project).
- The two workflow documents: `Consensus-Workflow.md` and `Agent-Collaboration.md`, placed somewhere both agents can read them. For convenience, these files could be placed at the root of your project folder.

### Configuration: The Insert

Both agents need to know the workflows exist and how to behave when you activate them. This is done through a configuration insert: a block of text that gets added to Claude's `CLAUDE.md` file and Codex's `AGENTS.md` file.

The insert tells both agents:
- What "Start Consensus Workflow" and "Start Agent Collaboration" mean.
- Which agent is Agent 1 and which is Agent 2.
- Where to find the workflow documents.
- How to behave at startup (Agent 2 enters polling immediately; Agent 1 enters Director-interaction mode for consensus, or executes first pass for collaboration).
- What not to do while polling (no status messages, no implementation, no edits).

Here is the full insert. Add this verbatim to both configuration files:

```
## Consensus Workflow & Agent Collaboration Mode

When the Director says "Start Consensus Workflow" or "Start Agent Collaboration" with a prompt file path:
- **Do not implement.** This is never an instruction to write code.
- Claude Code is **Agent 1**. Codex is **Agent 2**.
- For Consensus Workflow, read "Consensus-Workflow.md". For Agent Collaboration, read "Agent-Collaboration.md".
- Then read the prompt file once.
- Follow those directions explicitly.

### Startup Behavior (Asymmetric by Design)
- **Agent 2 (Codex):** Enter polling mode **immediately**. No analysis, no status output, no turn-ending messages. If the file already shows !NEXT: Agent 2, polling exits on its first iteration and Agent 2 acts normally.
- **Agent 1 (Claude Code):** Start in the role-appropriate mode: Director-interaction for CW, or execute first pass for AC. Agent 1 acts when routing hands off to Agent 1; otherwise Agent 1 polls.
- **Startup order is nondeterministic.** Neither agent assumes it started first. All decisions derive from current file state.

### While Polling: Hard Prohibitions
- Do not send completion/status messages that end the turn.
- Do not run implementation steps.
- Do not edit code or docs.
- Do not perform non-essential reads beyond the polling grep.
- After every complete log entry, immediately re-enter continuous polling mode.
- Do not stop polling unless the workflow reaches terminal completion state or the Director explicitly ends it.
- If handoff state is ambiguous, malformed, or missing, remain in polling mode: do not convert ambiguity into turn-ending output.
- Polling output is an internal control signal, not user-facing status.
```

### Launching the Consensus Workflow

1. Work with Claude to develop your initial prompt: the specification for what you want to build. Save it as a markdown file in your project. I keep mine in a `Prompts/` folder at the root of my project folder.

2. In Claude's window, type: `Start Consensus Workflow using: [path to your prompt file]`

3. In Codex's window, type the same command: `Start Consensus Workflow using: [path to your prompt file]`

4. Claude reads the workflow document and the prompt file, analyzes the current state, and presents its findings to you. Codex enters polling mode and waits.

5. You work with Claude through the Design phase. When you're ready to send the spec to Codex for review, tell Claude. Claude updates the shared file and Codex picks it up automatically.

6. The workflow proceeds through Design, Implement, and Test as described in Section 8. You stay in the loop throughout.

### Launching the Agent Collaboration

1. Work with Claude to develop a reasonable starting spec. Save it as a markdown file.

2. In both windows, type: `Start Agent Collaboration using: [path to your prompt file]`

3. Both agents read the workflow document and the prompt file. Claude writes the first pass. Codex enters polling, detects the handoff, and begins its review.

4. The agents iterate autonomously. Watch both windows to monitor progress. Interrupt if you want to provide input or redirect.

5. When both agents reach consensus, the collaboration completes. Review the final spec and decide what to do with it: feed it into the consensus workflow for implementation, hand it directly to one agent, or refine it further yourself.

### The Example Outputs

The GitHub repository includes two complete examples showing what the workflow actually produces:

**WordLabelManager Entity Refactor**: a pure consensus workflow example. This was a real task: eliminating a broker class that maintained fifteen arrays and five legacy collections by moving label references directly onto ECS entities. The prompt went through multiple Design passes between Claude and Codex before entering Implementation, which ran through fourteen passes total. The final result: all fifteen arrays and five collections eliminated, the broker class reduced to a pool creator, and the game running identically with significantly less overhead. The prompt file includes the complete iteration log showing every pass by both agents.

**RenderSortSystem Dirty Array Elimination**, an agent collaboration into consensus workflow example. This started with Claude analyzing the system and identifying roughly 605 KB of waste across word caches, typed input caches, and scale animation arrays, all sized for 5,120 entries when the actual problems involved one or two active nodes. The agent collaboration refined the prompt, which then fed into the consensus workflow for implementation.

Both examples demonstrate the process on real code with real results. They're not toy problems: they're actual optimization tasks from a game in active development.

### What to Expect on Your First Run

It may not be smooth. The agents might do unexpected things. They'll misinterpret a handoff, drop out of polling, or make assumptions they shouldn't make. That's normal. It happened to me on every iteration of the workflow documents, and it's why those documents have as many rules as they do.

When something goes wrong:
- Check the shared prompt file. The iteration log tells you exactly what happened.
- Identify the failure: was it a polling dropout, a misinterpreted handoff, an agent acting out of turn?
- Fix the workflow document with a new rule that prevents that specific failure.
- Try again.

This is how the workflow improves. Every rule is a bug fix. Every test run teaches you something. The documents I'm sharing have been through many iterations of exactly this process. Yours will go through the same thing, and they'll get better each time. Have Claude or Codex, or both, help you make the improvements.

---

## Section 11: What I've Learned

Five weeks ago I had never used an AI coding tool. I didn't know what a context window was. I didn't know what a prompt file was. I didn't know that the AI would agree to a constraint and then quietly forget about it twenty minutes later. I also had never developed anything in Unity or written a line of C#: so I was learning the AI tools, the engine, and the language all at the same time. I picked up these tools because I was bored, because I wanted to build something my wife might play, and because I thought it would be interesting to see what fifty years of programming experience looked like when applied to something completely new.

It has been interesting. It has also been frustrating, exhausting, occasionally maddening, and more rewarding than I expected.

Here's what I've taken away from it.

### Experience Becomes More Valuable, Not Less

The most important thing I can say to an experienced programmer considering these tools: your knowledge doesn't become less relevant. It becomes more relevant. The AI generates code fast enough that the bottleneck shifts from writing to evaluating. You can tell when the architecture is going sideways. You can spot defensive patterns that don't belong in your context. You recognize when the AI is looping on a problem it already tried to solve an hour ago. You can feel when something isn't right, even when the output compiles and runs.

Those skills come from experience. They can't be replaced by the tools. And they're exactly what the tools need from you in order to produce good results instead of plausible-looking results.

### Measure Twice, Cut Once

For those of you under 30 who may not have encountered this phrase, or who made it through this entire article without looking it up, it's the old carpenter's rule that says preparation prevents waste. It applies to software at least as much as it applies to wood.

This principle has been at the center of my work for decades, and it turns out to be even more important when working with AI. The cost of starting implementation before the specification is right arrives faster and more painfully than it ever did with human programmers. The AI can generate three thousand lines in a day or more. If those lines are built on a flawed spec, you're not unwinding a day of human typing: you're unwinding a day of machine-speed generation, and the rework is proportionally larger.

The two-agent consensus workflow is just a formalized way of measuring twice. Two agents with different strengths examine the same specification and catch different problems. A human director makes the decisions. Only when all three agree does implementation begin. It's not a new idea. It's an old idea applied to a new context.

### One Task at a Time

Context pressure is real and ongoing. The tools have a finite memory, and when it fills up, things go wrong quietly. My approach, short sessions, one task per session, save state to memory files, start fresh, works but requires discipline. Every time I'm tempted to push through a long session, I remind myself what happens when context compacts: the agent fabricates continuity, and I spend more time recovering than I saved by not stopping.

### The Tools Will Get Better

The specific frustrations I've described, context loss, agents dropping out of polling, instructions being ignored, timestamps in the wrong timezone, these are problems with the current generation of tools, not with the concept of AI-assisted development. The tools will improve. Context windows will get larger or smarter. Agent communication will become native rather than something you hack together with grep and sleep. The workflow documents I wrote in English will eventually be expressible in something more structured and more reliably executed.

When that happens, the principles behind what I've built will still apply. Two perspectives catch more than one. Specifications should be stress-tested before implementation. The human stays in the loop for decisions that matter. Those aren't tool-dependent insights: they're engineering insights.

### What I Haven't Figured Out Yet

I want to be honest about what's still rough.

The agent collaboration and consensus workflows are both less reliable than I'd like. The biggest issue is that both agents can and do drop out of polling whenever they feel like it. When that happens, the fix is usually simple, just type "continue" and hit enter, but it shouldn't be necessary. The agents maintain the protocol most of the time, but "most of the time" isn't enough when you want to trust a process. I'm still iterating on the workflow documents, still finding edge cases, still writing rules.

My memory system evolved ad hoc and could be more structured. I'm moving toward a shared single-index approach where both agents read and write to the same memory location. I just finished implementing the new system and will be posting an article shortly.

I can mostly monitor context usage, VS Code shows an indicator that tells me when context is getting high. When I see it climbing, I make sure both agents save their current state to the shared memory system and are prepared to switch to a fresh context without losing much. That's where the shared memory really starts paying off, it turns a potentially disruptive context switch into a routine handoff.

And I'm still spending 12 to 16 hours a day on this, which is not sustainable and my wife would like me to stop. She's right. I probably will, eventually. After the next feature.

### An Invitation

I wrote this because I thought what I learned might save someone else some time. Not the polished version: the real version, with the failures and the frustrations and the things I'm still figuring out.

If you try the workflows and they work for you, I'd like to hear about it. If you try them and they break in ways I haven't seen, I'd like to hear about that too. If you've been doing this longer than I have and you think I missed something important, I'm genuinely interested.

---

## Resources

All workflow documents, configuration files, and examples referenced in this article are available in the GitHub repository:

**Repository:** `github.com/TheGameWiz/Measure-Twice-Cut-Once`

| File | Description |
|------|-------------|
| [`Workflows/Consensus-Workflow.md`](../Workflows/Consensus-Workflow.md) | The full consensus workflow specification: Design, Implement, and Test phases with FSM, polling mechanism, handoff protocol, and hard prohibitions |
| [`Workflows/Agent-Collaboration.md`](../Workflows/Agent-Collaboration.md) | The autonomous agent collaboration specification: two agents iterate on a spec without Director involvement until they reach consensus |
| [`Config/Config-Insert.md`](../Config/Config-Insert.md) | The configuration block to paste into both CLAUDE.md (Claude Code) and AGENTS.md (Codex). Contains startup behavior, role assignments, and polling prohibitions |
| [`Examples/WordLabelManager-Refactor.md`](../Examples/WordLabelManager-Refactor.md) | A complete consensus workflow example: real task that eliminated 15 arrays and 5 legacy collections from a broker class. Includes the full spec and iteration log showing all passes between both agents |
| [`Examples/RenderSortSystem-DirtyArrayElim.md`](../Examples/RenderSortSystem-DirtyArrayElim.md) | An agent collaboration into consensus workflow example: identified ~605 KB of waste in cache arrays sized for 5,120 entries when only 1-2 nodes were active |
| [`Examples/Shared-Memory-Collaboration-Log.md`](../Examples/Shared-Memory-Collaboration-Log.md) | The design conversation between Claude and Codex that produced the shared memory system. A real-world example of the two-tabs technique. |
| [`Memory/Shared-Agent-Memory-Bootstrap-Kit.md`](../Memory/Shared-Agent-Memory-Bootstrap-Kit.md) | The complete bootstrap specification: hand it to an agent and it builds the entire shared memory system. Project-agnostic. |

### Companion Articles

| Article | Description |
|---------|-------------|
| [Two Tabs and a Log File](./Two-Tabs-and-a-Log-File.md) | The easiest entry point: one prompt, one shared file, no infrastructure. Try this first if you want to see the concept work before committing to the full system |
| [Two Agents, One Memory](./Two-Agents-One-Memory.md) | Unifying two agents' separate memory systems into a single shared architecture: designed using the two-tabs technique, with a bootstrap kit that sets it up for you |

---

## Important Notice

The observations and opinions in this article are strictly my own, based on approximately five weeks of personal experience with these tools on a single project. They are not intended as a thorough evaluation of Claude Code, Codex, or any other product, and they don't represent the views of Anthropic, OpenAI, or any other company. Where I describe one agent as better or worse at a particular task, that reflects my experience in my specific context, game development in Unity 6.2, and may not apply to yours. I'm not endorsing or disparaging any product. I use both tools daily and find genuine value in each.

The workflow documents and configuration files shared with this article are provided as-is, with no warranty of any kind. They were developed and tested in a specific environment, Windows, Unity 6.2, Claude Code, and Codex CLI, by a single developer over the course of several weeks. They work for me. They may not work for you without modification.

These documents instruct AI agents to execute bash commands, including shell loops, file reads, and file writes within your project directory. You should review any workflow document before loading it into an agent, understand what it does, and satisfy yourself that it's appropriate for your environment. I am not responsible for any damage, data loss, or unexpected behavior that results from using these materials. Use them at your own risk, modify them freely, and test them in a safe environment before relying on them for anything that matters.

---

## Copyright and License

Copyright (c) 2026 David Todd.

This article is licensed under the **Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)**.

See [LICENSE](../LICENSE) and [NOTICE](../NOTICE) for attribution guidance and terms.
