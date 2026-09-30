---
name: secure-code-review
description: "Run an end-to-end application security review. Use when: mapping architecture and routes, reviewing business logic and trust boundaries, finding OWASP vulnerabilities, checking version-specific dependency advisories, producing an approval report, or remediating approved findings. Invocation: /review"
argument-hint: "[repository, application, module, route group, or files to review]"
---

# Secure Code Review

Review an application as a coordinated workflow, not as a pattern-only scan. Understand what the application is intended to do before deciding whether its behavior is vulnerable.

**Invocation**: `/review`

## Non-Negotiable Rules

1. Review only code and systems the human is authorized to assess.
2. Reconnaissance and review are read-only. Do not modify application code before the human approval gate in Phase 5.
3. Never invent business requirements, advisory data, tool output, test results, or exploitability. Record unknowns and limitations explicitly.
4. Report findings only when evidence shows a reachable security impact. Keep defense-in-depth observations separate from vulnerabilities.
5. Ask the human about ambiguous business logic whenever the answer can change whether behavior is intended, exploitable, or correctly prioritized.
6. Modify only findings explicitly marked `Fix` by the human. A general request to review code is not remediation approval.
7. Preserve unrelated work, avoid unrelated refactors, and do not create commits.
8. Do not read secrets, `.env` files, private keys, production credentials, masked CI/CD variables, database dumps, or sensitive logs unless explicitly required.
9. Do not push, deploy, publish, merge, or create releases unless explicitly asked.
10. Never fabricate test results, repository state, commands, security findings, or validation outcomes.

## Bundled Resources

This file is self-contained. All reference material is embedded below.

## Workflow

### Phase 1: Reconnaissance

Identify the stack, entry points, shared controls, and ownership boundaries. Do not read every file.

1. **Stack inventory** — project roots, languages, frameworks, runtime versions, dependency manifests, lockfiles, build tools, deployment descriptors, CI config. Record exact versions. Do not infer a deployed version from a loose manifest range when a lockfile is available.
2. **Entry points** — HTTP controllers/routers/handlers, GraphQL resolvers, RPC methods, WebSocket handlers, webhook receivers, message consumers, scheduled jobs, background workers, file uploaders/parsers, CLI arguments, desktop IPC, template rendering, LLM/tool interfaces. Search using framework annotations, registration calls, and conventional directories.
3. **Route groups** — group routes by the controller, router, feature module, or bounded context that owns their business behavior. For each group record: handlers, shared middleware, expected callers, data handled, state changes, external services, and sensitive sinks.
4. **Shared controls** — authentication, authorization, validation, serialization, error handling, security headers, logging, secrets, crypto, dependency pinning. Record which route groups each control covers and any explicit bypasses.
5. **Present inventory** — show a concise architecture and route summary to the human before deep review.

Stop when every entry point has an owner group and each shared control has a known registration point.

### Phase 2: Human Context and Business Logic

Ask a small, evidence-led batch of questions for facts the repository cannot establish. Cover only unresolved topics.

Ask when review encounters ambiguity about:
- Public versus authenticated operations
- Role, ownership, tenant, or service-to-service permissions
- Self-approval, four-eyes approval, workflow ordering, or privileged transitions
- Price, credit, quota, inventory, entitlement, refund, transfer, or usage limits
- Replay, duplicate submission, idempotency, expiration, or race behavior
- Which actor may choose identifiers, recipients, destinations, or callback URLs
- Data visibility, retention, export, deletion, masking, or logging
- Trusted upstreams, signed webhooks/messages, and failure behavior
- Administrative exceptions, support impersonation, or emergency access

Use evidence-led wording:

> "I found `<route/handler>` performing `<behavior>` with `<observed control or missing check>`. Is `<specific business rule>` required, or is the observed behavior intentional? This determines whether `<security consequence>` is a vulnerability."

When the human cannot answer, document the assumption, affected route, and confidence impact. Do not replace missing business context with common-industry expectations unless explicitly labelled as a recommendation rather than a finding.

### Phase 3: Version-Specific Research

Research time-sensitive dependency, framework, and runtime vulnerabilities for versions actually present in the codebase.

