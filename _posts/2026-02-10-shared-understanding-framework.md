---
title:  "Shared Understanding: When AI Becomes Your Research Partner"
date:   2026-02-10 12:00:00 -0500
categories: ai collaboration knowledge-management
---

You know that feeling after a really productive meeting? Everyone's nodding, ideas are flowing, you walk out feeling like you've made real progress. Then two weeks later you realize half the team is building something completely different from what you expected.

I've been there too many times. It turns out we have a meeting problem disguised as an understanding problem.

Daniel Kahneman's work on System 1 versus System 2 thinking explains why this happens. In meetings, we're mostly running on System 1—fast, pattern-based thinking that *feels* like understanding but often isn't. Real understanding requires System 2: slow, deliberate analysis. And that rarely happens when we're busy talking.

Here's the thing: AI has similar problems. No persistent memory between sessions. Gets confused about when things happened. Loses context when you start a new conversation. Sound familiar?

Neither humans nor AI can solve complex problems alone. But together? That's where it gets interesting.

## Treating AI as a Teammate, Not a Tool

I've been working on a framework called [Shared Understanding](https://github.com/benthomasson/shared-understanding) that changes how I work with AI. Instead of treating it as a documentation tool that summarizes things for me, I treat it as an active participant in figuring things out.

The basic idea is simple: use a git repo with chronological markdown entries as external memory for both me and the AI. Everything gets written down in a structured way, dated and organized. This gives us both something to refer back to.

My typical workflow looks like this:

1. Before a meeting, I pull in the Jira context so the AI knows what we're discussing
2. The meeting happens, gets transcribed
3. After the meeting, I have a conversation with the AI about the transcript—not just "summarize this" but actually talking through what was discussed, correcting misunderstandings, probing assumptions
4. Then comes the discovery phase: systematic search through Jira, Google Drive, and especially Slack
5. Document what we learned in a structured entry
6. Share it back to Google Docs so others can give feedback
7. Archive the conversation to git so we don't lose it

## The Tools

I use a few CLI tools via `uvx` (so no installation hassle):

- **jirahhh** for pulling Jira context
- **gcmd** for exporting Google Docs
- **slacker** for searching Slack and setting reminders
- **claude-projects** for archiving conversations

Everything lives in markdown, organized by date. Simple, but it works.

## Why Bother?

Here's what I've realized: writing things down isn't just good practice. It's actually cognitive enhancement—for both me and the AI.

When I force myself to write things down, I engage System 2 thinking. I have to actually think through what I learned, not just feel like I understood it. And when the AI has access to those structured notes, it doesn't get confused about what happened when or lose important context.

The framework solves three problems at once:
- Memory that persists beyond any single session
- Clear timeline that prevents temporal confusion
- Forced deliberate thinking instead of pattern matching

## What Actually Happened

Let me tell you about a project where this really paid off. After several cycles through this workflow, we landed on a solution that:

- Required less implementation work than we originally planned
- Was technically better than our initial ideas
- Was actually easier for customers to use

None of us saw this solution at the start. It emerged from the systematic back-and-forth—human strategic thinking combined with AI synthesis across all the information we'd gathered.

Slack search was the surprise hero here. It consistently found relevant conversations I'd completely forgotten about. Decisions that were made months ago, discussions that informed those decisions, context that nobody remembered having. All that organizational memory, just sitting there, and the AI could help me make sense of it.

Each cycle through the workflow built on the previous one. Compound understanding, you could call it.

## Want to Try It?

1. Fork the [shared-understanding repo](https://github.com/benthomasson/shared-understanding)
2. Set up the integrations (Jira, Google Drive, Slack)
3. Pick a real problem you're working on
4. Go through the cycle a few times and see what emerges

A few things I've learned:

- Don't just ask for summaries. Have actual conversations with the AI about what you're learning. Correct its mistakes. Probe its understanding.
- Don't skip the search phase. You'll be amazed what's hiding in your Slack history.
- Be patient. The magic happens after multiple cycles, when understanding compounds.

The best solutions I've found didn't come from me or from the AI alone. They came from thinking together, systematically, over time.
