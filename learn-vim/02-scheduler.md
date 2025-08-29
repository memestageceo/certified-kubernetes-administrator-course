```markdown
# Kubernetes manual pod scheduling — important concepts and commands

## Concepts
- **Automatic scheduling:** The kube-scheduler assigns Pods to Nodes. If it’s not running, Pods stay Pending.
- **Pending indicator:** In `kubectl describe pod`, `Node: <none>` and no scheduling Events imply no assignment yet.
- **Manual scheduling:** Set `spec.nodeName` in the Pod manifest to bind directly to a specific Node.
- **Immutability of placement:** You can’t “move” a running Pod; delete and recreate it with the new target.
- **Node readiness matters:** The target node must exist and be Ready, or the Pod will remain Pending.

---

## Diagnose a pending pod
```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl get pods -n kube-system | grep -E 'scheduler|control'
```

- **Check status:** `STATUS = Pending`.
- **Describe details:** `Node: <none>`, `Events: <none>`.
- **Scheduler presence:** If the scheduler isn’t running, automatic placement won’t occur.

Example:

```bash
kubectl get pods
# NAME   READY   STATUS    RESTARTS   AGE
# nginx  0/1     Pending   0          46s

kubectl describe pod nginx
# Node: <none>
# Events: <none>
```

---

## Manually assign the pod to a node

- **Add nodeName:** Specify the exact node name under `spec.nodeName`.

Pod manifest (node01):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: node01
  containers:
    - name: nginx
      image: nginx
```

- **Apply when pod already exists:** Replace to delete-and-recreate in one step.

```bash
kubectl replace --force -f nginx.yaml
```

---

## Verify scheduling

```bash
kubectl get pods --watch
kubectl get pods -o wide
```

Example:

```
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE
nginx   1/1     Running   0          19s   10.244.0.4   node01
```

---

## Schedule on the control plane node

- **Update manifest:** Set `nodeName` to the control plane’s node name (e.g., controlplane).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: controlplane
  containers:
    - name: nginx
      image: nginx
```

- **Recreate and watch:**

```bash
kubectl replace --force -f nginx.yaml
kubectl get pods --watch
kubectl get pods -o wide
```

Example:

```
NAME    READY   STATUS    AGE   IP           NODE
nginx   1/1     Running   58s   10.244.0.4   controlplane
```

---

## Practical workflow

1. **Create:** Apply the Pod manifest without nodeName.
2. **Observe:** Confirm `Pending` and `Node: <none>` via describe.
3. **Target:** Add `spec.nodeName: <target-node>`.
4. **Recreate:** `kubectl replace --force -f <file>.yaml`.
5. **Watch:** `kubectl get pods --watch` until `Running`.
6. **Validate:** `kubectl get pods -o wide` to confirm Node placement.

---

## Notes and pitfalls

- **Exact node name:** `nodeName` must exactly match `kubectl get nodes` output.
- **Node readiness:** Ensure `kubectl get nodes` shows the node as `Ready`.
- **No live migration:** Pods aren’t movable; delete and recreate on the desired node.
- **Graceful termination:** Force replace sends a termination signal to the container; brief delays are normal.
- **Security constraints:** Taints, tolerations, or admission policies can still block scheduling despite nodeName.

---

## Commands cheat sheet

```bash
# List pods and statuses
kubectl get pods

# Inspect why a pod is pending
kubectl describe pod <pod-name>

# Delete & recreate in one step with updated manifest
kubectl replace --force -f <file>.yaml

# Watch pod state transitions
kubectl get pods --watch

# Confirm node and pod IP
kubectl get pods -o wide

# List nodes and their readiness
kubectl get nodes -o wide
```

1. create config for binding

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  conceptsame: node02
```

2. export config as json

```bash
kubectl create -f binding.yaml --dry-run=client -o json | binding.json
```

3. send POST request to api

```sh
curl --header "Content-Type: application/json" --request POST --data @binding.json http://$SERVER/api/v1/namespaces/default/pods/nginx/binding
```

## labels and selectors 🏷️

- **Labels:** Key-value pairs attached to Kubernetes objects (e.g., Pods, ReplicaSets, Services) to organize and group them by attributes such as `app`, `function`, or `env`.
- **Selectors:** Queries used to filter resources based on label values. Can be used in `kubectl` commands or inside object specifications.
- **Annotations:** Non-identifying metadata for storing extra information (e.g., build versions, contact info). Not used for selection or grouping.

---

## Labels in Kubernetes

- **Purpose:** Enable grouping and logical selection of resources.
- **Example use case:** Assigning `app: App1` to multiple Pods so they can be managed or queried together.
- **Syntax in Pod manifest:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
```

---

## Using Labels with `kubectl`

```bash
# Get pods with a specific label
kubectl get pods --selector app=App1

# Multiple label filtering (AND logic)
kubectl get pods --selector app=App1,function=Front-end
```

**Example Output:**

```
NAME            READY   STATUS      RESTARTS   AGE
simple-webapp   0/1     Completed   0          1d
```

---

## Labels and ReplicaSets

- **Internal mechanism:** ReplicaSets use labels and selectors to link to their Pods.
- **Where labels go:**
  1. **ReplicaSet metadata** — to help other resources reference the ReplicaSet.
  2. **Pod template metadata** — so created Pods inherit the labels.
- **Selector in ReplicaSet:** Must match the labels defined in the Pod template for correct association.

Example:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```

