
# ⭐ **1. What is an Annotation in Kubernetes?**

An **annotation** is a key/value pair attached to Kubernetes objects (Pods, Deployments, Namespaces, Services).

### ✔ Annotations are used to store *metadata*

Not used for scheduling
Not used for selectors
Not used for identifying objects

### ✔ They are meant for “extra instructions” or “hints” to tools

Examples:

* Monitoring agents
* Sidecar injectors
* Mutating webhooks
* Logging systems
* Service mesh controllers

### ✔ Annotation Example

```yaml
metadata:
  annotations:
    mycompany/log-level: "debug"
```

Annotations do **not affect Kubernetes behavior** directly, but external tools (like Dynatrace) read them.

---

# ⭐ **2. How Dynatrace Uses Annotations**

Dynatrace heavily uses annotations to control:

## **A. Selective OneAgent instrumentation (MOST IMPORTANT)**

Only pods with the annotation will be instrumented.

```yaml
metadata:
  annotations:
    oneagent.dynatrace.com/inject: "true"
```

### ✔ What this does:

* Tells Dynatrace Operator to inject OneAgent into this pod
* Enables **cloudNativeFullStack** or **application-only** monitoring for this pod
* Does it **without changing container images**

### ❌ If the annotation is missing → no injection

You can also disable instrumentation:

```yaml
metadata:
  annotations:
    oneagent.dynatrace.com/inject: "false"
```

---

## **B. Metadata Enrichment (Application tagging)**

Dynatrace can enrich metadata from annotations:

```yaml
metadata:
  annotations:
    tags.dynatrace.com/environment: "prod"
    tags.dynatrace.com/team: "frontend"
```

These appear in Dynatrace as:

* Tags
* Custom properties
* Process metadata

Useful for:

* Filtering
* Alerting
* Ownership assignment
* Cost allocation

---

## **C. Prometheus scraping via annotations**

If enabled in DynaKube:

```yaml
metrics:
  ingestPrometheusAnnotations: true
```

Then Pods can expose Prometheus metrics using annotations:

```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
```

Dynatrace will scrape these endpoints automatically.

---

# ⭐ **3. How to Enable Annotation-Based Injection in DynaKube**

In your DynaKube YAML:

```yaml
oneAgent:
  cloudNativeFullStack:
    useAnnotation: true
```

Now OneAgent is injected **ONLY** where annotation is present.

---

# ⭐ **4. How to Annotate Pods or Deployments**

### ✔ Method 1: Add annotation in Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    metadata:
      annotations:
        oneagent.dynatrace.com/inject: "true"
    spec:
      containers:
        - name: app
          image: python:3.9
```

Apply:

```bash
kubectl apply -f myapp.yaml
```

---

### ✔ Method 2: Add annotation to an existing Deployment

```bash
kubectl annotate deployment myapp oneagent.dynatrace.com/inject="true"
```

Restart pods:

```bash
kubectl rollout restart deploy myapp
```

---

### ✔ Method 3: Annotate Namespace

Apply injection to ALL pods in a namespace:

```bash
kubectl annotate namespace my-namespace oneagent.dynatrace.com/inject="true"
```

---

# ⭐ **5. How to Verify Dynatrace Annotation Injection is Working**

Pick a pod:

```bash
kubectl describe pod <pod-name>
```

Check for:

* `dynatrace-oneagent-init` container
* `dt-csi-volume` mounts
* Dynatrace environment variables
* `/opt/dynatrace/oneagent` directory

In Dynatrace UI →
**Kubernetes → Workloads → Processes → Services**
You should see:

* instrumented services
* traces
* CPU/memory
* network calls

---

# ⭐ **6. When Should You Use Annotations?**

Use annotation-based injection when:

✔ You want **only specific pods** to be instrumented
✔ You don’t want to monitor all workloads in a namespace
✔ You are in multi-tenant environments (AKS/EKS shared clusters)
✔ You need fine control for staging/production
✔ You want easy A/B rollout of Dynatrace injection

Annotations are the best way to control monitoring **per workload**.

---

# ⭐ Summary — What Annotations Do for Dynatrace

| Purpose                    | Annotation Example                       |
| -------------------------- | ---------------------------------------- |
| Enable OneAgent injection  | `oneagent.dynatrace.com/inject: "true"`  |
| Disable injection          | `oneagent.dynatrace.com/inject: "false"` |
| Add custom tags            | `tags.dynatrace.com/team: backend`       |
| Enable Prometheus scraping | `prometheus.io/scrape: "true"`           |
| Attach metadata            | `metadata.dynatrace.com/app: payment`    |

---


