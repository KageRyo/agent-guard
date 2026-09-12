# agent-guard

**Policy-as-code for AI agent permissions.**

Define bounded permissions for AI agents in YAML. Enforce at runtime. Prevent scope creep.

## Why

AI agents need bounded permissions. Current solutions are either:
- **Sandboxes** (E2B, Daytona) — heavy, require infrastructure
- **Custom code** — rebuilt per project, error-prone
- **Nothing** — agents run unrestricted

`agent-guard` is a lightweight CLI + library that lets you declare *exactly* what an agent can do, then enforce it on every tool call.

## Install

```bash
pip install agent-guard
```

## Quick Start

### 1. Generate a policy

```bash
agent-guard init my-agent --output .agent-guard.yaml
```

### 2. Define permissions

```yaml
# .agent-guard.yaml
name: code-reviewer
default_action: deny

max_tool_calls: 50
blocked_tools:
  - destructive_*
  - admin_*

rules:
  - action: allow
    tool: fs.read
    resource: "./**/*"
    description: "Read project files"

  - action: allow
    tool: shell
    resource: "git *"
    description: "Read-only git commands"

  - action: deny
    tool: shell
    resource: "rm *"
    description: "Prevent deletion"
```

### 3. Check calls

```bash
agent-guard check .agent-guard.yaml --tool fs.read --resource ./src/main.py
# ✅ True — allowed by rule: Read project files
#    risk: low

agent-guard check .agent-guard.yaml --tool shell --resource "rm -rf /"
# ❌ False — denied by rule: Prevent deletion
#    risk: medium
```

### 4. Audit your policy

```bash
agent-guard audit .agent-guard.yaml --format table
```

## Features

- **YAML policies** — version-controlled, reviewable
- **Rule ordering** — first match wins (like firewall rules)
- **Tool wildcards** — `fs.*` matches `fs.read`, `fs.write`
- **Resource patterns** — glob + regex support
- **Domain controls** — allowlist/blocklist for network tools
- **Execution limits** — max tool calls per session
- **Risk scoring** — LOW/MEDIUM/HIGH/CRITICAL per verdict
- **Multiple outputs** — JSON for automation, table for humans
- **Exit codes** — 0 = allowed, 1 = denied (CI-friendly)

## CLI Commands

| Command | Purpose |
|---------|---------|
| `check` | Test a tool call against policy |
| `explain` | Show which rule matched and why |
| `audit` | Review effective ruleset |
| `init` | Generate starter policy |

## Python API

```python
from agent_guard import Guard, Policy, ToolCall

policy = Policy.from_file(".agent-guard.yaml")
guard = Guard(policy)

call = ToolCall(tool="fs.read", resource="./src/main.py")
verdict = guard.check(call)

if verdict.allowed:
    print(f"OK: {verdict.reason}")
else:
    print(f"BLOCKED: {verdict.reason}")
```

## Use Cases

- **CI/CD pipelines** — restrict agent to read-only operations
- **Code review agents** — allow `git log` but block `git push`
- **Browser agents** — restrict to specific domains
- **Multi-agent systems** — different policies per agent
- **Compliance** — audit trail of all enforcement decisions

## Comparison

| Tool | Self-hosted | Declarative | Lightweight | Domain-aware |
|------|:-:|:-:|:-:|:-:|
| agent-guard | ✅ | ✅ | ✅ | ✅ |
| E2B | ❌ | ❌ | ❌ | ❌ |
| Daytona | ✅ | ❌ | ❌ | ❌ |
| OpenAI sandbox | ❌ | ❌ | ❌ | ❌ |
| Custom code | ✅ | ❌ | ✅ | ❌ |

## License

MIT © Yunare Maia
