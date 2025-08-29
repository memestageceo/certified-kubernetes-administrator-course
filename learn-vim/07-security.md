# Kubernetes Security

k run nginx --image nginx

k get pods

essential parts of pod configuration file: apiVersion, kind, metadata, spec

k create -f pod-definition.yaml

k describe pod myapp-pod

k expose pod nginx --port 80 --type NodePort

k run nginx --image=nginx

k get pods -o wide

k run redis --image=redis123 --dry-run=client -o yaml > redis.yaml

apiVersion for replicaset: `apps/v1`

for deployment & replicaset, field under which to define pod specification: spec > template

field to define what pods are linked to a deployment: `selector > matchLabels`

k scale --replicas=6 -f replicaset-definition.yml

k create deployment nginx --image=nginx --replicas=3

NodePort: Maps a port on the node to a port on a Pod.
ClusterIP: Creates a virtual IP for internal communication between services (e.g., connecting front-end to back-end servers).
LoadBalancer: Provisions an external load balancer (supported in cloud environments) to distribute traffic across multiple Pods.

With a NodePort service, there are three key ports to consider:

Target Port: The port on the Pod where the application listens (e.g., 80).
Port: The virtual port on the service within the cluster.
NodePort: The external port on the Kubernetes node (by default in the range 30000–32767).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    app: myapp
    type: front-end
```

While NodePort works, it forces users to remember multiple IP-port pairs, which can be inconvenient. ERGO LoadBalancer.

Endpoints represent the set of pod IP addresses and ports that receive traffic from the service. If the service’s selector mistakenly does not match any pods, the endpoints list will be empty.

k expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml

k edit deployment nginx

k set image deployment nginx nginx=nginx:1.18

k replace -f nginx.yaml

k run redis --image=redis:alpine --labels="tier=db"

k scale deployment webapp --replicas=3

k run custom-nginx --image=nginx --port=8080

k run httpd --image=httpd:alpine --port=80 --expose=true

Notice that the YAML configuration is converted to JSON and stored as the "last applied configuration" in an annotation. This information is used during subsequent updates to identify any differences.

This annotation is the key to performing a three-way merge in future apply operations. The process compares:

The local file.
The live object configuration.
The last applied configuration.

---

to manually assign pod to node, use the field `spec > nodeName`

to assign a running container to a node, create a Binding object, then convert to json and POST to pod's binding api using curl.

curl --header "Content-Type: application/json" --request POST --data @binding.json http://$SERVER/api/v1/namespaces/default/pods/nginx/binding

k get pods --selector app=App1

k get pods --selector env=dev --no-headers | wc -l

k get all --selector env=prod,bu=finance,tier=frontend

k taint nodes node1 app=blue:NoSchedule

**Other taint effects:**

- PreferNoSchedule
- NoExecute (evicts existing pods)

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

k describe node kubemaster | grep Taint

k taint node controlplane node-role.kubernetes.io/master:NoSchedule-

k label nodes node-1 size=Large

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  nodeSelector:
    size: Large
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values:
                  - Large
```

**Operators:**

- In
- Exists
- NotIn

**NodeAffinity Types:**

- requiredDuringSchedulingIgnoredDuringExecution
- preferredDuringSchedulingIgnoredDuringExecution

```yaml
resources:
  requests:
    memory: "4Gi"
    cpu: 2
```

If a pod exceeds its memory limit, it may be terminated due to an Out Of Memory (OOM) error.

```yaml
limits:
  memory: "2Gi"
  cpu: 2
```

LimitRanges are namespace-level objects that automatically assign default resource values to containers that do not specify them.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
    - default: # this section defines default limits
        cpu: 500m
      defaultRequest: # this section defines default requests
        cpu: 500m
      max: # max and min define the limit range
        cpu: "1"
      min:
        cpu: 100m
      type: Container
```

```yaml
# daemon-set-definition.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
        - name: monitoring-agent
          image: monitoring-agent
```

k describe ds kube-flannel-ds -n kube-system

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
        # This toleration ensures the DaemonSet can run on master nodes.
        # Remove it if your master nodes cannot run pods.
        - key: node-role.kubernetes.io/master
          operator: Exists
          effect: NoSchedule
      containers:
        - name: fluentd-elasticsearch
          image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
          resources:
            limits:
              memory: 200Mi
            requests:
              cpu: 100m
              memory: 200Mi
          volumeMounts:
            - name: varlog
              mountPath: /var/log
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
```

k get ds -n kube-system

**For static pods:**

- Pass path to kubelet: `--pod-manifest-path=/etc/kubernetes/manifests`
- Pass the path in the kubeconfig.yaml: `staticPodPath: /etc/kubernetes/manifests`

**How to identify a static pod:**

- It has the node name appended to it (e.g., `nginx-node01`)
- In the YAML output, `ownerReferences.kind: Node` (non-static pods have `ReplicaSet` as the kind)

**To permanently delete a static pod:**

- SSH into the node the pod is running on
- Delete the manifest file

---

k get priorityclass

