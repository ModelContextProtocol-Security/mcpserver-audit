---
title: Docker Security - Comprehensive Check
version: 1.0
date: 2025-09-24
tags: [security, vulnerability, MCP, docker, container, supply-chain, aivss]
aliases: [docker hardening, container security]
status: draft
cwe: [250, 16, 532, 693, 922, 400]
cwe-primary: 250
---

# Docker Security - Security Assessment Check

## Security Assessment Metadata

**AIVSS Scoring Capability**: Full (CVSS + AARS)
**Confidence Level**: High for build-time checks; Medium for runtime posture
**Complexity**: Moderate  
**Data Requirements**: Dockerfile and related manifests; optional CI logs/scans
**Time to Complete**: 5–10m automated + 10–20m manual
**Reliability Notes**: File heuristics are reliable; runtime posture requires env docs/compose/K8s
**Risk Level**: Medium–High (varies by deployment)

## Security Purpose & Context

Docker configuration directly affects attack surface, privilege boundaries, reproducibility, and data exposure. Misconfigurations (root user, leaked secrets, unpinned bases) commonly lead to privilege escalation, supply-chain compromise, and operational risk. This check evaluates build-time and runtime posture for MCP servers and extensions.

**Security Impact**: High — privilege, data exposure, supply-chain, and availability risks converge in containerization.
**When to Use**: Any MCP server with Docker artifacts (Dockerfile, compose, K8s, CI)
**When to Skip**: Non-containerized projects (use OS/host hardening checks instead)
**Threat Context**: Malicious images, registry poisoning, lateral movement via privileged containers, data exfiltration via logs or baked secrets, DoS from unbounded resources

## Security Assessment Criteria

### High Confidence Vulnerabilities
- Root user in final image; no `USER` directive
- Secrets baked via `ENV`/persisted `ARG`; `.env` copied into image
- Use of `ADD` for remote URLs or auto-extract without justification
- Unpinned base images (e.g., `:latest`) and no digest pin for production images
- Missing `.dockerignore`, or context includes `.git`, `.env`, `node_modules`, `.venv`

### Medium Confidence Security Risks
- Single-stage image with build tools present in runtime layer
- No healthcheck for HTTP services; misleading EXPOSE/ports
- No resource limits guidance (CPUs/memory/pids)
- No SBOM/scanning in CI; no image provenance/signing
- Container runs with excessive capabilities; no `no-new-privileges`

### Low Confidence / Expert Analysis Required
- Seccomp/AppArmor/SELinux profiles in prod (needs deployment info)
- Justified use of shell-form ENTRYPOINT/CMD and `ADD` edge-cases
- TLS termination strategy and network policy handled externally

## Automated Security Assessment (For AI Assistants)

### High Confidence Vulnerability Detection
```bash
# Discover Docker-related files
rg -n "^(FROM|USER|RUN|COPY|ADD|EXPOSE|ENTRYPOINT|CMD|HEALTHCHECK)" -S --glob "**/Dockerfile*" || true
rg -n "version:|services:|image:|build:" --glob "**/docker-compose*.yml" || true
rg -n "apiVersion:|kind: Deployment|securityContext" --glob "**/*.{yaml,yml}" || true

# Dangerous patterns
rg -n "^FROM\s+\S+:latest\b" --glob "**/Dockerfile*" || true
rg -n "^ADD\s+" --glob "**/Dockerfile*" || true
rg -n "ENV\s+.*(KEY|SECRET|TOKEN|PASSWORD)" --glob "**/Dockerfile*" || true
rg -n "ARG\s+.*(KEY|SECRET|TOKEN|PASSWORD)" --glob "**/Dockerfile*" || true
rg -n "chmod\s+.*(777|o\+w|0o777)" --glob "**/Dockerfile*" || true

# Privilege & user checks
rg -n "^USER\s+" --glob "**/Dockerfile*" || true
rg -n "cap_add|cap-drop|privileged|security_opt" --glob "**/docker-compose*.yml" || true

# Hygiene
test -f .dockerignore && echo ".dockerignore present" || echo ".dockerignore missing"
rg -n "\.env|\.git|node_modules|\.venv|__pycache__" .dockerignore || true

# Build order heuristic (Python): install before copying src is suspicious
rg -n "pip install\s+\.|python -m pip install\s+\." --glob "**/Dockerfile*" | sort -n || true
rg -n "COPY\s+src/|COPY\s+\.\s+/app|WORKDIR" --glob "**/Dockerfile*" | sort -n || true

# Healthcheck presence
rg -n "^HEALTHCHECK\s+" --glob "**/Dockerfile*" || true
```

