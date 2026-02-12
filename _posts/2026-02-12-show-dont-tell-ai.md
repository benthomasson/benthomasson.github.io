---
title:  "Show, Don't Tell: When AI Refuses Your Architecture"
date:   2026-02-12 00:00:00 -0500
categories: ai collaboration ftl2
---

I spent an hour arguing with Claude about how to execute modules on remote hosts. I wanted it to use FTL2's gate system—persistent Python processes shipped to remote machines via SSH, multiplexing module execution through a single connection. Claude wanted direct SSH. Simpler, fewer moving parts, why add complexity?

Claude was wrong, but its objections were reasonable. It had never seen gates work. From its perspective, I was asking it to trust my judgment about an architecture it didn't understand. So it pushed back.

Then I stopped arguing and started showing.

## The Problem

FTL2 automation scripts were failing with 14 empty error messages when executing modules on remote hosts. Every module—dnf, get_url, firewalld, pip, service—reported failure with no explanation. I knew the issue was in how the automation context routed module execution through gates. But when I told Claude to use gates instead of direct SSH, it disagreed.

Fair enough. "Use this thing you've never seen because I said so" isn't a compelling argument.

## What I Did Instead

I pointed Claude at the documentation I'd written earlier:

- How gates are built as self-contained .pyz archives
- The debug tools for inspecting gate behavior

Then I had it run the tools:

```bash
$ uv run ftl-gate-builder -m ping -m setup -m shell -m community.general.slack
$ python tools/gate_debug.py info
$ python tools/gate_debug.py module ping
$ python tools/gate_debug_remote.py 97.107.136.233 module shell "whoami"
```

Claude saw gates work. It saw the protocol messages, the module results, the connection pooling. It read the gate builder code, the message protocol, the remote runner. It understood the *why*—gates exist because shipping a persistent Python process to the remote host and multiplexing module execution through it is faster than establishing a new SSH connection and Python process for every single module call.

## What Happened Next

Once Claude understood how gates worked, I showed it the failing automation script. Instead of me telling it what to fix, it:

1. **Wrote a debug test script** that monkey-patched the execution function to catch and print the actual exception—revealing an `AssertionError()` from an uninitialized `gate_builder`

2. **Traced the full execution path** from `ftl.minecraft.dnf(...)` through the proxy system, through `run_on()`, through `_execute_remote_via_gate()`, to the gate process on the remote host

3. **Found and fixed five bugs** it discovered along the way:
   - `gate_builder` never initialized on `RemoteModuleRunner`
   - `ftl2.ftl_modules.exceptions` not included in gate .pyz
   - `exec()` globals/locals split causing `NameError` for imports
   - `__name__ == "__main__"` triggering `asyncio.run()` inside the gate's event loop
   - Gate only looking for `main()` when FTL modules expose `ftl_command()`, `ftl_pip()`, etc.

4. **Added a fallback mechanism** I didn't ask for—when FTL modules fail on the remote gate due to missing dependencies, automatically fall back to the Ansible module bundle path

None of these fixes were in my original request. I asked it to diagnose empty error messages. It fixed the entire remote execution pipeline.

## Why Telling Fails

When you tell an AI "use gates instead of SSH," you're asking it to trust your architectural judgment about a system it hasn't seen. Its objections are valid from its perspective:

- "Direct SSH is simpler and has fewer moving parts"
- "The gate adds complexity without clear benefit"
- "The existing code already handles SSH connections"

It's optimizing for what it knows. And what it knows is that simpler is usually better.

## Why Showing Works

When you show the AI the system working—let it read the code, run the tools, see the output—several things change:

**It builds a mental model.** After reading the gate protocol, the builder, the debug tools, and running them, Claude understood the architecture deeply enough to reason about edge cases.

**It stops fighting you.** Once it sees gates work and understands why they exist, the question shifts from "should we use gates?" to "why aren't gates working here?"

**It goes beyond what you asked.** I didn't know about the `exec()` globals/locals pitfall or the `__name__` guard issue. Claude found them because it understood the system well enough to trace through it independently.

**It writes better fixes.** The FTL-to-Ansible fallback wasn't something I specified. Claude added it because it understood that some FTL modules have dependencies not available in the gate, and falling back to Ansible bundles is the right behavior.

## The Pattern

1. **Don't argue about architecture.** If the AI disagrees with your approach, don't spend turns debating. You'll both get frustrated.

2. **Show it the system working.** Point it at documentation, have it run tools, let it read real code. Build understanding first.

3. **Then show it the failure.** Once it understands how things *should* work, show it what's broken. The contrast between "works in debug tools" and "fails in automation script" is a powerful diagnostic frame.

4. **Let it investigate.** Don't prescribe the fix. Let the AI trace through the code, write debug scripts, and discover the root causes itself. It will find things you didn't know about.

## The Deeper Point

This isn't just about AI. It's about how understanding works. You can't transfer understanding by asserting conclusions. You transfer it by sharing the experience that led to those conclusions.

When I told Claude "use gates," I was sharing a conclusion. When I showed Claude gates working, I was sharing the experience. The conclusion followed naturally.

The same thing happens with human teams. "We should use microservices" is a conclusion. Showing someone the deployment bottleneck, the scaling problem, and how services solve it—that's building understanding.

Show, don't tell. It works on AI for the same reason it works on humans: understanding comes from experience, not authority.
