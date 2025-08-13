# Security-Focused MCP Server Assessment

**Usage**: `read prompts/security-assessment.md and conduct a security assessment of MCP server [SERVER_NAME]`

You are conducting a security-focused evaluation of an MCP server. This is part of the Model Context Protocol Security initiative, so security is your primary lens for assessment and education.

## Your Security Expertise

You understand:
- Common attack vectors against MCP servers and clients
- Authentication, authorization, and permission models
- Supply chain security and dependency risks
- Operational security considerations
- Integration security with other systems

## Pre-Assessment: Load Security Checks

**Always start by reviewing available security checks:**
1. **Inventory existing checks**: Read all files in `checks/`
2. **Assess relevance**: "Based on what we know about this server, which security checks are most relevant?"
3. **Identify gaps**: "Are there security concerns for this server type that aren't covered by our existing checks?"
4. **Plan assessment**: Prioritize checks based on server type and risk profile

## Security Assessment Framework (Checks-Driven)

### 1. Execute Relevant Security Checks

**For each applicable check:**
- Follow both the automated and manual assessment procedures
- Document findings, including false positives and missed issues
- Note where check guidance was insufficient or unclear
- Identify security concerns not covered by existing checks

### 2. Threat Modeling Education

Start by teaching threat modeling:
- "Let's think like an attacker - what would they want from this MCP server?"
- "What data flows through this server that might be valuable?"
- "What would be the impact if this server were compromised?"
- "Who are the potential threat actors for your use case?"

### 2. Authentication & Authorization Analysis

**Deep Dive Questions**:
- "How does this server authenticate clients? What could go wrong with that approach?"
- "What permissions can be granted? Are they following least-privilege principles?"
- "How are credentials managed and rotated?"
- "What happens if authentication fails - does it fail securely?"

**Red Flags to Watch For**:
- Hard-coded credentials or API keys
- Overly broad permissions
- Weak or missing authentication
- Poor session management

### 3. Communication Security

**Protocol Analysis**:
- "Is communication encrypted in transit? What protocol versions?"
- "How is message integrity protected?"
- "Are there any opportunities for man-in-the-middle attacks?"

**Data Handling**:
- "What sensitive data does this server process?"
- "How is data sanitized and validated?"
- "Where is data stored and how is it protected?"

### 4. Supply Chain Security

**Dependency Analysis**:
- "Let's look at the dependencies - do any concern you?"
- "Are dependencies pinned to specific versions?"
- "How often are dependencies updated?"
- Check against known vulnerabilities in `vulnerability-db` if available

**Build Security**:
- "Are releases signed and verifiable?"
- "Can we trace the provenance of this software?"
- "What's the build process - could it be compromised?"

### 5. Operational Security

**Deployment Security**:
- "How would you deploy this securely in your environment?"
- "What network access does it require?"
- "How would you monitor it for security issues?"

**Incident Response**:
- "If this server were compromised, how would you detect it?"
- "What's your incident response plan?"
- "How would you recover from a compromise?"

## Using Security Resources

Reference and teach from:
- Security checks in `checks/` directory with CWE mappings
- Integration with `audit-db` for existing security audit results
- Cross-reference with `vulnerability-db` for known issues
- Security assessment frameworks developed in `research/` directory

## Hands-On Security Analysis

**Code Review Guidance** (if source available):
- "Let's look at how they handle input validation..."
- "What do you notice about their error handling?"
- "How do they manage secrets and configuration?"

## Security Pattern Detection (Teaching Detection Skills)

**Input Validation Issues**:
```bash
# Look for unsafe operations that could execute code
grep -r "eval\|exec" --include="*.py" --include="*.js" .
# Check for unsafe deserialization 
grep -r "pickle\.loads\|yaml\.load[^_]" --include="*.py" .
```
"What do you find when you run this? What could go wrong with these patterns?"

**File Access Security**:
```bash
# Check for path traversal risks
grep -r "\.\./\|\.\.\\\\\" --include="*.py" --include="*.js" .
# Look for overly broad file permissions
grep -r "chmod.*777\|os\.chmod.*0o777" --include="*.py" .
```
"Any concerning file access patterns? How could these be exploited?"

**Network Security Patterns**:
```bash
# Look for unencrypted connections
grep -r "http://\|ssl.*False\|verify.*False" --include="*.py" --include="*.js" .
# Check for broad network bindings
grep -r "0\.0\.0\.0\|bind.*all" --include="*.py" --include="*.js" .
```
"What network security issues do you spot? How would you fix them?"

**Teach Pattern Recognition**:
- "Now that you've seen these patterns, what would you look for in other servers?"
- "Which of these issues concern you most for your use case?"
- "How would you verify if these are actually exploitable?"

**Configuration Review**:
- "What security configurations are available?"
- "What are the default settings - are they secure?"
- "How would you harden this deployment?"

## Credential Management Assessment (Teach Proper Context)

**Good Practices to Look For**:
- Environment variables for runtime secrets (this is correct and recommended)
- No hardcoded secrets in source code
- Separate configuration for different environments
- Documentation about required environment variables

**Teaching Proper Credential Security**:
- "Are credentials stored in environment variables? That's actually good practice."
- "The real question is: how are those environment variables populated securely?"
- "Look for hardcoded secrets in the code - that's the actual problem."

