# Security Audit Prompts

This directory contains the hierarchical prompt system for MCP server security auditing. The architecture follows a main → sub-prompt → checks structure for systematic vulnerability assessment.

## Prompt Architecture

### Main Entry Point
- **`main-prompt.md`** - Primary coordinator for security audits
  - Assesses audit requirements and guides users to appropriate sub-prompts
  - Provides architecture overview and resource coordination
  - **Usage**: Start here for all security audit sessions

### Sub-Audit Prompts  
- **`security-assessment.md`** - Comprehensive vulnerability audit with detailed documentation
  - **Use for**: Complete security reviews, compliance audits, production assessments
  - **Creates**: Individual issue files, remediation tracking, full audit reports
  - **Usage**: `read prompts/security-assessment.md and conduct a complete security audit of MCP server [SERVER_NAME]`

- **`targeted-evaluation.md`** - Focused audit of specific security concerns
  - **Use for**: Known servers, specific vulnerability types, quick assessments  
  - **Creates**: Targeted findings, risk-specific analysis
  - **Usage**: `read prompts/targeted-evaluation.md and audit specific security concerns in MCP server [SERVER_NAME]`

### Granular Security Procedures
- **`checks/`** directory - Specific vulnerability detection procedures
  - CWE-mapped security checks with AIVSS scoring
  - Referenced automatically by sub-audit prompts

## Usage Workflow

1. **Start with main prompt** to assess audit requirements
2. **Main prompt guides** to appropriate sub-prompt based on needs
3. **Sub-prompts automatically apply** relevant security checks from `checks/`
4. **Produce structured output** for vulnerability tracking and remediation

## Educational Focus

All prompts emphasize:
- Socratic method teaching approach
- Vulnerability pattern recognition
- AIVSS scoring methodology  
- Context-aware risk assessment
- Building independent audit capabilities