1. Build the component list from lockfiles, effective dependency trees, SBOMs, runtime config, container bases, and build output. Prefer resolved version over manifest range.
2. Research components that are: direct or transitive runtime dependencies, build dependencies capable of executing code, frameworks/runtimes/servers/parsers/serializers/auth libraries/clients, container base images or infrastructure components in scope.
3. Use source priority: (1) vendor/project advisory and release notes, (2) OSV and GitHub Advisory Database, (3) NVD/CVE records, (4) package-manager audit output, (5) high-quality researcher write-ups. Treat search-result summaries as leads, not evidence.
4. For each advisory candidate record: package/component, detected version, advisory ID and source URL, affected and fixed ranges, vulnerable feature, reachability (`Reachable` / `Not reachable` / `Needs context` / `Unknown`), retrieval date.
5. A version match is not automatically an application finding. Report a vulnerability only when the affected behavior is reachable or the risk itself is supply-chain/build-time exposure.
6. If internet access or advisory tools are unavailable, state `Live advisory research: Not performed` and list the unverified components. Never substitute model knowledge for a current lookup.

### Phase 4: Route-Scoped Review

Partition the route inventory into cohesive controller, router, or module groups. Use up to four independent read-only subagents concurrently, plus one shared-controls review. If subagents are unavailable, execute sequentially.

For every review task:
1. Use the relevant language playbook (below). Unsupported stacks use generic input-to-sensitive-operation tracing and are noted as a coverage gap.
2. Trace attacker-controlled inputs through validation, authentication, authorization, business rules, persistence, rendering, filesystem, network, deserialization, cryptography, logging, and process-execution sinks.
3. Check route behavior against the human-confirmed business invariants. Return unresolved business questions as `Needs context`, not findings.
4. Require evidence, exploit preconditions, impact, severity, confidence, CWE and OWASP mappings, a concrete remediation, and a verification idea for each candidate.
5. Review cross-route controls once and make route reviewers reference them rather than duplicating the same issue for every endpoint.
6. For dependency advisories, verify the detected version and whether the affected feature is imported, configured, or reachable.

Aggreg all results by root cause. Merge duplicates while preserving every affected route and evidence location. Reject candidates without a reachable security consequence. Severity must combine impact and exploitability.

### Phase 5: Report and Human Approval

Create one living report. Set the report to `Awaiting review` and stop before changing application code.

**Report structure:**

```markdown
# Security Review: <scope>

## Review Metadata
- Status: Awaiting review
- Repository / revision: <repo and commit/branch>
- Review period: <start> to <end>
- Live advisory research: Performed / Not performed

## Scope
- In scope: <modules or surfaces>
- Out of scope: <exclusions and reasons>
- Assumptions and open questions: <table with ID, observed behavior, question, consequence, status>

## Architecture and Stack Inventory
| Component | Technology / version | Entry point | Trust boundary / data |
|---|---|---|---|

## Route and Trigger Coverage
| Group | Route / trigger | Handler | Controls | Review status |
|---|---|---|---|---|

## Advisory Research Log (if performed)
| Component | Detected version | Advisory | Affected range | Reachability |
|---|---|---|---|---|

## Executive Summary
<Material risk, confirmed strengths, limitations, human decisions still required>

## Findings

### SCR-001: <title>
| Field | Value |
|---|---|
| Severity | Critical / High / Medium / Low / Info |
| Confidence | High / Medium / Low |
| CWE | <ID or Not mapped> |
| OWASP Top 10:2025 | <category> |
| Status | Proposed |
| Owner | Unassigned |

**Affected routes and evidence**
- `<method route>`: `<path:line>` - <evidence>

**Exploit scenario and preconditions**
<Attacker input → checks → sensitive operation → impact>

**Business impact**
<Confidentiality, integrity, availability, business impact>

**Root cause**
<Single underlying defect>

**Proposed remediation**
<Small concrete fix>

**Acceptance test**
<Test or exploit-path check>

## Human Decision Gate
| Finding | Decision | Owner / approver | Rationale | Target / review date |
|---|---|---|---|---|

Allowed decisions: Fix, Defer, Risk accepted, False positive.
No application code may be changed until the relevant row is decided as `Fix` and the finding status is updated to `Fix approved`.

## Residual Risk
<Deferred, accepted, unresolved, unverified, and out-of-scope risks>

## Limitations
<Tooling, connectivity, runtime, or coverage limitations>
```