```yaml
# priority-class.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000000
description: "Priority class for mission critical pods"
# default value is 0, but we set this priorityClass as the default one now.
globalDefault: true
preemptionPolicy: PreemptLowerPriority # or Never is want the pod to wait in queue

---
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
  priorityClassName: high-priority
```

---

## 🧠 Multiple Schedulers in Kubernetes

Kubernetes allows the use of multiple schedulers. You can configure and run custom schedulers alongside the default one.

### 📝 Scheduler Configuration Files

```yaml
# my-scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
```

```yaml
# scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: default-scheduler
```

---

## 🚀 Running Scheduler as a Pod

You can deploy a scheduler as a pod inside the Kubernetes cluster:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
    - name: kube-scheduler
      image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
      command:
        - kube-scheduler
        - --address=127.0.0.1
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --config=/etc/kubernetes/my-scheduler-config.yaml
```

---

## 🛠️ Deploying the Custom Scheduler as a Deployment

### Step 1: Build and Push a Custom Scheduler Image

Create and push your custom scheduler image to a container registry.

### Step 2: Create ServiceAccount and RBAC Configurations

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-scheduler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-scheduler-as-kube-scheduler
subjects:
  - kind: ServiceAccount
    name: my-scheduler
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: system:kube-scheduler
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-scheduler-as-volume-scheduler
subjects:
  - kind: ServiceAccount
    name: my-scheduler
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: system:volume-scheduler
  apiGroup: rbac.authorization.k8s.io
```

### Step 3: Create a ConfigMap for Scheduler Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-scheduler-config
  namespace: kube-system
data:
  my-scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1beta2
    kind: KubeSchedulerConfiguration
    profiles:
      - schedulerName: my-scheduler
        leaderElection:
          leaderElect: false
```

k create configmap my-scheduler-config --from-file=/root/my-scheduler-config.yaml -n kube-system

### Step 4: Define the Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-scheduler
  namespace: kube-system
  labels:
    component: scheduler
    tier: control-plane
spec:
  replicas: 1
  selector:
    matchLabels:
      component: scheduler
      tier: control-plane
  template:
    metadata:
      labels:
        component: scheduler
        tier: control-plane
        version: second
    spec:
      serviceAccountName: my-scheduler
      containers:
        - name: kube-second-scheduler
          image: gcr.io/my-gcp-project/my-kube-scheduler:1.0
          command:
            - /usr/local/bin/kube-scheduler
            - --config=/etc/kubernetes/my-scheduler/my-scheduler-config.yaml
          livenessProbe:
            httpGet:
              path: /healthz
              port: 10259
              scheme: HTTPS
            initialDelaySeconds: 15
          readinessProbe:
            httpGet:
              path: /healthz
              port: 10259
              scheme: HTTPS
          volumeMounts:
            - name: config-volume
              mountPath: /etc/kubernetes/my-scheduler
      volumes:
        - name: config-volume
          configMap:
            name: my-scheduler-config
```

---

## 🔐 ClusterRole for Scheduler

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:kube-scheduler
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
rules:
  - apiGroups:
      - coordination.k8s.io
    resources:
      - leases
    verbs:
      - create
  - apiGroups:
      - coordination.k8s.io
    resourceNames:
      - kube-scheduler
      - my-scheduler
    resources:
      - leases
    verbs:
      - get
      - list
```

---

## 📦 Configuring Workloads to Use the Custom Scheduler

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
  schedulerName: my-custom-scheduler
```

---

## ✅ Verifying Scheduler Operation

```bash
k get events -o wide
k logs my-custom-scheduler --namespace=kube-system
```

k get sa my-scheduler

# [Configuring Scheduler Profiles - KodeKloud Notes](https://notes.kodekloud.com/docs/CKA-Certification-Course-Certified-Kubernetes-Administrator/Scheduling/Configuring-Scheduler-Profiles)

## Additional Scheduler Configuration Examples

```yaml
# my-scheduler-2-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler-2
  - schedulerName: my-scheduler-3
```

```yaml
# my-scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
```

```yaml
# scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: default-scheduler
```

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler-2
    plugins:
      score:
        disabled:
          - name: TaintToleration
        enabled:
          - name: MyCustomPluginA
          - name: MyCustomPluginB
  - schedulerName: my-scheduler-3
    plugins:
      preScore:
        disabled:
          - name: "*"
      score:
        disabled:
          - name: "*"
  - schedulerName: my-scheduler-4
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list", "get", "create", "update", "delete"]
```

**Built-in admission controllers:** AlwaysPullImages, DefaultStorageClass, EventRateLimit, NamespaceExists, etc.

kube-apiserver -h | grep enable-admission-plugins

**How to enable auto provisioning of namespace:**

In `spec.containers.command`:

- `--enable-admission-plugins=NodeRestriction,NamespaceAutoProvision`

**Admission Controller Flow:**
User runs k command → API server authenticates based on RBAC → Admission Controllers validate, mutate and auto-provision resources and provide extra layer of security.

k exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep 'enable-admission-plugins'

```yaml
spec:
  containers:
    - command:
        - --disable-admission-plugins=DefaultStorageClass
```