### AIVSS Risk Factor Assessment
```bash
# Indicators for AARS scoring (0.0, 0.5, 1.0)
# 1) Autonomy/Tool Access: Does the image allow broad host interaction (privileged/host mounts)?
rg -n "privileged: true|network_mode: host|/var/run/docker.sock" --glob "**/docker-compose*.yml" || true

# 2) Data Exposure: Secrets in ENV or baked files; logging of secrets
rg -n "ENV\s+.*(KEY|SECRET|TOKEN|PASSWORD)" --glob "**/Dockerfile*" || true

# 3) Availability: No healthcheck, no limits, unbounded processes
rg -n "pids_limit|deploy:|resources:" --glob "**/docker-compose*.yml" || true
```

**Automation Reliability**: High for syntactic patterns; medium where intent matters (EXPOSE vs stdio-only). Manually verify ambiguous cases.

## Manual Security Assessment (For Humans)

### Step-by-Step Security Checklist
- [ ] CVSS: Confirm non-root `USER` in final image; minimal capabilities at runtime
- [ ] CVSS: Verify no secrets baked in image; use BuildKit/runtime secrets
- [ ] CVSS: Ensure base image is pinned (tag + digest) and trusted; avoid `latest`
- [ ] CVSS: Use multi-stage; keep build tools out of runtime layer
- [ ] CVSS: `.dockerignore` excludes sensitive/large paths
- [ ] AARS: Assess privileged/host mounts; avoid unless strictly necessary
- [ ] AARS: Assess exposure of HTTP ports matches actual runtime; add HEALTHCHECK if HTTP server
- [ ] AARS: Resource limits strategy (CPUs/memory/pids); read-only rootfs if feasible
- [ ] AARS: Supply-chain defenses: SBOM, image scanning, and signing

### Key Security Questions to Ask
- High Confidence: "Does the final image run as non-root and drop unnecessary capabilities?"
- Medium Confidence: "Are any secrets introduced during build or persisted via ENV/ARG?"
- Expert Analysis: "What seccomp/AppArmor/SELinux profile is enforced in prod?"

### Human Security Verification Points
- Validate multi-stage boundaries and absence of compilers/tools in runtime
- Inspect CI for image scanning, SBOM generation, and signature verification
- Verify no `.env` or private keys are copied during build

## Secure vs. Vulnerable Patterns

### Excellent Security Examples
**Pattern**: Non-root image with multi-stage build and digest-pinned base  
**Why Secure**: Reduces privilege and supply-chain tampering risk; reproducible builds  
**AIVSS Context**: Low autonomy, lower data exposure, improved availability via smaller, clearer runtime  
**Confidence**: High  
**Example**:
```Dockerfile
FROM python:3.12-slim@sha256:<digest> AS builder
WORKDIR /build
COPY pyproject.toml .
RUN pip install --no-cache-dir build && python -m build

FROM python:3.12-slim@sha256:<digest>
RUN useradd -u 10001 -r app && mkdir -p /app && chown -R app:app /app
WORKDIR /app
COPY --chown=app:app --from=builder /build/dist/*.whl /tmp/
RUN pip install --no-cache-dir /tmp/*.whl && rm -rf /tmp/*.whl
USER app:app
HEALTHCHECK CMD python -c "import socket; s=socket.socket(); s.settimeout(2); s.connect(('127.0.0.1',8080)); s.close()" || exit 1
ENTRYPOINT ["spacebridge-mcp-server"]
```

### Vulnerable Security Examples
**Pattern**: Root user, unpinned base, secrets in ENV  
**Why Vulnerable**: High privilege, supply-chain drift, credential exposure  
**AIVSS Scoring**: Elevated autonomy/data exposure; increased availability risk  
**Confidence**: High  
**Example**:
```Dockerfile
FROM python:latest
WORKDIR /app
ENV OPENAI_API_KEY=sk-...  # BAD: secret in image
RUN pip install .
CMD spacebridge-mcp-server
```

### Ambiguous Security Cases
**Pattern**: `ADD` used for local archives  
**Context Factors**: May be acceptable for tar extraction with clear intent; prefer `COPY` otherwise  
**Assessment Approach**: Verify no remote URLs; document rationale

## Security Assessment Confidence Indicators

### High Confidence Security Results
**When to Trust**: Clear patterns (root user, ENV secrets, latest tags, no .dockerignore)  
**Minimum Security Data**: Dockerfile + compose/k8s manifest
**Strong Security Signals**: Digest pins, non-root user, multi-stage, SBOM/scanning in CI

### Low Confidence Security Warnings
**Security Assessment Red Flags**: Shell-form ENTRYPOINT/CMD with complex logic; implicit HTTP assumptions  
**Data Quality Issues**: Missing Dockerfile or externalized build scripts
**External Security Factors**: Runtime policies (seccomp/AppArmor) set outside repo