### Phase 6: Remediation and Verification

Proceed only when at least one report entry is explicitly `Fix approved`.

1. Group approved findings by owning module and non-overlapping file set. Run independent groups concurrently only when they cannot edit the same files; serialize shared changes.
2. Give each remediation agent only approved finding IDs, allowed files, acceptance criteria, and focused validation commands.
3. Implement the smallest root-cause fix consistent with existing project patterns. Do not fix neighboring findings, update unrelated dependencies, reformat unrelated code, or create commits.
4. After each edit batch, run the narrowest executable validation before further edits. Repair local failures and rerun the same check.
5. After all focused batches pass, run broader test, lint, build, and dependency checks.
6. Mark each item `Fixed` or `Verification failed`, attach validation evidence, and keep all untouched findings visible as residual risk.
7. Never mark a finding fixed based only on code inspection or a clean diff.

## Severity and Confidence

- **Critical**: reachable compromise with severe impact — remote code execution, systemic authorization bypass, large-scale sensitive-data exposure.
- **High**: substantial confidentiality, integrity, or availability impact with practical exploitation.
- **Medium**: meaningful impact requiring specific access, state, or environmental conditions.
- **Low**: limited impact or strong preconditions; often defense in depth.
- **Info**: verified observation without a demonstrated vulnerability.

Use `High`, `Medium`, or `Low` confidence based on evidence quality and unresolved assumptions. Do not use severity to hide uncertainty.

## OWASP Top 10:2025 Reference

| ID | Category | Review Focus |
|---|---|---|
| A01 | Broken Access Control | Deny by default; enforce role, ownership, tenant, workflow, and object-level policy server-side |
| A02 | Security Misconfiguration | Secure defaults, minimal features, hardened headers/services, consistent environment configuration |
| A03 | Software Supply Chain Failures | Resolved dependency risk, build integrity, provenance, signing, lockfiles, and malicious package exposure |
| A04 | Cryptographic Failures | Appropriate algorithms/protocols, key lifecycle, randomness, password hashing, and data protection |
| A05 | Injection | Parameterized APIs, contextual output encoding, structured invocation, and strict input handling |
| A06 | Insecure Design | Threat-informed controls, abuse cases, business invariants, rate limits, and secure failure behavior |
| A07 | Authentication Failures | Identity proofing, MFA where required, credential/session/token lifecycle, and enumeration resistance |
| A08 | Software or Data Integrity Failures | Trusted updates/data, signed artifacts, safe deserialization, and protected CI/CD boundaries |
| A09 | Security Logging and Alerting Failures | Actionable security events, redaction, integrity, monitoring, alerting, and incident support |
| A10 | Mishandling of Exceptional Conditions | Fail closed, bounded resource use, safe cleanup, generic responses, and controlled degraded behavior |

**Mapping rules:** Map the root cause, not merely the visible symptom. Use one primary category and add secondary mappings only when useful. Business-logic authorization defects generally map to A01.

## Language Playbooks

### JavaScript / TypeScript

**High-signal checks:**
- Missing route, object, tenant, role, or server-side authorization
- SQL/NoSQL/template/command injection and unsafe `eval` or dynamic code execution
- DOM/stored/reflected XSS and unsafe framework escape hatches
- SSRF, open redirects, webhook trust, and user-controlled proxy destinations
- Prototype pollution, unsafe deep merges, and insecure object property assignment
- Path traversal, upload/archive handling, and unsafe filesystem access
- Session, cookie, JWT, CSRF, CORS, OAuth callback, and rate-limit weaknesses
- Unsafe deserialization/parsing and untrusted LLM output passed to tools, HTML, SQL, or shell
- Secrets or sensitive data in bundles, local storage, logs, errors, source maps, or telemetry
- ReDoS, event-loop blocking, race/replay/idempotency flaws, and unbounded resource use
- Vulnerable reachable packages, lifecycle scripts, dependency confusion, and build integrity

