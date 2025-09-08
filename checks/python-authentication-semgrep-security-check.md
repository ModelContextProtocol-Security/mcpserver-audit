---
title: Python Authentication Security Check with Semgrep
version: 1.0
date: 2025-08-20
tags: [security, authentication, python, semgrep, static-analysis, MCP, aivss]
aliases: [python-auth-security, semgrep-authentication-check]
status: active
priority: high
cwe: [798, 327, 256, 862, 319, 532, 614, 338, 89]
cwe-primary: 862
vulnerability-db: []
---

# Python Authentication Security Check with Semgrep

## Security Assessment Metadata

**AIVSS Scoring Capability**: Full
**Confidence Level**: High for pattern detection, Medium for context interpretation
**Complexity**: Simple for automated detection, Moderate for risk assessment
**Data Requirements**: Always Available (source code)
**Time to Complete**: 5 minutes (automated) + 10 minutes (manual review)
**Reliability Notes**: Semgrep provides high confidence for syntactic patterns, manual review needed for business logic
**Risk Level**: High (authentication vulnerabilities can lead to complete system compromise)

## Security Purpose & Context

Authentication vulnerabilities in MCP servers present unique risks due to their privileged access to external services and AI agent interactions. This check uses Semgrep to systematically detect Python authentication anti-patterns that could compromise MCP server security.

**Security Impact**: High - Authentication flaws enable unauthorized access to MCP tools and connected services
**When to Use**: All Python-based MCP servers, especially those with custom authentication logic
**When to Skip**: Non-Python servers or servers with no authentication mechanisms
**Threat Context**: Addresses credential theft, privilege escalation, and unauthorized tool access

## Security Assessment Criteria

### High Confidence Vulnerabilities
- **Hardcoded Credentials**: Syntactically detectable password literals
- **Weak Hashing Algorithms**: Clear usage of MD5/SHA1 for passwords
- **Basic Auth over HTTP**: Observable URL patterns with credentials
- **SQL Injection in Auth**: String concatenation in authentication queries

### Medium Confidence Security Risks
- **Missing Authentication Decorators**: Flask routes without @login_required patterns
- **Credential Logging**: Variables matching password patterns in logging calls
- **Weak Random Generation**: Using `random` module for security tokens
- **Insecure Session Configuration**: Specific Flask security flag settings

### Low Confidence / Expert Analysis Required
- **Business Logic Authentication Flaws**: Complex authentication flows requiring domain expertise
- **Context-Dependent Auth Bypasses**: Authorization checks that depend on application logic
- **MCP-Specific Tool Authorization**: Permission checks for individual MCP tools

## Automated Security Assessment (For AI Assistants)

### Semgrep Rules Configuration

Save the following as `python-auth-security.yml`:

```yaml
rules:
  # Rule 1: Detect hardcoded passwords/secrets
  - id: hardcoded-password
    patterns:
      - pattern-either:
          - pattern: password = "..."
          - pattern: PASSWORD = "..."
          - pattern: secret = "..."
          - pattern: SECRET = "..."
          - pattern: api_key = "..."
          - pattern: API_KEY = "..."
          - pattern: token = "..."
          - pattern: TOKEN = "..."
    message: "Hardcoded credential detected. Use environment variables or secure configuration instead."
    languages: [python]
    severity: ERROR
    metadata:
      category: security
      cwe: "CWE-798: Use of Hard-coded Credentials"

  # Rule 2: Detect weak password hashing
  - id: weak-password-hashing
    patterns:
      - pattern-either:
          - pattern: hashlib.md5(...)
          - pattern: hashlib.sha1(...)
          - pattern: hashlib.md5($PASSWORD)
          - pattern: hashlib.sha1($PASSWORD)
    message: "Weak hashing algorithm detected for password. Use bcrypt, scrypt, or Argon2 instead."
    languages: [python]
    severity: ERROR
    metadata:
      category: security
      cwe: "CWE-327: Broken or Risky Cryptographic Algorithm"

  # Rule 3: Detect plaintext password storage
  - id: plaintext-password-storage
    patterns:
      - pattern-either:
          - pattern: |
              def $FUNC(..., password, ...):
                ...
                $DB.save($OBJ.password)
          - pattern: |
              user.password = $PASSWORD
              $DB.save(user)
    message: "Password appears to be stored in plaintext. Hash passwords before storage."
    languages: [python]
    severity: ERROR
    metadata:
      category: security
      cwe: "CWE-256: Unprotected Storage of Credentials"

  # Rule 4: Detect missing authentication checks
  - id: missing-auth-check
    patterns:
      - pattern: |
          @app.route($ROUTE)
          def $FUNC(...):
            ...
      - pattern-not: |
          @app.route($ROUTE)
          @$AUTH_DECORATOR
          def $FUNC(...):
            ...
      - pattern-not: |
          @app.route($ROUTE)
          def $FUNC(...):
            ...
            if not $AUTH_CHECK:
              ...
    message: "Route handler missing authentication check. Add @login_required or similar."
    languages: [python]
    severity: WARNING
    metadata:
      category: security
      cwe: "CWE-862: Missing Authorization"

  # Rule 5: Detect basic auth over HTTP
  - id: basic-auth-over-http
    patterns:
      - pattern-either:
          - pattern: |
              requests.get($URL, auth=($USER, $PASS))
          - pattern: |
              requests.post($URL, auth=($USER, $PASS))
      - metavariable-regex:
          metavariable: $URL
          regex: "^\"http://.*"
    message: "Basic authentication over HTTP detected. Use HTTPS to protect credentials in transit."
    languages: [python]
    severity: ERROR
    metadata:
      category: security
      cwe: "CWE-319: Cleartext Transmission of Sensitive Information"

  # Rule 6: Detect credential leakage in logs
  - id: credential-logging
    patterns:
      - pattern-either:
          - pattern: |
              logging.$LEVEL(..., $PASSWORD, ...)
          - pattern: |
              print(..., $PASSWORD, ...)
          - pattern: |
              logger.$LEVEL(..., $PASSWORD, ...)
      - metavariable-regex:
          metavariable: $PASSWORD
          regex: ".*(password|secret|token|key).*"
    message: "Potential credential leakage in logging. Avoid logging sensitive information."
    languages: [python]
    severity: WARNING
    metadata:
      category: security
      cwe: "CWE-532: Information Exposure Through Log Files"

  # Rule 7: Detect insecure session configuration
  - id: insecure-session-config
    patterns:
      - pattern-either:
          - pattern: app.config['SESSION_COOKIE_SECURE'] = False
          - pattern: app.config['SESSION_COOKIE_HTTPONLY'] = False
          - pattern: app.config['SESSION_COOKIE_SAMESITE'] = None
    message: "Insecure session cookie configuration. Enable Secure, HttpOnly, and SameSite flags."
    languages: [python]
    severity: ERROR
    metadata:
      category: security
      cwe: "CWE-614: Sensitive Cookie Without 'Secure' Flag"

  # Rule 8: Detect weak random number generation for security
  - id: weak-random-for-security
    patterns:
      - pattern-either:
          - pattern: random.random()
          - pattern: random.randint(...)
          - pattern: random.choice(...)
      - pattern-inside: |
          def $FUNC(...):
            ...
            $TOKEN = ...
            ...
      - metavariable-regex:
          metavariable: $FUNC
          regex: ".*(token|password|secret|session|auth).*"
    message: "Weak random number generator used for security purposes. Use secrets module instead."
    languages: [python]
    severity: ERROR
    metadata:
      category: security
      cwe: "CWE-338: Use of Cryptographically Weak Pseudo-Random Number Generator"

  # Rule 9: Detect SQL injection in authentication
  - id: sql-injection-auth
    patterns:
      - pattern: |
          $CURSOR.execute("SELECT * FROM users WHERE username = '" + $USER + "' AND password = '" + $PASS + "'")
      - pattern: |
          $CURSOR.execute(f"SELECT * FROM users WHERE username = '{$USER}' AND password = '{$PASS}'")
    message: "SQL injection vulnerability in authentication query. Use parameterized queries."
    languages: [python]
    severity: ERROR
    metadata:
      category: security
      cwe: "CWE-89: SQL Injection"

  # Rule 10: Detect MCP server specific auth issues
  - id: mcp-server-missing-auth-validation
    patterns:
      - pattern: |
          @mcp.tool()
          def $FUNC_NAME(...):
            ...
      - pattern-not: |
          @mcp.tool()
          def $FUNC_NAME(...):
            ...
            if not $AUTH_CHECK:
              ...
      - pattern-not: |
          @mcp.tool()
          def $FUNC_NAME(...):
            ...
            $AUTH_FUNC(...)
            ...
    message: "MCP tool missing authentication validation. Verify user permissions before executing sensitive operations."
    languages: [python]
    severity: WARNING
    metadata:
      category: security
      cwe: "CWE-862: Missing Authorization"
      note: "Specific to Model Context Protocol servers"
```

