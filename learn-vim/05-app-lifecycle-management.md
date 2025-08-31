# App Lifecycle Management

## Rollout and Rollback

These commands are used to manage the state and history of a Kubernetes Deployment during a rollout.

```bash
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
```

To update the image of a container within a deployment:

```bash
kubectl set image deployment/myapp-deployment nginx-container=nginx:1.9.1
```

To revert to the previous revision:

```bash
kubectl rollout undo deployment/myapp-deployment
```

The [.spec.strategy.type](https://spec.strategy.type) field can be `Recreate` or `RollingUpdate`. `RollingUpdate` is the default value.

## Dockerfile CMD and ENTRYPOINT Instructions

The `CMD` instruction provides default parameters for an executable, while `ENTRYPOINT` defines the executable itself.

```dockerfile
CMD ["bash"]
CMD sleep 5
CMD ["sleep", "5"]
```

Example of a Dockerfile with both ENTRYPOINT and CMD:

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

When you run a container from this image without any extra parameters, it executes `sleep 5`.

```bash
docker run ubuntu-sleeper
```

You can override the CMD default parameters at runtime by specifying new arguments:

```bash
docker run ubuntu-sleeper 10
```

### Overriding ENTRYPOINT

To override the ENTRYPOINT with a different executable, use the `--entrypoint` flag:

```bash
docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```

### Pod Definition: command and args

Any argument provided in a `docker run` command correlates with the `args` property in a pod definition. When you append an argument to a `docker run` command, it overrides the default parameters defined by the `CMD` instruction in the Dockerfile.

Example `args` field in a pod definition:

```yaml
containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    args: ["10"]
```

Example of a pod overriding both ENTRYPOINT and CMD:

```yaml
containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    command: ["sleep2.0"]
    args: ["10"]
```

Remember that specifying the `command` in a pod definition replaces the Dockerfile's `ENTRYPOINT` entirely, while the `args` field only overrides the default parameters defined by `CMD`. All elements in the `command` and `args` arrays must be provided as strings.

### The kubectl run command

```bash
kubectl run webapp-green --image=kodekloud/webapp-color -- --color green
```

## ConfigMap

Sure! Here's your content formatted in clean, copy-ready **Markdown** for your notes:

---

### 🐳 Docker: Set Environment Variable

```bash
docker run -e APP_COLOR=pink simple-webapp-color
```

---

### 📦 Kubernetes: ConfigMap Commands

```bash
kubectl get configmaps

kubectl create configmap app-config \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_MOD=prod

kubectl create configmap app-config \
  --from-file=app_config.properties
```

---

### 📄 ConfigMap Manifest

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

---

## 🧬 Pod Spec: Injecting Environment Variables

#### Direct Key-Value Pair

```yaml
spec:
  containers:
    - env:
        - name: APP_COLOR
          value: pink
```

#### From ConfigMap Key

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: color
```

#### From Secret Key

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
        name: app-secrets
        key: color
```

#### From Entire ConfigMap

```yaml
envFrom:
  - configMapRef:
      name: app-color
```

#### 🔐 Alternate Methods of Injecting Secrets

```yaml
envFrom:
  - configMapRef:
      name: app-config

env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR

volumes:
  - name: app-config-volume
    configMap:
      name: app-config
```

### Create Kubernetes Secrets

```bash
kubectl create secret generic app-secret --from-literal=DB_Host=mysql

kubectl create secret generic app-secret --from-file=app_secret.properties

echo -n 'mysql' | base64
echo -n 'bXlzcWw=' | base64 --decode

```

```yaml
envFrom:
  - secretRef:
      name: app-secret

Mounting Secrets as Volumes
volumes:
  - name: app-secret-volume
    secret:
      secretName: app-secret

```

**Notes**

- Each key in the Secret becomes a separate file when mounted as a volume.
- Listing the mounted directory will show each key as a file.
- Kubernetes Secrets use Base64 encoding only.
- For enhanced security, enable encryption at rest for etcd.

### Encryption at Rest - etcd

**Check if encryption is enabled:**

```bash
ps -aux | grep kube-api | grep "encryption-provider-config"
```

**Generate encryption key:**

```bash
head -c 32 /dev/urandom | base64
```

**Create encryption configuration:**

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: y0xTt+U6xgRdNxe4nDYYsijOGgRDoUYC+wAwOKeNfPs= # Replace with your generated key
      - identity: {}
```

**Add to kube-apiserver configuration:**

```yaml
# Add to kube-apiserver.yaml
spec:
  containers:
    - command:
        - kube-apiserver
        # ... other flags ...
        - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
      volumeMounts:
        # ... other volume mounts ...
        - name: enc
          mountPath: /etc/kubernetes/enc
          readOnly: true
  volumes:
    # ... other volumes ...
    - name: enc
      hostPath:
        path: /etc/kubernetes/enc
        type: DirectoryOrCreate
```

### Horizontal/Vertical Pod Autoscaling

**Set namespace context:**

```bash
kubectl config set-context --current --namespace=elastic
```

#### Cluster Infrastructure Horizontal Scaling

```bash
kubeadm join <master-node-endpoint> --token <token> --discovery-token-ca-cert-hash <hash>
```

#### Workload Horizontal Scaling

**Manual scaling:**

```bash
kubectl scale --replicas=<number> <workload-type>/<workload-name>
kubectl edit <workload-type>/<workload-name>
```

**Monitor pod metrics:**

```bash
kubectl top pod my-app-pod
```

#### Horizontal Pod Autoscaler (HPA)

HPA continuously monitors pod metrics—such as CPU, memory, or custom metrics—using the metrics-server.

**Create HPA:**

```bash
kubectl autoscale deployment my-app --cpu-percent=50 --min=1 --max=10
```

**Manage HPA:**

```bash
kubectl get hpa
kubectl delete hpa my-app
```

**HPA YAML configuration:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

#### Vertical Pod Autoscaler (VPA)

**VPA YAML configuration:**

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
      - containerName: "my-app"
        minAllowed:
          cpu: "250m"
        maxAllowed:
          cpu: "2"
        controlledResources: ["cpu"]
```