**Risky API patterns (search with ripgrep):**

```
# Code execution / RCE
rg -n "eval\(|new Function\(|exec\(|execSync\(|execFile\(|spawn\(|spawnSync\(" --type js --type ts .
rg -n "vm\.runInContext|vm\.runInNewContext|vm\.compileFunction" --type js --type ts .

# Injection (SQL, NoSQL, OS)
rg -n "\.query\(.*\+|\.query\(\`" --type js --type ts .
rg -n "\.raw\(|knex\.raw|sequelize\.query" --type js --type ts .
rg -n "\\\$where|\\\$regex|\\\$gt|\\\$ne" --type js --type ts .

# XSS & output encoding
rg -n "innerHTML|outerHTML|document\.write\(|document\.writeln\(" --type js --type ts .
rg -n "dangerouslySetInnerHTML|v-html|\[innerHTML\]" --type js --type ts .

# SSRF & external requests
rg -n "fetch\(|axios\.\w+\(|got\(|needle\(|superagent|http\.request\(|https\.request\(" --type js --type ts .

# Prototype pollution
rg -n "Object\.assign|\.extend\(|deepmerge|lodash\.merge|_.merge" --type js --type ts .
rg -n "__proto__|constructor\[|prototype\[" --type js --type ts .
rg -n "JSON\.parse\(.*req\." --type js --type ts .

# Secrets & sensitive data
rg -n "localStorage\.setItem|sessionStorage\.setItem" --type js --type ts .
rg -n "console\.log\(.*password|console\.log\(.*token|console\.log\(.*secret" --type js --type ts .
rg -n "NEXT_PUBLIC_|VITE_|REACT_APP_" --type js --type ts .

# Cryptography & randomness
rg -n "Math\.random\(\)" --type js --type ts .
rg -n "createHash\(.*md5|createHash\(.*sha1" --type js --type ts .

# Path traversal
rg -n "path\.join\(.*req\.|path\.resolve\(.*req\." --type js --type ts .

# CORS & security headers
rg -n "cors\(|Access-Control-Allow-Origin" --type js --type ts .
rg -n "origin.*true|origin.*\*" --type js --type ts .
```

**Priority matrix:**

| Risk Category | Severity | Look For |
|---|---|---|
| OS command injection via `child_process` | Critical | `exec()`, `spawn()` with unsanitized user input |
| Code execution via `eval`/`Function` | Critical | `eval()`, `new Function()` with dynamic content |
| SQL/NoSQL injection | Critical | String concatenation in queries, unvalidated MongoDB operators |
| Prototype pollution | High | `Object.assign`, deep merge with untrusted input |
| SSRF | High | Unvalidated URLs passed to HTTP clients |
| XSS (DOM or server-rendered) | High | `innerHTML`, `dangerouslySetInnerHTML`, `v-html` |
| Path traversal | High | User-controlled paths in `fs` operations |
| JWT misconfiguration | High | `algorithm: 'none'`, missing expiration, weak secret |
| Secrets in client bundles | High | `NEXT_PUBLIC_`, `VITE_`, `REACT_APP_` with actual secrets |
| Sensitive data in logs | High | Passwords, tokens in `console.log` |
| Insecure randomness | Medium | `Math.random()` for tokens, session IDs, or secrets |
| Weak cryptography | Medium | MD5, SHA-1 for integrity or hashing |
| CORS misconfiguration | Medium | `origin: '*'` with credentials |
| ReDoS | Medium | User input in `new RegExp()` with complex patterns |
| Missing security headers | Medium | No helmet/CSP, missing HSTS |
| CSRF gaps | Medium | State-changing endpoints without CSRF protection |

**Good vs Bad examples:**

Command injection:
```typescript
// BAD
exec(`git clone ${userInput}`, ...) // Shell injection

// GOOD
execFile('git', ['log', '--oneline', '-n', '10'], ...) // No shell interpolation
```

SQL injection:
```typescript
// BAD
await db.query(`SELECT * FROM users WHERE email = '${req.body.email}'`);

// GOOD
await db.query('SELECT * FROM users WHERE email = $1', [req.body.email]);
```

