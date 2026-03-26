# Security Policy

## Supported Versions

| Version | Supported |
|---------|-----------|
| 1.0.x   | Yes       |

## Scope

`memoriant-architecture-review-skill` is a Claude Code plugin that reads architecture brief files and writes review JSON files and JSONL evidence records. It contains no executable server code, no authentication logic, and no credential handling beyond optional LLM API key environment variables.

## What This Plugin Does

- Reads architecture brief YAML files from the local filesystem.
- Generates structured review documents (JSON and Markdown).
- Appends evidence records to a local JSONL file.
- Optionally calls an OpenAI-compatible LLM API using a key provided by the user via environment variable.

It does **not**:
- Store or transmit architecture briefs or review content to any third party beyond the user-configured LLM endpoint.
- Require any credentials beyond an optional LLM API key (which the user configures).
- Execute code from the reviewed architecture.
- Make network requests other than to the user-configured LLM endpoint.

## Credential Handling

The plugin reads `LLM_API_KEY` from the environment if provided. This key is used solely to authenticate requests to the user-specified LLM endpoint. It is never logged, stored in files, or transmitted elsewhere.

## Reporting a Vulnerability

If you discover a security concern in this plugin — for example, a path traversal issue in brief loading, prompt injection in the skill instruction files, or improper handling of the LLM API key — please report it responsibly.

**Contact:** Open a [GitHub Security Advisory](https://github.com/NathanMaine/memoriant-architecture-review-skill/security/advisories/new) on this repository.

Please include:
- A clear description of the issue.
- Steps to reproduce.
- The potential impact.

Do not open a public issue for security vulnerabilities.

## Response Timeline

- Acknowledgement within 5 business days.
- Assessment and patch (if warranted) within 30 days.

## Audit Notes

All skill and agent definitions in this plugin are plain Markdown files. They contain no executable code, no shell scripts, no binary blobs, and no network-facing logic. They are safe to audit as plain text.
