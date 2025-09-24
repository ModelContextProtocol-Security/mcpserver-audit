---
title: CI/CD Secrets & Pipeline Security - Comprehensive Check
version: 1.0
date: 2025-09-24
tags: [security, vulnerability, MCP, ci, cd, secrets, supply-chain, aivss]
aliases: [pipeline secrets, ci security]
status: draft
cwe: [798, 522, 532, 16]
cwe-primary: 798
---

# CI/CD Secrets & Pipeline Security - Security Assessment Check

## Security Assessment Metadata

**AIVSS Scoring Capability**: Full (CVSS + AARS)  
**Confidence Level**: Medium–High (files/logs reliable; hosted runners policies vary)  
**Complexity**: Moderate  
**Data Requirements**: CI config files (.github/workflows, .gitlab-ci.yml), scripts, docs, sample logs

## Security Purpose & Context

Pipelines often handle credentials for registries, clouds, and package managers. Leaks via env, logs, or image layers are common. This check focuses on secret handling, build-time vs runtime use, and artifact provenance.

## Security Assessment Criteria

### High Confidence Vulnerabilities
- Hardcoded credentials in CI files/scripts  
- Secrets passed as `ARG` then persisted as `ENV` or copied into images  
- Secrets printed to logs (lack of masking)  

### Medium Confidence Security Risks
- Tokens passed as generic build-args instead of BuildKit secrets  
- Missing scanning (image/deps), missing SBOM, unsigned artifacts  
- Over-broad token scopes; long-lived tokens; secrets in fork PRs

## Automated Security Assessment

```bash
# GitHub Actions / GitLab CI / general yaml
rg -n "name:|on:|jobs:|stages:|image:" --glob ".github/workflows/*.yml" --glob ".github/workflows/*.yaml" --glob "**/.gitlab-ci.yml" || true

# Secret anti-patterns
rg -n "(API|OPENAI|SECRET|TOKEN|PASSWORD|KEY)\s*[:=]\s*['\"][^'\"]+" --glob ".github/workflows/*" --glob "**/.gitlab-ci.yml" --glob "**/*.sh" || true
rg -n "echo\s+\$\{?(API|OPENAI|SECRET|TOKEN|PASSWORD|KEY)" --glob ".github/workflows/*" --glob "**/.gitlab-ci.yml" --glob "**/*.sh" || true
rg -n "--build-arg\s+\w*TOKEN=|--build-arg\s+\w*KEY=" --glob ".github/workflows/*" --glob "**/.gitlab-ci.yml" || true

# BuildKit secrets usage
rg -n "RUN\s+--mount=type=secret|id=\w+" --glob "**/Dockerfile*" || true

# Scanning/SBOM
rg -n "trivy|grype|docker\s+scout|sbom|cyclonedx" --glob ".github/workflows/*" --glob "**/.gitlab-ci.yml" || true
```

## Manual Security Assessment

- [ ] Replace hardcoded creds with CI secret stores; assert masking enabled  
- [ ] Use BuildKit secrets for build-time use; do not persist in layers  
- [ ] Avoid printing secrets; validate log redaction  
- [ ] Limit token scopes and lifetimes; disallow secrets in fork PR workflows  
- [ ] Add scanning (deps/images) and SBOM generation; store artifacts  
- [ ] Sign images/artifacts; verify on pull/deploy

## References
- GitHub Actions secrets: https://docs.github.com/actions/security-guides/using-secrets-in-github-actions  
- GitLab CI variables: https://docs.gitlab.com/ee/ci/variables/  
- BuildKit secrets: https://docs.docker.com/build/buildkit/secrets/  
- Docker Scout (scan/SBOM): https://docs.docker.com/scout/  
- OWASP Secrets Management Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html

---
*This check is part of the MCP Server Audit framework. Last updated: 2025-09-24*