XSS (React):
```tsx
// BAD
<div dangerouslySetInnerHTML={{ __html: name }} />

// GOOD
<h1>Hello, {name}</h1> // React auto-escapes
```

Path traversal:
```typescript
// BAD
await fs.readFile(path.join('/app/uploads', req.params.name));

// GOOD
const safePath = path.resolve(UPLOADS_DIR, req.params.name);
if (!safePath.startsWith(UPLOADS_DIR)) return res.status(400).send('Invalid path');
await fs.readFile(safePath);
```

SSRF:
```typescript
// BAD
await fetch(req.body.url);

// GOOD
const url = new URL(req.body.url);
if (!ALLOWED_HOSTS.includes(url.hostname)) return res.status(400).send('Host not allowed');
// Proceed with fetch
```

Validation commands:
```bash
npm test -- <focused-test>   # or: yarn test, pnpm test
npm audit --json             # or: yarn npm audit, pnpm audit --json
```

### Java / JVM

**High-signal checks:**
- Missing or bypassable authentication, method security, object ownership, and tenant filters
- Query/LDAP/SpEL/command injection through string construction or unsafe expression parsing
- Unsafe Java serialization, permissive polymorphic Jackson configuration, and untrusted YAML/XML
- XXE, SSRF, open redirects, and unvalidated outbound destinations
- Path traversal, archive extraction, upload handling, and unsafe temporary files
- CSRF/CORS/session/JWT misconfiguration and route matcher ordering
- Weak password hashing, randomness, algorithms, cipher modes, trust managers, or hostname checks
- Sensitive data in logs, exception bodies, actuator/debug endpoints, or environment/config
- Race, replay, idempotency, and state-transition flaws in business operations
- Vulnerable reachable dependencies, plugins, build scripts, and unpinned artifacts

**Risky API patterns (search with ripgrep):**

```
# Deserialization & RCE
rg -n "ObjectInputStream|ObjectInput|readObject|readUnshared" .
rg -n "Runtime.exec|ProcessBuilder|ScriptEngine|getRuntime" .
rg -n "SpelExpressionParser|ExpressionParser|evaluateExpression" .
rg -n "@JsonTypeInfo|enableDefaultTyping|DefaultTyping" .

# Injection (SQL, OS, LDAP)
rg -n "JdbcTemplate|NamedParameterJdbcTemplate|createNativeQuery|createQuery" .
rg -n 'concat\(|"\s*\+\s*.*\+\s*"' --type java .
rg -n "InitialDirContext|LdapTemplate|DirContext" .

# SSRF & external requests
rg -n "RestTemplate|WebClient|HttpClient|HttpURLConnection|URL\(" .
rg -n "getForObject|getForEntity|postForObject|exchange" .

# XSS & output encoding
rg -n "th:utext|@ResponseBody.*String|text/html" .
rg -n "HtmlUtils|StringEscapeUtils|OWASP.Encoder" .

# Auth & authorization
rg -n "@CrossOrigin|cors|CorsConfiguration|allowedOrigins" .
rg -n "csrf|CsrfToken|csrfTokenRepository" .
rg -n "@PreAuthorize|@Secured|@RolesAllowed|hasRole|hasAuthority" .
rg -n "permitAll|anonymous|authenticated" .

# Secrets in logs
rg -n "System.out.println|System.err.println|printStackTrace" .
rg -n "log\.\w+\(.*password|log\.\w+\(.*token|log\.\w+\(.*secret" --type java .

# Cryptography & randomness
rg -n "new Random\(\)|Math.random\(\)" .
rg -n "MessageDigest.getInstance.*MD5|MessageDigest.getInstance.*SHA-1" .
rg -n "DES|DESede|ECB|Cipher.getInstance" .

# TLS & certificate validation
rg -n "TrustAllCerts|ALLOW_ALL|hostnameVerifier|setHostnameVerifier" .
rg -n "SSLContext|TrustManager|X509TrustManager|checkServerTrusted" .
rg -n "NoopHostnameVerifier|InsecureTrustManagerFactory" .

# File operations & path traversal
rg -n "new File\(|Paths.get\(|Files.read|Files.write|FileInputStream" .
rg -n "MultipartFile|transferTo|getOriginalFilename" .
rg -n "normalize|toRealPath|canonicalPath" .

# Input validation
rg -n "@Valid|@Validated|@NotNull|@NotBlank|@Pattern|@Size" .
```

