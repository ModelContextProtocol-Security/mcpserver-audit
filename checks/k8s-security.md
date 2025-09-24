---
title: Kubernetes Security - Comprehensive Check
version: 1.0
date: 2025-09-24
tags: [security, vulnerability, MCP, kubernetes, k8s, container, runtime, aivss]
aliases: [k8s hardening, kubernetes security]
status: draft
cwe: [250, 16, 693, 732, 922]
cwe-primary: 250
---

# Kubernetes Security - Security Assessment Check

## Security Assessment Metadata

**AIVSS Scoring Capability**: Full (CVSS + AARS)
**Confidence Level**: Medium–High (manifests reliable; cluster policy requires context)
**Complexity**: Moderate  
**Data Requirements**: K8s manifests/Helm charts; optional cluster policy docs
**Time to Complete**: 10–20m automated + 20–30m manual
**Reliability Notes**: Static manifest analysis is strong; runtime/PSA enforcement needs env details
**Risk Level**: Medium–High (privilege, data, availability)

## Security Purpose & Context

Kubernetes securityContext and pod-level controls govern privilege boundaries for workloads. Misconfigurations (root users, added capabilities, writable rootfs, host mounts) significantly increase the blast radius of a compromise.

**Security Impact**: High — privilege escalation and data exposure  
**When to Use**: Any repo deploying MCP servers to Kubernetes  
**Threat Context**: Lateral movement, host escape attempts, secret exposure via mounts/env

## Security Assessment Criteria

### High Confidence Vulnerabilities
- `securityContext.runAsNonRoot: false` or absent with root images
- `allowPrivilegeEscalation: true` (or absent defaults to true for some runtimes)
- `privileged: true`, `hostNetwork: true`, `hostPID: true`, `hostIPC: true`
- Added Linux capabilities (e.g., `CAP_SYS_ADMIN`) without justification
- `readOnlyRootFilesystem: false` for services that can run read-only
- HostPath mounts, Docker socket mounts, broad emptyDir usage for persistent data

### Medium Confidence Security Risks
- Missing seccomp/AppArmor profiles  
- Secrets provided via env vars instead of volume mounts with least-privileged access  
- Lack of resource limits/requests and liveness/readiness probes

### Low Confidence / Expert Analysis Required
- Pod Security Admission (PSA) level enforcement (Baseline/Restricted)  
- NetworkPolicies and egress restrictions  
- ServiceAccount scopes and RBAC bindings

## Automated Security Assessment (For AI Assistants)

### High Confidence Vulnerability Detection
```bash
# Find Kubernetes manifests
rg -n "apiVersion:|kind:\s+(Deployment|StatefulSet|DaemonSet|Pod|Job|CronJob)" --glob "**/*.{yaml,yml}" || true

# SecurityContext checks
rg -n "runAsNonRoot:\s*false|privileged:\s*true|allowPrivilegeEscalation:\s*true" --glob "**/*.{yaml,yml}" || true
rg -n "hostNetwork:\s*true|hostPID:\s*true|hostIPC:\s*true" --glob "**/*.{yaml,yml}" || true
rg -n "capabilities:\s*\n\s*add:|cap_add" --glob "**/*.{yaml,yml}" || true
rg -n "readOnlyRootFilesystem:\s*false" --glob "**/*.{yaml,yml}" || true
rg -n "hostPath:|/var/run/docker.sock" --glob "**/*.{yaml,yml}" || true

# Probes and resources
rg -n "livenessProbe:|readinessProbe:" --glob "**/*.{yaml,yml}" || true
rg -n "resources:\s*\n\s*limits:|resources:\s*\n\s*requests:" --glob "**/*.{yaml,yml}" || true

# Secrets via env
rg -n "env:\n\s*-\s*name:.*(KEY|SECRET|TOKEN|PASSWORD)" --glob "**/*.{yaml,yml}" || true
```

### AIVSS Risk Factor Assessment
```bash
# Autonomy/Tool Access (1.0 if privileged/host* true)
rg -n "privileged:\s*true|host(Network|PID|IPC):\s*true" --glob "**/*.{yaml,yml}" || true

# Data Exposure (0.5–1.0 if secrets via env; 1.0 if hostPath/socket mounted)
rg -n "hostPath:|/var/run/docker.sock|env:\n\s*-\s*name:.*(KEY|SECRET|TOKEN|PASSWORD)" --glob "**/*.{yaml,yml}" || true

# Availability (0.5–1.0 if no probes/limits)
rg -n "livenessProbe:|readinessProbe:|resources:\s*(limits|requests)" --glob "**/*.{yaml,yml}" || true
```

**Automation Reliability**: High for presence/absence; confirm necessity and environment policies manually.

## Manual Security Assessment (For Humans)

### Step-by-Step Security Checklist
- [ ] Non-root user enforced (`runAsNonRoot: true`; `runAsUser` set)  
- [ ] `allowPrivilegeEscalation: false`; `privileged: false`; no host* flags  
- [ ] Drop capabilities; add none unless justified  
- [ ] `readOnlyRootFilesystem: true` with tmpfs/emptyDir for needed writes  
- [ ] Probes: liveness/readiness; Resources: limits/requests  
- [ ] Secrets via volumes (mounted files), not env; restrict access  
- [ ] Seccomp/AppArmor/SELinux profiles set  
- [ ] PSA enforced (Baseline/Restricted); NetworkPolicies in place

### Key Security Questions to Ask
- “Why does this workload need elevated privileges or host access?”  
- “Can the rootfs be read-only with a writable temp directory?”  
- “How are secrets rotated and mounted at runtime?”

## Secure vs. Vulnerable Patterns

### Excellent Security Example
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 10001
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop: ["ALL"]
```

### Vulnerable Security Example
```yaml
securityContext:
  privileged: true
  allowPrivilegeEscalation: true
  capabilities:
    add: ["SYS_ADMIN"]
```

## Security Justification & Evidence

**Why It Matters**: K8s is the enforcement point for least privilege and isolation. Weak settings enable container escapes and lateral movement.

**CWE Mapping**:  
- CWE-250 (Unnecessary Privileges), CWE-16 (Configuration), CWE-693 (Missing Protection), CWE-732 (Permissions), CWE-922 (Transport/Secrets)

## References
- Pod Security Admission (PSA): https://kubernetes.io/docs/concepts/security/pod-security-admission/  
- Configure a Security Context: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/  
- Pod Security Standards: https://kubernetes.io/docs/concepts/security/pod-security-standards/  
- Seccomp: https://kubernetes.io/docs/tutorials/security/seccomp/  
- AppArmor: https://kubernetes.io/docs/tutorials/security/apparmor/  
- Secrets: https://kubernetes.io/docs/concepts/configuration/secret/

---
*This check is part of the MCP Server Audit framework. Last updated: 2025-09-24*