### Running Automated Analysis

```bash
# Install Semgrep
pip install semgrep

# Run authentication security check
semgrep --config=python-auth-security.yml --json --output=auth-results.json /path/to/mcp/server

# Quick summary of critical issues
semgrep --config=python-auth-security.yml --severity=ERROR /path/to/mcp/server

# Check specific patterns
semgrep --config=python-auth-security.yml --include="*.py" --exclude="test_*" /path/to/mcp/server
```

**Automation Reliability**: High confidence for syntactic patterns, requires manual review for business logic vulnerabilities

## Manual Security Assessment (For Humans)

### Step-by-Step Security Checklist

- [ ] **CVSS Base Score Assessment**: Review Semgrep ERROR-level findings for immediate threats
- [ ] **AARS Factor Evaluation**: Assess MCP tool authorization and agent interaction risks
- [ ] **Threat Model Integration**: Consider deployment context and attack vectors
- [ ] **Context-Specific Risk Assessment**: Evaluate authentication flow completeness

### Key Security Questions to Ask

- **High Confidence (CVSS)**: "Are there any hardcoded credentials or weak cryptographic algorithms?"
- **Medium Confidence (AARS)**: "Do MCP tools properly validate caller permissions before execution?"
- **Expert Analysis**: "Does the authentication flow handle edge cases and prevent bypasses?"

### Human Security Verification Points

- Manually review authentication business logic that Semgrep cannot parse
- Verify that MCP tool authorization aligns with intended access controls
- Check authentication error handling and information disclosure patterns
- Validate that session management follows security best practices

## Secure vs. Vulnerable Patterns

### Excellent Security Examples

**Pattern**: Proper environment variable usage with validation
```python
import os
import secrets

def load_auth_config():
    api_key = os.environ.get('MCP_API_KEY')
    if not api_key:
        raise ValueError("MCP_API_KEY environment variable required")
    return api_key

# Secure token generation
def generate_session_token():
    return secrets.token_urlsafe(32)
```
**Why Secure**: Uses environment variables, proper validation, cryptographically secure randomness
**AIVSS Context**: Addresses both traditional credential management (CVSS) and agent access control (AARS)
**Confidence**: High in identifying this secure pattern

### Vulnerable Security Examples

**Pattern**: Hardcoded credentials with weak hashing
```python
# VULNERABLE - Don't do this
API_KEY = "sk-1234567890abcdef"  # Hardcoded secret
user_password_hash = hashlib.md5(password.encode()).hexdigest()  # Weak hashing

@app.route('/admin')
def admin_panel():  # Missing authentication
    return sensitive_data()
```
**Why Vulnerable**: Credentials in source code, weak MD5 hashing, missing authentication
**AIVSS Scoring**: High CVSS score for credential exposure, high AARS score for missing authorization
**Confidence**: High in identifying these vulnerability patterns

### Ambiguous Security Cases

**Pattern**: Authentication decorators with complex logic
```python
@app.route('/api/mcp-tool')
@requires_permission('tool_access')  # Custom decorator
def execute_tool():
    # Complex authorization logic here
    pass
```
**Context Factors**: Security depends on the implementation of `requires_permission`
**Threat Model Impact**: Risk varies based on tool sensitivity and user privileges
**Assessment Approach**: Manual code review required for custom authentication logic

## Security Assessment Confidence Indicators

### High Confidence Security Results

**When to Trust**: Semgrep ERROR-level findings for syntactic patterns (hardcoded secrets, weak crypto)
**Minimum Security Data**: Access to complete Python source code
**Strong Security Signals**: Clear patterns matching established vulnerability signatures

### Low Confidence Security Warnings