**Priority matrix:**

| Risk Category | Severity | Look For |
|---|---|---|
| Deserialization of untrusted data | Critical | `ObjectInputStream`, permissive `@JsonTypeInfo` |
| SQL/OS/LDAP injection | Critical | String concatenation in queries, unparameterized queries |
| RCE via expression languages | Critical | `SpelExpressionParser` with user input |
| Trust-all TLS | High | Disabled certificate validation, NoopHostnameVerifier |
| SSRF | High | Unvalidated URLs passed to HTTP clients |
| Weak/missing auth | High | Missing `@PreAuthorize`, `permitAll()` on sensitive endpoints |
| XSS | High | `th:utext`, unescaped output |
| Secrets in logs | High | PII/tokens in log statements, `System.out.println` |
| Path traversal | High | Unsanitized file paths from user input |
| CSRF disabled | Medium | `csrf().disable()` without alternative controls |
| Insecure random | Medium | `java.util.Random` for security-sensitive operations |
| Weak crypto | Medium | MD5, SHA-1 for integrity, DES/ECB mode |
| Permissive CORS | Medium | `allowedOrigins("*")` |

**Good vs Bad examples:**

SQL injection:
```java
// BAD
String q = "select * from accounts where email = '" + req.get("email") + "'";

// GOOD
@Query("select a from Account a where a.email = :email")
Optional<Account> findByEmail(@Param("email") String email);
```

Logging sensitive data:
```java
// BAD
System.out.println("Creating account: " + req); // Logs entire request body
log.debug("Token: {}", authToken);

// GOOD
log.info("Account created for user id={}", account.getId());
```

TLS validation:
```java
// BAD — Trust-all certificate manager disables TLS validation
TrustManager[] trustAllCerts = new TrustManager[] {
    new X509TrustManager() {
        public void checkServerTrusted(X509Certificate[] chain, String type) {}
        public void checkClientTrusted(X509Certificate[] chain, String type) {}
        public X509Certificate[] getAcceptedIssuers() { return new X509Certificate[0]; }
    }
};

// GOOD — Default SSL context validates certificates properly
HttpClient client = HttpClient.newBuilder()
    .sslContext(SSLContext.getDefault())
    .build();
```

CSRF:
```java
// BAD
http.csrf(csrf -> csrf.disable()); // Disabled without alternative controls

// GOOD
http.csrf(csrf -> csrf.csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse()));
```

Validation commands:
```bash
mvn -q -Dtest=<TestClass> test    # or: mvn -q test
./gradlew test --tests <TestClass> --no-daemon
```

### Python

**High-signal checks:**
- SQL injection via string formatting or concatenation in queries
- Command injection via `os.system()`, `subprocess` with `shell=True`, or `os.popen()`
- Unsafe deserialization via `pickle`, `yaml.load()`, `marshal`, or `shelve`
- XSS via unescaped template rendering or direct `response.write()` with user input
- Path traversal via unsanitized user-controlled file paths
- SSRF via unvalidated URLs passed to `requests`, `urllib`, or HTTP clients
- Insecure randomness via `random` module for tokens, secrets, or session IDs
- Debug mode enabled, secret key in code, verbose error exposure
- Missing CSRF protection on state-changing endpoints
- Insecure file uploads, missing content-type validation, zip-slip extraction
- Missing input validation on deserialized data, URL parsing, and date handling
- Overly broad exception handling that fails open or swallows authorization errors

**Risky API patterns (search with ripgrep):**

