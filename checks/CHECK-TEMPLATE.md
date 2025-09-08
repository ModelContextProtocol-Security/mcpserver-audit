---
title: [Check Name] - TEMPLATE
version: 1.0
date: [YYYY-MM-DD]
tags: [security, vulnerability, MCP, [specific-security-area], aivss]
aliases: [[alternative-names]]
status: template
cwe: [[XXX, XXX]]  # Applicable Common Weakness Enumerations
cwe-primary: [XXX]  # Primary CWE this check addresses
---

# [Check Name] - Security Assessment Check

## Security Assessment Metadata

**AIVSS Scoring Capability**: Full/CVSS-Only/AARS-Only
**Confidence Level**: High/Medium/Low/Variable
**Complexity**: Simple/Moderate/Complex/Expert  
**Data Requirements**: Always/Usually/Sometimes/Rarely Available
**Time to Complete**: X minutes (automated) + Y minutes (manual)
**Reliability Notes**: [What factors affect confidence in this security assessment]
**Risk Level**: Critical/High/Medium/Low (based on AIVSS scoring)

## Security Purpose & Context

[Explain why this security aspect matters for MCP server risk assessment, including both traditional vulnerabilities and AI risks]

**Security Impact**: [How this vulnerability affects overall security posture - Critical/High/Medium/Low impact]
**When to Use**: [Specific security scenarios where this check is most valuable]
**When to Skip**: [Situations where this security check might not be relevant]
**Threat Context**: [What threat actors or attack scenarios this addresses]

## Security Assessment Criteria

### High Confidence Vulnerabilities
- **Clear Security Issues**: [Specific, objective indicators of vulnerabilities]
- **CVSS Quantifiable**: [Traditional security metrics that can be scored confidently]
- **Definite Threat Patterns**: [Obvious signs of security weaknesses]

### Medium Confidence Security Risks  
- **Probable Vulnerabilities**: [Patterns that suggest security risks but require interpretation]
- **AARS Factor Assessment**: [AI risk factors that depend on server context]
- **Contextual Security Signals**: [Security indicators that depend on threat model]

### Low Confidence / Expert Analysis Required
- **Expert Security Judgment Required**: [Areas needing significant security experience to assess]
- **Context-Dependent Risks**: [Security factors that vary greatly by deployment scenario]
- **Advanced Threat Analysis**: [Signals that require specialized security knowledge]

## Automated Security Assessment (For AI Assistants)

### High Confidence Vulnerability Detection
```bash
# [Specific commands, scripts, or patterns for detecting clear security issues]
# [Clear thresholds for vulnerability classification]
# [Objective security metrics extraction]
```

### AIVSS Risk Factor Assessment
```bash
# [Commands to assess AI risk factors: autonomy, tool use, etc.]
# [Guidance for scoring AARS factors (0.0, 0.5, 1.0)]
# [Context analysis for AI behavior patterns]
```

**Automation Reliability**: [How much to trust automated security results vs. expert security review]

## Manual Security Assessment (For Humans)

