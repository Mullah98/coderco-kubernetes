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
                               │      Control Plane     │
                               │------------------------│
                               │  API Server            │
                               │  Scheduler             │
                               │  Controller Manager    │
                               │  etcd (data store)     │
                               └─────────┬──────────────┘
                                         │
                           ┌─────────────┴─────────────┐
                           │                           │
                       ┌─────────┐                 ┌─────────┐
                       │ Worker  │                 │ Worker  │
                       │ Node 1  │                 │ Node 2  │
                       │---------│                 │---------│
                       │ Kubelet │                 │ Kubelet │
                       │ Kube-   │                 │ Kube-   │
                       │ proxy   │                 │ proxy   │
                       │ Runtime │                 │ Runtime │
                       │---------│                 │---------│
                       │ Pods    │                 │ Pods    │
                       │ - nginx │                 │ - nginx │
                       │ - api   │                 │ - api   │
                       └─────────┘                 └─────────┘
```

- **Control Plane:** the brain, decides what happens.
- **Nodes:** the workers, actually run your applications in pods.
- **Kubelet:** talks to control plane to run pods.
- **Kube-proxy:** handles network communication.

---

Here’s a clean, compact one-page cheat sheet from your notes:

---

## KIND setup

**1. Local Cluster Setup**

**Install KIND & kubectl (Ubuntu):**
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/

sudo snap install kubectl --classic
```

**Docker must be running** before KIND.

**Create KIND cluster:**
`conf.yaml` example:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

```bash
kind create cluster --name k8s-demo --config conf.yaml
```

**Verify cluster:**
```bash
kubectl cluster-info
kubectl config use-context kind-k8s-demo
kubectl get nodes
```

**Delete cluster:**
```bash
kind delete cluster --name k8s-demo
```

---

**2. Pods**
- Smallest deployable unit; can contain multiple containers.
- Containers share **network namespace** (localhost) and must mount volumes explicitly.

**Imperative pod:**

```bash
kubectl run nginx --image=nginx
kubectl get pods -w
kubectl describe pod nginx
kubectl get pods -o wide   # shows node placement
```

**Declarative pod YAML:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
```

```bash
kubectl apply -f nginx-pod.yaml
```

---

## **Deployments**
- Controller managing multiple pods: scaling, updates, rollbacks.
- Top-level `metadata` → Deployment info; `spec.template` → pod blueprint.

**Imperative deployment:**
```bash
kubectl create deployment nginx-deployment --image=nginx --replicas=3
kubectl get deployments
kubectl get pods
kubectl scale deployment nginx-deployment --replicas=5
```

**Declarative deployment YAML:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

```bash
kubectl apply -f nginx-deployment.yaml
```

**Tips:**
- `kubectl get pods -o wide` → see node placement.
- Deleting a Deployment removes all its pods.
- `spec.selector` must match `spec.template.metadata.labels`.

---

## ReplicaSets vs Deployments
| Feature / Concept       | ReplicaSet                                             | Deployment                                                       |
| ----------------------- | ------------------------------------------------------ | ---------------------------------------------------------------- |
| **Primary Role**        | Ensures a specific number of pod instances are running | Manages ReplicaSets and the desired state of pods                |
| **Pod Replacement**     | Automatically replaces failed pods                     | Handles pod replacement via underlying ReplicaSets               |
| **Updates & Rollbacks** | Not supported                                          | Supports **rolling updates** and **rollbacks**                   |
| **Version Management**  | Manages pods of a single version                       | Can manage multiple ReplicaSets for different app versions       |
| **Complexity**          | Simple, low-level controller                           | Higher-level abstraction with more features                      |
| **Use Case**            | Maintain pod count                                     | Easier scaling, updates, and management for production workloads |

---

## Services
- To connect pods internally or externally
- Allows loose coupling between application components
- Services use *labels* to select the correct pods
- Operates at Layer 3 (Network) of OSI model

**Types of Services**
- **ClusterIP (default)** 
  - Provides an internal IP for communication within the cluster only
  - Not accessible externally -> secure by default
  - Ideal for internal services like backend or database Pods

- **NodePort**
  - Opens a specific port on every node
  - Maps external traffic to internal Pod target port
  - Port range: 30000-32767
  - Useful for simple external access or local testing

- **Load Balancer**
  - Creates an external cloud load balancer
  - Distributes traffic across Pods automatically
  - Needed for production external access in cloud environments
  - Works with AWS, Azure, GCP

*ClusterIP -> internal only, NodePort -> external via nde port, Load Balancer -> full external cloud exposure.*

---