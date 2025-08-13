# Security Checks

This directory contains specific, actionable security verification procedures for MCP server assessment. Each check provides both automated analysis instructions for AI assistants and manual steps for human reviewers.

## Purpose

Security checks are modular, repeatable verification procedures that:
- Focus on specific security concerns or attack vectors
- Provide concrete detection methods and scripts
- Include context about risk levels and remediation
- Distinguish between real security issues and security theater

## Active Development Workflow

**This directory is under active development and improvement:**

### Population Phase
- **New checks needed**: Many security areas are not yet covered by dedicated checks
- **Gap identification**: Each security assessment may reveal areas needing new checks
- **Community contribution**: Real-world MCP server assessments drive check creation

### Continuous Improvement Phase
- **Check refinement**: Existing checks are regularly improved based on assessment results
- **Example updates**: Good/bad examples are added from real assessments
- **Accuracy improvements**: False positive/negative rates are reduced through iteration
- **Coverage expansion**: Checks are enhanced to cover more security scenarios

### Long-term Integration Goals
- **CWE Coverage**: Every applicable Common Weakness Enumeration (CWE) should be represented
- **Vulnerability Integration**: Checks will reference `vulnerability-db` for known issues
- **Community Intelligence**: Checks will incorporate real-world attack patterns and defenses

## Check Format

Each check follows a standardized format:
- **Obsidian-formatted markdown** with proper frontmatter including:
  - `cwe:` tags referencing applicable [Common Weakness Enumeration](https://cwe.mitre.org/) IDs
  - `vulnerability-db:` references (planned for future integration)
  - Standard metadata (version, date, tags, priority, status)
- **Purpose and context** explaining why the check matters
- **AI instructions** with specific commands and scripts
- **Human assessment steps** for manual verification
- **Risk assessment framework** with clear severity levels
- **Good and bad examples** demonstrating secure vs. insecure patterns
- **Remediation guidance** for addressing findings
- **Teaching points** for building security understanding

## CWE Integration

**Goal**: Comprehensive coverage of applicable Common Weakness Enumerations for MCP servers.

### Current CWE Mapping
- `credential-management-security.md`: CWE-798 (Hard-coded Credentials), CWE-200 (Information Exposure), CWE-522 (Insufficiently Protected Credentials)

### Priority CWEs for MCP Servers
High-priority CWEs that need dedicated checks:
- **CWE-79**: Cross-site Scripting (XSS) - for web-based MCP servers
- **CWE-89**: SQL Injection - for database-accessing servers  
- **CWE-94**: Code Injection - for servers executing dynamic code
- **CWE-287**: Improper Authentication - for authentication mechanisms
- **CWE-352**: Cross-Site Request Forgery - for web interfaces
- **CWE-22**: Path Traversal - for file-accessing servers
- **CWE-502**: Unsafe Deserialization - for data processing servers

### CWE Documentation Standard
Each check should include in frontmatter:
```yaml
cwe: [798, 200]  # List of applicable CWE IDs
cwe-primary: 798  # Primary CWE this check addresses
```

## Examples in Security Checks

**Every check should include concrete examples:**

### Good Examples (Secure Patterns)
- Real code snippets showing proper implementation
- Configuration examples with security best practices
- Documentation patterns that indicate quality security practices
- Server behaviors that demonstrate good security hygiene

### Bad Examples (Insecure Patterns)  
- Code patterns that create security vulnerabilities
- Configuration mistakes that expose systems to risk
- Documentation red flags that indicate security problems
- Server behaviors that suggest poor security practices

### Example Guidelines
- **Use real-world patterns** encountered in actual MCP servers
- **Anonymize** sensitive information while preserving the security lesson
- **Explain the why** - don't just show good/bad, explain the security implications
- **Keep current** - update examples as new patterns emerge in the ecosystem
- **Show context** - include enough surrounding code/config to make examples actionable

## Available Checks

- **`credential-management-security.md`** - Evaluates how servers handle API keys, tokens, and sensitive configuration

## Usage with Prompts

These checks are referenced by the security assessment prompts in `prompts/` and provide the detailed technical guidance for conducting thorough security evaluations.

## Contributing New Checks

When adding new security checks:
1. Use the established Obsidian format with proper tags
2. Include both automated and manual assessment methods
3. Provide clear risk assessment criteria
4. Explain the security context and common misconceptions
5. Include practical remediation steps
6. Add teaching elements for skill development