---

## Annotations

- **Purpose:** Store auxiliary, non-identifying metadata.
- **Example:** Build information, version numbers, or maintainer details.
- **Difference from labels:** Annotations are not used by selectors for grouping or filtering.

Example with annotation:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
  annotations:
    buildversion: "1.34"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```

---

## Practical Workflow Summary

1. **Define labels** in the object’s `metadata.labels`.
2. **Use selectors** in:
   - `kubectl` commands to filter objects.
   - Controller specs (e.g., ReplicaSet, Service) to match specific Pods.
3. **Apply annotations** for extra metadata, not selection.
4. **Verify grouping** with `kubectl get <resource> --selector <key>=<value>`.

---

## Commands Cheat Sheet

```bash
# Get all pods with a specific label
kubectl get pods --selector app=App1

# Get pods matching multiple labels
kubectl get pods --selector app=App1,function=Front-end

# Show labels of all pods
kubectl get pods --show-labels

# Describe a resource and see labels/annotations
kubectl describe pod <pod-name>

# Apply manifest with labels and annotations
kubectl apply -f <file>.yaml
```

### solutions

**Q:** How do you list all pods in the `dev` environment?  
**A:**  

```bash
kubectl get pods --selector env=dev
```

**Q:** How do you count them without headers?  
**A:**  

```bash
kubectl get pods --selector env=dev --no-headers | wc -l
```

💡 *Example:* Output shows `7` pods in `dev`.

---

**Q:** How to count pods in the `finance` BU?  
**A:**  

```bash
kubectl get pods --selector bu=finance --no-headers | wc -l
```

💡 Output: `6` pods.

---

**Q:** How to list only `prod` pods?  
**A:**  

```bash
kubectl get pods --selector env=prod --no-headers
```

**Q:** How to include all object types (`pods`, `services`, `ReplicaSets`) in `prod`?  
**A:**  

```bash
kubectl get all --selector env=prod --no-headers
```

Then pipe to:

```bash
... | wc -l
```

💡 Example: `7` objects.

---

**Q:** How to find the pod in `prod` **AND** `finance` BU **AND** `frontend` tier?  
**A:**  

```bash
kubectl get all --selector env=prod,bu=finance,tier=frontend
```

💡 Example output:

```
pod/app-1-zzxdf  1/1 Running  0  4m7s
```

---

**Q:** Error:  

```
Invalid value: selector does not match template labels
```

**Cause:** `spec.selector.matchLabels` **must exactly match** `spec.template.metadata.labels`.

**Original YAML (mismatch)**  

```yaml
selector:
  matchLabels:
    tier: front-end
template:
  metadata:
    labels:
      tier: nginx
```

**Fix:** Align template label with selector:  

```yaml
selector:
  matchLabels:
    tier: front-end
template:
  metadata:
    labels:
      tier: front-end
```

**Steps to apply fix:**

```bash
kubectl create -f replicaset-definition-1.yaml
kubectl get rs
```

💡 *Tip:* Selector–template label mismatch is one of the most common ReplicaSet creation errors.

---

## ⚡ Key Takeaways

- `--selector key=value` filters Kubernetes objects by label.
- Combine labels with commas for logical AND.
- Use `--no-headers | wc -l` for quick counts.
- Always ensure **ReplicaSet selector labels** match **template labels** exactly.

#### Concept: How Taints & Tolerations Work

- **Taint**: Applied to a node to repel pods unless they have matching tolerations.
- **Toleration**: Set on a pod to allow scheduling on a tainted node.
- **Purpose**: Controls **scheduling only**, not security.
- **Mechanism**: Scheduler checks node taints → pod must have matching **key**, **value**, and **effect**.

---

#### Command: Taint a Node

```bash
kubectl taint nodes <node-name> key=value:taint-effect
```

Example:

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

---

#### Taint Effects

| Effect               | Behavior |
|----------------------|----------|
| **NoSchedule**       | Blocks non‑tolerating pods from being scheduled. |
| **PreferNoSchedule** | Avoids scheduling non‑tolerating pods when possible. |
| **NoExecute**        | Evicts running non‑tolerating pods and blocks new ones. |

---

#### Manifest Snippet: Add Toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
```

---

#### Example: NoExecute Taint

```bash
kubectl taint nodes node1 app=blue:NoExecute
```

- **Without toleration**: Pods evicted or blocked.
- **With toleration**: Pods continue running.

---

#### Master Node Default Taint

Check:

```bash
kubectl describe node <master-node> | grep Taint
```

Output:

```
Taints: node-role.kubernetes.io/master:NoSchedule
```

---

#### Key Takeaways

- Taints **repel** pods; tolerations **permit** them.
- Matching **key, value, effect** is required.
- Default master node taint reserves control plane for system components.
- Use **node affinity** for guaranteed scheduling on specific nodes.
