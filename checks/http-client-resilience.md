---
title: HTTP Client Resilience (Timeouts, Retries, Async) - Comprehensive Check
version: 1.0
date: 2025-09-24
tags: [security, vulnerability, MCP, availability, http, client, resilience, aivss]
aliases: [timeouts and retries]
status: draft
cwe: [400, 703, 307]
cwe-primary: 400
---

# HTTP Client Resilience - Security Assessment Check

## Security Assessment Metadata

**AIVSS Scoring Capability**: Full (CVSS + AARS)  
**Confidence Level**: High for code-level detection  
**Complexity**: Moderate  
**Data Requirements**: Source code (Python/JS) making outbound HTTP calls

## Security Purpose & Context

Missing timeouts, unbounded retries, and blocking I/O in async contexts lead to availability risks (DoS), hung sessions, and resource starvation. MCP servers commonly call external APIs; resilience is part of security.

## Security Assessment Criteria

### High Confidence Vulnerabilities
- Outbound HTTP calls without explicit timeouts  
- Async handlers calling sync/blocking HTTP libraries  
- Retrying non-idempotent operations by default  

### Medium Confidence Security Risks
- No backoff/jitter for retries; global retry loops  
- No rate/concurrency limits for tool calls  

## Automated Security Assessment

```bash
# Python examples
rg -n "requests\.(get|post|put|delete|request)\(" --glob "**/*.py" || true
rg -n "httpx\.(get|post|put|delete|request)\(" --glob "**/*.py" || true
rg -n "async\s+def|await\s+" --glob "**/*.py" || true

# Missing timeout heuristics (requests/httpx calls without timeout=)
rg -n "requests\.(get|post|put|delete|request)\([^)]*\)$" --glob "**/*.py" || true
rg -n "httpx\.(get|post|put|delete|request)\([^)]*\)$" --glob "**/*.py" || true

# Retries/backoff indications
rg -n "Retry|backoff|tenacity|urllib3.util.retry|httpx.Retry" --glob "**/*.py" || true
```

## Manual Security Assessment

- [ ] Enforce connect/read timeouts (e.g., 5s/20s) on all calls  
- [ ] Use async HTTP in async handlers or offload sync calls via threadpool  
- [ ] Add bounded retries for idempotent GETs with exponential backoff + jitter  
- [ ] Avoid retries for non-idempotent requests; implement idempotency keys where needed  
- [ ] Add per-user and global rate/concurrency limits for tools  
- [ ] Standardize error taxonomy; provide user-safe messages, log details internally

## Secure vs. Vulnerable Patterns

### Secure (requests)
```python
resp = session.request("GET", url, timeout=(5, 20))
```

### Secure (httpx async)
```python
async with httpx.AsyncClient(timeout=httpx.Timeout(20.0, connect=5.0)) as client:
    resp = await client.get(url)
```

### Vulnerable
```python
# No timeout, called from async handler
data = requests.get(url).json()
```

## References
- Requests timeouts: https://requests.readthedocs.io/en/latest/user/advanced/#timeouts  
- urllib3 Retry: https://urllib3.readthedocs.io/en/stable/reference/urllib3.util.html#urllib3.util.retry.Retry  
- httpx timeouts: https://www.python-httpx.org/advanced/#timeout-configuration  
- Exponential backoff (Google SRE): https://sre.google/sre-book/handling-overload/  
- Idempotency keys: https://developer.mozilla.org/docs/Glossary/Idempotent

---
*This check is part of the MCP Server Audit framework. Last updated: 2025-09-24*

