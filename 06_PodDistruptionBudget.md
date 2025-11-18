A **PodDisruptionBudget (PDB)** in Kubernetes is a policy that protects your application from **unplanned downtime** during **voluntary disruptions**, such as:

* Node drain
* Cluster upgrades
* Evictions
* Scaling events
* OS updates
* Maintenance operations

It **limits how many pods can be unavailable at the same time**.

---

# ⭐ **1. What is a PodDisruptionBudget?**

A **PDB** tells Kubernetes:

> "Keep at least this many replicas of my application running even during maintenance."

Kubernetes will **block disruptions** (like `kubectl drain`) if doing so would violate the PDB.

---

# ⭐ **2. Why PDB is Important**

PDB is mainly used for **high availability workloads**:

* Frontend services
* Backends
* Databases
* State-dependent apps
* Monitoring agents (like Dynatrace ActiveGate)
* Controllers (operators)

Without a PDB, draining a node can take down **all replicas**, causing **downtime**.

---

# ⭐ **3. How PDB Works**

PDB defines **one of two rules**:

### ✔ **minAvailable**

Minimum number of pods that must stay running.

```yaml
minAvailable: 2
```

### ✔ **maxUnavailable**

Maximum pods allowed to be unavailable.

```yaml
maxUnavailable: 1
```

---

# ⭐ **4. Example: PDB for a 3-Replica App**

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: frontend-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: frontend
```

Meaning:

* Out of 3 pods, **at least 2 must remain running**.
* Only 1 pod can be disrupted at a time.

---

# ⭐ **5. PDB Example with maxUnavailable**

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: backend-pdb
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: backend
```

Meaning:

* At most 1 pod can be removed at a time.

---

# ⭐ **6. How to Apply PDB in Rolling Updates**

If you have a Deployment:

```bash
kubectl apply -f pdb.yaml
```

Then during:

* `kubectl drain node1`
* `kubectl rollout restart deploy/frontend`
* Cluster upgrade

Kubernetes will **wait** and avoid killing too many pods.

---

# ⭐ **7. Check PDB Status**

```bash
kubectl get pdb
kubectl describe pdb <name>
```

You will see fields:

* **Allowed disruptions** (how many pods can be removed)
* **Current healthy** (running pods)
* **Expected** (desired replicas)

Example output:

```
Allowed Disruptions: 1
Current Healthy: 3
Desired Healthy: 2
```

This means draining nodes is safe.

---

# ⭐ **8. PDB for Dynatrace Components**

Dynatrace recommends PDB for:

### ✔ ActiveGate (StatefulSet)

Ensures no interruption of:

* Prometheus ingestion
* Kubernetes API monitoring
* Routing

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: activegate-pdb
  namespace: dynatrace
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: dynatrace-activegate
```

### ✔ Dynatrace Operator

You usually run **only 1 operator**, so a PDB does not help — but operator must be high priority.

### ✔ OneAgent DaemonSet

DaemonSets ignore PDBs (they run one pod per node).
PDB is unnecessary here.

---

# ⭐ **9. When NOT to use PDB**

Do NOT create PDB if:

* Your Deployment has **only 1 replica**
* Your pods are not critical
* You want updates to proceed quickly

Example: test apps.

---

# ⭐ Summary

### PodDisruptionBudget:

* Protects your app from voluntary disruptions
* Ensures minimum pods always stay alive
* Required for HA workloads
* Controls rolling updates, node drains & cluster maintenance

---