### Improving Security Assessment Confidence
**Additional Security Data**: CI pipelines, registry policies, runtime orchestration manifests  
**Cross-Validation**: Supply-chain tools (Trivy/Grype/Scout) logs, SBOMs  
**Threat Evolution Factors**: Base image CVEs, registry compromises, policy changes

## Security Justification & Evidence

### Why This Security Assessment Matters
**Impact on Security Posture**: Container config is a primary control surface for privilege, data, and resilience  
**Risk Implications**: Weak images ease exploitation and amplify blast radius  
**Threat Actor Interest**: Privileged containers, secrets in layers, vulnerable bases are high-value targets  
**AIVSS Relevance**: Affects autonomy/tool access, data exposure, and availability elements

### Supporting Security Evidence
**Research Basis**: Industry consensus on container hardening and supply-chain security  
**Security Standards**: 
- Dockerfile best practices (official)  
- Docker Engine security (official)  
- Rootless mode (official)  
- Seccomp/AppArmor (official)  
- BuildKit secrets (official)  
- .dockerignore (official)
**CVE References**: Frequent base image CVEs; dependency CVEs impact runtime
**Real-World Exploits**: Compromised base images, leaked ENV secrets, privileged containers used for host escapes

### Security Decision Weighting
**Critical Security Factor**: Root/privileged containers; secrets baked into image; untrusted/unpinned bases  
**Supporting Risk Factor**: Missing SBOM/scanning, missing healthchecks  
**Context Dependent Risk**: HTTP exposure, healthchecks, and resource limits vary by deployment

## Security Integration Guidance

### Using High Confidence Security Results
- Treat root user, baked secrets, and unpinned bases as immediate remediation items
- Weigh CVSS (privilege, exposure) with AARS (autonomy, data handling) per context

### Handling Low Confidence Security Results
- Request deployment manifests and policy docs to firm up runtime posture
- Seek security sign-off for intentional deviations (document exceptions)

### Combining with Other Security Checks
- Pair with credential/security checks; analyze CI/CD supply-chain; network binding checks
- Use SBOM and scanners to complement static Dockerfile analysis

## Common Security Assessment Pitfalls

### Security Assessment Errors
- Assuming EXPOSE implies active HTTP server (may be stdio-only)
- Overlooking `.dockerignore` leading to context leaks (e.g., `.env`)
- Installing app before copying source, producing stale/empty installs

### Security Context Mismatches
- Dev containers are looser; prod needs strict policies  
- Some orchestrators enforce policies outside Dockerfile

### Security Confidence Misjudgments
- Over-trusting base images without digest/signature  
- Under-valuing SBOM/scanning as “optional”

## Basic Remediation Guidance

### Critical Issues Found
- Root or privileged runtime: add non-root `USER`, drop capabilities, avoid `--privileged`
- Secrets in image: remove, use BuildKit/runtime secrets; rotate exposed keys
- Unpinned/untrusted base: pin to digest; use official/maintained source

### Next Steps for Fixing
**For detailed remediation guidance**: Use mcpserver-builder for hardened Dockerfile and compose/K8s examples  
**For secure deployment**: Coordinate with mcpserver-operator for runtime flags, profiles, and limits  
**Immediate priority**: Address root/privileged, secrets, and base pinning first

## Integration with MCP Security Ecosystem

- Cross-reference with `vulnerability-db` for container patterns  
- Document findings in `audit-db`  
- Flag for enhanced monitoring when privileged containers or missing scans are found

## AIVSS & CVSS Reference Materials

For AI assistants conducting security assessments, these markdown versions and official docs are recommended:

- AIVSS (AI Vulnerability Scoring System): https://github.com/CloudSecurityAlliance-DataSets/dataset-public-laws-regulations-standards/tree/main/tools-resources/owasp.org/AIVSS
- CVSS v4.0 (Common Vulnerability Scoring System): https://github.com/CloudSecurityAlliance-DataSets/dataset-public-laws-regulations-standards/tree/main/tools-resources/first.org/CVSS

Official Docker References (Key):
- Dockerfile best practices: https://docs.docker.com/build/guide/dockerfile-best-practices/
- .dockerignore and build context: https://docs.docker.com/build/building/context/#dockerignore-files
- Docker Engine security: https://docs.docker.com/engine/security/
- Rootless mode: https://docs.docker.com/engine/security/rootless/
- Seccomp: https://docs.docker.com/engine/security/seccomp/
- AppArmor: https://docs.docker.com/engine/security/apparmor/
- BuildKit secrets: https://docs.docker.com/build/buildkit/secrets/
- Docker Scout (scan/SBOM): https://docs.docker.com/scout/

---

*This check is part of the MCP Server Audit security assessment framework. Last updated: 2025-09-24*

