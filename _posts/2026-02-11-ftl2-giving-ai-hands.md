---
title:  "FTL2: Giving AI Hands"
date:   2026-02-11 00:00:00 -0500
categories: ftl2 automation ai
---

We're almost there with AI. Claude can understand what you want to build, design architectures, write code, debug problems, and explain complex systems. But there's a gap. Claude can think, plan, and generate—but it can't **do**. It can write a script that provisions infrastructure, but it can't reach out and actually create the servers, configure the databases, deploy the applications.

AI needs hands.

## The Vision

**Give people the power to build whatever they want to build.**

The path is simple:

1. You tell Claude what you want
2. Claude figures out how to build it
3. FTL2 makes it happen

```
┌─────────────────────────────────────────────────────────┐
│                     What you want                        │
│         "Build me a production-ready web app"           │
│     "Set up a Kubernetes cluster with monitoring"       │
│       "Deploy my code to AWS with auto-scaling"         │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                       Claude                             │
│                                                          │
│              Understands intent                          │
│              Designs architecture                        │
│              Generates execution plan                    │
│              Handles errors and adapts                   │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                        FTL2                              │
│                                                          │
│     The bridge between AI and the real world            │
│                                                          │
│     • 3-17x faster than Ansible                         │
│     • Access to every automation module that exists     │
│     • Clean Python API that AI generates naturally      │
│     • Secrets handled safely, never in generated code   │
│     • Check mode for safe preview before execution      │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                    The real world                        │
│                                                          │
│              Servers provisioned                         │
│              Applications deployed                       │
│              Databases configured                        │
│              DNS updated                                 │
│              Monitoring enabled                          │
│              Users served                                │
└─────────────────────────────────────────────────────────┘
```

## Why FTL2?

### The Module Ecosystem Already Exists

Ansible has 20+ years of community work building modules for everything:

| Domain | Examples |
|--------|----------|
| Cloud | AWS, Azure, GCP, DigitalOcean, Linode, Vultr |
| Virtualization | VMware, Proxmox, libvirt, Hyper-V |
| Containers | Kubernetes, Docker, Podman |
| Networking | Cisco, Juniper, Palo Alto, F5 |
| Databases | PostgreSQL, MySQL, MongoDB, Redis |
| Monitoring | Prometheus, Grafana, Datadog, Splunk |
| DNS | Route53, Cloudflare, Namecheap |
| Bare Metal | Files, users, packages, services, commands |

Thousands of modules. Every cloud. Every service. Every operating system.

FTL2 doesn't reinvent this—it makes it accessible to AI.

### Why Not Just Use Ansible?

Ansible was built for humans writing YAML playbooks, not AI agents executing Python:

| Ansible | FTL2 |
|---------|------|
| YAML (indentation errors) | Python (natural for AI) |
| Subprocess per module | In-process (3-17x faster) |
| Fork-based parallelism | Async/await |
| Secrets in playbooks | Secret bindings (never in code) |
| Designed for static playbooks | Designed for dynamic AI execution |

### Performance Benchmarks

Real benchmarks from `ftl2-performance` repo:

| Benchmark | Ansible | FTL2 | Speedup |
|-----------|---------|------|---------|
| file_operations (30 tasks) | 6.17s | 0.43s | **14.2x** |
| local_facts (1 task) | 0.73s | 0.22s | **3.3x** |
| template_render (10 tasks) | 3.22s | 0.19s | **16.6x** |
| uri_requests (15 requests) | 3.75s | 0.30s | **12.4x** |

Why does speedup scale with task count? Ansible forks a subprocess per task (Python startup + import ansible + run module + serialize result), while FTL2 calls modules in-process.

## The AI-Native Interface

```python
async with ftl2.automation(
    secret_bindings={
        "amazon.aws.*": {"aws_access_key_id": "AWS_KEY"},
        "community.general.slack": {"token": "SLACK_TOKEN"},
    }
) as ftl:
    # AI generates this naturally
    vpc = await ftl.amazon.aws.ec2_vpc(cidr_block="10.0.0.0/16")

    subnet = await ftl.amazon.aws.ec2_subnet(
        vpc_id=vpc["vpc_id"],
        cidr_block="10.0.1.0/24",
    )

    instance = await ftl.amazon.aws.ec2_instance(
        instance_type="t3.micro",
        image_id="ami-12345",
        subnet_id=subnet["subnet_id"],
    )

    await ftl.community.general.slack(
        channel="#deployments",
        msg=f"Server {instance['public_ip']} is ready",
    )
```

- Clean syntax that AI generates correctly
- Secrets injected automatically, never visible
- Results chain naturally between calls
- Errors handled gracefully
- Check mode for safe preview

## AI-First Design Philosophy

AI should be treated as the primary user of software, not as a secondary interface or afterthought. This isn't about replacing humans—it's recognizing that there will be more instances of AI using software than there are human users.

### The Multiplier Effect

Every human spawns multiple AI sessions. A single developer might run dozens of Claude Code sessions per day, each one a distinct "user" of the underlying tools. Most human users will interact with software through AI, making AI the actual consumer of APIs, CLIs, and error messages.

### Design Implications

When AI is the primary user, different things matter:

