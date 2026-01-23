# Kubernetes (a.k.a K8s)
- Open-source container orchestration platform.
- Manages, scales, and runs containers reliably.
- Core responsibilities: scheduling containers, maintaining healthy containers, replacing dead/unhealthy containers.

---

**Problem without K8s**
- Each container often requires its own node/VM.
- Scaling new versions wastes resources (eg: 3 containers using 6 cores & 12GB RAM inefficiently).
- Low resource utilization, nodes are underused.

**How K8s helps resolve this**
- Efficiently packes *pods* onto nodes.
- A *pod* (smallest deployable unit in K8s) can contain *multiple* containers.
- Allows multiple versions and services to run on the same node.
- Improve horizontal scaling and resource usage.

---

## Kubernetes Architecture
- **Cluster**
    - Collection of *nodes* providing compute, memory, storage, and networking
    - Can consist fo multiple clusters in advanced setups

- **Nodes**
    - Worker nodes run *pods* and K8s components

- **Master Node / Control Plane**
    - The brain of the cluster; manages what happens and delegates to worker nodes
    - *Key components:*
        - **kube-api-server:** Entry point for administrative commands
        - **etcd:** Stores cluster data and state
        - **kube-controller-manager:** Ensures desired state matches reality
        - **kube-scheduler:** Decides which worker node pods run on
        - **cloud-controller-manager:** Connects cluster to cloud provider

- **Worker Node components**
    - **Kubelet** - Agent that ensures pods/containers are running
    - **Kube-proxy** - Handles networking, services, and pod communication

- **Pod**
    - Smallest deployable unit
    - Can contain 1 or more containers
    - Managed by K8s for scaling and orchastration


```
                                    ┌────────────────────────┐
                                    │       Control Plane    │
                                    │------------------------│
                                    │  API Server            │
                                    │  Scheduler             │
                                    │  Controller Manager    │
                                    │  etcd (data store)     │
                                    └─────────┬──────────────┘
                                              │
                            ┌─────────────────┴─────────────┐
                            │                               │
                        ┌─────────┐                     ┌─────────┐
                        │Worker   |                     | Worker  |
                        |Node 1   │                     │  Node 2 │
                        │---------│                     │---------│
                        │ Kubelet │                     │ Kubelet │
                        │ Kube-   │                     │ Kube-   │
                        │ proxy   │                     │ proxy   │
                        │ Runtime │                     │ Runtime │
                        │ Pods    │                     │ Pods    │
                        └─────────┘                     └─────────┘
```

Control Plane: the brain, decides what happens.

Nodes: the workers, actually run your applications in pods.

Kubelet: talks to control plane to run pods.

Kube-proxy: handles network communication.

---

## Creating a local Cluster
1. **Installation**
```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x kind
sudomv kind /usr/local/bin/

# kubectl
sudo snap install kubectl --classic
```

2. **Have Docker Desktop running**

3. **Create KIND cluster**
```
# Config file: conf.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker

# Create cluster
kind create cluster --name k8s-demo --config conf.yaml
```

4. **Verify cluster**
```
kubectl cluster-info
kubectl config use-context kind-k8s-demo
kubectl get nodes
```

5. **Work with pods**
- `kubectl rung nginx --image=nginx` -> Run nginx pod
- `kubectl get pods -w` -> Watch pod status
- `kubectl get pods -o wide` -> Shows which Node a pod is on
- `kubectl describe pod nginx` -> Check pod details

6. **Deployments**
```
kubectl create deployment nginx --image=nginx
kubectl get deployments
kubectl get pods
kubectl scale deployment nginx --replicas=3
```

7. **Delete cluster**
`kind delete cluster --name k8s-demo`

---

## Pods & YAML
- Pod is the smallest deployable unit in K8s
- Wraps *one or more* containers
- Containers in a pod are always scheduled together

**Pod Networking & Storage**
- Containers share the *same IP and port space*
- Communicate via the localhost
- Storage is *not* automatic; volumes must be explicitly mounted

**K8s YAML***
- Requires 4 top level fields:
    1. **apiVersion** - K8s api version
    2. **kind** - Type of object (pod, service etc)
    3. **metadata** - Name, labels, annotations, namespace
    4. **spec** - Desired state (containers, images, volumes etc)

- `containers` is a list because pods can run multiple containers
- Image defaults to `latest` if no tag is specified


**Creating Pods declaratively**
```
apiVersion:v1
kind: Pod
metadata:
    name: nginx-pod
    labels:
      app:nginx
spec:
 containers:
  - name: nginx-container
    image: nginx
    ports:
       - containerPort:80

```

**Creating Pods imperatively**
- `kubectl run nginx-imperative --image=nginx`

---

## K8s Deployments
- K8s controller managing *multiple pods*
- Ensures desired number of pods are always running
- Handles scaling, updates, and rollbacks
- Simplifies large-scale ops, automates pod creation and management, and supports scaling up/down without manual intervention

**Rolling update** - upgrade pods without downtime
**Rollbacks** - revert to previous version if an update fails

**K8s Deployment yaml**
- Structure:
    - **apiVersion**: `apps/v1` for deployments
    - **kind**: `Deployment`
    - **metadata**: Name, labels, namespace
    - **spec**:
        - **replicas**: Number of pods to maintain
        - **selector**: Matches deployment to pods via labels
        - **template**: Pod blueprint (nested pod spec)
            - Contains **metadata** (labels) and **spec** (containers, images, volumes)

- `kubectl apply -f <deployment.yaml>` creates or updates the deployment
- Deployment creates a **replica set**
- Replica set ensures the correct number of pods are running
- Pods are named based on deployment + unique identifiers

---

**Creating Deployments declaratively**
```
apiVersion:apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
    replicas:3
    selector:
     matchLabels:
      app: nginx
    template:
     metadata:
        labels:
          app:nginx
     spec:
      containers:
       - name: nginx
         image: nginx:latest
         ports:
           - containerPort: 80
```

**Creating Deployments imperatively**
- `kubectl create deployment nginx-deployment --image=nginx --replicas=3`