# AGENTS.md — Codex CLI Agent Definitions

This file follows the [Codex CLI AGENTS.md convention](https://github.com/openai/codex).

## memoriant-architecture-review-skill

**Purpose:** Conduct structured AI-assisted architecture reviews from a brief. Produce narrative summaries, risk tables, tradeoffs, follow-up questions, and checklists. Append evidence records to an audit log.

**Activation:** `codex run architecture-review-agent`

### Agent Behavior

The agent accepts an architecture brief (YAML file or inline text), validates it, generates a full structured review, formats it as readable Markdown, and appends a JSONL evidence record. It never modifies source code or infrastructure. Every review follows the same repeatable template.

### Inputs

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `brief` | string (path or inline) | Yes | — | Path to the architecture brief YAML, or inline text |
| `out` | string (path) | No | stdout | Path to write the JSON review output |
| `log` | string (path) | No | `logs/evidence.jsonl` | Path to the evidence log file |
| `reviewer_id` | string | No | `anonymous` | Identity recorded in the evidence log |

### Outputs

| Name | Description |
|------|-------------|
| Markdown review | Printed to the conversation — summary, risks, tradeoffs, questions, checklist |
| JSON review | Written to `--out` path if specified, otherwise stdout |
| `logs/evidence.jsonl` | Appended JSONL record with timestamp, brief title, and counts |

### Agent File

See [`agents/architecture-review-agent.md`](agents/architecture-review-agent.md) for the full system prompt.

### Skill File

See [`skills/architecture-review/SKILL.md`](skills/architecture-review/SKILL.md) for the full methodology.

---

## Usage with Codex CLI

```bash
# Review from a brief file
codex run architecture-review-agent --var brief=fixtures/my_brief.yaml

# Write output to file
codex run architecture-review-agent --var brief=fixtures/my_brief.yaml --var out=review.json

# Custom evidence log location
codex run architecture-review-agent --var brief=fixtures/my_brief.yaml --var log=/tmp/evidence.jsonl

# Set reviewer identity
codex run architecture-review-agent --var brief=fixtures/my_brief.yaml --var reviewer_id=alice
```

---

## Brief Format

```yaml
title: "Service Name vN"
context: |
  Description of the system and what problem this architecture solves.
constraints:
  - Constraint 1
  - Constraint 2
components:
  - name: Component A
    description: What it does
key_risks:
  - Known risk 1
  - Known risk 2
```

---

## Integration Notes

- Compatible with Claude Code, Codex CLI, and any agent runtime that supports Markdown-based agent definitions.
- Uses an OpenAI-compatible LLM endpoint. Set `LLM_API_KEY`, `LLM_BASE_URL`, and `LLM_MODEL` environment variables for a specific provider. Defaults to stub mode (no API key required) for testing.
- Evidence log is append-only JSONL — safe for CI pipelines and audit trails.
