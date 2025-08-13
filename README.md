# MCP Server Audit

**Code audit tool that finds security vulnerabilities in MCP servers and Claude Desktop Extensions - because anyone can build them, but not everyone builds them safely.**

## Why This Tool Exists

**Anyone can create MCP servers and Desktop Extensions** - no programming experience required. Here's how easy it is:

### Example: Building a Desktop Extension (No Programming Needed)

As Anthropic states in their [official blog post](https://www.anthropic.com/engineering/desktop-extensions):

> "Internally at Anthropic, we have found that Claude is great at building extensions with minimal intervention. If you too want to use Claude Code, we recommend that you briefly explain what you want your extension to do and then add the following context to the prompt:"

Simply paste this into Claude or any AI assistant:

```
I want to build this as a Desktop Extension, abbreviated as "DXT". Please follow these steps:

1. **Read the specifications thoroughly:**
   - https://github.com/anthropics/dxt/blob/main/README.md - DXT architecture overview, capabilities, and integration patterns
   - https://github.com/anthropics/dxt/blob/main/MANIFEST.md - Complete extension manifest structure and field definitions
   - https://github.com/anthropics/dxt/tree/main/examples - Reference implementations including a "Hello World" example

2. **Create a proper extension structure:**
   - Generate a valid manifest.json following the MANIFEST.md spec
   - Implement an MCP server using @modelcontextprotocol/sdk with proper tool definitions
   - Include proper error handling and timeout management

3. **Follow best development practices:**
   - Implement proper MCP protocol communication via stdio transport
   - Structure tools with clear schemas, validation, and consistent JSON responses
   - Make use of the fact that this extension will be running locally
   - Add appropriate logging and debugging capabilities
   - Include proper documentation and setup instructions

4. **Test considerations:**
   - Validate that all tool calls return properly structured responses
   - Verify manifest loads correctly and host integration works

Generate complete, production-ready code that can be immediately tested. Focus on defensive programming, clear error messages, and following the exact DXT specifications to ensure compatibility with the ecosystem.
```

**The AI will write the entire extension for you.** No coding knowledge required.

## The Problem: Easy to Build, Hard to Build Safely

Since anyone can create these tools by asking AI assistants, we're seeing an explosion of MCP servers and extensions. **But many are poorly built and potentially dangerous:**

- **Security vulnerabilities** - AI-generated code often has security holes
- **Data exposure risks** - Tools may leak sensitive information
- **Poor quality** - Many tools break, perform poorly, or aren't maintained
- **Unknown trustworthiness** - Hard to tell which tools are safe to use

## The Solution: Code Security Audit

MCP Server Audit **audits MCP server source code to find security vulnerabilities**. Rather than just being an automated scanner, it serves as a knowledgeable security tutor who teaches you how to identify code vulnerabilities, understand AIVSS scoring, and develop the skills to audit server code independently.

This tool helps you **audit MCP server code** to find vulnerabilities in servers that are:
- Built by AI assistants (like the example above)
- Downloaded from the community
- Created by professional developers
- Something you built yourself

The tool combines vulnerability detection with security education, helping you understand not just whether code has security issues, but how to identify and score vulnerabilities systematically using AIVSS (Agentic AI Vulnerability Scoring System).

## MCP Security Ecosystem Integration

MCP Server Audit is part of a complete security toolkit:

- **mcpserver-audit** (this tool): **Finds security vulnerabilities** in existing code
- **mcpserver-builder**: **Fixes vulnerabilities and builds secure code** from scratch or updates existing code
- **mcpserver-operator**: **Deploys and operates servers securely**, provides runtime security controls and workarounds

**Complete workflow**: Audit finds problems → Builder fixes them in code → Operator handles secure deployment

## Expert-Guided Approach

This MCP server is designed as an **expert advisor and tutor** in its domain, with the primary focus on providing guidance, education, and practical recommendations rather than being a comprehensive implementation tool. While it includes enough functional capability to be immediately useful and demonstrate real expertise, the long-term vision is for this server to serve as your domain expert who helps you understand the landscape, evaluate options, and find the right tools for your specific needs. As the MCP ecosystem evolves, this server will learn about new tools and approaches, recommending the best solutions while teaching you how to evaluate and use them effectively. The broader Model Context Protocol Security initiative may also develop comprehensive implementation tools, but these specific repositories are focused on being practical expert guidance systems—combining deep domain knowledge with enough hands-on capability to provide genuine value, not just theoretical advice.

## Key Capabilities

### What This Tool Does

- **Audits source code** - Scans MCP server code for security vulnerabilities  
- **Scores vulnerabilities** - Uses AIVSS to rate severity of security issues found
- **Identifies dangerous patterns** - Spots code that creates security risks
- **Teaches vulnerability recognition** - Learn what security issues look like in code
- **Limited fix guidance** - Basic recommendations (detailed fixes handled by mcpserver-builder)

### Code Vulnerability Analysis
- **Source Code Scanning**: Automated detection of security vulnerabilities in code
- **Dependency Analysis**: Checks for known vulnerabilities in third-party libraries
- **Configuration Review**: Identifies insecure configuration patterns
- **AIVSS Scoring**: Rates vulnerabilities using both traditional CVSS and agentic AI risk factors
- **CWE Mapping**: Links findings to Common Weakness Enumeration categories

### Security Education
- **Vulnerability Recognition**: Teaches you to identify security issues in MCP server code
- **Code Security Patterns**: Shows secure vs. insecure coding patterns
- **AIVSS Understanding**: Explains how to score agentic AI security risks
- **Audit Methodology**: Builds skills for systematic code security review

### Ecosystem Integration
- **Vulnerability Database**: Contributes findings to audit-db and vulnerability-db for community benefit
- **mcpserver-builder Integration**: Hands off vulnerabilities for detailed fix recommendations
- **mcpserver-operator Integration**: Coordinates with deployment security guidance
- **Community Intelligence**: Shares vulnerability patterns to help everyone stay safer

## Who Needs This Tool

**Everyone in the MCP ecosystem:**

- **Non-programmers creating tools** - Verify your AI-generated extensions are safe
- **Users downloading tools** - Check if community extensions are trustworthy  
- **Security-conscious organizations** - Evaluate tools before business use
- **Developers** - Systematic approach to tool evaluation and creation

## Goals and Teaching Approach

### Primary Educational Goals

1. **Develop Security Thinking**: Teach users to approach MCP servers with a security mindset
2. **Build Risk Assessment Skills**: Help users evaluate and communicate security risks effectively
3. **Foster Threat Awareness**: Educate about MCP-specific threats and attack vectors
4. **Promote Security Best Practices**: Share knowledge about secure development and deployment
5. **Enable Independent Analysis**: Provide frameworks for conducting security evaluations
6. **Build Community Security Knowledge**: Contribute to collective understanding of MCP security

### Teaching Strategies

#### 1. Interactive Security Learning
- **Threat Modeling Sessions**: Walks you through identifying and analyzing threats
- **Risk Assessment Practice**: Hands-on experience with security risk evaluation
- **Vulnerability Analysis**: Guided examination of real security issues
- **Attack Scenario Planning**: Explores how attacks might unfold
- **Defense Strategy Development**: Teaches mitigation and protection approaches

#### 2. Practical Security Application
- **Real-World Examples**: Uses actual MCP servers to illustrate security concepts
- **Case Study Analysis**: Examines both secure and vulnerable implementations
- **Scenario-Based Learning**: Presents different security contexts and challenges
- **Problem-Solving**: Helps you work through actual security assessment challenges
- **Skill Building**: Develops your ability to evaluate security independently

#### 3. Security Knowledge Sharing
- **Expert Insights**: Provides professional security perspectives and experience
- **Community Wisdom**: Shares collective knowledge from the security community
- **Emerging Threats**: Keeps you informed about new attack vectors and risks
- **Defense Evolution**: Teaches how security practices evolve with threats
- **Continuous Learning**: Stays current with security developments and research

## Expert Knowledge Areas

### MCP-Specific Security Threats
- **Prompt Injection**: Understanding direct and indirect prompt manipulation attacks
- **Confused Deputy**: Recognizing authorization bypass and privilege escalation risks
- **Token Theft**: Evaluating OAuth token security and credential protection
- **Data Exfiltration**: Assessing risks of unauthorized data access and leakage
- **Protocol Violations**: Identifying deviations from MCP security specifications
- **Cross-Origin Issues**: Understanding cross-domain and cross-service security risks

### AI-Specific Vulnerabilities
- **Prompt Injection**: Malicious prompts that manipulate server behavior
- **Model Manipulation**: Attacks targeting the underlying language models
- **Training Data Poisoning**: Compromised training data affecting model behavior
- **Output Manipulation**: Attacks that modify or intercept model responses
- **Context Poisoning**: Malicious context injection affecting decision-making
- **Indirect Prompt Injection**: Attacks via external data sources
- **Model Denial of Service**: Resource exhaustion attacks against AI systems
- **Privacy Attacks**: Extraction of sensitive information from models

### Security Assessment Methodologies
- **Threat Modeling**: Systematic identification and analysis of security threats
- **Risk Analysis**: Quantitative and qualitative approaches to risk assessment
- **Vulnerability Assessment**: Techniques for identifying security weaknesses
- **Code Security Review**: Methods for evaluating code security
- **Configuration Security**: Assessing security-relevant configuration options
- **Dependency Security**: Evaluating third-party library and dependency risks

### Security Tool Ecosystem
- **Static Analysis Tools**: Understanding and using code analysis tools
- **Dynamic Testing**: Approaches for runtime security testing
- **Vulnerability Scanners**: Leveraging automated vulnerability detection
- **Security Frameworks**: Using established security assessment frameworks
- **Monitoring Solutions**: Implementing security monitoring and alerting
- **Incident Response**: Handling security incidents and breaches

### Comprehensive Security Coverage

**Known Vulnerabilities**: We systematically cover established security patterns:
- **Standard CWE categories** - All applicable Common Weakness Enumerations 
- **AI-specific threats** - Prompt injection, model manipulation, training data poisoning
- **MCP protocol risks** - Transport security, authentication flows, message validation
- **Supply chain issues** - Dependency vulnerabilities, maintainer trust, build security

**Emerging Threats**: We train assessment capabilities to recognize new and evolving risks:
- **Pattern recognition** - Identify suspicious code structures that don't match known vulnerabilities
- **Contextual analysis** - Spot concerning combinations of features that individually seem safe
- **Behavioral assessment** - Detect unusual server behaviors that could indicate novel attack vectors
- **Ecosystem monitoring** - Flag new trends in the MCP community that might introduce risks

## Practical Workflow

### Phase 1: Security Education and Threat Modeling
1. **Security Awareness Building**: Introduce MCP-specific security concepts and risks
2. **Threat Model Development**: Guide you through identifying relevant threats
3. **Risk Assessment Framework**: Teach systematic approaches to risk evaluation
4. **Attack Vector Analysis**: Explore how attacks might target the specific server
5. **Security Objectives**: Define what security means for your use case

### Phase 2: Guided Security Analysis
1. **Analysis Planning**: Develop a systematic approach to security evaluation
2. **Basic Security Scanning**: Perform fundamental security checks
3. **Code Review Guidance**: Help you understand what to look for in code
4. **Configuration Assessment**: Evaluate security-relevant settings and options
5. **Dependency Review**: Check for known vulnerabilities in dependencies

### Phase 3: Risk Evaluation and Prioritization
1. **Finding Classification**: Categorize and prioritize discovered issues
2. **Risk Assessment**: Evaluate the real-world impact and likelihood of threats
3. **Vulnerability Analysis**: Deep dive into identified security weaknesses
4. **Exploitability Assessment**: Determine how easily issues could be exploited
5. **Business Impact**: Understand how security issues affect your specific use case

### Phase 4: Remediation Guidance and Next Steps
1. **Mitigation Strategy**: Develop approaches to address identified risks
2. **Remediation Planning**: Prioritize and plan security improvements
3. **Compensating Controls**: Identify alternative security measures when needed
4. **Monitoring Recommendations**: What to watch for in ongoing operations
5. **Learning Reinforcement**: Summarize key security lessons and skills developed

## Basic Functional Capabilities

### Security Analysis Functions
- **Static Code Analysis**: Basic scanning for common security patterns and anti-patterns
- **Dependency Scanning**: Checking for known vulnerabilities in third-party libraries
- **Configuration Review**: Assessment of security-relevant configuration options
- **Protocol Compliance**: Verification of MCP security specification adherence
- **Permission Analysis**: Evaluation of access controls and privilege models

### Educational Functions
- **Interactive Threat Modeling**: Guided sessions for identifying and analyzing threats
- **Risk Assessment Tools**: Frameworks for evaluating and communicating security risks
- **Vulnerability Explanation**: Detailed explanations of security issues and their implications
- **Mitigation Guidance**: Practical advice on addressing identified security concerns
- **Security Skill Assessment**: Evaluation of user security knowledge and gaps

### Orchestration Functions
- **Security Tool Integration**: Recommendations for specialized security analysis tools
- **Expert Consultation**: Guidance on when to involve security professionals
- **Resource Recommendations**: Pointing to relevant security documentation and resources
- **Community Intelligence**: Access to collective security knowledge and experience
- **Incident Response Planning**: Guidance on handling security issues and breaches

## How to Use This System

### Quick Walkthrough

**1. Choose Your Code Audit Approach:**

- **Comprehensive Code Audit**: Start with `read prompts/security-assessment.md` for complete vulnerability scanning and code analysis
- **Targeted Code Review**: Use `read prompts/targeted-evaluation.md` if you already have a specific server to audit  
- **General Audit Guidance**: Use `read prompts/main-prompt.md` for overall MCP server code audit guidance

**2. Apply Code Security Checks:**

- Browse `checks/` directory for available vulnerability detection checks
- Each check includes AIVSS scoring, CWE mappings, and code scanning methods
- Current checks: credential management vulnerabilities, network security issues
- Use AIVSS scores (Critical/High/Medium/Low) to prioritize vulnerability fixes

**3. Reference Audit Resources:**

- `research/` contains vulnerability analysis templates and audit frameworks
- `resources/` has code security checklists and vulnerability assessment frameworks  
- Many vulnerability types exist but don't have dedicated check files yet - great contributor opportunity!

**4. Leverage Code Analysis Tools:**

- Code vulnerability scanning and analysis capabilities are built into Claude Desktop and other MCP clients
- AIVSS and CVSS reference materials provided for accurate vulnerability scoring
- This system provides the audit methodology; your AI assistant provides the code analysis power

**5. Contribute and Hand Off:**

- Submit vulnerability findings to `audit-db` and `vulnerability-db` for community benefit
- **Hand off to mcpserver-builder** for detailed vulnerability fixes and secure coding guidance
- **Coordinate with mcpserver-operator** for secure deployment and runtime security controls
- Help us learn from your discoveries and improve the vulnerability detection framework

### Quick Start Examples

**Comprehensive Code Audit** (full vulnerability scan):
```
read prompts/security-assessment.md and conduct a security audit of MCP server [SERVER_NAME]
```

**Targeted Code Review** (focused vulnerability assessment):
```
read prompts/targeted-evaluation.md and audit the security of MCP server [SERVER_NAME]
```

**Apply Vulnerability Check** (scan for specific vulnerability type):
```
read checks/credential-management-security.md and check for credential vulnerabilities in [SERVER_NAME]
```

### System Requirements

- **Claude Desktop** or other MCP-compatible AI client
- **File system access** to read MCP server source code and vulnerability checks
- **Internet access** for vulnerability database lookups and dependency analysis (optional but helpful)

## Learn More

### Key Resources
- **[Desktop Extensions Blog Post](https://www.anthropic.com/engineering/desktop-extensions)** - Official introduction to building Claude Desktop Extensions
- **[DXT GitHub Repository](https://github.com/anthropics/dxt)** - Complete specifications and examples for Desktop Extensions
- **[Model Context Protocol](https://modelcontextprotocol.io/)** - Official MCP documentation and specifications

### Our Security-First Approach

This tool is part of the **Model Context Protocol Security** initiative - a community project focused on making MCP tools safer and more reliable. We emphasize:

- **Education over automation** - Teaching you to evaluate tools yourself
- **Security awareness** - Helping you recognize and avoid risks  
- **Community knowledge** - Sharing insights about tool quality and safety
- **Continuous improvement** - Getting better through real-world usage

### Personalized Learning (Future Vision)

**Long-term goal**: Make the teaching style flexible and personalized to each user's needs, experience level, and learning preferences.

Since this is an AI-powered tool, we can build assessment capabilities that customize themselves to you:
- **Adapt to your skill level** - Beginners get more explanation, experts get concise analysis
- **Match your role** - Different guidance for developers vs. business users vs. security professionals
- **Learn your preferences** - Remember what teaching methods work best for you
- **Focus on your goals** - Emphasize the security aspects most relevant to your use case
- **Evolve with you** - Become more advanced as your expertise grows

**AI enables personalization at scale** - the same tool can be a patient tutor for newcomers and an efficient expert advisor for professionals, adapting in real-time to what each user needs and wants.

## Integration with Ecosystem

### Security Framework Integration

We're building comprehensive mappings between vulnerabilities and security frameworks:

**Standards Alignment**:
- **CWE Mapping** - Every vulnerability linked to Common Weakness Enumeration categories
- **NIST Framework** - Map findings to NIST Cybersecurity Framework controls
- **OWASP Integration** - Align with OWASP Top 10 and Application Security standards
- **ISO 27001 Controls** - Connect vulnerabilities to specific ISO security controls

**Threat Intelligence Integration**:
- **MITRE ATT&CK** - Map attack patterns to MITRE ATT&CK framework tactics and techniques
- **Threat Modeling** - Link vulnerabilities to specific threat actors and attack scenarios
- **Risk Quantification** - Calculate business impact and likelihood scores
- **Control Effectiveness** - Measure how security controls reduce specific risks

**Enterprise Security Integration**:
- **Compliance Reporting** - Generate reports for SOC 2, PCI-DSS, HIPAA, etc.
- **Security Control Mapping** - Show which controls address which vulnerabilities
- **Risk Register Integration** - Feed findings into enterprise risk management systems
- **Policy Alignment** - Map findings to organizational security policies

This creates a **comprehensive security intelligence system** that doesn't just find problems, but explains their business impact, regulatory implications, and exactly which security controls need to be implemented to address them.

### Ecosystem Integration and Handoffs
- **mcpserver-finder**: Integrate with discovery patterns that indicate security concerns
- **mcpserver-builder**: Hand off vulnerabilities for detailed fix guidance and secure development
- **mcpserver-operator**: Coordinate on operational security requirements and deployment risks
- **vulnerability-db**: Contribute to and access known vulnerabilities and security intelligence
- **audit-db**: Share and learn from community vulnerability assessments and findings

### Code Audit Preparation for Next Steps
- **Vulnerability Fix Planning**: Prepare detailed findings for mcpserver-builder remediation
- **Deployment Security**: Document runtime security controls needed for mcpserver-operator  
- **Incident Response**: Ensure users understand how to handle discovered vulnerabilities
- **Continuous Monitoring**: Teach ongoing code audit and vulnerability monitoring practices

## Expert Development and Learning

### Continuous Security Knowledge Updates
- **Threat Intelligence**: Stay current with emerging threats and attack vectors
- **Vulnerability Research**: Monitor new vulnerabilities and security research
- **Tool Evolution**: Learn about new security analysis tools and techniques
- **Community Engagement**: Incorporate insights from security researchers and practitioners
- **Best Practice Evolution**: Adapt guidance based on evolving security landscape

### Teaching Methodology Improvement
- **Learning Assessment**: Evaluate how well users develop security thinking skills
- **Approach Refinement**: Improve security education methods based on user success
- **Curriculum Development**: Enhance security education content and progression
- **Feedback Integration**: Incorporate user feedback into teaching approaches
- **Skill Development**: Better understanding of security knowledge gaps and needs

## Security Assessment Types

### Assessment Purposes
- **Security Validation**: Verify that servers meet security requirements
- **Risk Assessment**: Evaluate security risks for deployment decisions
- **Vulnerability Discovery**: Identify potential security weaknesses
- **Compliance Evaluation**: Assess adherence to security standards and best practices
- **Incident Investigation**: Analyze security incidents and breaches

### Assessment Approaches
Each assessment type receives tailored security education, appropriate analysis depth, and specific guidance based on the security objectives and risk context.

## Security Assessment Types

### Assessment Purposes
- **Security Validation**: Verify that servers meet security requirements
- **Risk Assessment**: Evaluate security risks for deployment decisions
- **Vulnerability Discovery**: Identify potential security weaknesses
- **Compliance Evaluation**: Assess adherence to security standards and best practices
- **Incident Investigation**: Analyze security incidents and breaches

### Assessment Approaches
Each assessment type receives tailored security education, appropriate analysis depth, and specific guidance based on the security objectives and risk context.

## Usage and Access

This MCP server is designed to be used with MCP-compatible clients and requires:
- **Interactive Capabilities**: For conversational security guidance and tutoring
- **Basic Code Access**: For performing security analysis and review
- **Vulnerability Intelligence**: For accessing known vulnerability information
- **Security Tool Integration**: For orchestrating with specialized security analysis tools

## Contributing

We welcome contributions that help users make safer choices:
- **Security insights** from real tool assessments
- **Examples** of good and bad tool patterns
- **Teaching improvements** based on user feedback
- **New security checks** for emerging threats
- **Community intelligence** about tool reputation and quality

This tool is part of the broader Model Context Protocol Security initiative. We welcome contributions that enhance the security expertise and teaching capabilities:
- **Security Expertise**: Insights about MCP-specific threats and vulnerabilities
- **Teaching Methods**: Improved approaches for educating users about security
- **Analysis Techniques**: Better methods for evaluating MCP server security
- **Threat Intelligence**: Information about emerging threats and attack vectors
- **Security Best Practices**: Proven approaches for secure MCP server development and deployment

---

*Part of the [Model Context Protocol Security](https://modelcontextprotocol-security.io/) initiative - A Cloud Security Alliance community project.*

