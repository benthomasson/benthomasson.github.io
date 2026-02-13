---
title:  "Claude Is Your User"
date:   2026-02-13 00:00:00 -0500
categories: ai development ftl2
---

What happens when the entire SDLC runs at conversation speed?

For the past week, I've been building [FTL2](https://github.com/benthomasson/ftl2)—a Python automation framework—with AI as the primary user. Not AI-assisted development. AI *is* the user. The one running the tools, hitting the errors, requesting the features, testing the code, and providing UX feedback.

The results: 1,200+ lines of production code in a single day. Six bugs diagnosed and fixed in one debugging session. Performance benchmarks showing 14-17x speedup over Ansible. Multi-host scale tests running on real infrastructure.

All documented. All tested. All by 11pm.

## The Insight

There will be more instances of AI using software than human users. Every developer spawns multiple AI sessions daily. Most users will interact with tools through AI. Background agents multiply horizontally.

So I asked: what if we designed software for that reality?

**Traditional SDLC** (weeks to months):

![Traditional SDLC](/assets/images/traditional-sdlc.png)

**AI-as-user SDLC** (minutes):

![AI-as-user SDLC](/assets/images/ai-sdlc.png)

The feedback loop collapses from weeks to minutes. The user is always present. The round-trip time for "what's wrong with this?" is seconds.

## What This Looks Like in Practice

### Day 1: The Debugging Session

I pointed Claude at a failing automation script. 14 modules failing with empty error messages. Instead of me telling it what to fix, I *showed* it how the system worked—let it run the debug tools, read the code, see the output.

Once it understood the architecture, it:

1. Wrote a debug test script to catch the actual exception
2. Traced the full execution path through six components
3. Found and fixed **five bugs** I didn't know existed
4. Added a fallback mechanism I hadn't designed

None of these were in my original request. I asked about empty error messages. It fixed the entire remote execution pipeline.

### Day 2: Performance Testing

Created [benchmarks](https://github.com/benthomasson/ftl2-performance) comparing FTL2 to Ansible:

| Benchmark | Speedup |
|-----------|---------|
| file_operations (30 tasks) | 14.2x |
| template_render (10 tasks) | 16.6x |
| uri_requests (15 HTTP calls) | 12.4x |

The AI ran the benchmarks, analyzed the results, generated the charts, and explained why the speedup scales with task count (subprocess overhead per task in Ansible vs in-process calls in FTL2).

Then it suggested the next benchmarks to run.

### Day 3: Scale Testing

Spun up 5 Linodes with `uv run provision.py 5`. Ran parallel execution across all hosts. Hit three bugs:
- Missing collection module (not committed to git)
- `ping()` only checking first result, not all hosts
- Gate result wrapper not unwrapped

Claude found all three, fixed them, and suggested testing on Fedora 42 instead of 43 because Python 3.14 doesn't have `python3-dnf` yet.

It knew this because it tried, failed, read the error, and reasoned about it.

## The Key Principle: Show, Don't Tell

When you *tell* an AI "use this architecture instead of that one," it pushes back. From its perspective, it doesn't understand why your approach is better. It will suggest alternatives, argue, or implement something different.

But when you *show* it—let it run the tools, see the output, trace through the code—it builds genuine understanding. The AI stops fighting the architecture and starts extending it.

This is usability testing at zero cost. If the AI struggles with your documentation, your documentation is unclear. If the AI can't parse your error messages, neither can humans under pressure. If it takes five attempts to get the config right, your config is too implicit.

## Designing for AI as User

When AI is the primary user, different things matter:

**Error Messages:**
- Human-first: "Connection failed"
- AI-first: "SSH connection to 'web01' failed: authentication rejected. Tried: [publickey]. Expected key at ~/.ssh/id_rsa (exists: true, permissions: 0600). Server offered: [publickey, password]."

**Validation:**
- Human-first: Fail at point of use
- AI-first: Fail fast with pre-flight checks, enumerate all issues upfront

**Output:**
- Human-first: Pretty tables and progress spinners
- AI-first: Structured JSON with consistent schemas

AI debugging is expensive (tokens, time, context). Catching problems before execution saves entire investigation cycles.

## The Documentation That Writes Itself

Every debugging session becomes a documented entry. Every feature request includes:
- Impact assessment
- Code examples
- Priority ranking by actual time wasted
- Implementation path

This repository now has 90+ entries documenting real debugging sessions, feature implementations, and design decisions. All written collaboratively with AI. All providing context for future sessions.

The conversation is ephemeral. The markdown endures.

## What Changes

**Old model:** Build features, hope users find them useful, wait months for feedback.

**New model:** AI uses the feature, tells you immediately what's wrong, you fix it before lunch.

The AI articulates friction precisely: "This error message doesn't tell me what's wrong." "I don't know which file to check." "The validation should happen before connection attempts."

These become documented issues with root cause analysis, feature requests with priorities, and fixes implemented the same day.

## The Speed

Traditional estimate for FTL2's feature set: months with a team.

Actual time: one week, solo, with AI as the user.

- 1,200+ lines of production code in a single day session
- 71 tests written alongside the implementations
- 8 documentation entries per major session
- Performance benchmarks, scale tests, charts

This isn't about AI writing code faster. It's about the feedback loop. When your user is always present and can articulate exactly what's wrong, you stop guessing about priorities.

## Try It

Design your next feature with AI as the user. Not AI-assisted. AI *is* the user.

Let it run your tools. Watch where it struggles. Ask it what information was missing. Fix that first.

The ratio of AI instances to human users is already shifting. Build for that reality.
