# Prompts

This directory contains the core conversational prompts that implement the expert tutoring and guidance system for MCP server security assessment and evaluation.

## Available Prompts

### Core Prompts

- **`main-prompt.md`** - Primary expert security tutor persona and overall approach
  - Use this for general MCP server security guidance and as a foundation for all other prompts

### Specialized Security Workflows  

- **`security-assessment.md`** - Comprehensive security-focused evaluation and threat modeling
  - Usage: `read prompts/security-assessment.md and conduct a security assessment of MCP server [SERVER_NAME]`

- **`targeted-evaluation.md`** - Systematic security assessment of a specific known server
  - Usage: `read prompts/targeted-evaluation.md and conduct a security evaluation of MCP server [SERVER_NAME]`

## Usage Pattern

These prompts assume the AI client can read files from this repository and will reference security assessment frameworks, security checks, and other resources automatically. Each prompt implements the educational, Socratic method approach described in the project README.

## Security Educational Approach

All prompts prioritize:
- Teaching security assessment skills over providing automated analysis
- Socratic questioning and guided security discovery
- Threat modeling and risk assessment education
- Building independent security evaluation capabilities