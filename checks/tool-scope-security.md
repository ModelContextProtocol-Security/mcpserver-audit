---
title: MCP Server Tool Scope & Privilege Escalation Security Check
version: 1.0
date: 2026-07-21
tags: [security, privilege-escalation, tool-scope, least-privilege, MCP, aivss]
aliases: [tool-scope, privilege-escalation, least-privilege, permission-scope]
status: active
priority: critical
cwe: [269, 272, 284, 732]
cwe-primary: 272
vulnerability-db: []
---

# MCP Server Tool Scope & Privilege Escalation Security Check

## Security Assessment Metadata

**AIVSS Scoring Capability**: Full (CVSS + AARS)
**Confidence Level**: High (tool permissions are structurally verifiable from source code and schemas)
**Complexity**: Moderate
**Data Requirements**: Usually Available (tool definitions and permission scopes are in source)
**Time to Complete**: 15 minutes (automated) + 30 minutes (manual)
**Reliability Notes**: Tool scope assessment is generally high-confidence because permissions are declared in code. The harder question is whether granted permissions are justified by the use case — that requires manual review of the server's stated purpose.
**Risk Level**: Critical when broad permissions are granted without access controls; High when permissions exceed stated use case.

## Security Purpose & Context

Every tool exposed by an MCP server represents a permission grant to the LLM agent that connects to it. Unlike traditional software where a human decides each action, an LLM agent may call tools autonomously and at scale. This changes the risk calculus dramatically: a tool that allows file deletion, code execution, or unauthenticated API calls becomes a direct attack vector if:

- The MCP server is compromised (supply chain attack)
- The LLM agent is deceived via prompt injection (see `prompt-injection-security` check)
- The LLM makes an incorrect autonomous decision

**Privilege escalation** in this context means: the combined effect of the tools available to an LLM agent exceeds what is necessary for the server's stated purpose, creating opportunities for misuse that the server operator did not intend to grant.

**Security Impact**: Critical — an MCP server with overly broad tool scope acts as a privilege amplifier, turning any compromise of the agent (prompt injection, jailbreak, buggy LLM decision) into a high-impact event.

**When to Use**: For every MCP server before production deployment, especially when connected to LLM agents operating autonomously.

**When to Skip**: Sandboxed development environments with no production credentials or data.

**Threat Context**: Supply chain attacks targeting MCP servers, prompt injection escalation, insider threat via malicious MCP server, buggy LLM agent over-calling destructive tools.

## Security Assessment Criteria

### High Confidence Vulnerabilities

- **Unrestricted write/delete operations**: Tools that can write or delete files, records, or data without any scope restriction, allowlist, or confirmation requirement.
- **Shell or code execution tools**: Tools exposing `subprocess`, `eval`, `exec`, or OS shell commands to the LLM with no argument sanitization.
- **Credentials passed as tool output**: Tools that return API keys, passwords, tokens, or private keys in their output where they could be exfiltrated via injection.
- **No authentication on tool calls**: An MCP server that connects to authenticated backends but does not itself enforce who can call each tool (any MCP client gets full access).
- **Missing scope separation**: A single server exposes both read-only tools and destructive write/execute tools, violating the principle of separating concerns.

### Medium Confidence Security Risks

- **Overly broad path or resource parameters**: Tools that accept arbitrary file paths, SQL queries, or resource identifiers without allowlisting or parameterized access.
- **Implicit admin-level credentials**: Server uses production admin credentials for all operations, even for tools that only require read-only access.
- **Tool descriptions encouraging autonomous use**: Tool descriptions that explicitly invite the LLM to call them without user confirmation ("automatically", "always call this when", "proactively").
- **Chaining risk**: Multiple individually-benign tools that, when chained by an LLM, produce a dangerous composite operation (read secret → send to external endpoint).

### Low Confidence / Expert Analysis Required

- **Threat model appropriateness**: Whether the scope is acceptable depends on the use case, user base, and deployment environment — a tool that is acceptable in an enterprise internal tool may be unacceptable in a public-facing agent.
- **LLM-specific calling patterns**: Some LLMs are more likely than others to autonomously call destructive tools without prompting.

## Automated Security Assessment (For AI Assistants)

### High Confidence Vulnerability Detection