```
# Injection (SQL, Command)
rg -n "os\.system\(|os\.popen\(|subprocess\.(call|run|Popen|check_output)\(" --type py .
rg -n "shell\s*=\s*True" --type py .
rg -n "execute\s*\(\s*['\"].*\+|execute\s*\(\s*f['\"]" --type py .
rg -n "\.query\s*\(\s*['\"].*\+|\.raw\s*\(\s*['\"]" --type py .
rg -n "format\s*\(|%\s*\(|f['\"].*\{.*\}" --type py .

# Deserialization
rg -n "pickle\.loads?\(|yaml\.load\s*\(|marshal\.loads?\(|shelve\.open\(" --type py .
rg -n "BaseHTTPRequestHandler|SimpleXMLRPCServer" --type py .

# XSS & output encoding
rg -n "response\.write\(|response\.data\s*=|\.write\s*\(" --type py .
rg -n "render_template_string\s*\(" --type py .
rg -n "html\.unescape|bleach" --type py .

# SSRF & external requests
rg -n "requests\.(get|post|put|delete|patch|head)\(\s*.*req|requests\.(get|post|put|delete|patch|head)\(\s*user" --type py .
rg -n "urllib\.request\.urlopen\(|urllib\.request\.Request\(" --type py .
rg -n "httpx\.|aiohttp\." --type py .

# Path traversal
rg -n "open\s*\(\s*.*req|open\s*\(\s*user|open\s*\(\s*request" --type py .
rg -n "os\.path\.join\s*\(\s*.*req|shutil\.copy2\s*\(\s*.*req" --type py .
rg -n "zipfile\.ZipFile|tarfile\.open" --type py .

# Cryptography & randomness
rg -n "random\.(random|randint|choice|sample|randrange)\(" --type py .
rg -n "hashlib\.(md5|sha1)\(" --type py .
rg -n "DES\.|AES\.new.*MODE_ECB|DES_ECB" --type py .
rg -n "Crypto\.Cipher\.AES" --type py .

# Auth & session
rg -n "session\[" --type py .
rg -n "Flask\.secret_key\s*=\s*['\"]|SECRET_KEY\s*=\s*['\"]" --type py .
rg -n "@login_required|@requires_auth|auth\.login_required" --type py .

# Debug & secrets
rg -n "debug\s*=\s*True|DEBUG\s*=\s*True|app\.debug\s*=\s*True" --type py .
rg -n "print\s*\(\s*.*req|print\s*\(\s*.*request|print\s*\(\s*.*password|print\s*\(\s*.*secret" --type py .
rg -n "__version__|__author__" --type py .

# Exception handling
rg -n "except.*:.*return.*True|except.*:.*pass" --type py .
rg -n "except Exception|except:" --type py .
```

**Priority matrix:**

| Risk Category | Severity | Look For |
|---|---|---|
| SQL injection via string formatting | Critical | String concat/format in `execute()` or query builder |
| Command injection | Critical | `os.system()`, `subprocess` with `shell=True`, user input |
| Unsafe deserialization | Critical | `pickle.loads()`, `yaml.load()` with `Loader=Loader` |
| SSRF | High | Unvalidated URLs in `requests`, `urllib`, HTTP clients |
| Path traversal | High | User-controlled paths in `open()`, `shutil`, archive extraction |
| XSS (stored/reflected) | High | Unescaped user input in template or direct response write |
| Debug mode in production | High | `debug=True` in deploy config, verbose error pages |
| Secrets in code | High | `SECRET_KEY`, passwords, API keys hardcoded |
| Sensitive data in logs | High | `print()` or `logging` with passwords, tokens, full requests |
| Insecure randomness | Medium | `random` module for tokens, session IDs, or secrets |
| Weak cryptography | Medium | MD5, SHA-1, DES, ECB mode |
| Overly broad exception handling | Medium | `except:` swallowing auth errors or failing open |
| Missing CSRF protection | Medium | `@csrf.exempt` or no CSRF token on state-changing views |
| Insecure file uploads | Medium | No content-type validation, no size limits, zip-slip |

**Good vs Bad examples:**

SQL injection:
```python
# BAD
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
cursor.execute("SELECT * FROM users WHERE name = '%s'" % user_input)

# GOOD
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

Command injection:
```python
# BAD
os.system(f"convert {filename} output.png")
subprocess.run(f"rm {user_input}", shell=True)

# GOOD
subprocess.run(["convert", filename, "output.png"], shell=False)
```

Deserialization:
```python
# BAD
data = pickle.loads(user_input)
data = yaml.load(user_input)  # unsafe by default

