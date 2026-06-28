# Cost Controls for Harness

> **Status:** Active · **Owner:** revfactory · **Last updated:** 2026-06-28

Harness orchestrates multi-agent teams. A single task can fan out to 5+ parallel Claude calls, and a complex ticket can consume 50K–200K tokens. This document explains why costs spike and what you can do about it.

---

## Why harness costs more than a single-agent call

Each team member is a separate Claude invocation with its own context window. A three-agent Producer-Reviewer-QA team on one ticket calls Claude **at least three times**; if agents exchange messages (`SendMessage`) or the orchestrator retries on error, the number climbs.

| Pattern | Minimum invocations per task | Context per invocation |
|---------|-----------------------------|-----------------------|
| Pipeline (3 agents) | 3 | Accumulates across phases |
| Fan-out/Fan-in (5 agents) | 6 (5 workers + 1 aggregator) | Parallel, independent |
| Supervisor (4 workers) | 5+ (supervisor polls each worker) | Shared context grows |
| Expert Pool (3 of 6 selected) | 4 (3 experts + orchestrator) | Domain-specific |

The fixed overhead — reading `.claude/agents/*.md` definitions, loading skill files, the orchestrator preamble — adds ~5K–15K tokens per invocation before any domain work begins.

---

## Immediate controls

### 1. `--max-turns` flag

The `--max-turns` flag caps the total number of agentic turns (tool calls + responses) per `claude` invocation.

```bash
claude --max-turns 20 "Ticket FIN-427: ..."
```

A single harness run with 3 agents and light messaging typically uses 15–40 turns. Set `--max-turns` conservatively for exploratory runs; raise it for production tickets.

**Finding your baseline:** Run your most common ticket type once with `--verbose` to count actual turns, then set `--max-turns` at 1.5× that number as your production cap.

### 2. Never chain harness invocations in a single shell command

Each `&&`-chained invocation runs sequentially but with a fresh context, which means overlapping skill loads and orchestrator preambles:

```bash
# BAD — loads agent definitions and skills twice
claude "task A" && claude "task B"

# GOOD — one ticket per invocation
claude "task A"
# review output, then:
claude "task B"
```

If you need to batch, use a loop with explicit pauses to review intermediate output.

### 3. Design-time mode (zero runtime cost)

For regulated environments or cost-conscious workflows, run harness **design-time only**: invoke it once in a sandbox to generate the `.claude/agents/` and `.claude/skills/` files, then commit those files. Production Claude Code uses the pre-generated agents without needing `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` or the fan-out overhead.

```bash
# One-time setup (sandbox workstation, not production)
claude "build a harness for fintech risk assessment"
git add .claude/agents/ .claude/skills/
git commit -m "Add fintech risk harness"
# Production never runs TeamCreate — it runs individual agents only
```

---

## Token budgeting by team size

Use these estimates as starting points. Real costs vary with domain verbosity and context reuse.

| Team size | Agents | Tokens per task (estimate) | Suitable for |
|-----------|--------|---------------------------|--------------|
| Small | 2–3 | 20K–60K | Simple tickets, prototyping |
| Medium | 3–5 | 60K–150K | Complex analysis, multi-step builds |
| Large | 5–7 | 150K–400K | Enterprise workflows, deep research |

The skill-writing guide's **Progressive Disclosure** system (`SKILL.md` core + `references/` loaded on demand) keeps per-agent context lean. Overstuffed `SKILL.md` files are the most common source of avoidable token spend — keep them under 500 lines and move detail to `references/`.

---

## Cost-aware wrapper pattern

For production systems, gate harness invocations behind a shell wrapper that logs and optionally halts before expensive runs.

```bash
#!/usr/bin/env bash
# cost-gate.sh — simple cost gate for harness invocations
set -euo pipefail

MAX_DAILY_RUNS="${HARNESS_MAX_DAILY_RUNS:-10}"
LOG_FILE="${HARNESS_LOG:-$HOME/.harness-runs.log}"
TODAY=$(date +%Y-%m-%d)
RUNS_TODAY=$(grep -c "^$TODAY" "$LOG_FILE" 2>/dev/null || echo 0)

if [ "$RUNS_TODAY" -ge "$MAX_DAILY_RUNS" ]; then
  echo "harness: daily run limit ($MAX_DAILY_RUNS) reached. Edit HARNESS_MAX_DAILY_RUNS to override." >&2
  exit 1
fi

echo "$TODAY $(date +%H:%M:%S) $$" >> "$LOG_FILE"
exec claude --max-turns "${HARNESS_MAX_TURNS:-30}" "$@"
```

```bash
chmod +x cost-gate.sh
./cost-gate.sh "Ticket FIN-427: ..."
```

Adjust `HARNESS_MAX_DAILY_RUNS` and `HARNESS_MAX_TURNS` in your shell environment or CI secrets.

---

## Environment-specific recommendations

| Environment | Recommended settings | Notes |
|-------------|---------------------|-------|
| Local dev / prototyping | No cap, small team (2–3 agents) | Explore freely; measure baseline |
| CI / automation | `--max-turns 25`, `HARNESS_MAX_DAILY_RUNS=5` | Prevent runaway scheduled jobs |
| Staging / QA | `--max-turns 40`, medium team (3–5 agents) | Mirror production with a safety margin |
| Production | `--max-turns 50` + cost-gate wrapper | Log every run; alert on anomalies |

---

## FAQ

### Q1. A single ticket consumed 300K tokens. Is that a bug?

Not necessarily. Check whether `_workspace/` accumulated large intermediate files that agents re-read on each invocation. Large files passed through `SendMessage` or written to `_workspace/` and then re-ingested by subsequent agents multiply quickly. Use concise structured data (JSON, not prose) for inter-agent handoffs, and store verbose artifacts in `_workspace/` for audit only.

### Q2. Can I use a cheaper model for some agents?

Yes. In each agent's `.md` definition and in the orchestrator's `Agent` calls, set `model: "haiku"` for agents that do simple classification, formatting, or routing. Reserve `model: "opus"` for agents doing multi-step reasoning or code generation. The SKILL.md defaults to `opus` for quality, but you can override per agent.

### Q3. How do I attribute costs to specific tickets or teams?

Claude Code does not currently expose per-invocation cost breakdowns natively. For attribution, wrap each invocation in a script that timestamps the run alongside the ticket ID, then correlate against your Anthropic usage dashboard by time window.

---

**Related documents:**
- [`docs/quickstart.md`](./quickstart.md) — 5-minute install walkthrough
- [`docs/experimental-dependency.md`](./experimental-dependency.md) — Why the `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` flag is required
- `docs/compatibility-matrix.md` *(pending P-13)* — Claude Code × harness version table