```bash
# 1. Find shell execution exposure
grep -rn -E "(subprocess|os\.system|os\.popen|exec\(|eval\(|shell=True)" \
  --include="*.py" --include="*.js" --include="*.ts" --include="*.go" . \
  | grep -v "test\|spec\|#"

# 2. Find file write and delete operations exposed as tools
grep -rn -E "(open\(.*['\"]w|os\.remove|os\.unlink|shutil\.(rmtree|move)|unlink|writeFile|deleteFile|rm\b)" \
  --include="*.py" --include="*.js" --include="*.ts" --include="*.go" . \
  | grep -v "test\|spec\|#"

# 3. Find unrestricted path parameters in tool definitions
grep -rn -B2 -A2 -E "(path|file_path|filepath|directory|dir)\s*[=:]\s*" \
  --include="*.py" --include="*.js" --include="*.ts" . \
  | grep -i "param\|arg\|schema\|input" | grep -v "test\|spec"

# 4. Find credentials returned in tool output (exfiltration risk)
grep -rn -E "(api_key|secret|password|token|private_key|credential)" \
  --include="*.py" --include="*.js" --include="*.ts" . \
  | grep -i "return\|result\|output\|response"

# 5. Check for authentication on the MCP server's own transport
# (Not just the backend — is the MCP endpoint itself authenticated?)
grep -rn -i -E "(auth|bearer|api.key|token|oauth|jwt|Authorization)" \
  --include="*.py" --include="*.js" --include="*.ts" . \
  | grep -v "test\|spec\|comment" | head -20
```

### AIVSS Risk Factor Assessment

```bash
# AARS-1: Autonomy — does the server encourage unsupervised LLM operation?
grep -rn -i -E "(automat|proactive|always|continuous|background|schedule|cron)" \
  --include="*.py" --include="*.js" --include="*.ts" . | grep -v test

# AARS-2: Tool use breadth — count distinct tool categories (read vs write vs execute)
grep -rn -E "(@mcp\.tool|@tool|Tool\(|def tool_|function_declarations)" \
  --include="*.py" --include="*.js" --include="*.ts" . | wc -l

# AARS-3: External connectivity — can tools call external network endpoints?
grep -rn -E "(requests\.|httpx\.|fetch\(|axios\.|urllib\.request|http\.get)" \
  --include="*.py" --include="*.js" --include="*.ts" . | grep -v test

# AARS-4: Data sensitivity — does the server handle PII or secrets?
grep -rn -i -E "(ssn|email|password|credit.card|personal|private|confidential|pii)" \
  --include="*.py" --include="*.js" --include="*.ts" . | grep -v test

# Count and categorize tool risk levels
echo "=== Tool risk inventory ==="
echo "Destructive (write/delete/exec):"
grep -rn -l -E "(os\.remove|unlink|delete|write.*file|exec|subprocess)" \
  --include="*.py" --include="*.js" --include="*.ts" . | grep -v test
echo "Read-only:"
grep -rn -l -E "(os\.path|open.*r|read|list|get|fetch)" \
  --include="*.py" --include="*.js" --include="*.ts" . | grep -v test
```

**Automation Reliability**: Tool permission inventory is high-confidence from code analysis. Chaining-risk and autonomy assessment require manual judgment about real-world use cases.

## Manual Security Assessment (For Humans)

### Step-by-Step Security Checklist

- [ ] **Inventory all exposed tools**: List every tool the MCP server exposes. Categorize each as Read, Write, Delete, Execute, or External-Call.
- [ ] **Apply least-privilege test**: For each Write/Delete/Execute tool, ask "Is this the minimum permission needed for the server's stated purpose?" If not, flag for removal or restriction.
- [ ] **Check path/resource scope**: For tools accepting file paths, SQL queries, or resource IDs — are inputs restricted to an allowlist, a specific directory, or parameterized queries? Or does the tool accept arbitrary user-controlled values?
- [ ] **Evaluate credential exposure**: Do any tools return credentials, tokens, or keys in their output? If so, this is an exfiltration risk when combined with prompt injection.
- [ ] **Assess MCP transport authentication**: Is the MCP server itself authenticated? Can any MCP client connect and call any tool, or is there per-client authorization?
- [ ] **Review tool descriptions for autonomous use**: Tool descriptions that tell the LLM to "proactively" or "automatically" call a destructive tool without user confirmation are a design smell.
- [ ] **Test chaining attack**: Identify whether a sequence of individually-safe tools can be combined by an LLM to produce a dangerous result (e.g., "list secrets" → "write to external API" as two separate tools).

### Key Security Questions to Ask

- **High Confidence**: "Which tools can write, delete, or execute? Are they separated from read-only tools? Are they individually justified by the server's stated purpose?"
- **High Confidence**: "Do tools accept arbitrary file paths or SQL strings from the LLM without validation?"
- **Medium Confidence**: "Do the tool descriptions discourage unsupervised autonomous use of destructive operations?"
- **Expert Analysis**: "Does the composite permission surface of all tools together exceed what the use case requires?"

### Human Security Verification Points

