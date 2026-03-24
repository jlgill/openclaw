---
title: "Ollama Self-Learning Implementation Plan"
summary: "Deferred implementation plan for OpenClaw self-learning best practices using local Ollama to reduce token cost"
---

# Ollama Self-Learning Implementation Plan

This document captures the full setup plan so it can be implemented later.

Goal:

- Apply OpenClaw self-learning best practices from the Reddit workflow.
- Keep costs low by using local Ollama for both chat and memory embeddings.
- Keep memory durable, structured, and retrievable on demand (not preloaded dumps).

## Notes on Current Limitation

The previous attempt could not directly scaffold or edit `~/.openclaw` due to permission boundaries in the active session.

Use this plan as the implementation checklist once permissions are resolved.

## 1) Configure OpenClaw for Ollama chat + local embeddings

Run:

```bash
openclaw config set models.providers.ollama.api "ollama"
openclaw config set models.providers.ollama.baseUrl "http://owui-ollama:11434"
openclaw config set models.providers.ollama.apiKey "ollama-local"
openclaw config set agents.defaults.model.primary "ollama/qwen3.5:2b"
openclaw config set agents.defaults.memorySearch.provider "ollama"
openclaw config set agents.defaults.memorySearch.model "nomic-embed-text"
openclaw config set agents.defaults.memorySearch.remote.baseUrl "http://owui-ollama:11434"
openclaw config set agents.defaults.memorySearch.remote.apiKey "ollama-local"
openclaw config set plugins.slots.memory "memory-core"
```

Pull required models:

```bash
ollama pull qwen3.5:2b
ollama pull nomic-embed-text
```

## 2) Apply memory-as-index workspace structure

Run:

```bash
WORKSPACE="${OPENCLAW_WORKSPACE:-$HOME/.openclaw/workspace}"
mkdir -p "$WORKSPACE/memory/people" "$WORKSPACE/memory/projects" "$WORKSPACE/memory/decisions" "$WORKSPACE/reflections"

cat > "$WORKSPACE/MEMORY.md" <<'EOF'
# Memory Index

## People
- Store person-specific stable context in memory/people/.

## Projects
- Store project-level durable notes in memory/projects/.

## Decisions
- Store key decisions and rationale in memory/decisions/.

## Daily Logs
- Append daily notes to memory/YYYY-MM-DD.md.

## Retrieval Rule
- Keep this file short.
- Use memory search/get tools to drill into detailed files on demand.
EOF

cat > "$WORKSPACE/HEARTBEAT.md" <<'EOF'
# Heartbeat Checklist

- Check if memory index is still clean and not bloated.
- Move durable facts from daily logs into project/people/decision files.
- Archive stale or redundant notes.
- If repeated friction appears, propose one small tool or rule change.
EOF

cat > "$WORKSPACE/meditations.md" <<'EOF'
# Reflection Loop

## Active Questions
- What repeated failure happened this week?
- What rule would have prevented it?
- What should be promoted to operating behavior?

## Promotion Rule
Promote only if:
1. Pattern repeats across multiple days.
2. It changes actual behavior.
3. It is concise enough to become an operating rule.
EOF
```

## 3) Reduce token pressure from bootstrap injection

Run:

```bash
openclaw config set agents.defaults.bootstrapMaxChars 8000
openclaw config set agents.defaults.bootstrapTotalMaxChars 40000
```

Rationale:

- Keeps injected startup files constrained.
- Preserves on-demand drill-down via memory tools.

## 4) Keep pre-compaction memory flush on

Run:

```bash
openclaw config set agents.defaults.compaction.memoryFlush.enabled true
openclaw config set agents.defaults.compaction.memoryFlush.softThresholdTokens 4000
openclaw config set agents.defaults.compaction.memoryFlush.prompt "Write durable facts to memory files now. Reply NO_REPLY if nothing to store."
```

## 5) Validate end-to-end

Run:

```bash
openclaw models list
openclaw memory status --deep
openclaw memory index --force
openclaw memory search "recent decisions"
```

Expected outcomes:

- Ollama provider appears and selected model resolves.
- Memory index probes succeed.
- Search returns snippets from `MEMORY.md` and `memory/*.md` after indexing.

## Optional Follow-up

After permissions are fixed, add a compact startup policy in `AGENTS.md` that enforces:

- Read small core files only on startup.
- Write durable memory to files, not transient context.
- Use memory tools for drill-down instead of loading everything.
- Promote repeated insights from reflections into operating behavior.
