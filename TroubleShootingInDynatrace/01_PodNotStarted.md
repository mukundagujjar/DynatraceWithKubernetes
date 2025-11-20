## To troubleshoot **why your pod works WITHOUT Dynatrace OneAgent injection but fails WITH OneAgent injection**.

This problem is **common** and can be caused by:

* networking changes after injection
* initContainer failure
* CSI mount failure
* library injection breaking your app
* environment variable conflicts
* liveness/readiness issues
* permission issues
* language-runtime problems (Java, Node, Python, Go, etc.)

I will walk you through **exact checks** to identify the root cause.

---

# ⭐ Questions

Before I give the exact fix, I need **one detail**:

### 👉 Is your POD stuck in:

**A)** `Init:CrashLoopBackOff`
**B)** `CrashLoopBackOff` after start
**C)** Running, but your app is not accessible?
**D)** Liveness/Readiness failing?
**E)** Application error logs?



--

# ⭐ Step 1 — Check Dynatrace Init Container Status

Run:

```bash
kubectl describe pod <pod-name>
```

Look for `dynatrace-oneagent-init`.

### If it shows:

```
Error
CrashLoopBackOff
```

Then Dynatrace injection is failing **before** your app starts.

This indicates:

* CSI mount failed
* Token issues
* Init script failure
* Permissions or capabilities missing

If so → Tell me the logs:

```bash
kubectl logs <pod> -c dynatrace-oneagent-init
```

---

# ⭐ Step 2 — Check CSI Volume Mount Issues

Look for errors like:

```
MountVolume.SetUp failed
rpc error: failed to mount dynatrace volume
```

Or:

```
cannot read agent binaries
no such file or directory: /opt/dynatrace/oneagent
```

This means:

* CSI Node Plugin is broken
* Node has network / CNI issues
* OneAgent download failed
* CSI DaemonSet not running

Check:

```bash
kubectl get pods -n dynatrace | grep csi
```

If any CSI pods are **NotReady**, your injection will fail.

---

# ⭐ Step 3 — Check if the Application Container Fails After Injection

If your app fails AS SOON AS OneAgent is injected, check logs:

```bash
kubectl logs <pod> -c <app-container>
```

Common root causes:

### ❌ Language runtime conflicts

Java, Node, Python apps may break if:

* they load native libraries incorrectly
* LD_PRELOAD conflicts
* old glibc version
* custom entrypoint scripts break preload

### ❌ Application binds to 127.0.0.1 only

Some apps break when LD_PRELOAD affects socket operations.

### ❌ App uses static musl (Alpine)

Alpine + musl → occasionally incompatible with code module injection.

If your container uses Alpine:

```bash
cat /etc/os-release
```

Let me know — I will give Alpine-specific fixes.

---

# ⭐ Step 4 — Check Container Start Hooks

Run:

```bash
kubectl get pod <pod> -o json | jq '.spec.containers[].env'
```

Dynatrace injects environment variables like:

* LD_PRELOAD
* JAVA_TOOL_OPTIONS
* DT_* variables

If your app script overrides these → it breaks.

---

# ⭐ Step 5 — Check Liveness/Readiness Probes

Sometimes after injection the app starts slower → probes fail → pod restarts.

Check:

```bash
kubectl describe pod <pod> | grep -i probe -A5
```

If probes are too aggressive:

* increase initialDelaySeconds
* increase timeoutSeconds

Example fix:

```yaml
readinessProbe:
  initialDelaySeconds: 20
```

---

# ⭐ Step 6 — Check Networking Changes After Injection

If the app is running but network not working:

### Check if IPTables is modified

Dynatrace uses eBPF / socket hooks
Older kernels + custom CNI may conflict.

### Run inside pod:

```bash
curl -v https://google.com
```

If outbound traffic fails only AFTER injection →
→ eBPF or network hook conflict.

Also test DNS:

```bash
nslookup kubernetes.default
```

---

# ⭐ Step 7 — Disable Code Module Injection Temporarily (Diagnostic)

Add annotation:

```yaml
oneagent.dynatrace.com/inject: "false"
```

If pod works → root cause = code module injection.

Then we test:

### Option 1: Disable only specific languages

```yaml
oneagent.dynatrace.com/oneagent-ignore-proc: "node"
```

### Option 2: Disable tracing only

```yaml
oneagent.dynatrace.com/automatic-injection: "false"
```

### Option 3: Enable logs only

```yaml
oneagent.dynatrace.com/logs: "true"
```

I can generate proper annotations after I see your logs.

---

# ⭐ Step 8 — Collect Full Dynatrace Troubleshooting Log Bundle

Run:

```bash
kubectl logs deploy/dynatrace-operator -n dynatrace | grep <pod-name> -A5
```

Operator log will show **WHY** it injected and if anything failed.

---

# ⭐ FINAL — I Can Diagnose This 100%

All I need from you is:

### ✔ pod name + namespace

### ✔ output of:

```bash
kubectl describe pod <pod>
```

### ✔ logs of init container:

```bash
kubectl logs <pod> -c dynatrace-oneagent-init
```

### ✔ logs of your app container:

```bash
kubectl logs <pod> -c <app-container>
```

Once you provide this, I will tell you:

* whether the issue is **network**, **CSI**, **probe**, **runtime**, or **injection conflict**,
* and give the exact fix.

---

# ❗ Waiting for your details

Reply with:

**A/B/C/D/E** from the earlier list **and** the pod logs/describe info.
I will troubleshoot and fix it precisely.
