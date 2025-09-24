---
title: Docker Compose Security - Comprehensive Check
version: 1.0
date: 2025-09-24
tags: [security, vulnerability, MCP, docker, compose, runtime, aivss]
aliases: [compose hardening]
status: draft
cwe: [250, 16, 693, 532, 922]
cwe-primary: 250
---

# Docker Compose Security - Security Assessment Check

## Security Assessment Metadata

**AIVSS Scoring Capability**: Full (CVSS + AARS)  
**Confidence Level**: High for compose file analysis  
**Complexity**: Moderate  
**Data Requirements**: `docker-compose*.yml` files and related docs
**Time to Complete**: 10–15m automated + 10–20m manual
**Risk Level**: Medium–High depending on privileges and mounts

## Security Purpose & Context

Compose controls networking, volumes, privileges, and environment for multi-service deployments. Misuse (privileged, host network, docker.sock mounts, secrets in env) increases risk.

## Security Assessment Criteria

### High Confidence Vulnerabilities
- `privileged: true`; `network_mode: host`  
- Mounts of `/var/run/docker.sock`, sensitive host paths, or broad device access  
- Secrets/keys in `environment:` or `.env` files referenced directly  
- `cap_add` adds powerful capabilities; absence of `cap_drop`  

### Medium Confidence Security Risks
- Missing resource limits (`deploy.resources.limits`)  
- Missing healthchecks for HTTP services  
- No `read_only: true` where feasible; writable bind mounts for app code

### Low Confidence / Expert Analysis Required
- Logging drivers and isolation options; external secrets stores

## Automated Security Assessment

```bash
rg -n "version:|services:|image:|build:" --glob "**/docker-compose*.yml" || true
rg -n "privileged:\s*true|network_mode:\s*host" --glob "**/docker-compose*.yml" || true
rg -n "/var/run/docker.sock|host.docker.internal|/etc|/var" --glob "**/docker-compose*.yml" || true
rg -n "cap_add:|cap_drop:" --glob "**/docker-compose*.yml" || true
rg -n "environment:|env_file:" --glob "**/docker-compose*.yml" || true
rg -n "read_only:\s*true|tmpfs:" --glob "**/docker-compose*.yml" || true
rg -n "healthcheck:|deploy:\s*\n\s*resources:\s*\n\s*limits:" --glob "**/docker-compose*.yml" || true
```

### AIVSS Risk Factor Assessment
```bash
# Autonomy/Tool Access: privileged/host networking/docker.sock mounts
rg -n "privileged:\s*true|network_mode:\s*host|/var/run/docker.sock" --glob "**/docker-compose*.yml" || true

# Data Exposure: secrets in env or env_file
rg -n "environment:|env_file:" --glob "**/docker-compose*.yml" || true

# Availability: no healthcheck/resource limits
rg -n "healthcheck:|deploy:\s*\n\s*resources:\s*\n\s*limits:" --glob "**/docker-compose*.yml" || true
```

## Manual Security Assessment

- [ ] Avoid privileged and host networking unless strictly required  
- [ ] Drop all capabilities; add only what is necessary  
- [ ] Avoid docker.sock/host mounts; use volumes with least privilege  
- [ ] Keep secrets out of env/env_file; use secret stores or mounted files  
- [ ] Set resource limits and healthchecks; consider `read_only: true` and `tmpfs`

## Secure vs. Vulnerable Patterns

### Excellent
```yaml
services:
  app:
    read_only: true
    tmpfs:
      - /tmp
    cap_drop: ["ALL"]
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
```

### Vulnerable
```yaml
services:
  app:
    privileged: true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - OPENAI_API_KEY=sk-...
```

## References
- Compose file reference: https://docs.docker.com/compose/compose-file/  
- Docker Engine security: https://docs.docker.com/engine/security/  
- Secrets management: https://docs.docker.com/engine/swarm/secrets/

---
*This check is part of the MCP Server Audit framework. Last updated: 2025-09-24*