### Step-by-Step Security Checklist
- [ ] **CVSS Base Score Assessment**: [Traditional vulnerability scoring - High confidence]
- [ ] **AARS Factor Evaluation**: [AI risk factor scoring - Medium confidence]  
- [ ] **Threat Model Integration**: [How this fits user's threat model - Expert judgment]
- [ ] **Context-Specific Risk Assessment**: [Deployment-specific security considerations]

### Key Security Questions to Ask
- **High Confidence (CVSS)**: "[Clear vulnerability question with objective answer]"
- **Medium Confidence (AARS)**: "[AI risk question requiring interpretation]"  
- **Expert Analysis**: "[Question requiring specialized security expertise]"

### Human Security Verification Points
- [When to double-check automated vulnerability detection]
- [Subjective security assessments that require expert analysis]
- [Context-specific threat considerations]

## Secure vs. Vulnerable Patterns

### Excellent Security Examples
**Pattern**: [Specific observable secure pattern]
**Why Secure**: [Explanation of why this prevents vulnerabilities]
**AIVSS Context**: [How this addresses both CVSS and AARS risks]
**Confidence**: High/Medium/Low in identifying this secure pattern
**Example**: [Real-world secure implementation if possible]

### Vulnerable Security Examples  
**Pattern**: [Specific observable vulnerable pattern]
**Why Vulnerable**: [Explanation of why this creates security risks]
**AIVSS Scoring**: [How this scores in both CVSS and AARS factors]
**Confidence**: High/Medium/Low in identifying this vulnerability
**Example**: [Real-world vulnerable example if possible]

### Ambiguous Security Cases
**Pattern**: [Patterns that could be secure or vulnerable depending on context]
**Context Factors**: [What determines if this is a security risk]
**Threat Model Impact**: [How different threat models affect risk assessment]
**Assessment Approach**: [How to evaluate these ambiguous security cases]

## Security Assessment Confidence Indicators

### High Confidence Security Results
**When to Trust**: [Conditions where this security assessment is highly reliable]
**Minimum Security Data**: [Minimum information needed for reliable vulnerability assessment]
**Strong Security Signals**: [Clear indicators that increase confidence in security findings]

### Low Confidence Security Warnings
**Security Assessment Red Flags**: [When to be skeptical of vulnerability results]
**Data Quality Issues**: [Problems that undermine security assessment reliability]
**External Security Factors**: [Things outside the check that affect security reliability]

### Improving Security Assessment Confidence
**Additional Security Data**: [What extra information would improve vulnerability assessment]
**Cross-Validation**: [Other security checks that can confirm/contradict findings]
**Threat Evolution Factors**: [How time and evolving threats affect assessment confidence]

## Security Justification & Evidence

### Why This Security Assessment Matters
**Impact on Security Posture**: [How this vulnerability affects overall server security]
**Risk Implications**: [What security problems arise when this area is weak]
**Threat Actor Interest**: [Why attackers would target this vulnerability]
**AIVSS Relevance**: [How this relates to both traditional and AI security risks]

### Supporting Security Evidence
**Research Basis**: [Academic/industry security research supporting this assessment]
**Security Standards**: [Industry security standards, CWEs, or frameworks]
**CVE References**: [Known vulnerabilities that relate to this pattern]
**Real-World Exploits**: [Evidence from actual security incidents]

### Security Decision Weighting
**Critical Security Factor**: [When this vulnerability should heavily influence security decisions]
**Supporting Risk Factor**: [When this is important but not the primary security concern]  
**Context Dependent Risk**: [When security importance varies by threat model]

## Security Integration Guidance

### Using High Confidence Security Results
- [How to incorporate reliable vulnerability findings into risk decisions]
- [Weight to give CVSS vs. AARS scores in overall AIVSS assessment]
- [Clear security decision rules based on confident vulnerability findings]

### Handling Low Confidence Security Results
- [How to treat uncertain security assessments]
- [When to seek additional security expert validation]
- [How to communicate security uncertainty to users]

### Combining with Other Security Checks
- [Which other security checks complement this one]
- [How to resolve conflicts between different security assessments]
- [Holistic AIVSS evaluation approaches across multiple checks]

## Common Security Assessment Pitfalls

### Security Assessment Errors
- [Common mistakes in applying this security check]
- [Security misinterpretations to avoid, especially around AIVSS scoring]
- [Cognitive biases that can affect security judgment]

### Security Context Mismatches
- [When this security check doesn't apply well]
- [MCP server types where this vulnerability is less relevant]  
- [Deployment scenarios that change the security assessment criteria]

### Security Confidence Misjudgments
- [When people over-trust uncertain security results]
- [When people under-value reliable vulnerability findings]
- [How to calibrate security confidence appropriately between CVSS and AARS factors]

## Basic Remediation Guidance

### Critical Issues Found
- **[Specific vulnerability type]**: [Brief description of immediate security risk]
- **[Another vulnerability type]**: [Brief description of security issue]

### Next Steps for Fixing
**For detailed remediation guidance**: Use mcpserver-builder for step-by-step secure coding fixes
**For secure deployment**: Coordinate with mcpserver-operator for runtime security controls
**Immediate priority**: Focus on the highest AIVSS-scored vulnerabilities first

## Integration with MCP Security Ecosystem

- **Cross-reference with `vulnerability-db`** for known vulnerability patterns
- **Document findings in `audit-db`** for community benefit
- **Update `server-db`** with security assessment results
- **Flag for enhanced monitoring** if security practices are concerning

## AIVSS & CVSS Reference Materials

For AI assistants conducting security assessments, these markdown versions of the standards are optimized for AI processing:

- **AIVSS (AI Vulnerability Scoring System)**: https://github.com/CloudSecurityAlliance-DataSets/dataset-public-laws-regulations-standards/tree/main/tools-resources/owasp.org/AIVSS
- **CVSS v4.0 (Common Vulnerability Scoring System)**: https://github.com/CloudSecurityAlliance-DataSets/dataset-public-laws-regulations-standards/tree/main/tools-resources/first.org/CVSS

---

*This check is part of the MCP Server Audit security assessment framework. Last updated: [YYYY-MM-DD]*

**Template Usage Notes:**
- Replace all [bracketed items] with specific security content
- Choose appropriate AIVSS scoring capability and confidence levels
- Include real vulnerability examples where possible
- Focus on observable, actionable security criteria
- Emphasize security education over just vulnerability detection
- Feature AIVSS scoring as the primary approach for MCP server security assessment
- Use the above reference materials to ensure accurate AIVSS and CVSS scoring