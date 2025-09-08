# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is **mcpserver-audit**, a security auditing tool for Model Context Protocol (MCP) servers and Claude Desktop Extensions. It serves as an expert security tutor that teaches users how to audit MCP server source code for vulnerabilities using AIVSS (AI Vulnerability Scoring System) scoring.

**Key Role**: Find security vulnerabilities in existing code (not fix them - that's handled by mcpserver-builder)

## Architecture Overview

This is a **prompt-driven security education system** built around three core components:

### 1. Security Assessment Prompts (`prompts/`)
- **Conversational AI instructions** that implement expert security tutoring
- Use **Socratic method** to teach vulnerability identification rather than just providing answers
- Each prompt targets specific security assessment scenarios:
  - `main-prompt.md`: Core security tutor persona
  - `security-assessment.md`: Comprehensive vulnerability scanning workflow  
  - `targeted-evaluation.md`: Focused security assessment of known servers
- **Usage pattern**: `read prompts/[file].md and [security assessment action]`

### 2. Modular Security Checks (`checks/`)
- **Technical vulnerability detection procedures** with both automated and manual steps
- Each check targets specific CWE categories with AIVSS scoring
- Include concrete detection scripts, risk assessment, and remediation guidance
- **Format**: Obsidian-style markdown with CWE mappings and vulnerability-db integration
- **Current coverage**: Credential management, network port binding (many gaps remain)

### 3. Knowledge Frameworks (`research/`, `resources/`, `tools/`)
- Supporting materials for threat modeling, risk assessment, and compliance mapping
- Templates and checklists for systematic security evaluation
- Placeholder for future MCP tool implementations

## MCP Security Ecosystem Integration

This tool is part of a three-tool workflow:
1. **mcpserver-audit** (this repo): Finds vulnerabilities → 
2. **mcpserver-builder**: Fixes vulnerabilities and builds secure code →
3. **mcpserver-operator**: Deploys and operates securely

## Common Usage Patterns

### Running Security Audits
```bash
# Comprehensive vulnerability scan
read prompts/security-assessment.md and conduct a security audit of MCP server [SERVER_NAME]

# Focused assessment  
read prompts/targeted-evaluation.md and audit the security of MCP server [SERVER_NAME]

# Apply specific vulnerability check
read checks/credential-management-security.md and check for credential vulnerabilities in [SERVER_NAME]
```

### System Improvement Workflow
The system is designed to **continuously improve** through use:
1. Run security assessment using existing prompts/checks
2. Document gaps and ineffective detection methods
3. Create/improve security checks based on real findings
4. Validate against known good/bad MCP servers
5. Update documentation and share knowledge

### Working with Security Checks
Each security check follows standardized format:
- **Frontmatter**: CWE mappings, AIVSS scoring capability, metadata
- **Automated detection**: Scripts and commands for AI assistants
- **Manual assessment**: Human verification steps
- **Risk framework**: Clear severity classification
- **Examples**: Real secure vs. vulnerable patterns
- **Ecosystem handoffs**: Limited remediation, clear handoff to builder/operator

## Key Security Concepts

### AIVSS Scoring
- **Primary approach** for vulnerability scoring in this system
- Extends CVSS v4.0 with AI risk factors (AARS)
- Reference materials: CloudSecurityAlliance datasets for AI-optimized markdown versions

### Educational Philosophy  
- **Teach vulnerability recognition** rather than just scan for problems
- **Socratic questioning** to help users discover security issues themselves
- **Progressive complexity** from basic patterns to sophisticated analysis
- **Context sensitivity** adapting to user skill level and server risk profile

### CWE Coverage Goals
System aims for comprehensive Common Weakness Enumeration coverage:
- Priority CWEs: 79 (XSS), 89 (SQL Injection), 94 (Code Injection), 22 (Path Traversal), 287 (Auth), 352 (CSRF), 502 (Deserialization), 798 (Hardcoded Creds)
- Each CWE should have dedicated security check with AIVSS scoring
- Current coverage is minimal - major contribution opportunity

## Development Approach

This is **not a traditional software project** - it's a knowledge system:
- **No build/test/deploy cycle** - changes are documentation and prompt improvements
- **No package dependencies** - pure markdown documentation system  
- **Quality measured by**: Assessment effectiveness, false positive rates, user skill development
- **Primary iteration**: Real security assessments → identify gaps → improve checks/prompts

## File Organization Logic

```
prompts/          # Conversational AI instructions for security tutoring
├── main-prompt.md              # Core security tutor persona
├── security-assessment.md      # Comprehensive audit workflow  
├── targeted-evaluation.md      # Focused server assessment
└── PROMPT-TEMPLATE.md         # Template for new prompts

checks/           # Technical vulnerability detection procedures  
├── credential-management-security.md    # CWE-798, 200, 522
├── network-port-binding-security-check.md
├── CHECK-TEMPLATE.md          # Template for new checks
└── README.md                  # Check development guidelines

research/         # Draft security research and exploratory work
resources/        # Security checklists and assessment frameworks
tools/           # Future MCP tool implementations
```

## Contributing Security Knowledge

When improving the system:
1. **New security checks** should target specific CWE categories with clear AIVSS scoring
2. **Prompt improvements** should enhance educational effectiveness and vulnerability detection
3. **Real-world examples** from actual MCP server assessments are highest value
4. **Templates provide structure** - follow CHECK-TEMPLATE.md and PROMPT-TEMPLATE.md formats
5. **Focus on teaching** - help users develop independent security assessment skills

The system succeeds when users can confidently audit MCP server code for vulnerabilities and understand how to score them using AIVSS methodology.