- Attempt to call a destructive tool (e.g., file write) via MCP without any authentication — does it succeed?
- Try an arbitrary path traversal (e.g., `../../../../etc/passwd`) as a file path argument — does the tool block it?
- Check whether the server uses production admin credentials where read-only credentials would suffice.

## Secure vs. Vulnerable Patterns

### Excellent Security Examples

**Pattern**: Read/write tool separation with explicit scoping  
**Why Secure**: Separating read and write tools allows deployers to grant only the permissions their use case requires.  
**AIVSS Context**: Directly reduces AARS autonomy amplification by limiting blast radius.  
**Confidence**: High  
**Example**:
```python
# GOOD: Tools are scoped to specific, narrow operations
@mcp.tool()
def read_customer_record(customer_id: str) -> dict:
    """Read a single customer record by ID. Read-only."""
    validate_customer_id(customer_id)  # allowlist check
    return db.query("SELECT name, email FROM customers WHERE id = ?", [customer_id])

# Write tool is separate and explicit — deployer can choose not to include it
@mcp.tool()
def update_customer_email(customer_id: str, new_email: str) -> dict:
    """Update customer email. Requires confirmation before calling."""
    validate_customer_id(customer_id)
    validate_email(new_email)
    return db.execute("UPDATE customers SET email = ? WHERE id = ?", [new_email, customer_id])
```

**Pattern**: Directory allowlisting for file-access tools  
**Why Secure**: Restricts the tool to a known-safe directory, preventing path traversal and LLM-driven access to sensitive filesystem locations.  
**Confidence**: High  
**Example**:
```python
ALLOWED_DIRS = ["/app/user-data", "/app/exports"]

def read_file(path: str) -> str:
    resolved = os.path.realpath(path)
    if not any(resolved.startswith(d) for d in ALLOWED_DIRS):
        raise PermissionError(f"Path not in allowed directories: {resolved}")
    with open(resolved) as f:
        return f.read()
```

**Pattern**: Parameterized queries for database tools  
**Why Secure**: Prevents both SQL injection and scope creep — the LLM cannot craft arbitrary queries.  
**Confidence**: High  
**Example**:
```python
# GOOD: LLM provides structured parameters, not raw SQL
@mcp.tool()
def search_products(category: str, max_price: float) -> list:
    return db.query(
        "SELECT id, name, price FROM products WHERE category = ? AND price <= ?",
        [category, max_price]
    )
```

### Vulnerable Security Examples

**Pattern**: Arbitrary shell command execution  
**Why Vulnerable**: The LLM (or an attacker via injection) can execute any OS command with the server's privileges.  
**AIVSS Scoring**: CVSS Critical (9.0+), AARS amplification maximum.  
**Confidence**: High  
**Example**:
```python
# BAD: LLM can run any OS command
@mcp.tool()
def run_command(cmd: str) -> str:
    """Execute a shell command and return output."""
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout
```

**Pattern**: Unrestricted file path access  
**Why Vulnerable**: LLM can read `/etc/passwd`, SSH keys, `.env` files, or any sensitive file on the server.  
**Confidence**: High  
**Example**:
```python
# BAD: no path restriction
@mcp.tool()
def read_file(path: str) -> str:
    with open(path) as f:    # attacker/LLM provides path
        return f.read()
```

**Pattern**: Credentials in tool output  
**Why Vulnerable**: Enables exfiltration when combined with prompt injection — injected instructions can direct the LLM to relay the credentials to an external endpoint.  
**Confidence**: High  
**Example**:
```python
# BAD: returns the API key in tool output
@mcp.tool()
def get_config() -> dict:
    return {"api_key": os.environ["SECRET_API_KEY"], "endpoint": "..."}
```

**Pattern**: No authentication on MCP transport  
**Why Vulnerable**: Any process on the network can connect and invoke all tools, bypassing any user-level authorization.  
**Confidence**: High when the server listens on a network interface.

### Ambiguous Security Cases

**Pattern**: A tool that accepts a "query" string for database search  
**Context Factors**: Whether this is safe depends entirely on whether "query" is passed through parameterized SQL (safe) or interpolated into a raw SQL string (SQL injection).  
**Assessment Approach**: Trace the query string through the code. If it ever appears in string concatenation with SQL keywords, flag it.

**Pattern**: Admin tools exposed "for development purposes"  
**Context Factors**: Development-only tools that persist in production builds are a common privilege escalation vector.  
**Assessment Approach**: Check whether tool exposure is controlled by environment variables or build flags. If admin tools are always compiled in, flag as medium risk.

## Security Assessment Confidence Indicators

### High Confidence Security Results

**When to Trust**: When the code directly shows a tool accepting an LLM-provided value and passing it to `subprocess`, `exec`, `open()` without validation, or a SQL string without parameterization.

