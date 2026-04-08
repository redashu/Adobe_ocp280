# OpenShift Daily Ops Checklist

## 1. Daily Mindset

As an SRE/Admin, ask every day:

- Is the cluster healthy?
- Is it performing well?
- Is anything likely to break soon?

## 2. Daily Health Checklist (Golden Routine)

Run these checks in order.

### 2.1 Cluster Health (Top Priority)

Command:

```bash
oc get co
```

What to check:

| Field | Meaning |
|---|---|
| `Available=True` | Healthy |
| `Progressing=False` | Stable |
| `Degraded=False` | No issues |

Red flags:

- `Degraded=True`
- `Progressing=True` for a long time
- `Available=False`

Example healthy output:

```text
authentication   True   False   False   OK
etcd             True   False   False   OK
```

If something is wrong:

```bash
oc describe co <operator-name>
```

### 2.2 Node Health

Command:

```bash
oc get nodes
```

Expected:

- `STATUS=Ready`

Common issues:

| Status | Meaning |
|---|---|
| `NotReady` | Node problem |
| `SchedulingDisabled` | Node is cordoned |

Resource usage:

```bash
oc adm top nodes
```

Watch for:

- CPU > 80%
- Memory > 80%

Example:

```text
NAME        CPU(%)   MEMORY(%)
worker-1    75%      82%
```

### 2.3 Pod Health

Command:

```bash
oc get pods -A
```

Look for:

- `CrashLoopBackOff`
- `Error`
- `Pending`

Debug commands:

```bash
oc describe pod <pod>
oc logs <pod>
```

### 2.4 Cluster Version and Updates

Command:

```bash
oc get clusterversion
```

Check:

- `Available=True`
- `Progressing=False`
- Upgrade availability

Upgrade info:

```bash
oc adm upgrade
```

### 2.5 Events (Hidden Problems)

Command:

```bash
oc get events -A --sort-by=.lastTimestamp
```

This helps surface:

- Failures
- Scheduling issues
- Crashes

### 2.6 Storage Health

Commands:

```bash
oc get pvc
oc get pv
```

Watch for:

- Pending PVC
- Lost PV

### 2.7 Networking Check

Routes:

```bash
oc get routes -A
```

Ingress:

```bash
oc get pods -n openshift-ingress
```

### 2.8 etcd Health (Critical)

Command:

```bash
oc get pods -n openshift-etcd
```

Expected:

- All etcd pods should be running.

## 3. Weekly / Advanced Checks

Resource pressure:

```bash
oc adm top pods -A
```

Node disk usage:

```bash
oc debug node/<node>
chroot /host
df -h
```

Certificate activity:

```bash
oc get certificatesigningrequests
```

## 4. Real Incident Scenarios

### Case 1: Node NotReady

```bash
oc describe node <node>
journalctl -u kubelet
```

### Case 2: Pod CrashLoop

```bash
oc logs <pod>
oc describe pod <pod>
```

### Case 3: Operator Degraded

```bash
oc describe co <operator>
```

## 5. Scaling and Maintenance Tasks

Drain node for maintenance:

```bash
oc adm drain <node> --ignore-daemonsets --delete-emptydir-data
```

Uncordon node:

```bash
oc adm uncordon <node>
```

Scale workloads:

```bash
oc scale deployment my-app --replicas=3
```

## 6. Logs and Debugging

Pod logs:

```bash
oc logs <pod>
```

Node debug:

```bash
oc debug node/<node>
```

## 7. Security Checks

Who can create pods:

```bash
oc adm policy who-can create pods
```

SCC usage:

```bash
oc get scc
```

## 8. SRE Best Practices

Always monitor:

- CPU
- Memory
- Disk

Always check:

- Operators
- Nodes
- Events

Automate where possible:

- Prometheus alerts
- Log aggregation
- Backups (etcd + applications)

## Final Daily Routine (Copy/Paste)

```bash
oc get co
oc get nodes
oc adm top nodes
oc get pods -A
oc get events -A --sort-by=.lastTimestamp
oc get pvc
oc get clusterversion
```

## Final Mental Model

- Operators = brain health
- Nodes = infrastructure health
- Pods = workload health
- Events = hidden issues