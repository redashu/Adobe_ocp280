# OpenShift SCC (Security Context Constraints)

## 1. What is SCC?

### Concept

SCC is a security policy that controls how pods run.

It defines:

- whether a container can run as root
- whether privileged mode is allowed
- whether host paths can be mounted
- what UID/GID the workload runs with

### Why OpenShift Introduced SCC

In older Kubernetes setups, pods could run as root by default, which is risky.

OpenShift enforces a more restricted, non-root model by default.

## 2. Where SCC Fits

### Admission Flow

`User/ServiceAccount -> Pod creation request -> SCC validation -> Allowed/Denied`

If SCC rules fail, you get a pod forbidden error (SCC violation).

## 3. Default SCCs in OpenShift

### List SCCs

```bash
oc get scc
```

### Common SCCs

| SCC | Purpose |
| --- | --- |
| restricted | Default non-root profile |
| anyuid | Allows root UID |
| privileged | Near full host-level access |
| hostnetwork | Allows host network usage |

### Most Important Default: restricted

- no root
- no privilege escalation
- random UID enforcement

## 4. How SCC is Applied

### Key Rule

SCC is typically evaluated against:

- ServiceAccounts (most common)
- Users (less common)

Flow:

`Pod -> ServiceAccount -> SCC -> Validation`

## 5. Check Which SCC a Pod Uses

```bash
oc describe pod <pod-name>
```

Look for:

```text
openshift.io/scc: restricted
```

## 6. Common Problem Scenario

### Pod Fails to Start

Example error:

```text
container has runAsUser=0 but SCC does not allow root
```

Common causes:

- container image expects root
- assigned SCC is restricted

## 7. Fix Options

### Option 1 (Best Practice): Run Non-Root

Update application security context:

```yaml
securityContext:
  runAsUser: 1001
```

### Option 2: Allow Root via anyuid (Use Carefully)

```bash
oc adm policy add-scc-to-user anyuid -z default
```

Result:

- pods using that ServiceAccount can run as root

Risk:

- less secure
- avoid in production unless required

## 8. Create Custom SCC (Advanced)

### Example SCC

```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: custom-scc
allowPrivilegedContainer: false
runAsUser:
  type: MustRunAsRange
```

Apply:

```bash
oc apply -f scc.yaml
```

Bind to a ServiceAccount:

```bash
oc adm policy add-scc-to-user custom-scc -z my-sa
```

## 9. Demo: Secure vs Insecure Deployment

### Insecure Example

```yaml
securityContext:
  runAsUser: 0
```

Result: blocked by restricted SCC.

### Secure Fix

```yaml
securityContext:
  runAsNonRoot: true
```

## 10. Real-World Use Cases

### Banking App

- Use restricted SCC
- Enforce non-root
- Enforce no privilege escalation

### Legacy App

- Some workloads may require root
- Use anyuid only where necessary

### DevOps Tools (for example, Jenkins)

- May need additional capabilities beyond restricted

## 11. Common Mistakes

Dangerous example:

```bash
oc adm policy add-scc-to-user privileged -z default
```

Avoid these mistakes:

- granting privileged SCC broadly
- skipping SCC validation before deployment
- running root containers without justification

## 12. Debug SCC Issues

Step 1: Check pod error details

```bash
oc describe pod <pod>
```

Step 2: Check available SCCs

```bash
oc get scc
```

Step 3: Check pod ServiceAccount

```bash
oc get pod <pod> -o yaml | grep serviceAccount
```

## Final Mental Model

SCC is a gatekeeper:

`Pod -> ServiceAccount -> SCC -> Allowed/Denied`