# GOOD
data = json.loads(user_input)
validate_schema(data)  # then use validated data
```

SSRF:
```python
# BAD
response = requests.get(request.args['url'])

# GOOD
url = validate_url(request.args['url'])  # allowlist host, scheme
if url.hostname not in ALLOWED_HOSTS:
    abort(400)
response = requests.get(url)
```

Path traversal:
```python
# BAD
with open(os.path.join('/app/uploads', request.args['filename'])) as f:
    return f.read()

# GOOD
safe_path = os.path.realpath(os.path.join(UPLOADS_DIR, filename))
if not safe_path.startswith(os.path.realpath(UPLOADS_DIR)):
    abort(400)
with open(safe_path) as f:
    return f.read()
```

Exception handling (fail-closed):
```python
# BAD — Fail-open
def check_permission(user, resource):
    try:
        return auth.check(user, resource)
    except:
        return True  # Allow on error

# GOOD — Fail-closed
def check_permission(user, resource):
    try:
        return auth.check(user, resource)
    except Exception as e:
        logger.error(f"Auth check failed: {e}")
        return False  # Deny on error
```

Validation commands:
```bash
python -m pytest <focused-test>    # or: pytest <focused-test>
python -m pytest                   # full suite
pip-audit                          # dependency scan
bandit -r <path>                   # static security scan (if installed)
```

## Finding Template

Every candidate finding must include:

```markdown
### SCR-<id>: <title>
- **Severity:** Critical / High / Medium / Low / Info
- **Confidence:** High / Medium / Low
- **Status:** Proposed / Fix approved / Fixed / Deferred / Risk accepted / False positive
- **CWE:** <ID or Not mapped>
- **OWASP Top 10:2025:** <category>
- **Advisory:** <ID or Not applicable>

**Affected routes and evidence**
- `<method route>`: `<path:line>` - <evidence>

**Exploit scenario and preconditions**
<Attacker input → checks → sensitive operation → impact>

**Business impact**
<Confidentiality, integrity, availability, business impact>

**Root cause**
<Single underlying defect>

**Proposed remediation**
<Small concrete fix>

**Acceptance test**
<Test or exploit-path check that retraces the original exploit>
```

## Anti-Patterns

- Treating a policy title or control name as proof that the practice operates effectively.
- Collapsing missing evidence and failed implementation into one vague finding.
- Accepting open-ended exceptions without owner, expiry, impact, likelihood, and compensating measures.
- Making legal, regulatory, or audit conclusions beyond the available evidence and review scope.
- Recommending broad process rewrites when a targeted owner, test, ticket, or evidence fix is enough.
- Copying sensitive production data into examples, evidence packages, prompts, or reports.
- Using pattern matches as findings without tracing inputs, controls, and impact.
- Reporting a vulnerability solely because a vulnerable dependency version is present without proving reachability.
- Converting an assumption into a finding without stating it or reducing confidence.
- Using severity to hide uncertainty — a Low severity with Low confidence is not the same as High severity with High confidence.

## Decision Rules

- If source diff or route inventory is missing for a critical service, raise at least a high-severity readiness gap.
- If authorization check cannot be tied to an owner and approval, treat the outcome as unauditable until corrected.
- If HTTP client is present but expired or untested, require validation before accepting residual risk.
- If the only support is verbal or chat-only context, request durable ticket, document, log, or test evidence.
- If remediation would require a process or architecture decision, assign a decision owner instead of prescribing fixes.
- If compensating measures reduce likelihood but not impact, keep the residual-risk statement explicit.
- If the only evidence is a pattern match (e.g., ripgrep hit) without input-to-sink tracing, downgrade to a review lead, not a finding.

## Completion Criteria

A review is complete only when the report records:

- Scope and limitations
- Route coverage with status per group
- Advisory research status (performed / not performed)
- All human decisions (Fix / Defer / Risk accepted / False positive) per finding
- Remediation and validation evidence per fixed item
- Residual risk listing every unresolved item
- Final status: `Complete`, `Partially remediated`, or `Blocked`

If no vulnerabilities remain after evidence review, state that no security issues were found within the reviewed scope and list the coverage limitations.
