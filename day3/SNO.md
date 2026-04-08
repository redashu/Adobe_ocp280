# OKD SNO (Single Node OpenShift)

## What Is SNO?

SNO means Single Node OpenShift.

One VM runs:

- Control plane (master)
- Worker workloads

It is not HA, but it is perfect for:

- Labs
- Dev/Test
- Learning

## 1. Minimum Requirements

### VM Specs

| Resource | Minimum | Recommended |
|---|---|---|
| CPU | 4 | 8 |
| RAM | 16 GB | 24-32 GB |
| Disk | 120 GB | 200 GB |
| OS | Rocky Linux 8/9 | Yes |

### Network

- Static IP is recommended.
- Required DNS entries:
  - `api.okd.lab.example.com`
  - `*.apps.okd.lab.example.com`

Both should point to your VM IP.

## 2. Install Required Tools

Download OKD installer and client:

```bash
wget https://github.com/okd-project/okd/releases/latest/download/openshift-install-linux.tar.gz
wget https://github.com/okd-project/okd/releases/latest/download/openshift-client-linux.tar.gz
```

Extract and place binaries:

```bash
tar -xvf openshift-install-linux.tar.gz
tar -xvf openshift-client-linux.tar.gz
mv oc kubectl openshift-install /usr/local/bin/
```

Verify:

```bash
oc version
openshift-install version
```

## 3. Create Install Directory

```bash
mkdir okd-sno
cd okd-sno
```

## 4. Generate install-config.yaml

Command:

```bash
openshift-install create install-config
```

Choose:

- Platform: `None` (bare-metal)
- Base domain: `lab.example.com`
- Cluster name: `okd`
- Pull secret: optional for OKD (skip or provide)
- SSH key: your public key

Result:

- `install-config.yaml`

## 5. Modify for SNO

Open the config:

```bash
vi install-config.yaml
```

Set replicas as follows:

```yaml
controlPlane:
  replicas: 1

compute:
  replicas: 0
```

This makes it a single-node cluster.

## 6. Generate Manifests

```bash
openshift-install create manifests
```

Optional but important tweak for SNO:

- Keep masters schedulable.

```bash
vi manifests/cluster-scheduler-02-config.yml
```

Set:

```yaml
mastersSchedulable: true
```

## 7. Generate Ignition

```bash
openshift-install create ignition-configs
```

Generated files:

- `bootstrap.ign`
- `master.ign` (used for SNO)
- `worker.ign` (not used)

## 8. Serve Ignition over HTTP

Quick Python HTTP server:

```bash
python3 -m http.server 8080
```

Or with Apache:

```bash
cp master.ign /var/www/html/
systemctl start httpd
```

## 9. Boot VM with RHCOS

Recommended: ISO boot.

Example ISO download:

```bash
wget https://github.com/okd-project/okd/releases/.../rhcos-live.iso
```

Boot VM using the ISO, then add kernel arguments:

```text
coreos.inst.install_dev=/dev/sda
coreos.inst.ignition_url=http://<your-ip>:8080/master.ign
```

This tells the node to:

- Install OS
- Fetch ignition

## 10. Installation Flow

1. Boot ISO
2. Fetch ignition
3. Install RHCOS
4. Configure node
5. Start control plane
6. Cluster becomes ready

## 11. Monitor Installation

Run from installer node:

```bash
openshift-install wait-for bootstrap-complete --log-level=debug
openshift-install wait-for install-complete --log-level=debug
```

## 12. Access Cluster

Get credentials:

```bash
cat auth/kubeconfig
cat auth/kubeadmin-password
```

Login:

```bash
oc login https://api.okd.lab.example.com:6443
```

## 13. Verify Cluster

```bash
oc get nodes
oc get pods -A
```

Expected:

- Single node running master and worker roles.

## 14. Access Apps

Ensure DNS resolves:

- `*.apps.okd.lab.example.com -> VM IP`

Test route:

```bash
oc new-app nginx
oc expose svc nginx
```

## 15. Common Issues

- Ignition not reachable: HTTP server issue
- DNS not configured: API/routes inaccessible
- Not enough RAM: unstable cluster
- Wrong disk path: `/dev/sda` vs `/dev/vda`

## Final Mental Model

```text
install-config -> manifests -> ignition
                 ->
ISO boot -> fetch ignition
          ->
single node becomes full OpenShift cluster
```