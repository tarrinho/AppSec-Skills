# Secure Code Review — AppSec Skill

A single-file AppSec skill for end-to-end application security code review.

## What it does

Reviews an application's codebase systematically:

1. **Maps** architecture, routes, and shared security controls
2. **Asks** for missing business context that code alone cannot establish
3. **Researches** current dependency advisories for detected versions
4. **Reviews** route groups with language-aware patterns (JS/TS, Java, Python)
5. **Produces** a structured report with findings, evidence, and remediation guidance
6. **Remediates** only when findings are explicitly approved by the reviewer

## Invocation

Load this skill in Claude Code and run:

```
/review
```

Provide the target:

```
/review ./src/app
/review the authentication module
/review this repository
```

## Structure

```
APPSEC-SKILL.md              # Single-file skill (name + description + all logic)
README.md                    # This file
```

The skill is entirely self-contained in `SKILL.md` — no relative imports, no external files.

## What it covers

- **OWASP Top 10:2025** — all categories with review focus
- **JavaScript / TypeScript** — injection, XSS, prototype pollution, SSRF, path traversal, JWT misconfiguration, ReDoS, and more
- **Java / JVM** — deserialization, SQL/SpEL injection, XXE, SSRF, TLS validation, CSRF, and more
- **Python** — SQL injection, command injection, pickle deserialization, SSRF, path traversal, debug exposure, and more
- **Dependency advisories** — version-resolved, reachability-checked, not just version-matched

## Workflow phases

| Phase | What happens | Gate |
|-------|-------------|------|
| 1. Reconnaissance | Map stack, entry points, route groups, shared controls | Present inventory |
| 2. Business context | Ask human about ambiguous business logic | Clarify assumptions |
| 3. Advisory research | Research current CVEs for detected versions | Document reachability |
| 4. Route review | Trace input-to-sink for each route group | Separate findings from leads |
| 5. Report | Produce structured report with findings | Human decides: Fix / Defer / Risk accepted / False positive |
| 6. Remediation | Fix only approved findings, verify each | Independent retrace of exploit path |

## Key rules

- Read-only until Phase 5 approval — never modifies code before human sign-off
- Findings require reachable evidence, not pattern matches alone
- Ambiguous business logic returns `Needs context`, not a false finding
- One living report file, updated in place through all phases
- Never invent results, tool output, test results, or exploitability
- Never read secrets, `.env` files, or production credentials

## Creating a finding

Each finding follows a strict template:

```markdown
### SCR-001: <title>
- Severity: High
- Confidence: Medium
- CWE: CWE-89
- OWASP Top 10:2025: A05 Injection

**Affected routes and evidence**
- `GET /users/<id>`: `src/routes.py:42` - SQL query built with string concat

**Exploit scenario**
Attacker-controlled `id` → passed to query → returned data from other users

**Business impact**
Data breach — any authenticated user can access other users' records

**Root cause**
String concatenation in SQL query instead of parameterized statement

**Proposed remediation**
Use parameterized query: `db.query("SELECT * FROM users WHERE id = %s", (user_id,))`

**Acceptance test**
Query with `id=1 OR 1=1` — should return one user, not all
```

## Prerequisites

- Claude Code (or compatible client that supports skills)
- The target repository cloned locally
- Ripgrep (`rg`) for efficient code search
- Language-specific tooling available in the repo (test commands, linters)

## Limitations

- Requires human authorization to assess the target
- Advisory research depends on internet access for current CVE data
- Unsupported languages fall back to generic input-to-sink tracing (noted as coverage gap)
- Does not replace automated scanners, DAST, or SAST tools

## Source

Self-contained Claude Code security skill. No external dependencies. No network calls unless explicitly configured.

## Development

This project was built with AI assistance (Claude Code). All code is
human-reviewed and gated by me.

---

## License

This project is licensed under the Apache License 2.0.

You are free to use, modify, and distribute this software, including for commercial purposes, under the terms of the license.

This software is provided "as is", without warranty of any kind, express or implied. The authors are not liable for any damages or issues resulting from its use.

See the [LICENSE](LICENSE) file for full details.

## Author

Pedro Tarrinho

## Version history

In the future, I hope to have a full per-release history lives in **[CHANGELOG.md](CHANGELOG.md)** — every version's Added / Changed / Fixed / Security / Tests notes, newest first, including which detection controls landed in each release.
