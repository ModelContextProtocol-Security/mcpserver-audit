# Draft: Description-First AI Prompt/Check Format Codex

## Executive Summary
- We defined a simple, markdown-only, Description-first format for all AI-facing artifacts (prompts, checks, future exceptions).
- The first thing the AI sees is an Instruction sentence under a unified Description section, followed by a stable set of bullets (Purpose, When To Use, Inputs, Outputs, Dependencies, Safety Notes, Related, Quick Start).
- This balances human readability with predictable structure for future indexing, without introducing YAML/frontmatter or tooling friction.

## Why We’re Doing This
- Clarity for humans: Writers and readers can quickly understand each artifact’s intent, use cases, and outputs.
- Predictability for AIs: A consistent Instruction sentence sets role, action, and deliverables; bullets provide structured context the AI can follow.
- Low overhead: Pure Markdown, minimal ceremony, no new dependencies.
- Future-friendly: The stable bullet keys enable optional, later auto-indexing (JSON/Markdown resource lists) if we choose, without changing authored content.

## Core Standard
- Placement: Immediately after the H1 title of the file, add a Description section.
- First line: Instruction sentence (imperative; sets the AI’s role, what to do, and what to produce).
- Then the bullet list, using these stable keys:
  - Purpose: 1–2 sentences describing what the artifact is for.
  - When To Use: Concrete scenarios where it applies.
  - Inputs: What the AI (or user) needs to run it effectively.
  - Outputs: The artifacts or structured results expected.
  - Dependencies: Other files/tools/commands it references; keep minimal.
  - Safety Notes: Privacy, data-sharing, non-destructive guidance, boundaries.
  - Related: Links to relevant checks/prompts/exceptions in-repo.
  - Quick Start: 1–2 lines that show the minimal invocation or first steps.

Notes
- Keep the Instruction sentence imperative and explicit (role + action + outputs).
- Keep bullet content concise, concrete, and scannable.
- Use relative repo paths in Related for easy navigation.

## Canonical Skeleton
```
# Description
Instruction: [One sentence telling the AI what role to adopt, what to do, and what to produce.]

- Purpose: [1–2 sentences]
- When To Use: [Scenarios]
- Inputs: [Repo/context needed]
- Outputs: [Artifacts/results]
- Dependencies: [Files/tools/commands]
- Safety Notes: [Privacy and non-destructive guidance]
- Related: [Repo-relative links]
- Quick Start: [1–2 lines to run/invoke]
```

## Asset Types and How to Write Them

### Prompts (mcpserver-audit/prompts/*.md)
Use for conversational guidance and orchestration. The AI must teach, ask questions, and produce artifacts.

Suggested Description block for prompts:
- Instruction: “Act as an MCP server security auditor; guide the user through [scope], ask Socratic questions, and produce [outputs].”
- Purpose: Explain the audit’s intent (education + outputs).
- When To Use: Comprehensive vs targeted; production readiness; compliance.
- Inputs: Server name, repo path, constraints, deployment context.
- Outputs: SECURITY-ISSUE-NNN.md, TODO.md, [server]-audit-report-detailed.md.
- Dependencies: checks/ directory; specific checks often used.
- Safety Notes: Provide guidance only; do not execute code or use network.
- Related: Link to checks and sibling prompts.
- Quick Start: “read prompts/security-assessment.md and …”.

Example (prompt):

# Description
Instruction: Act as an MCP server security auditor; guide the user through a full audit, ask Socratic questions, and produce issue files, a remediation TODO, and a detailed report.

- Purpose: Full code security audit with CWE/AIVSS scoring and actionable remediation.
- When To Use: Production readiness reviews, compliance assessments, or new server vetting.
- Inputs: Server name, repo path, deployment context (Local/Remote/Enterprise), time constraints.
- Outputs: SECURITY-ISSUE-NNN.md files, TODO.md, [server]-audit-report-detailed.md.
- Dependencies: checks/ directory; targeted-evaluation.md for focused follow-ups.
- Safety Notes: Provide guidance only; do not execute code or access external networks.
- Related: prompts/targeted-evaluation.md; checks/docker-security.md; checks/http-client-resilience.md
- Quick Start: read prompts/security-assessment.md and conduct a complete security audit of MCP server [SERVER_NAME]

### Checks (mcpserver-audit/checks/*.md)
Use for focused, repeatable security assessments of a particular area (e.g., Docker, K8s, CI secrets).

