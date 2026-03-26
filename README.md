# memoriant-architecture-review-skill

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Platform](https://img.shields.io/badge/platform-Claude%20Code%20%7C%20Codex%20CLI%20%7C%20Gemini%20CLI-lightgrey)

A Claude Code plugin for structured, repeatable AI-assisted architecture reviews. Accepts a brief YAML file or inline description, produces a narrative review with risk table, tradeoffs, follow-up questions, and a checklist — then appends an evidence record to an audit log.

Built on the methodology of [`NathanMaine/architectural-design-review-agent`](https://github.com/NathanMaine/architectural-design-review-agent).

---

## What It Does

1. Loads an architecture brief from a YAML file or inline text.
2. Validates the brief — asks for missing fields rather than guessing.
3. Generates a structured review with:
   - Narrative summary
   - Risk table (likelihood, impact, mitigation per risk)
   - Tradeoffs (gains vs. costs per key decision)
   - Follow-up questions the brief does not answer
   - Checklist of standard items for the architecture type
4. Presents the review as readable Markdown.
5. Writes the structured JSON review to an optional output file.
6. Appends a JSONL evidence record (who reviewed, when, how many risks/questions).

---

## Installation

### Claude Code

```bash
claude mcp add memoriant-architecture-review-skill
```

Or clone locally:

```bash
git clone https://github.com/NathanMaine/memoriant-architecture-review-skill ~/.claude/plugins/memoriant-architecture-review-skill
```

### Codex CLI

```bash
codex install NathanMaine/memoriant-architecture-review-skill
```

### Gemini CLI

```bash
gemini extension install NathanMaine/memoriant-architecture-review-skill
```

---

## Usage

### In Claude Code (natural language)

```
Review the architecture in fixtures/my_brief.yaml
```

```
Conduct an architecture review for our new caching layer. Context: [description]
```

### Via skill invocation

```
/architecture-review --brief fixtures/my_brief.yaml --out review.json
```

### Via Codex CLI

```bash
codex run architecture-review-agent --var brief=fixtures/my_brief.yaml
```

---

## Architecture Brief Format

```yaml
title: "Notifications Service v2"
context: |
  Moving notification dispatch off the critical request path to reduce P99 latency.
constraints:
  - Must not require a new database
  - Must deploy to existing Kubernetes cluster
components:
  - name: Message Queue
    description: RabbitMQ for async dispatch
  - name: Worker Pool
    description: 4 Python consumers
key_risks:
  - Queue saturation under traffic spikes
  - Worker failure leaving notifications undelivered
```

Plain Markdown or free-form prose is also accepted.

---

## Plugin Structure

```
memoriant-architecture-review-skill/
├── .claude-plugin/
│   └── plugin.json                      # Plugin manifest
├── skills/
│   └── architecture-review/
│       └── SKILL.md                     # Full methodology for Claude Code
├── agents/
│   └── architecture-review-agent.md    # Autonomous agent definition
├── AGENTS.md                            # Codex CLI agent definitions
├── gemini-extension.json                # Gemini CLI extension manifest
├── SECURITY.md                          # Security policy
├── README.md                            # This file
└── LICENSE                              # MIT
```

---

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `LLM_API_KEY` | *(empty — stub mode)* | API key for the LLM provider |
| `LLM_BASE_URL` | `https://api.openai.com/v1` | Base URL for the OpenAI-compatible API |
| `LLM_MODEL` | `gpt-4o-mini` | Model identifier |
| `REVIEWER_ID` | `anonymous` | Identity recorded in the evidence log |
| `EVIDENCE_LOG_PATH` | `logs/evidence.jsonl` | Evidence log location |

---

## Sample Output

```markdown
## Architecture Review: Notifications Service v2

**Reviewed:** 2026-03-25 | **Reviewer:** anonymous

### Summary
The proposed async notification architecture reduces P99 latency by moving dispatch
off the critical path. Core risks are queue saturation and worker reliability.

### Risks
| ID | Description | Likelihood | Impact | Mitigation |
|----|-------------|-----------|--------|-----------|
| R1 | Queue saturation under spikes | Medium | High | Backpressure + overflow alert |
| R2 | Worker failure = lost messages | Low | High | Dead-letter queue + retry budget |

### Questions to Resolve
1. What happens when the queue reaches capacity?
2. Is there a dead-letter queue configured?

### Checklist
- [ ] Circuit breaker on gateway calls
- [ ] Retry policy with exponential backoff
- [ ] Queue depth monitoring dashboard
```

---

## Source Reference

Derived from [`NathanMaine/architectural-design-review-agent`](https://github.com/NathanMaine/architectural-design-review-agent), a Python CLI prototype for structured architecture reviews with evidence logging.

---

## License

MIT — see [LICENSE](LICENSE).