**Minimum Security Data**: Source code for tool handler functions and their MCP schema definitions.

**Strong Security Signals**:
- Shell=True in subprocess calls with any user-provided argument
- File open with user-provided path and no path validation
- SQL string built via f-string or concatenation with user data
- Credentials present in tool return values

### Low Confidence Security Warnings

**Security Assessment Red Flags**:
- Cannot access the actual tool implementation (only the schema) — can't verify sanitization.
- Server uses external libraries as tool backends — need to check the library's own permission model.

**External Security Factors**: MCP transport layer security (authenticated vs. unauthenticated) is set by the deployment environment, not the server code — check deployment configuration separately.

### Improving Security Assessment Confidence

**Additional Security Data**: Deployment configuration (which credentials are provided, which network the server is on, which LLM model is used and with what system prompt).

**Cross-Validation**: Combine with `prompt-injection-security` check — injection risk is multiplicative with broad tool scope. An injection that succeeds against a read-only server is low impact; the same injection against a shell-execution server is catastrophic.

## Security Justification & Evidence

### Why This Security Assessment Matters

**Impact on Security Posture**: Overly broad tool scope is the force multiplier for every other MCP server vulnerability. Prompt injection, supply chain attacks, and LLM misbehavior are all orders of magnitude more dangerous when the server has admin-level permissions.

**Threat Actor Interest**: MCP servers are high-value targets because they often hold long-lived credentials and can act autonomously. Compromising a single MCP server may grant access to every downstream system it connects to.

**AIVSS Relevance**: Traditional CVSS under-scores this risk because it models a human attacker making deliberate choices. The AI agent threat model introduces a new class of autonomous actors who may inadvertently exercise permissions they should not have.

### Supporting Security Evidence

**Research Basis**: OWASP Top 10 for LLM Applications (LLM07: Insecure Plugin Design), OWASP Top 10 for LLM Applications (LLM08: Excessive Agency). CWE-272 (Least Privilege Violation), CWE-269 (Improper Privilege Management).

**Security Standards**: NIST SP 800-53 AC-6 (Least Privilege), CWE-284 (Improper Access Control).

**Real-World Exploits**: Malicious MCP servers that over-claim permissions have been publicly documented in the MCP ecosystem. The "ToolPoisoning" attack class (malicious tool descriptions that instruct the LLM to exfiltrate data during other operations) relies on tool scope being broad enough to include exfiltration-capable tools.

## Basic Remediation Guidance

### Critical Issues Found

- **Shell execution exposed**: Remove `shell=True` and arbitrary command execution from MCP-facing tools. If system commands must be called, use a fixed allowlist of permitted commands and parameterized arguments.
- **Unrestricted file path**: Add `os.path.realpath` + directory allowlist validation before any file open operation with an LLM-provided path.
- **Credentials in tool output**: Move secrets to server-side-only usage. Never return raw credentials in tool output; return only derivative values (signed tokens, operation results).
- **Unauthenticated MCP transport**: Add bearer token, OAuth, or mTLS authentication to the MCP transport layer so only authorized clients can call tools.
- **No SQL parameterization**: Replace all string-interpolated SQL with parameterized queries or ORM methods.

### Next Steps for Fixing

**For detailed remediation guidance**: Use mcpserver-builder for step-by-step secure coding fixes  
**For secure deployment**: Coordinate with mcpserver-operator for runtime credential management and network isolation  
**Immediate priority**: Address shell execution and credential exfiltration risks first — these have the highest impact if exploited

## Integration with MCP Security Ecosystem

- **Cross-reference with `vulnerability-db`** for known privilege escalation CVEs in MCP servers
- **Document findings in `audit-db`** for community benefit
- **Combine with `prompt-injection-security` check**: scope and injection risk are multiplicative — assess both together for a realistic AIVSS score
- **Update `server-db`** with tool scope assessment results

## AIVSS & CVSS Reference Materials

- **AIVSS (AI Vulnerability Scoring System)**: https://github.com/CloudSecurityAlliance-DataSets/dataset-public-laws-regulations-standards/tree/main/tools-resources/owasp.org/AIVSS
- **CVSS v4.0 (Common Vulnerability Scoring System)**: https://github.com/CloudSecurityAlliance-DataSets/dataset-public-laws-regulations-standards/tree/main/tools-resources/first.org/CVSS
- **OWASP LLM07 (Insecure Plugin Design)**: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- **OWASP LLM08 (Excessive Agency)**: https://owasp.org/www-project-top-10-for-large-language-model-applications/

---

*This check is part of the MCP Server Audit security assessment framework. Last updated: 2026-07-21*
