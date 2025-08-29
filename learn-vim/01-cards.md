## What is etcd in Kubernetes?

Consistent and highly-available key-value store used as Kubernetes' backing store for all cluster data [#kubernetes]()

## What type of data store is etcd?

Key-Value Store (stores data in JSON or YAML format) [#etcd]()

# What is etcd's role in Kubernetes cluster?

Single source of truth for all cluster configuration and state [#etcd]()

## Export etcd API version %

How do you set the etcd API version to 3?

```bash
export ETCDCTL_API=3
```

[#etcd]()

## Set key in etcd %

What command sets a key-value pair in etcd?

```bash
./etcdctl put key1 value1
```

[#etcd]()

## Get key from etcd %

What command retrieves a key's value from etcd?

```bash
./etcdctl get key1
```

[#etcd]()

## View all etcd keys %

What command shows all keys in etcd via kubectl?

```bash
kubectl exec etcd-master -n kube-system -- etcdctl get / --prefix --keys-only
```

[#etcd]()

## What is the Kube API Server?

Front-end for Kubernetes control plane that exposes the Kubernetes API [#kubernetes]()

## API Server request lifecycle

1. Authentication & Authorization 2. Data Retrieval from etcd 3. Coordination with other components [#api-server]()

## What does the Controller Manager do?

Runs controller processes that watch cluster state and make changes to move current state towards desired state [#controller-manager]()

## Name two types of controllers

Node Controller (manages nodes) and Replication Controller (ensures correct pod replicas) [#controller-manager]()

## What is the Kubelet?

Agent that runs on each node ensuring containers are running in Pods [#kubelet]()

## Kubelet responsibilities

Registers node with cluster, manages pod lifecycle, reports status to API server [#kubelet]()

## What does the Scheduler do?

Watches for newly created Pods with no Node assigned and finds the best Node for them to run on [#scheduler]()

## Scheduler process steps

1. Filter Nodes (by CPU, memory, taints) 2. Rank Nodes (score 0-10) 3. Select highest scoring Node [#scheduler]()

## What is a Pod?

Smallest and simplest unit in Kubernetes that can contain one or more containers sharing storage and network [#pods]()

## How do you scale in Kubernetes?

Create more Pods, not by adding containers to existing Pod [#pods]()

## Run nginx pod %

What command creates a single nginx pod?

```bash
kubectl run nginx --image=nginx
```

[#kubectl]()

## Generate pod YAML %

What command generates a pod manifest without creating it?

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

[#kubectl]()

## What is a ReplicaSet?

Maintains a stable set of replica Pods running at any given time [#replicaset]()

## Scale ReplicaSet declaratively %

How do you scale a ReplicaSet by updating the YAML file?

```bash
kubectl replace -f replicaset-definition.yml
```

[#replicaset]()

## Scale ReplicaSet imperatively %

What command scales a ReplicaSet to 6 replicas?

```bash
kubectl scale --replicas=6 replicaset/myapp-replicaset
```

[#replicaset]()

## {{c1::spec.nodeName}} field manually assigns a Pod to a specific Node

[#manual-scheduling]()

## Manual scheduling limitation

You cannot {{c1::move}} a running Pod; you must delete and recreate it [#manual-scheduling]()

## Pending pod indicators

{{c1::Node: <none>}} and no scheduling Events in kubectl describe [#manual-scheduling]()

## Force replace pod %

What command deletes and recreates a pod with updated manifest?

```bash
kubectl replace --force -f nginx.yaml
```

[#manual-scheduling]()

## Watch pod status %

What command continuously monitors pod state changes?

```bash
kubectl get pods --watch
```

[#manual-scheduling]()

## View pod node assignment %

What command shows which node each pod is running on?

```bash
kubectl get pods -o wide
```

[#manual-scheduling]()

## What are Labels?

Key-value pairs attached to Kubernetes objects for organization and grouping [#labels]()

## What are Selectors?

Queries used to filter resources based on label values [#labels]()

## What are Annotations?

Non-identifying metadata for storing extra information like build versions [#labels]()

## Get pods by label %

What command gets pods with label app=App1?

```bash
kubectl get pods --selector app=App1
```

[#labels]()

## Multiple label filtering %

What command gets pods with app=App1 AND function=Front-end?

```bash
kubectl get pods --selector app=App1,function=Front-end
```

[#labels]()

## Show pod labels %

What command displays all pods with their labels?

```bash
kubectl get pods --show-labels
```

[#labels]()

## Count pods by label %

How do you count pods in dev environment without headers?

```bash
kubectl get pods --selector env=dev --no-headers | wc -l
```

[#labels]()

## ReplicaSet selector requirement

{{c1::spec.selector.matchLabels}} must exactly match {{c1::spec.template.metadata.labels}} [#labels]()

## What is a Taint?

Applied to a node to repel pods unless they have matching tolerations [#taints]()

## What is a Toleration?

Set on a pod to allow scheduling on a tainted node [#taints]()

## Taint a node %

What command taints node1 with app=blue:NoSchedule?

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

[#taints]()

## NoSchedule taint effect

{{c1::Blocks}} non-tolerating pods from being scheduled [#taints]()

## PreferNoSchedule taint effect

{{c1::Avoids}} scheduling non-tolerating pods when possible [#taints]()

## NoExecute taint effect

{{c1::Evicts}} running non-tolerating pods and blocks new ones [#taints]()

## Check master node taint %

What command shows the taint on a master node?

```bash
kubectl describe node <master-node> | grep Taint
```

[#taints]()

## Default master node taint

{{c1::node-role.kubernetes.io/master:NoSchedule}} [#taints]()

## Toleration matching requirement

Pod toleration must match taint's {{c1::key}}, {{c1::value}}, and {{c1::effect}} [#taints]()

## What provides logical isolation in Kubernetes?

Namespaces [#namespaces]()

## Default namespace behavior

Automatically used when {{c1::no namespace is specified}} [#namespaces]()

## kube-system namespace purpose

Contains {{c1::core system components}} like DNS and network [#namespaces]()

## Cross-namespace service access format

{{c1::db-service.dev.svc.cluster.local}} [#namespaces]()

## Get pods in specific namespace %

What command lists pods in kube-system namespace?

```bash
kubectl get pods --namespace=kube-system
```

[#namespaces]()

## Get pods in all namespaces %

What command lists pods across all namespaces?

```bash
kubectl get pods --all-namespaces
```

[#namespaces]()

## Create namespace %

What command creates a namespace called dev?

```bash
kubectl create namespace dev
```

[#namespaces]()

## Set default namespace %

What command sets dev as the default namespace for current context?

```bash
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

[#namespaces]()

## ResourceQuota purpose

Controls and limits {{c1::resource consumption}} within a namespace [#namespaces]()

## Create deployment %

What command creates an nginx deployment?

```bash
kubectl create deployment --image=nginx nginx
```

[#kubectl]()

## Generate deployment YAML %

What command generates deployment manifest and saves to file?

```bash
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

[#kubectl]()

## Create deployment with replicas %

What command creates deployment with 4 replicas and saves YAML?

```bash
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```

[#kubectl]()

## Apply manifest file %

What command creates resources from a YAML file?

```bash
kubectl create -f nginx-deployment.yaml
```

[#kubectl]()
