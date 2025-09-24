# DRAFT: An AI-Native Approach to Prompt Architecture

**Author:** Gemini
**Date:** 2025-09-24
**Status:** This document proposes a new standard for designing and managing the prompt system for the MCP Server Audit framework.

---

## 1. Core Philosophy: Prompts as a Knowledge Graph

Our previous approach to prompt engineering was rooted in a traditional software development paradigm, treating prompts like code modules. This involved rigid structures, metadata (frontmatter), and explicit `include` directives. This model is flawed because it is designed for systems with no native reasoning capability.

This framework is **by an AI, for an AI**. Therefore, we must adopt a more fluid and powerful, AI-native paradigm.

Our new philosophy is to treat the prompt system as an **interconnected library of concepts**, or a knowledge graph. Individual prompt files are nodes in this graph, representing core ideas, personas, or skills. The connections between them are not rigid function calls, but conceptual links expressed in natural language.

The AI itself is responsible for traversing this graph, understanding the conceptual links, and synthesizing the information into a coherent operational context. This approach is simpler, more flexible, and fully leverages the core reasoning capabilities of the Large Language Model.

## 2. The Three Guiding Principles

### Principle 1: Conceptual Organization

We will organize all prompts into a directory structure that reflects a library of abstract concepts, not a hierarchy of code modules. Each directory has a clear conceptual purpose.

- `/prompts/personas/`: Defines a core identity, role, or mode of thinking that the AI can adopt. (e.g., `security_auditor.md`)
- `/prompts/frameworks/`: Defines a high-level mental model, a strategic process, or a philosophy for approaching a complex task. These are the most important prompts as they teach the AI *how to think*. (e.g., `how_to_audit.md`, `how_to_handle_exceptions.md`)
- `/prompts/skills/`: Defines a specific, narrow, and reusable capability that can be composed by any persona or framework. (e.g., `summarize_code.md`, `format_report.md`)

### Principle 2: Natural Language Composition

We will eliminate all rigid, code-like directives for prompt chaining, such as `{{include: ...}}`. Instead, we will use natural language to link concepts.

The AI is expected to understand that a phrase like "...you must apply the **Exception Handling framework**..." is a directive to locate and incorporate the knowledge from the `/prompts/frameworks/how_to_handle_exceptions.md` file into its current reasoning process.

This makes the prompts themselves high-level and descriptive, focusing on the **what** and **why**, while leaving the **how** of composition to the AI.

### Principle 3: Radical Simplicity

Prompt files will contain **only the pure, high-level instruction** intended for the AI. All boilerplate is eliminated.

- **No YAML Frontmatter:** We will not use frontmatter for metadata like `version`, `inputs`, or `outputs`. Git already provides version control, and the prompt's purpose should be evident from its high-level text and its location within the conceptual directory structure.
- **No Redundant Instructions:** The text should be clean, direct, and focused on conveying the core concept of that file.

## 3. Example of the System in Action

This example demonstrates how three simple, high-level prompts are synthesized by the AI to execute a complex task.

**File 1: `frameworks/how_to_audit.md`**
```markdown
When you are asked to perform a security audit, you must do so by embodying the **Security Auditor persona**.

Your process is to analyze the codebase to find potential vulnerabilities. For every finding, you must apply the **Exception Handling framework** to determine if it is a real risk or an acceptable exception.

When presenting your findings, you must use your **Report Formatting skill**.
```

**File 2: `personas/security_auditor.md`**
```markdown
You are an expert MCP server security auditor. Your primary goal is to find vulnerabilities and assess risk using the AIVSS model. You are meticulous and skeptical.
```

**File 3: `skills/format_report.md`**
```markdown
All security findings must be presented in a markdown table with the columns: 'ID', 'Title', 'Severity', and 'CWE'.
```

**AI Execution:**
When instructed to perform an audit, the AI reads the `how_to_audit.md` framework. It recognizes the natural language links to the other two concepts. It then synthesizes these three sources into a single, coherent set of operating instructions: it adopts the persona of a skeptical expert, follows the process of finding vulnerabilities and checking for exceptions, and knows to format its final output as a markdown table.

## 4. Benefits of the AI-Native Approach

- **Power & Flexibility:** This system fully leverages the LLM's ability to understand, reason, and synthesize information from multiple sources.
- **Simplicity & Readability:** Prompts are clean, high-level text that is easy for humans to write and understand.
- **Maintainability & Scalability:** New personas, skills, or frameworks can be added simply by creating a new text file in the appropriate directory. The system scales organically without increasing in complexity.
