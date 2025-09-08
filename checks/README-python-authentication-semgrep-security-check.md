# README: Python Authentication Semgrep Security Check

## Overview

This security check uses [Semgrep](https://semgrep.dev/) to automatically detect common authentication vulnerabilities in Python-based MCP servers. It combines static analysis pattern matching with MCP-specific security knowledge to identify both traditional web security issues and AI risks.

## What This Check Detects

### 🔴 Critical Issues (ERROR Level)
- **Hardcoded credentials** - API keys, passwords, tokens in source code
- **Weak cryptographic algorithms** - MD5, SHA1 used for password hashing
- **Insecure transport** - Basic authentication over HTTP
- **SQL injection** - String concatenation in authentication queries
- **Insecure session configuration** - Missing security flags on cookies
- **Weak randomness** - Using `random` module for security tokens

### 🟡 Important Issues (WARNING Level)
- **Missing authentication decorators** - Flask routes without `@login_required`
- **Credential logging** - Passwords/secrets accidentally logged
- **MCP tool authorization gaps** - `@mcp.tool()` functions without permission checks

## Prerequisites

```bash
# Install Semgrep
pip install semgrep

# Or using pipx (recommended for isolated installation)
pipx install semgrep

# Verify installation
semgrep --version
```

## Quick Start

### 1. Extract the Semgrep Rules

The security check file contains embedded Semgrep rules. Extract them:

```bash
# Navigate to the checks directory
cd /path/to/mcpserver-audit/checks

# Extract rules from the security check file
grep -A 200 "Save the following as" python-authentication-semgrep-security-check.md | \
grep -A 180 "python-auth-security.yml" | \
tail -n +3 > python-auth-security.yml
```

Or manually copy the YAML rules from the security check file into `python-auth-security.yml`.

### 2. Run Basic Scan

```bash
# Scan a single MCP server
semgrep --config=python-auth-security.yml /path/to/mcp-server/

# Scan multiple servers
semgrep --config=python-auth-security.yml /path/to/mcp-servers/*/

# Focus on critical issues only
semgrep --config=python-auth-security.yml --severity=ERROR /path/to/mcp-server/
```

### 3. Example Usage on SearXNG MCP Servers

```bash
# Scan the downloaded SearXNG MCP servers
cd /Volumes/MacMiniData/Users/kurt/GitHub/example/

# Quick critical issues check
semgrep --config=python-auth-security.yml --severity=ERROR ./searxng-simple-mcp/
semgrep --config=python-auth-security.yml --severity=ERROR ./searxng-mcp-server/

# Full security scan with output to file
semgrep --config=python-auth-security.yml --json --output=auth-scan-results.json ./
```

## Understanding the Output

### Sample Output Explanation

```
Findings:

  python-auth-security.yml
    hardcoded-password
        example/server.py:15
         API_KEY = "sk-1234567890abcdef"
         
    weak-password-hashing  
        example/auth.py:23
         password_hash = hashlib.md5(password.encode()).hexdigest()
         
    missing-auth-check
        example/routes.py:45
         @app.route('/admin')
         def admin_panel():
```

**What this means:**
- **hardcoded-password**: Line 15 has an API key in source code - **CRITICAL**
- **weak-password-hashing**: Line 23 uses MD5 for passwords - **CRITICAL** 
- **missing-auth-check**: Line 45 has an admin route with no authentication - **HIGH**

### Risk Prioritization

**Fix immediately:**
1. ERROR-level findings (hardcoded secrets, weak crypto)
2. Authentication bypasses in sensitive endpoints
3. SQL injection in auth queries

**Fix soon:**
4. WARNING-level findings (missing decorators, logging issues)
5. MCP tool authorization gaps
6. Session security improvements

## Common Usage Patterns

### Development Workflow Integration

```bash
# Pre-commit hook
semgrep --config=python-auth-security.yml --error --quiet src/

# CI/CD pipeline
semgrep --config=python-auth-security.yml --sarif --output=auth-results.sarif .

# IDE integration (VS Code with Semgrep extension)
# Automatically runs on file save
```

### Filtering Results

```bash
# Exclude test files
semgrep --config=python-auth-security.yml --exclude="test_*.py" --exclude="*_test.py" ./

# Focus on specific rule types
semgrep --config=python-auth-security.yml --include="hardcoded-password" --include="weak-password-hashing" ./

# Scan only MCP server files
semgrep --config=python-auth-security.yml --include="*server*.py" --include="*mcp*.py" ./
```

### Output Formats

```bash
# Human-readable (default)
semgrep --config=python-auth-security.yml ./

# JSON for tool integration
semgrep --config=python-auth-security.yml --json --output=results.json ./

# SARIF for GitHub Security tab
semgrep --config=python-auth-security.yml --sarif --output=results.sarif ./

# JUnit XML for CI dashboards
semgrep --config=python-auth-security.yml --junit-xml --output=results.xml ./
```

## Interpreting Results for MCP Servers

### MCP-Specific Patterns

**MCP Tool Authorization (`mcp-server-missing-auth-validation`):**
```python
# ❌ VULNERABLE: No permission check
@mcp.tool()
def delete_all_files():
    shutil.rmtree('/')  # Dangerous operation without auth

# ✅ SECURE: Proper authorization
@mcp.tool()  
def delete_all_files():
    if not verify_admin_permission():
        raise PermissionError("Admin required")
    shutil.rmtree('/')
```

**Environment Variable Usage:**
```python
# ❌ FLAGGED: Hardcoded credential
SEARXNG_URL = "https://searx.example.com"
SEARXNG_PASSWORD = "secret123"

# ✅ SECURE: Environment variables  
SEARXNG_URL = os.environ.get('SEARXNG_URL', 'https://searx.space/random')
SEARXNG_PASSWORD = os.environ.get('SEARXNG_PASSWORD')
```

## Troubleshooting

### Common Issues

**"No rules found"**
```bash
# Check if rules file exists and has correct YAML format
cat python-auth-security.yml | head -10
semgrep --validate --config=python-auth-security.yml
```

**"Too many false positives"**
```bash
# Add exclusions to rules or command line
semgrep --config=python-auth-security.yml --exclude="examples/" --exclude="docs/" ./
```

**"Missing Python files"**
```bash
# Verify file discovery
semgrep --config=python-auth-security.yml --verbose ./
```

### Performance Optimization

```bash
# For large codebases
semgrep --config=python-auth-security.yml --max-target-bytes=1MB --jobs=4 ./

# Skip large files
semgrep --config=python-auth-security.yml --max-lines-per-finding=100 ./
```

## Integration with MCP Security Framework

### Workflow Integration

1. **Run this check first** to catch obvious authentication issues
2. **Combine with credential-management-security.md** for comprehensive credential assessment  
3. **Follow up with manual review** using the security assessment steps in the main check file
4. **Document findings** in audit-db for community benefit

### AIVSS Scoring Context

**High CVSS Scores:** Traditional vulnerabilities (hardcoded secrets, SQL injection)
**High AARS Scores:** MCP-specific risks (unauthorized tool access, agent privilege escalation)  
**Combined AIVSS:** Authentication flaws in MCP servers have both high technical risk and high AI risk

### Remediation Workflow

```bash
# 1. Run scan and save results
semgrep --config=python-auth-security.yml --json --output=findings.json ./

# 2. Prioritize by severity  
cat findings.json | jq '.results[] | select(.extra.severity=="ERROR")'

# 3. Fix critical issues
# 4. Re-run scan to verify fixes
semgrep --config=python-auth-security.yml ./

# 5. Document remaining risks in audit report
```

## Advanced Usage

### Custom Rule Development

```yaml
# Add your own MCP-specific rules
rules:
  - id: custom-mcp-permission-check
    patterns:
      - pattern: |
          @mcp.tool()
          def $FUNC(...):
            ...
            os.system($CMD)
            ...
      - pattern-not: |
          @mcp.tool()  
          def $FUNC(...):
            ...
            if not check_permission('system_access'):
              ...
            os.system($CMD)
            ...
    message: "MCP tool executes system commands without permission check"
    languages: [python]
    severity: ERROR
```

### Continuous Monitoring

```bash
# Schedule daily scans
echo "0 2 * * * semgrep --config=python-auth-security.yml /path/to/mcp-servers/ --json >> /var/log/mcp-security.json" | crontab -

# Set up alerts for new findings
semgrep --config=python-auth-security.yml --baseline=previous-scan.json ./
```

## Security Context

This check addresses authentication vulnerabilities that are particularly dangerous in MCP servers because:

- **Credential Aggregation**: MCP servers often manage multiple service credentials
- **Agent Access**: Compromised authentication affects AI agent capabilities  
- **Privilege Escalation**: Authentication bypasses can lead to tool misuse
- **Supply Chain Risk**: Authentication flaws expose downstream services

## Resources

- [Semgrep Documentation](https://semgrep.dev/docs/)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [MCP Security Best Practices](https://github.com/modelcontextprotocol/specification)
- [Python Security Best Practices](https://python.org/dev/security/)

## Support

For issues with this security check:
1. Check the main security check file for detailed assessment guidance
2. Review Semgrep documentation for rule syntax issues
3. Consult the MCP security framework documentation for context-specific guidance

---

*Last updated: 2025-08-20 | Part of the MCP Server Audit security assessment framework*