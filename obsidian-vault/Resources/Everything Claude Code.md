---
tags: [claude, skills, agents, hooks, mcp, ai, framework, security, performance]
created: 2026-04-17
source: https://github.com/affaan-m/everything-claude-code
author: affaan-m (Anthropic hackathon winner)
---

# Everything Claude Code — Comprehensive AI Agent Framework

Production-ready plugin system for Claude Code and compatible AI harnesses.

| Stat | Count |
|------|-------|
| Specialized agents | 48 |
| Skills | 183 |
| Legacy command shims | 79 |
| MCP servers shipped | 14+ |
| Languages supported | 12 ecosystems |

---

## Core Components

### Agents
Delegate specialized tasks with limited scope — code reviewers, build resolvers, security auditors, and domain-specific analyzers.

Supported languages: TypeScript, Python, Go, Java, Kotlin, Rust, C++, PHP.

### Skills
Primary workflow surface replacing the legacy command system. Enable:
- Test-driven development
- Feature planning
- Code review
- Documentation updates
- Continuous learning patterns

### Hooks
Automate trigger-based actions on tool events:
- Prevent `console.log` statements
- Detect secrets in prompts
- Format code
- Manage session state across contexts

### Rules
Always-follow guidelines organized into:
- Language-agnostic common principles
- Language-specific directories: TypeScript, Python, Go, Swift, PHP, Java, Perl

> **Note:** Rules cannot distribute through plugins due to upstream limitations — must be installed separately.

---

## Installation

```bash
# Via Claude Code marketplace
/plugin marketplace add https://github.com/affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

Manual: clone repo and copy components to `~/.claude/` directories.

---

## Key Features

### Token Optimization
- Sonnet model defaults (~60% cheaper than Opus)
- Thinking token caps (10k)
- Early compaction triggers
- Keep active MCPs < 10 and tools < 80 to preserve ~70k of the 200k context window

### Memory Persistence
Hooks automatically save/load context across sessions.

### Continuous Learning v2
Instinct-based pattern extraction with confidence scoring.

### AgentShield
102 static analysis rules, 1282 tests covering:
- Secret detection
- Permission auditing
- Vulnerability and misconfiguration scanning across CLAUDE.md, settings.json, MCP configs, hooks, and skill definitions
- Three Claude Opus agents run adversarial analysis (`--opus` flag)

---

## Cross-Platform Support

| Platform | Hook Events |
|----------|-------------|
| Claude Code | 8 |
| Cursor IDE | 15 |
| OpenCode | 11 |
| Codex CLI | instruction-based (no executable hooks) |
| Gemini | supported |
| Antigravity | supported |

DRY adapter patterns prevent hook duplication across harnesses.

---

## MCP Servers Included

GitHub, Supabase, Vercel, Exa, Playwright, Sequential Thinking, Context7 (and more).

---

## Practical Workflows

**Feature development:**
```
/ecc:plan  →  /tdd  →  /code-review
```

**Security preparation:**
```
/security-scan  →  /e2e  →  /test-coverage
```

**Cost management:**
- Default: Sonnet for most work
- Switch to Opus for deep architectural decisions
- `/clear` between unrelated tasks
- `/compact` at logical breakpoints

---

## Hook Runtime Controls

```bash
# Enable strict profile
ECC_HOOK_PROFILE=strict

# Disable specific hooks
ECC_DISABLED_HOOKS=hook1,hook2
```

---

## Dashboard GUI

Tabbed browsing of agents, skills, commands, rules, and settings with dark/light themes and customizable fonts.

---

## Related

- [[Claude Skills Resources]] — official Claude Skills docs and links