**Security Assessment Red Flags**: Custom authentication logic, complex authorization flows
**Data Quality Issues**: Incomplete source code, dynamically loaded authentication modules
**External Security Factors**: Third-party authentication providers, runtime configuration

### Improving Security Assessment Confidence

**Additional Security Data**: Configuration files, environment variable documentation, deployment scripts
**Cross-Validation**: Combine with credential management checks, runtime testing
**Threat Evolution Factors**: Monitor for new Python authentication vulnerability patterns

## Security Justification & Evidence

### Why This Security Assessment Matters

**Impact on Security Posture**: Authentication vulnerabilities enable complete MCP server compromise
**Risk Implications**: Unauthorized access to AI agents, connected services, and sensitive operations
**Threat Actor Interest**: Authentication flaws are primary targets for initial access
**AIVSS Relevance**: Addresses both credential security (CVSS) and agent authorization (AARS)

### Supporting Security Evidence

**Research Basis**: OWASP Top 10, CWE/SANS Top 25, Python security best practices
**Security Standards**: NIST Authentication Guidelines, OAuth 2.1 Security Best Practices
**CVE References**: CVE-2021-44228 (Log4j), CVE-2020-1472 (Zerologon) for authentication bypass patterns
**Real-World Exploits**: GitHub secret scanning reports, credential stuffing attack analyses

## Security Integration Guidance

### Using High Confidence Security Results

- Immediately remediate ERROR-level Semgrep findings (hardcoded secrets, weak crypto)
- Weight CVSS scores heavily for traditional authentication vulnerabilities
- Prioritize findings in MCP tool authorization logic for AARS assessment

### Handling Low Confidence Security Results

- Manual code review required for custom authentication implementations
- Security expert consultation for complex authorization flows
- Penetration testing recommended for business logic authentication

### Combining with Other Security Checks

- **Credential Management Check**: Validates secure secret handling practices
- **Network Security Check**: Ensures HTTPS usage for authentication endpoints
- **Input Validation Check**: Prevents authentication bypass through injection attacks

## Common Security Assessment Pitfalls

### Security Assessment Errors

- Over-relying on static analysis for business logic vulnerabilities
- Missing context-dependent authentication requirements
- False positives from test code or example configurations

### Security Context Mismatches

- Applying web application auth patterns to MCP STDIO transport
- Missing MCP-specific authorization requirements for tool access
- Inconsistent authentication between MCP transport types

### Security Confidence Misjudgments

- Under-valuing clear Semgrep findings due to false positive concerns
- Over-trusting custom authentication implementations without validation
- Inadequate weighting of AARS factors for MCP-specific risks

## Basic Remediation Guidance

### Critical Issues Found

- **Hardcoded Credentials**: Immediate security risk - move to environment variables
- **Weak Cryptographic Algorithms**: Replace MD5/SHA1 with bcrypt/Argon2 for passwords
- **Missing Authentication**: Add proper authorization checks to all sensitive endpoints
- **Insecure Transport**: Ensure all credential transmission uses HTTPS/TLS

### Next Steps for Fixing

**For detailed remediation guidance**: Use mcpserver-builder for step-by-step secure coding fixes
**For secure deployment**: Coordinate with mcpserver-operator for runtime security controls
**Immediate priority**: Focus on ERROR-level Semgrep findings with highest AIVSS scores

## Integration with MCP Security Ecosystem

- **Cross-reference with `vulnerability-db`** for known Python authentication CVEs
- **Document findings in `audit-db`** for community benefit and pattern recognition
- **Update `server-db`** with authentication security assessment results
- **Flag for enhanced monitoring** if authentication practices show concerning patterns

## Teaching Points

**For Security Novices:**
- Static analysis tools like Semgrep catch common authentication mistakes reliably
- Focus on ERROR-level findings first - these represent clear security vulnerabilities
- Environment variables are the standard approach for credential management in MCP servers
- Authentication and authorization are different - both are critical for MCP server security

**For Security Professionals:**
- MCP servers require authorization at both transport and tool levels
- Combine static analysis with runtime testing for comprehensive authentication assessment
- Consider AI risks (AARS factors) alongside traditional web security (CVSS factors)
- Python's `secrets` module should be used for all cryptographic operations

---

*This check is part of the MCP Server Audit security assessment framework. Last updated: 2025-08-20*