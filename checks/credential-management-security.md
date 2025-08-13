---
title: MCP Server Credential Management Security Check
version: 1.0
date: 2025-08-07
tags: [security, credentials, secrets, environment-variables, MCP]
aliases: [secrets-management, credential-security]
status: active
priority: critical
cwe: [798, 200, 522]
cwe-primary: 798
vulnerability-db: []
---

# MCP Server Credential Management Security Check

## Purpose

Evaluate how an MCP server handles credentials, API keys, and sensitive configuration data. This check distinguishes between secure practices (environment variables, vault systems) and dangerous anti-patterns (hardcoded secrets, committed credentials).

## Why This Matters

MCP servers frequently act as intermediaries between LLMs and external services, requiring them to handle multiple sets of sensitive credentials simultaneously. Poor credential management in MCP servers can lead to:

- **Broad credential exposure** - Single compromise exposes multiple downstream systems
- **Git repository leaks** - Hardcoded secrets permanently stored in version control
- **Container escape attacks** - Though environment variables are still the recommended approach
- **Supply chain attacks** - Compromised dependencies accessing credential stores

## For AI Assistants: Automated Analysis

### Critical Red Flags (Immediate Security Concerns)

```bash
# Search for hardcoded API keys, tokens, and passwords
grep -r -E "(api[_-]?key|secret|password|token)\s*[:=]\s*['\"][^'\"]{10,}" --include="*.py" --include="*.js" --include="*.ts" --include="*.json" --include="*.yaml" --include="*.yml" .

# Look for common secret patterns
grep -r -E "(sk-[a-zA-Z0-9]{32,}|xoxb-[0-9]+-[a-zA-Z0-9]+|ghp_[a-zA-Z0-9]{36})" --include="*.py" --include="*.js" --include="*.ts" .

# Check for credentials in configuration files
find . -name "*.env*" -o -name "config.json" -o -name "settings.yaml" | xargs grep -l -E "(key|secret|password|token)"
```

### Positive Security Patterns

```bash
# Look for proper environment variable usage
grep -r "os\.environ\|process\.env\|getenv" --include="*.py" --include="*.js" --include="*.ts" .

# Check for credential validation and error handling
grep -r -A 3 -B 1 "environ.*get\|getenv" --include="*.py" .
```

### Configuration Analysis

**Check for MCP-specific credential patterns:**
```bash
# Look for Claude Desktop configuration patterns
find . -name "*claude*config*" -o -name "mcp_config*" | xargs cat 2>/dev/null

# Check for environment variable documentation
grep -r -E "(export|ENV|environment)" README.md docs/ 2>/dev/null
```

## For Humans: Manual Assessment Steps

### Step 1: Repository-Wide Secret Scan

1. **Clone the repository** and scan the entire git history:
   ```bash
   git log --all --full-history -p | grep -E "(api[_-]?key|secret|password|token)"
   ```

2. **Check current codebase** for obvious credential leaks:
   - Look in source files for string literals containing keys/tokens
   - Examine configuration files for embedded credentials
   - Review environment file examples for real secrets

### Step 2: Credential Configuration Review

**Good Patterns to Verify:**
- [ ] Credentials loaded from environment variables
- [ ] Environment variables documented in README
- [ ] Example configurations use placeholder values
- [ ] No `.env` files committed to git
- [ ] Proper error handling for missing credentials

**Questions to Ask:**
- "How do I provide credentials to this server?"
- "What happens if required credentials are missing?"
- "Are there separate credential requirements for dev/staging/production?"

### Step 3: MCP-Specific Security Assessment

**For MCP HTTP Transport:**
- [ ] Follows OAuth 2.1 specification guidance
- [ ] Implements PKCE for authorization flows
- [ ] Uses short-lived access tokens
- [ ] Validates token audiences properly
- [ ] Requires HTTPS for authorization endpoints

**For MCP STDIO Transport:**
- [ ] Uses environment-based credentials (as recommended by spec)
- [ ] Documents required environment variables
- [ ] Handles missing credentials gracefully

