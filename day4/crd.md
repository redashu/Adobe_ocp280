# Custom Resource Definitions (CRDs)

A CRD lets you extend Kubernetes/OpenShift APIs with your own resource types. Just like built-in resources such as `ConfigMap`, `Secret`, or `Deployment`, you can define custom types for your platform.

Think of it this way:

- `ConfigMap`: built-in resource for key-value configuration.
- `MyCRD`: your custom resource type with your own schema.

## How It Works

```text
CRD (schema definition) -> CR (instance of that schema) -> consumed by workloads
```

A CRD teaches the API server about a new type. A Custom Resource (CR) is an actual object of that type.

## Step-by-Step: Use a CRD for App Environment Variables

## 1. Define the CRD

Create `app-config-crd.yaml`:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: appconfigs.myorg.io
spec:
  group: myorg.io
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                envVars:
                  type: object
                  additionalProperties:
                    type: string
  scope: Namespaced
  names:
    plural: appconfigs
    singular: appconfig
    kind: AppConfig
    shortNames:
      - ac
```

Apply and verify:

```bash
oc apply -f app-config-crd.yaml
oc get crd appconfigs.myorg.io
```

## 2. Create a Custom Resource (CR)

Create `my-app-config.yaml`:

```yaml
apiVersion: myorg.io/v1
kind: AppConfig
metadata:
  name: my-app-config
  namespace: default
spec:
  envVars:
    APP_ENV: "production"
    LOG_LEVEL: "debug"
    DB_HOST: "postgres.default.svc"
    MAX_RETRIES: "3"
```

Apply and verify:

```bash
oc apply -f my-app-config.yaml
oc get appconfig
oc describe appconfig my-app-config
```

## 3. Bridge CR Data to Pod Environment

Important: Pods cannot natively read CRs the same way they read ConfigMaps. You need a bridge.

### Option A: Sync CR to ConfigMap (simplest)

One-time manual sync for testing:

```bash
oc get appconfig my-app-config -o jsonpath='{.spec.envVars}' \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
pairs = '\n'.join(f'  {k}: \"{v}\"' for k,v in data.items())
print(f'apiVersion: v1\nkind: ConfigMap\nmetadata:\n  name: my-app-env\ndata:\n{pairs}')
" | oc apply -f -
```

Then load this ConfigMap from your Pod (Step 4).

### Option B: Init Container Pulls CR at Startup

```yaml
initContainers:
  - name: fetch-config
    image: bitnami/kubectl
    command:
      - sh
      - -c
      - |
        oc get appconfig my-app-config -o jsonpath='{.spec.envVars.APP_ENV}' > /config/APP_ENV
    volumeMounts:
      - name: config-vol
        mountPath: /config
```

## 4. Pod Using the Synced ConfigMap

Create `app-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "env && sleep 3600"]
      envFrom:
        - configMapRef:
            name: my-app-env
```

Apply and validate:

```bash
oc apply -f app-pod.yaml
oc exec my-app -- env | grep -E "APP_ENV|LOG_LEVEL|DB_HOST"
```

## Summary

| Step | What You Do |
| --- | --- |
| CRD | Teach OpenShift about your new type (`AppConfig`) |
| CR | Create an instance with actual configuration values |
| Bridge | Sync CR to ConfigMap (script, operator, or init container) |
| Pod | Read env vars from ConfigMap using `envFrom` |

Most teams automate the bridge step by building an Operator (for example with Operator SDK or Kopf).