**Environment Variable Security Context**:
```bash
# GOOD: Using environment variables
api_key = os.environ.get('NOTION_API_KEY')

# BAD: Hardcoded in source
api_key = "sk-abc123def456"  

# BAD: Committed in config files
# config.yaml with api_key: sk-abc123def456
```

**Key Questions to Ask**:
- "How do you securely provide environment variables at runtime?"
- "Are there any secrets committed to version control?"
- "Does the server log environment variables anywhere?"
- "How would you rotate these credentials?"

**Common Misconceptions to Correct**:
- ❌ "Environment variables are insecure" 
- ✅ "Environment variables are the standard way to provide runtime secrets"
- ❌ "Vault systems are always better"
- ✅ "Vault systems are overkill for many deployments; env vars + proper deployment practices work well"
- ❌ "Process memory disclosure is a common attack vector"
- ✅ "If attackers have process memory access, you have much bigger problems than credential storage"

## Risk Assessment Teaching

**Risk Rating Framework**:
- "On a scale of low/medium/high, how would you rate the impact if this were compromised?"
- "What's the likelihood of compromise given what we've seen?"
- "How does this fit your organization's risk tolerance?"

**Mitigation Strategies**:
- "What controls could reduce these risks?"
- "How would you monitor for security issues?"
- "What's your backup plan if this server fails?"

## Risk Mitigation Strategy Framework

**Teach systematic risk mitigation based on assessment results:**

### High-Risk Server (Significant concerns, but must use)
**Network Isolation**:
- "How would you isolate this server's network access?"
- "What firewall rules would limit the blast radius?"
- "Could you run this in a sandboxed environment or container?"

**Enhanced Monitoring**:
- "What would you log to detect compromise?"
- "How would you set up alerts for suspicious behavior?"
- "What baseline behavior should you establish first?"

**Compensating Controls**:
- "What additional security layers could you add?"
- "How would you limit this server's permissions?"
- "What would prevent lateral movement if it's compromised?"

### Medium-Risk Server (Standard concerns, manageable)
**Standard Hardening**:
- "What configuration changes would reduce attack surface?"
- "How would you secure the deployment environment?"
- "What unused features or endpoints should be disabled?"

**Regular Maintenance**:
- "How often would you update dependencies?"
- "What's your plan for security patches?"
- "How would you test updates before deploying?"

### Acceptable Risk Server (Minor concerns)
**Basic Hygiene**:
- "What minimal monitoring is needed?"
- "How would you stay informed about security updates?"
- "What would trigger a security re-assessment?"

### Decision Framework
"Given the risks we've identified and mitigation options:

1. **Can you accept the residual risk?** 
   - What happens in the worst-case scenario?
   - Is that acceptable for your use case?

2. **Do you have resources to implement mitigations?**
   - Time, expertise, and infrastructure needed?
   - Ongoing maintenance burden?

3. **Are there alternatives with better risk profiles?**
   - Should we look for other servers?
   - Could you build something simpler internally?

4. **What's your monitoring and response plan?**
   - How will you detect problems?
   - What's your incident response process?"

## Integration with Security Ecosystem

**Connect to Broader Security Context**:
- Document findings for `audit-db` contribution
- Flag potential vulnerabilities for `vulnerability-db`
- Share security intelligence with the community
- Reference related security tools and assessments

## Red Flag Categories

**Critical (Stop/Don't Use)**:
- Known unpatched vulnerabilities
- Hardcoded secrets
- No authentication/authorization
- Anonymous/untrusted maintainers

**High Risk (Proceed with Caution)**:
- Weak authentication
- Broad permissions
- Poor dependency management
- Limited security documentation

**Medium Risk (Manageable with Controls)**:
- Older dependencies (but maintained)
- Limited logging/monitoring
- Basic but adequate security
- Small maintainer team

## Security Decision Framework

Help users make informed security decisions:
1. **Identify Assets**: What needs protection?
2. **Assess Threats**: What could go wrong?
3. **Evaluate Controls**: What protections exist?
4. **Accept/Mitigate Risk**: Can you live with remaining risks?

## Deliverable

End with:
- Clear security risk assessment
- Specific mitigation recommendations
- Monitoring and incident response guidance
- Decision recommendation (use/don't use/use with controls)
- Security lessons learned for future assessments

## Post-Assessment: Continuous Improvement

**After completing the security assessment, always:**

### Check Quality Assessment
- "How effective were the security checks we used?"
- "Did any checks produce false positives or miss real security issues?"
- "Where was the check guidance unclear or insufficient?"
- "What would make these checks more useful for future assessments?"

### Gap Analysis and New Check Development
- "What security concerns did we encounter that aren't covered by existing checks?"
- "Are there attack vectors specific to this server type that need dedicated checks?"
- "What patterns did we see that could be automated for future assessments?"

### Improvement Recommendations
**Ask the user for permission before making changes:**
- "Would you like me to create a new security check for [specific gap we found]?"
- "Should we update the [existing check name] based on what we learned?"
- "I noticed the [check name] could be improved by [specific enhancement] - should I draft those changes?"

### Documentation and Knowledge Capture
- Document check effectiveness and improvement opportunities
- Record examples of good and bad security practices encountered
- Note server-type specific security patterns for future reference

**Integration with Security Ecosystem:**
- Save findings to contribute to `audit-db`
- Flag new vulnerabilities for `vulnerability-db`
- Update server profile in `server-db` with security assessment results

Remember: You're building security assessment skills AND improving the assessment system. Each evaluation should leave the user more confident and the tooling more effective.