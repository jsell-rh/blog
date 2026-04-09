---
title: "NASA Stopped Writing Buggy Software. So Can AI Agents."
date: 2026-04-07
draft: true
summary: "NASA's on-board shuttle group used process to eliminate production bugs. That same process might be the key to writing bug-free code with AI agents."
tags: ["autonomous-development"]
image: /blog/images/nasa-ralph-loop-header.png
---

![NASA Ralph Loop](/blog/images/nasa-ralph-loop-header.png)

I recently [watched the crew of the Orion spacecraft execute a beautiful choreography](https://www.youtube.com/watch?v=Pebd3doNTtI)
as they ascended to the crew capsule and were strapped into an incredibly complex
machine, carrying over 3 million pounds of fuel.

Compared to the Apollo missions, the technological complexity of this spacecraft is
unfathomable.

As I saw the smiles and felt the excitement of the crew members, I couldn't help but
think about the software that they were trusting to deliver them to the void of space, and 
to bring them home.

**Alive.**

The more I thought about it, the smaller I felt. How could they be so confident in the software
driving their spacecraft? 

Are the software developers behind that incredible software just _that much better_
than "regular" developers like myself?

I used to think that it was impossible to write code without bugs.

Then I read [They Write the Right Stuff](https://www.eng.auburn.edu/~kchang/comp6710/readings/They%20Write%20the%20Right%20Stuff.pdf).

It turns out that a sufficiently **mature process** _has been proven_ to all but eliminate bugs.

---

## The Process

So why doesn't everyone follow such a process? I'd wager the usual suspects: time and cost. 

The cost, rigor and discipline required to write software for NASA feel entirely
unjustified for my personal inventory system, or an [experimental knowledge graph platform](https://github.com/openshift-hyperfleet/kartograph).

But the calculus changes when it's no longer humans writing code, but AI Agents.

**Can _all code_ be written with the rigor of a spacecraft control system?**

To find out, I built a simple agentic [ralph loop](https://ghuntley.com/loop/) that 
encoded the NASA process.

```bash
while true; do 
  claude < specs/prompts/project-manager.md  # Decompose tasks
  claude < specs/prompts/implementation.md   # Write code
  claude < specs/prompts/verifier.md         # Identify flaws
  claude < specs/prompts/process-revision.md # Prevent future flaws <- NASA
done
```

<details>
<summary style="opacity: 50%;  cursor: pointer;">See Detailed Loop</summary>

```bash
while true; do 
  claude --model opus[1m] --dangerously-skip-permissions < specs/prompts/project-manager.md
  claude --model opus[1m] --dangerously-skip-permissions < specs/prompts/implementation.md
  claude --model opus[1m] --dangerously-skip-permissions < specs/prompts/verifier.md
  claude --model opus[1m] --dangerously-skip-permissions < specs/prompts/process-revision.md
done
```


> [!IMPORTANT]
> Input redirection (`claude < file.md`) is strongly preferred
> to command substitution (`claude $(cat file.md)`).
>
> Using command substitution, I began to get the following 
> error once the size of the `verifier.md` exceeded 140KB.
>
> ```
> bash: ~/.local/bin/claude: Argument list too long
> ```

</details>

<br/>

**And it mostly works.**

I used this loop to build [Stego](https://github.com/jsell-rh/stego/tree/f7174e7b999376bd1bfa6a66c8f7218b7907dd4c), a declarative code generator <span style="opacity: 50%; font-style: italic;">[with a reconciler]</span> that eliminates accidental complexity from agent-written code.


Before getting into the details, here's an example of the `bug` -> `process revision` flow: 

In STEGO, the compiler generates a program by
assembling pieces from multiple independent code generators. On [round 9 of one task](https://github.com/jsell-rh/stego/blob/5bfd45bd8f27992f6b55b40fbe1cabe95bc52ade/specs/reviews/task-013.md?plain=1#L67),
the verifier discovered that the assembler was outputting these pieces in the wrong order.
It placed a piece of code that *uses* the database appeared before the line that *creates* it.

Each piece compiled fine on its own, and all unit tests passed. But the assembled program
wouldn't run. The `verifier` step caught the bug and the [process-revision step](https://github.com/jsell-rh/stego/commit/e66eff4#diff-eee8e72ea7b7c8756b90d9bc864491335620f835533bdea29969470822b81bdeR270)
ensured the class of error was documented for future prevention.

... which worked. But the bug could also have been caught by the `implementation` 
step running a simple compile step (see [Observations & Lessons-Learned](#observations--lessons-learned)).

---

## Running the Process

The input to the system was [a single spec file](https://github.com/jsell-rh/stego/blob/890da11d464764181667d81e8925922618524874/specs/spec.md), the result of several 
hours of back & forth with `Claude Opus 4.6 [1m]`.

The [project manager](https://github.com/jsell-rh/stego/blob/main/specs/prompts/project-manager.md) decomposed the spec
into [16 tasks](https://github.com/jsell-rh/stego/tree/be975d356e5a46b8a9b0ba2505a3bc36b90d4a32/specs/tasks): 

| Task | Title |
|------|-------|
| 001 | Go Module & Project Structure |
| 002 | Core Domain Types |
| 003 | YAML Schema Parsing |
| 004 | Registry Structure & Loading |
| 005 | Port Resolution Engine |
| 006 | Generator Interface & gen.Context |
| 007 | Slot/Fill Proto Contract & Interface Generation |
| 008 | rest-crud Archetype Definition |
| 009 | rest-api Component Generator |
| 010 | postgres-adapter Component Generator |
| 011 | jwt-auth Component Generator |
| 012 | Fill Wiring & main.go Assembly |
| 013 | Compiler Core — Plan/Apply Reconciler |
| 014 | Validation & Drift Detection |
| 015 | CLI — All 11 Commands |
| 016 | Example Service — End-to-End Demonstration |

Over 27.5 hours, the loop produced 10K lines of production code (+ 22K lines of test code).

More importantly, it engaged in **[95 rounds of review](https://github.com/jsell-rh/stego/tree/be975d356e5a46b8a9b0ba2505a3bc36b90d4a32/specs/reviews) and [process revision](https://github.com/jsell-rh/stego/compare/2365b0b5b38e0927637f9b846481c212eed82b70..be975d356e5a46b8a9b0ba2505a3bc36b90d4a32#diff-eee8e72ea7b7c8756b90d9bc864491335620f835533bdea29969470822b81bde)**. 

<details>
<summary style="cursor: pointer">See Detailed Stats</summary>

```bash
STEGO Build Stats
2026-04-08 16:49

Progress
  [##############################] 100% (16/16 tasks)
  16 complete  0 in review  0 in progress  0 not started

Timeline
  Total wall clock:  47h 52m 6s
  Total commits:     442
  Checklist items:   92 (accumulated process learnings)

Code
  Production:  10629 lines
  Test:        21957 lines
  Test ratio:  67% of total
  Throughput:  ~222 prod lines/hr

Task Breakdown
  TASK      TITLE                                   STATUS            COMMITS  ROUNDS  FINDINGS     ACTIVE TIME
  -------------------------------------------------------------------------------------------------------------
  task-001  Go Module & Project Structure           complete                0       0         0              --
  task-002  Core Domain Types                       complete               12       1         5         22m 36s
  task-003  YAML Schema Parsing                     complete               11       1         5         22m 44s
  task-004  Registry Structure & Loading            complete                9       2         7         26m 42s
  task-005  Port Resolution Engine                  complete               13       1         5         31m 51s
  task-006  Generator Interface & gen.Context       complete                8       2         4         20m 33s
  task-007  Slot/Fill Proto Co..terface Generation  complete                5       1         1         14m 44s
  task-008  rest-crud Archetype Definition          complete                4       0         0          4m 16s
  task-009  rest-api Component Generator            complete               80      22        45      5h 28m 16s
  task-010  postgres-adapter Component Generator    complete               48      13        31       3h 13m 5s
  task-011  jwt-auth Component Generator            complete                3       1         0           3m 0s
  task-012  Fill Wiring & main.go Assembly          complete               52      16        30      4h 49m 54s
  task-013  Compiler Core - Plan/Apply Reconciler   complete               42      14        31       5h 36m 4s
  task-014  Validation & Drift Detection            complete               33      15        28       4h 6m 37s
  task-015  CLI - All 11 Commands                   complete               11       5         5        1h 5m 5s
  task-016  Example Service - ..-End Demonstration  complete               10       1         3          51m 6s
  -------------------------------------------------------------------------------------------------------------
  TOTAL                                                                   341      95       200     27h 36m 33s

Review Efficiency
  Total findings:        200
  Total review rounds:   95
  Findings per round:    2.1
  Avg rounds per task:   5.9 (completed tasks only)
```
</details>

### What It Missed

Despite the explicit verification step, the loop left several spec requirements unimplemented:

| Spec Requirement | State at Completion |
|---|---|
| Remote git registry with url: + ref: | Types exist, not implemented |
| Per-component SHA pins | Types exist, not implemented |
| Multiple registries with precedence | Types exist, not implemented |
| stego plan warns on stale pins | Not implemented |
| Promotion path (fill -> component) | Documented, no tooling |
| Brownfield adoption (wraps: on components) | Documented, no tooling |
| map operator | Not implemented (gate, chain, fan-out are) |

### What It Got Wrong

- Missed implementing the remote git registry feature
- Added an un-spec'd `STEGO_REGISTRY` env var


## Observations & Lessons-Learned 

The critical component of the NASA process was documenting failures and updating
their process to prevent similar failures in the future. 

Not surprisingly, I didn't nail encoding this process into an agentic step.

Instead of 1) noting specific failure classes, why they happened, where they might happen in the future
and/or 2) adding mechanical verification scripts, the **verification step composed an
exhaustive and specific "verification checklist"** within the `implementation` step's prompt. 

This led to the prompt growing from [16KB](https://github.com/jsell-rh/stego/blob/2365b0b5b38e0927637f9b846481c212eed82b70/specs/prompts/implementation.md) 
to [366 KB](https://github.com/jsell-rh/stego/blob/be975d356e5a46b8a9b0ba2505a3bc36b90d4a32/specs/prompts/implementation.md).


<details>
<summary style="cursor: pointer; font-style: italic;">Aside: The size of the prompt would often confuse the process revision step as it had trouble reading the entire file at once.</summary>

```bash 
● The implementation prompt is very long. Let me check the specific area around checklist items related to migration and test fixtures, since the Round 3 findings involve those issues.

● Search(pattern: "migration|deprecated|renamed|backward.compat|yaml.*key.*reject|silent.*accept|silent.*ignor", path: "stego/specs/prompts/implementation.md")


  ⎿  Found 37 lines
    160:[Omitted long matching line]
    162:[Omitted long matching line]
    163:[Omitted long matching line]
    176:[Omitted long matching line]
    184:[Omitted long matching line]
    190:[Omitted long matching line]
    196:[Omitted long matching line]
    200:[Omitted long matching line]
    204:[Omitted long matching line]
    211:[Omitted long matching line]
    215:[Omitted long matching line]
    218:[Omitted long matching line]
    219:[Omitted long matching line]
    225:[Omitted long matching line]
    227:[Omitted long matching line]
    228:[Omitted long matching line]
    232:[Omitted long matching line]
    233:[Omitted long matching line]
    235:[Omitted long matching line]
    241:[Omitted long matching line]
    246:[Omitted long matching line]
    250:[Omitted long matching line]
    255:[Omitted long matching line]
    258:[Omitted long matching line]
    260:[Omitted long matching line]
    268:[Omitted long matching line]
    275:[Omitted long matching line]
    281:[Omitted long matching line]
    288:[Omitted long matching line]
    291:[Omitted long matching line]
    296:[Omitted long matching line]
    297:[Omitted long matching line]
    300:[Omitted long matching line]
    301:[Omitted long matching line]
    303:[Omitted long matching line]
    304:[Omitted long matching line]
● Let me read the end of the implementation prompt to find how checklist items are numbered and to see the full list.

```
</details>

<br/>

While the checklist did result in the implementation agent largely avoiding the specific
errors in subsequent loops (and fixing the original findings), it was overfitting. Keeping
process improvements pattern-based, and leveraging deterministic scripts, would help the process
evolution scale.

---


I found that **the loop was inefficient by design**.  

When issues were identified by the `verifier` (especially [trivial ones](https://github.com/jsell-rh/stego/blob/be975d356e5a46b8a9b0ba2505a3bc36b90d4a32/specs/reviews/task-002.md?plain=1#L5)),
the loop had to go through `process improvement` -> `project manager` -> `implementation` before the issue was resolved.

Process improvement is a necessary step that shouldn't be skipped, but there's no reason why the `verification` step couldn't have
submitted a fix itself (or passed a suggestion to the implementation step) if the fix is trivial.

---

As a general finding, **any deterministic piece of the process would benefit from 
being mechanical** (rather than handled by the agent.)

As a concrete example, the `implementation` agent prompt includes [instructions for identifying its next task](https://github.com/jsell-rh/stego/blob/be975d356e5a46b8a9b0ba2505a3bc36b90d4a32/specs/prompts/implementation.md?plain=1#L146).
These instructions are deterministic, and could be handled by a pre-init script that injects
the task into the initial prompt. This saves tokens & increases loop velocity. 

Deterministic verification steps are one other component of the process that would
benefit from being moved from the `implementation` prompt to mechanical scripts. This
is the responsibility of the `process improvement` step.

Somewhat related, [stats.sh](https://github.com/jsell-rh/stego/blob/be975d356e5a46b8a9b0ba2505a3bc36b90d4a32/scripts/stats.sh) was helpful for monitoring
the progress of tasks through the loop. However, the prompts and the script did not agree on a task format contract, making the status script
brittle and less portable.

---

Generally, this loop worked well. 

It left me itching to parallelize it. 

With a formalized dependency graph, perhaps produced up-front by the `project manager`, 
the inner loop (`implementation` -> `verifier` -> `process improvement`) could
run in parallel for independent tasks. 

To prevent races, the task decomposition/dependency identification would need to be
sequential.


**The obvious next question**: What if we wiped the code, and re-ran with the revised process?

