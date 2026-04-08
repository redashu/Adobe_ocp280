# OpenShift Service Mesh 3.x Installation Guide

To install Red Hat OpenShift Service Mesh 3.x, you no longer follow the strict "install 4 operators first" rule used in 2.x. The 3.x process is centered on the **Red Hat OpenShift Service Mesh 3 Operator**, which independently manages the Istio control plane.

## Step 1: Install the Service Mesh 3 Operator

1. In the OpenShift console, go to **Operators -> OperatorHub**.
2. Search for **Red Hat OpenShift Service Mesh 3 Operator**.
3. Choose the `stable` (or `stable-3.0`) channel.
4. Install in **All namespaces** (default: `openshift-operators`).

## Step 2: Create Required Projects

Unlike 2.x, you must create two separate projects:

- `istio-system` for the control plane
- `istio-cni` for the CNI plugin

```bash
oc new-project istio-system
oc new-project istio-cni
```

## Step 3: Deploy the Istio CNI Resource

`IstioCNI` is now a standalone component.

1. Go to **Installed Operators -> Red Hat OpenShift Service Mesh 3 Operator**.
2. Select the `istio-cni` project.
3. Click **Create IstioCNI**.
4. Use the default name: `default`.

## Step 4: Deploy the Istio Control Plane Resource

The `Istio` resource replaces the old `ServiceMeshControlPlane`.

1. In the same operator menu, select the `istio-system` project.
2. Click **Create Istio**.
3. Use the default YAML, or customize it with Discovery Selectors to control which namespaces are watched.
4. Wait until status is **Healthy**.

## Step 5: (Optional) Install Observability Components

For full dashboards and tracing, install these standalone operators as needed:

- **Kiali Operator**: visualization
- **Red Hat build of OpenTelemetry & Tempo Operator**: distributed tracing (replaces Jaeger/Elasticsearch)

## Step 6: Enable Sidecar Injection

In 3.x, add a project to the mesh by labeling its namespace:

```bash
oc label namespace <your_app_project> istio-discovery=enabled istio-injection=enabled
```

## Important Note

Do not run version 2.x and 3.x in the same cluster unless you are explicitly following a migration path, as the versions may conflict.



## OpenShift Route vs Istio Gateway

### 1) OpenShift Route (The Standard Way)

- **Layer**: Operates at the edge of the cluster
- **Handled by**: `ingress-controller` (HAProxy)
- **Pros**: Native to OpenShift, easy to use, integrates with AWS Load Balancers automatically
- **Cons**: It does not "know" about the Service Mesh. Traffic goes from the router directly to your Nginx pod, bypassing Istio features like mutual TLS (mTLS) and fine-grained traffic shifting.

### 2) Istio Gateway (The Mesh Way)

- **Layer**: Operates at the edge of the Service Mesh
- **Handled by**: `istio-ingressgateway` pod (Envoy)
- **Pros**: Supports advanced Istio features such as canary deployments (for example, send 10% traffic to v2), header-based routing, and full mTLS from gateway to pod
- **Cons**: More complex to set up. It usually requires an OpenShift Route to forward traffic from the AWS Load Balancer into the mesh.

### Comparison Table

| Feature | OpenShift Route | Istio Gateway + VirtualService |
| --- | --- | --- |
| Configuration | Simple YAML (`kind: Route`) | Two YAMLs (`Gateway` + `VirtualService`) |
| Traffic Shifting | Simple weight-based | Advanced (headers, cookies, regex) |
| mTLS Support | Edge/Re-encrypt/Passthrough | Full mesh mTLS (identity-based) |
| Observability | Basic HAProxy stats | Full Kiali/Jaeger distributed tracing |
| Path | Client -> Router -> Pod | Client -> Router -> Istio Ingress -> Pod |

### How They Work Together (Recommended Pattern)

In most OpenShift installations, you use both:

- **OpenShift Route (Passthrough)**: Send encrypted traffic directly to the Istio Ingress Gateway.
- **Istio Gateway**: Receive that traffic and use a `VirtualService` to route to the correct Nginx app version.

### Why Use Istio Gateway for Nginx?

If you only need a simple website, an OpenShift Route is enough. Use Istio Gateway when you need to:

- Inject faults (for example, simulate HTTP 500 errors for testing)
- Set request timeouts or retries
- View a real-time Kiali graph of requests to your Nginx app

In those cases, you need an Istio Gateway.