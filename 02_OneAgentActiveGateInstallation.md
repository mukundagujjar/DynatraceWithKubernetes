---

# **1️⃣ Confirm the operator is fully healthy**

Run:

```bash
kubectl -n dynatrace get pods
```

You must see these pods in **Running** state:

| Component                  | Required? | Purpose                                    |
| -------------------------- | --------- | ------------------------------------------ |
| `dynatrace-operator-xxxxx` | ✔         | Core controller                            |
| `dynatrace-webhook-xxxxx`  | ✔         | Validates DynaKube CR  |
| `dynatrace-csi-driver-*`   | Optional  | CSI driver for code modules                |

If any pod is **CrashLoop**, send logs:

```bash
kubectl -n dynatrace logs <pod-name>
```

---

# **2️⃣ apply your DynaKube configuration**

Now that webhook is running:

```bash
kubectl apply -f dynakube.yaml
```

This time it should create a **DynaKube CR** successfully.

Verify:

```bash
kubectl -n dynatrace get dynakube
```

It should show:

```
NAME          PHASE    PROGRESSING   ...
akscluster    Running  True
```

---

# **3️⃣ Wait for OneAgent to be deployed on all nodes**

Dynatrace Operator now creates **OneAgent DaemonSet**:

Check:

```bash
kubectl -n dynatrace get pods | grep oneagent
```

If using **cloudNativeFullStack**, you’ll also see **init containers** in workloads.

You should see:

* OneAgent pods running on every node
* No errors like image pull issues, permissions issues, or missing mounts

---

# **4️⃣ Check ActiveGate deployment**

Since your YAML includes:

```yaml
activeGate:
  capabilities:
    - routing
    - kubernetes-monitoring
    - dynatrace-api
```

Check ActiveGate pods:

```bash
kubectl -n dynatrace get pods | grep activegate
```

Expected:

* 1 ActiveGate pod running (unless you increased replicas)
* No CrashLoopBackOff

---

# **5️⃣ Validate in Dynatrace UI**

Go to Dynatrace → **Kubernetes** → you should see:

* Your **cluster**
* All **nodes**
* All **pods**
* All **workloads**
* **Metrics**, logs, traces flowing in

If the cluster shows as “Unhealthy” or “Partially connected,” ping me — it’s usually a token or connectivity issue.

---

# **6️⃣ (Optional but recommended) Clean up deprecated API usage**

You are using:

```
apiVersion: dynatrace.com/v1beta3
```

The operator warns this is **deprecated**.
Later, you should migrate to:

```
apiVersion: dynatrace.com/v1beta1
```

But **do NOT change it until everything is running**.

---

# **7️⃣ Start monitoring your applications**

Deploy any app (nginx, node app, java app).
Check Dynatrace to see:

* Service flows
* Traces
* Logs
* CPU/memory
* Topology map

---

# **8️⃣ If anything fails, check Operator logs**

```bash
kubectl -n dynatrace logs deploy/dynatrace-operator
```

If the DynaKube CR is stuck in error state:

```bash
kubectl -n dynatrace describe dynakube akscluster
```

---

# 👉 **If you send me your current output of:**

```
kubectl -n dynatrace get pods
kubectl -n dynatrace get dynakube
```