## Security Context and Best Practices

### Environment Variables: The Right Approach

**✅ SECURE - Environment Variable Usage:**
```python
import os

# Correct approach
api_key = os.environ.get('NOTION_API_KEY')
if not api_key:
    raise ValueError("NOTION_API_KEY environment variable required")
```

**❌ INSECURE - Hardcoded Secrets:**
```python
# Never do this
api_key = "sk-abc123def456789"
config = {
    "token": "xoxb-1234567890-abcdefghijklmnop"
}
```

### Common Misconceptions to Correct

| ❌ **Myth** | ✅ **Reality** |
|-------------|----------------|
| "Environment variables are insecure" | Environment variables are the industry standard for 12-factor apps and containerized deployments |
| "Vault systems are always better" | Vault systems add complexity and are overkill for many MCP server deployments |
| "Process memory disclosure is a common attack vector" | If attackers have process memory access, credential storage method is irrelevant - the system is already compromised |
| "Container escape makes env vars vulnerable" | Container escapes expose the entire system; the storage method doesn't matter at that point |

### Risk Assessment Framework

**🔴 HIGH RISK (Do Not Use)**:
- Hardcoded credentials in source code
- API keys committed to git repository
- Credentials in public Docker images
- No credential rotation capability

**🟡 MEDIUM RISK (Use with Caution)**:
- Credentials in unencrypted config files
- Overly broad credential scopes
- No credential expiration policies
- Poor error handling for auth failures

**🟢 LOW RISK (Acceptable)**:
- Environment variables with proper error handling
- Documented credential requirements
- Separate dev/staging/production credentials
- Integration with standard secret management tools

## Implementation Examples

### Secure MCP Server Credential Setup

**Docker Deployment:**
```bash
docker run -e OPENAI_API_KEY="${OPENAI_API_KEY}" \
           -e DATABASE_URL="${DATABASE_URL}" \
           my-mcp-server
```

**Claude Desktop Configuration:**
```json
{
  "mcpServers": {
    "my-server": {
      "command": "python",
      "args": ["/path/to/server.py"],
      "env": {
        "API_KEY": "your-actual-key-here"
      }
    }
  }
}
```

### Advanced Security Patterns

**Credential Rotation Support:**
```python
# Check for credential refresh capabilities
def refresh_credentials():
    new_token = get_fresh_token()
    os.environ['API_TOKEN'] = new_token
```

**Least Privilege Access:**
```python
# Verify minimal scope requirements
required_scopes = ['read:files', 'write:comments']
if not validate_token_scopes(token, required_scopes):
    raise PermissionError("Insufficient token permissions")
```

## Integration with MCP Security Ecosystem

- **Cross-reference with `vulnerability-db`** for known credential exposure CVEs
- **Document findings in `audit-db`** for community benefit
- **Update `server-db`** with credential management assessment results
- **Flag for enhanced monitoring** if credential practices are concerning

## Remediation Guidance

### Immediate Actions for Hardcoded Secrets

1. **Remove from git history**: Use `git filter-branch` or BFG Repo-Cleaner
2. **Rotate all exposed credentials**: Assume they are compromised
3. **Add to .gitignore**: Prevent future commits of credential files
4. **Implement pre-commit hooks**: Scan for secrets before commits

### Migration to Secure Practices

1. **Environment variable conversion**: Move hardcoded values to env vars
2. **Documentation updates**: Document all required environment variables
3. **Example configuration**: Provide templates with placeholder values
4. **Testing**: Verify server works with environment-based credentials

## Teaching Points

**For Security Novices:**
- Environment variables are **secure and recommended** for credential management
- The real risk is credentials committed to git, not how they're stored at runtime
- Focus on preventing credential leaks, not over-engineering credential storage

**For Security Professionals:**
- MCP servers present unique risks due to their role as credential aggregators
- Standard cloud security practices (IAM, least privilege, rotation) apply
- Consider workload identity and service mesh approaches for advanced deployments

---

*This check is part of the MCP Server Audit security assessment framework. Last updated: 2025-08-07*