Suggested Description block for checks:
- Instruction: “Act as a [domain] security auditor; apply this check to the provided repo, ask clarifying questions, then report findings with CWE/AIVSS and remediation.”
- Purpose: What it detects and why it matters.
- When To Use: Trigger conditions (present files/tech), pre-release, pre-deploy.
- Inputs: File patterns to scan; optional artifacts (CI logs/SBOMs).
- Outputs: Findings with code refs, CWE/AIVSS, remediation steps.
- Dependencies: ripgrep suggested commands; no network/builds required.
- Safety Notes: Static analysis only; no secrets in logs.
- Related: Neighbor checks that complement this one.
- Quick Start: 1–2 rg commands to start.

We created new checks aligned to this approach:
- checks/docker-security.md
- checks/k8s-security.md
- checks/compose-security.md
- checks/ci-secrets.md
- checks/http-client-resilience.md

### Exceptions (mcpserver-audit/exceptions/*.md) – future
Use for documented, time-bounded risk acceptance or suppression of specific check findings.

Suggested Description block for exceptions:
- Instruction: “Treat this as an explicit, time-bounded risk acceptance; apply only within the defined scope and record compensating controls.”
- Purpose: State the exception clearly.
- When To Use: Conditions where deviation is acceptable.
- Inputs/Scope: Paths/tags/check IDs this exception applies to.
- Outputs: Rationale, expiry date, compensating controls, owner/approval.
- Safety Notes: State blast radius and rollback plan; do not extend to prod unless approved.
- Related: Checks and prompts it modifies.
- Quick Start: “Apply this exception when [conditions]; review on [expires].”

## Safety and Ethics
- Data handling: Call out when any step could expose data (LLM calls, logs). Default to “no external network, no execution” unless explicitly permitted.
- Non-destructive defaults: Checks are static analysis; do not run builds or write artifacts outside designated audit outputs.
- Clarity about uncertainty: Encourage “ask clarifying questions before acting” to reduce missteps.

## Validation Checklist (for authors and reviewers)
- Instruction line is present, imperative, and accurate.
- All eight bullets exist and contain specific, scannable content.
- Safety Notes included when relevant (LLM, network, secrets, builds).
- Related links resolve to real files in the repo.
- Quick Start examples are actionable and aligned with the file’s purpose.

## Discovery and Future Indexing (optional; not required now)
- Because Description blocks use stable keys, we can later add an indexer script to build:
  - resources/INDEX.md (human overview) and resources/index.json (machine-readable).
  - Group by kind (prompt/check/exception), tags, and CWE where applicable.
- This is intentionally out of scope for now to keep authoring friction low.

## Migration Plan (incremental)
1) Add Description blocks to the new checks (done) and to existing prompts:
   - prompts/main-prompt.md
   - prompts/security-assessment.md
   - prompts/targeted-evaluation.md
2) Keep the rest of each file intact; only prepend the new Description section under the H1.
3) When creating new assets, authors must include the Description block.
4) (Optional) Add a short CONTRIBUTING note and reviewer checklist to enforce consistency.

## Examples to Retrofit (recommended wording)
- Prompts/security-assessment.md Instruction:
  - “Act as an MCP server security auditor; guide the user through a full audit, ask Socratic questions, and produce issue files, a remediation TODO, and a detailed report.”
- Checks/docker-security.md Instruction:
  - “Act as a Docker security auditor; apply this check to the provided repo, ask clarifying questions as needed, then report findings and remediation with CWE/AIVSS context.”

## Decision Log (what we chose and did not choose)
- Chosen: Markdown-only, no YAML/frontmatter. Reason: simplicity, consistency with repo style, less friction.
- Chosen: Instruction + bullets under a Description section as the standard block.
- Deferred: Auto-indexing generator script and exceptions/ directory; we may add later.
- Not chosen: Embedding executable commands/scripts in prompts (we keep suggested rg commands as examples, not mandatory tooling steps).

## References & Inspirations
- MCP init-style messaging adapted into a concise “Instruction” line for AI.
- Docker Official Docs (for check contents and references):
  - Dockerfile best practices: https://docs.docker.com/build/guide/dockerfile-best-practices/
  - .dockerignore & build context: https://docs.docker.com/build/building/context/#dockerignore-files
  - Engine security overview: https://docs.docker.com/engine/security/
  - BuildKit secrets: https://docs.docker.com/build/buildkit/secrets/
- Kubernetes Security references:
  - Security Context: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
  - Pod Security Admission: https://kubernetes.io/docs/concepts/security/pod-security-admission/
- Secrets management guidance:
  - OWASP Secrets Management Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html

## Next Steps
- Confirm this Description-first standard and begin retrofitting key files.
- Optionally draft a short CONTRIBUTING section describing the format and review checklist.
- Revisit indexing once a few cycles of authoring show stable patterns and needs.

