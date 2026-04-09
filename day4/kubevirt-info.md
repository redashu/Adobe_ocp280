# OpenShift Virtualization Operator Components

The core components of OpenShift Virtualization are described below.

## Core KubeVirt Components

### `virt-api`

A cluster-level component that provides an HTTP REST API entry point to manage VM and VM-related workflows in the cluster.

- Updates virtualization Custom Resource Definitions (CRDs).
- Handles defaulting and validation for VirtualMachineInstance (VMI) resources.

### `virt-controller`

A cluster-level component responsible for cluster-wide virtualization control.

- Manages lifecycle of pods associated with VMIs.
- Coordinates pod creation for VM objects.

### `virt-handler`

A host-level `DaemonSet` running on each node.

- Monitors VM object state changes.
- Executes node-level operations to match desired VM state.

### `virt-launcher`

The primary container in a VMI-associated pod.

- Provides cgroups and namespaces for the VMI process.
- Receives VM object context through `virt-handler` workflows.
- Uses a pod-local `libvirtd` instance to start the VMI.
- Monitors the VMI process until termination.

### `libvirtd`

Each VMI pod includes a `libvirtd` instance used by `virt-launcher` to manage VMI lifecycle operations.

## HyperConverged Operator (HCO)

OpenShift Virtualization requires the HyperConverged Operator (HCO), which bundles and manages related resources.

### HCO-Managed Resources

| Resource | Purpose |
| --- | --- |
| `deployment/hco-webhook` | Validates HyperConverged custom resource contents. |
| `deployment/hyperconverged-cluster-clidownload` | Provides `virtctl` binaries for direct cluster download. |
| `kubevirt/kubevirt-kubevirt-hyperconverged` | Contains required operators, CRs, and objects for OpenShift Virtualization. |
| `ssp/ssp-kubevirt-hyperconverged` | Scheduling, Scale, and Performance (SSP) custom resource. |
| `cdi/cdi-kubevirt-hyperconverged` | Containerized Data Importer (CDI) custom resource. |
| `networkaddonsconfig/cluster` | Custom resource managed by Cluster Network Addons Operator for network add-ons configuration. |