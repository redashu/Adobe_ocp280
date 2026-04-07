# OpenShift RBAC Guide

## 1. Why New Users Can Create Projects

### Root Cause

In OpenShift, there is a default ClusterRoleBinding that grants project creation rights:

- ClusterRole: self-provisioner
- Group: system:authenticated:oauth

### What This Means

Every authenticated user (including HTPasswd users) typically belongs to:

- system:authenticated
- system:authenticated:oauth

That default binding allows:

- Resource: projectrequests
- Verb: create

So users can run commands like:

```bash
oc new-project test
```

## 2. How to Check This Permission

### Step 1: Check ClusterRoleBinding

```bash
oc get clusterrolebinding self-provisioners -o yaml
```

Expected subject example:

```yaml
subjects:
  - kind: Group
    name: system:authenticated:oauth
```

### Step 2: Check What the Role Allows

```bash
oc describe clusterrole self-provisioner
```

Look for:

- Resources: projectrequests
- Verbs: create

### Step 3: Validate as a User

```bash
oc auth can-i create projectrequests --as=ashu
```

If the output is yes, the user can create projects.

## 3. How to Remove This Permission

### Recommended Approach (Production Safe)

Remove the default role from authenticated OAuth users:

```bash
oc adm policy remove-cluster-role-from-group \
  self-provisioner \
  system:authenticated:oauth
```

### Result

- Regular users cannot create projects.
- Admins can still create projects.

### Verify

```bash
oc auth can-i create projectrequests --as=ashu
```

Expected output:

```text
no
```

### Important Note

There are two similar groups:

- system:authenticated
- system:authenticated:oauth

OpenShift commonly uses system:authenticated:oauth for this behavior.

## 4. Roles vs ClusterRoles

### Difference

| Type | Scope |
| --- | --- |
| Role | Namespace |
| ClusterRole | Entire cluster |

### Explore Default Roles

```bash
oc get clusterroles
```

Important built-in roles include:

- admin
- edit
- view
- cluster-admin

Example:

```bash
oc describe clusterrole edit
```

Typical output includes actions like:

- Verbs: get, list, create, delete
- Resources: pods, services, and more

## 5. RoleBinding vs ClusterRoleBinding

### Difference

| Binding | Scope |
| --- | --- |
| RoleBinding | Namespace |
| ClusterRoleBinding | Cluster-wide |

### Rule of Thumb

Prefer RoleBinding whenever possible (least privilege).

## 6. Assign Permissions to a User

### Case 1: Namespace Access

```bash
oc new-project dev-project
oc adm policy add-role-to-user edit ashu -n dev-project
```

What happens:

- A RoleBinding is created.
- User gets namespace-level edit capabilities.

Verify:

```bash
oc get rolebinding -n dev-project
```

### Case 2: Read-only Access

```bash
oc adm policy add-role-to-user view ashu -n dev-project
```

### Case 3: Cluster Admin (Use Carefully)

```bash
oc adm policy add-cluster-role-to-user cluster-admin ashu
```

This should be avoided in production unless explicitly required.

## 7. Use Groups (Best Practice)

### Why

Instead of assigning roles directly to each user:

- Avoid: per-user role sprawl
- Prefer: group-based role assignment

### Create Group

```bash
oc adm groups new dev-team ashu user2
```

### Assign Role to Group

```bash
oc adm policy add-role-to-group edit dev-team -n dev-project
```

## 8. Custom Role (Advanced)

Use custom roles when built-in roles are too broad.

Example goal:

- Allow pod read access
- Deny delete operations

### Example Role Manifest

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pod-reader
  namespace: dev-project
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
```

Apply role:

```bash
oc apply -f role.yaml
```

Bind role to user:

```bash
oc create rolebinding pod-reader-binding \
  --role=pod-reader \
  --user=ashu \
  -n dev-project
```

## 9. How to Check Permissions

Use this command to test effective RBAC:

```bash
oc auth can-i create pods --as=ashu -n dev-project
```

This is the most practical command for real-world RBAC debugging.

## 10. Real-World RBAC Design

### Example Team Mapping

| Team | Role |
| --- | --- |
| Dev | edit |
| QA | view |
| DevOps | admin |
| Platform | cluster-admin |

### Recommended Pattern

User -> Group -> RoleBinding -> Role

## 11. Common Mistakes

- Direct user assignment everywhere (hard to manage)
- Overusing cluster-admin (security risk)
- Not using namespaces (no isolation)
- Over-permissioning (violates least privilege)
- Not validating with can-i checks

## 12. Debug Scenario

### Problem

User cannot deploy an app or gets forbidden errors.

### Check

```bash
oc auth can-i create deployment --as=ashu -n dev-project
```

Then verify:

- RoleBinding exists
- Correct namespace is used
- Correct role is assigned

## 13. Controlled Project Creation Strategy

### Core Rule

OpenShift does not automatically assign custom permissions per user. A scalable model is:

User -> Group -> RoleBinding/ClusterRoleBinding -> Role/ClusterRole

### Option 1: Group-Based Project Creators (Recommended)

Step 1: Remove default access

```bash
oc adm policy remove-cluster-role-from-group \
  self-provisioner \
  system:authenticated:oauth
```

Step 2: Create a dedicated group

```bash
oc adm groups new project-creators ashu user2
```

Step 3: Grant self-provisioner to that group

```bash
oc adm policy add-cluster-role-to-group \
  self-provisioner \
  project-creators
```

Result:

| User | Can Create Project |
| --- | --- |
| ashu | Yes |
| user2 | Yes |
| others | No |

### Option 2: Direct User Binding (Not Recommended)

```bash
oc adm policy add-cluster-role-to-user \
  self-provisioner \
  ashu
```

Drawbacks:

- Hard to manage at scale
- No group governance
- Difficult audits

## 14. Remove Project Creation for a Specific User

### If Using Group-Based Access

```bash
oc adm groups remove-users project-creators ashu
```

### If Using Direct User Binding

```bash
oc adm policy remove-cluster-role-from-user \
  self-provisioner \
  ashu
```

## 15. Verification Checklist

Check permission:

```bash
oc auth can-i create projectrequests --as=ashu
```

Check group:

```bash
oc get group project-creators -o yaml
```

Check bindings:

```bash
oc get clusterrolebinding | grep self-provisioner
```

## 16. Final Mental Model

- Authentication answers: Who are you?
- RBAC answers: What can you do?

Default OpenShift behavior:

- Everyone authenticated can create projects (unless changed)

Controlled enterprise behavior:

- Only selected groups can create projects