**Error Messages:**
```bash
# Human-first (minimal context)
Error: Module ping failed on db01

# AI-first (full context)
Error: Module 'ping' failed on host 'db01'
  Exit Code: 1
  Error: Connection timeout after 30s

  Context:
    Host: db01 (192.168.1.10:22)
    User: ansible
    Module: ping (hash: abc123)

  Possible Causes:
    - Network connectivity issue
    - SSH daemon not running
    - Firewall blocking port 22

  Suggested Actions:
    1. Verify host is reachable: ping 192.168.1.10
    2. Check SSH access: ssh ansible@192.168.1.10
```

**Output Format:**
- Human-first: Pretty tables, colors, progress spinners
- AI-first: Structured JSON with consistent schemas, parseable by code

Humans can always ask the AI to summarize structured output. AI cannot reliably parse pretty output.

**Validation:**
- Human-first: Fail at the point of use
- AI-first: Fail fast with pre-flight checks, enumerate all issues upfront

AI debugging is expensive (tokens, time, context). Catching problems before execution saves entire investigation cycles.

## Features Designed for AI

FTL2 includes features specifically for AI-assisted development:

### Structured Output Formats

```bash
# JSON output for programmatic parsing
ftl2 -m ping -i hosts.yml --format json
{
  "total_hosts": 10,
  "successful": 9,
  "failed": 1,
  "results": {...},
  "duration": 2.5
}
```

### Dry-Run Mode

```bash
ftl2 -m file -i hosts.yml -a "path=/tmp/test state=absent" --dry-run
Would execute on 10 hosts:
  - web01: Would remove /tmp/test (exists, 1.2KB)
  - web02: No changes (file doesn't exist)
```

### Module Discovery

```bash
ftl2 --list-modules
Available modules:
  ping         - Test connectivity to hosts
  setup        - Gather host facts
  file         - Manage files and directories
  copy         - Copy files to remote hosts

ftl2 --module-doc ping
Module: ping
Arguments:
  data (optional): Data to send with ping
Idempotent: Yes
```

### Smart Retry Logic

```bash
ftl2 -m copy -i hosts.yml -a "src=app.tgz dest=/opt/" --smart-retry
web01: Connection timeout - will retry
web02: File already exists - no retry (permanent error)
web03: SSH handshake failed - will retry
```

## Installation

```bash
# Using uvx (recommended)
uvx --from "git+https://github.com/benthomasson/faster-than-light2" ftl2 --help

# Or install with pip
pip install git+https://github.com/benthomasson/faster-than-light2
```

## Quick Start

### 1. Create an inventory file

```yaml
# hosts.yml
all:
  hosts:
    web01:
      ansible_host: 192.168.1.10
      ansible_user: admin
    web02:
      ansible_host: 192.168.1.11
      ansible_user: admin
```

### 2. Test connectivity

```bash
ftl2 -m ping -i hosts.yml
```

### 3. Gather facts

```bash
ftl2 -m setup -i hosts.yml --format json
```

### 4. Run a command

```bash
ftl2 -m shell -i hosts.yml -a "cmd='uptime'"
```

## Using FTL2 with Claude

When working with Claude Code, FTL2 becomes incredibly powerful:

```
You: "Set up a web server on web01"

Claude: I'll help you set up a web server. Let me:
1. Check connectivity
2. Install nginx
3. Start the service
4. Verify it's running

[Claude generates and executes FTL2 commands]

Done! Nginx is running on web01:80
```

The key insight: Claude writes Python naturally. FTL2 is Python. There's no translation layer, no YAML indentation to get wrong, no DSL to learn.

## Architecture

FTL2 uses a "gate" system for remote execution:

1. **Gate Builder**: Creates a zipapp containing the modules needed
2. **Gate Transfer**: Sends the gate to remote hosts via SSH
3. **Gate Execution**: Runs the gate, which manages module lifecycle
4. **Message Protocol**: JSON over stdin/stdout for module I/O

The gate stays running on the remote host, so multiple modules can execute without re-transferring the zipapp. This is one reason FTL2 is so much faster than Ansible.

## Native Fluency

AI natively "speaks" Python and markdown. There's no learning curve, no onboarding, no documentation to read first.

| Format | AI Fluency | Implication |
|--------|------------|-------------|
| Markdown | Native | Documentation, configs work immediately |
| Python | Native | Code examples work on first try |
| JSON/YAML | Native | Structured data parses cleanly |
| Custom DSLs | Learned | Requires examples, trial and error |

FTL2 embraces this: Python for automation, JSON for data exchange, markdown for documentation.

## Links

- **Repository**: [github.com/benthomasson/faster-than-light2](https://github.com/benthomasson/faster-than-light2)
- **Module Utils**: [github.com/benthomasson/ftl_module_utils](https://github.com/benthomasson/ftl_module_utils)
- **Performance Benchmarks**: [github.com/benthomasson/ftl2-performance](https://github.com/benthomasson/ftl2-performance)

## What's Next

FTL2 is mostly built. What remains:

1. **Declarative resources** - Terraform-style dependency graphs
2. **MCP server** - So Claude can discover FTL2 tools at runtime
3. **More demos** - Prove it works with impressive examples

The vision is close to reality: AI that can not just think and plan, but actually **do**.
