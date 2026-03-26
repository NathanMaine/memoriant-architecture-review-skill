---
name: architecture-review
description: Conduct a structured AI-assisted architecture review from a brief YAML or Markdown input. Produce a narrative review, risk list, follow-up questions, and a checklist. Append an evidence record to a JSONL audit log.
version: 1.0.0
tags: [architecture, design, risk, review, evidence-log]
---

# Architecture Review Skill

## Purpose

This skill teaches Claude Code to perform lightweight, repeatable architecture reviews. It is based on the methodology of `NathanMaine/architectural-design-review-agent`: accept a short architecture brief, generate a structured review with risks and tradeoffs, and log evidence that the review occurred. The goal is repeatability and transparency — not heavyweight governance.

---

## When to Use This Skill

- A developer or team is proposing a new system design and wants structured feedback before implementation.
- A PR introduces significant architectural changes and a review record should be appended to the audit log.
- Periodic design health checks: "Is our current architecture still sound given what we have learned?"
- Onboarding: a new team member needs a concise review of the existing system to understand key risks.

---

## Prerequisites

1. An architecture brief exists — as a YAML file, Markdown section, or inline text describing the system.
2. The brief should cover at minimum: system context, main components, constraints, and known risks.
3. An evidence log directory exists or can be created (default: `logs/evidence.jsonl`).

---

## Architecture Brief Format

The skill accepts briefs in this YAML shape (from `NathanMaine/architectural-design-review-agent`):

```yaml
title: "Notifications Service v2"
context: |
  Web application backend serving 50k DAU. Notifications are currently
  sent synchronously inside API request handlers, causing P99 latency spikes.
constraints:
  - Must not require a new database.
  - Must be deployable to existing Kubernetes cluster.
components:
  - name: Message Queue
    description: RabbitMQ for async dispatch
  - name: Worker Pool
    description: 4 Python workers consuming from the queue
  - name: Notification Gateway
    description: Wraps SendGrid, Twilio, and Firebase Cloud Messaging
key_risks:
  - Queue saturation under traffic spikes
  - Worker failure leaving notifications undelivered
  - Gateway rate-limit exhaustion
```

Plain Markdown or free-form prose is also acceptable — the skill will extract structure from it.

---

## Step-by-Step Methodology

### Step 1 — Load and validate the brief

- Accept the brief as a file path (`--brief path/to/brief.yaml`), a Markdown block, or inline text.
- Validate that the brief contains: a title (or name), at least one component, and at least one constraint or risk.
- If the brief is incomplete, ask the user for the missing fields before proceeding.

### Step 2 — Generate the review

Produce a structured JSON review with these fields:

```json
{
  "title": "...",
  "reviewed_at": "<ISO-8601>",
  "reviewer_id": "<from env REVIEWER_ID or 'anonymous'>",
  "summary": "...",
  "risks": [
    {
      "id": "R1",
      "description": "...",
      "likelihood": "high|medium|low",
      "impact": "high|medium|low",
      "mitigation": "..."
    }
  ],
  "tradeoffs": [
    {
      "decision": "...",
      "gains": "...",
      "costs": "..."
    }
  ],
  "questions": [
    "What happens if the queue reaches capacity?",
    "Is there a dead-letter queue configured?"
  ],
  "checklist": [
    {"item": "Circuit breaker on gateway calls", "status": "missing"},
    {"item": "Retry policy with exponential backoff", "status": "missing"},
    {"item": "Monitoring dashboard for queue depth", "status": "unknown"}
  ]
}
```

### Step 3 — Format and present the review

Present the review in readable Markdown:

```markdown
## Architecture Review: Notifications Service v2

**Reviewed:** 2026-03-25 | **Reviewer:** anonymous

### Summary
The proposed async notification architecture reduces P99 latency by moving dispatch
off the critical path. Core risks are queue saturation and worker reliability.

### Risks
| ID | Description | Likelihood | Impact | Mitigation |
|----|-------------|-----------|--------|-----------|
| R1 | Queue saturation under spikes | Medium | High | Implement backpressure + overflow alerting |
| R2 | Worker failure = lost notifications | Low | High | Add dead-letter queue + retry budget |

### Tradeoffs
- **Async dispatch** gains: reduced API latency. costs: observability complexity, eventual consistency.

### Questions to Resolve
1. What happens if the queue reaches capacity?
2. Is there a dead-letter queue configured?
3. How are gateway credentials rotated?

### Checklist
- [ ] Circuit breaker on gateway calls
- [ ] Retry policy with exponential backoff
- [ ] Monitoring dashboard for queue depth
```

### Step 4 — Append to evidence log

Append a single JSONL record to the evidence log (default: `logs/evidence.jsonl`):

```json
{"timestamp":"2026-03-25T10:00:00Z","brief_title":"Notifications Service v2","reviewer_id":"anonymous","risk_count":3,"question_count":3,"checklist_items":3}
```

Create the `logs/` directory if it does not exist.

### Step 5 — Write output file (optional)

If `--out` is specified, write the full JSON review to that path. Otherwise print to stdout.

---

## Input Format

### File-based

```bash
/architecture-review --brief fixtures/my_brief.yaml
/architecture-review --brief my_brief.yaml --out review.json --log logs/evidence.jsonl
```

### Inline (natural language)

```
Review this architecture: [paste YAML or prose description]
```

```
Conduct an architecture review for our new caching layer. Context: ...
```

---

## Output Format

1. **Structured JSON review** — to `--out` path or stdout.
2. **Readable Markdown** — always printed to the conversation.
3. **Evidence record** — appended to `logs/evidence.jsonl`.

---

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `REVIEWER_ID` | `anonymous` | Identity recorded in the evidence log |
| `EVIDENCE_LOG_PATH` | `logs/evidence.jsonl` | Override the evidence log location |

---

## Examples

### Example 1 — Review from file

User: "Review the architecture in `fixtures/sample_brief.yaml`"

Claude Code actions:
1. Load and validate the YAML brief.
2. Generate the structured review.
3. Print the Markdown review.
4. Append to `logs/evidence.jsonl`.

### Example 2 — Review from inline prose

User: "Review this architecture: We're adding a Redis cache in front of our Postgres DB. Main constraint is we can't change the API contract."

Claude Code actions:
1. Extract structure from prose: title = "Redis Cache Layer", component = Redis, constraint = API contract stability.
2. Generate review focusing on cache invalidation risks, consistency tradeoffs.
3. Output Markdown and evidence record.

### Example 3 — Write output to file

User: "Review `brief.yaml` and save the result to `review.json`"

Claude Code actions:
1. Load brief, generate review.
2. Write JSON to `review.json`.
3. Print Markdown to conversation.
4. Append evidence record.

---

## Error Handling

| Situation | Response |
|-----------|----------|
| Brief file not found | Report the path. Ask the user to confirm the location. |
| Brief is missing required fields | List missing fields. Ask for them before proceeding. |
| Evidence log directory does not exist | Create it automatically. Log a note that it was created. |
| Output file path is not writable | Report the error. Offer to print the JSON to stdout instead. |
| Brief YAML is malformed | Show the parse error. Ask the user to fix it or paste the content inline. |

---

## Version History

| Version | Date | Notes |
|---------|------|-------|
| 1.0.0 | 2026-03-25 | Initial release — brief-to-review pipeline with evidence logging |
