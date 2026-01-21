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



            ┌────────────────────────┐
            │       Control Plane    │
            │------------------------│
            │  API Server            │
            │  Scheduler             │
            │  Controller Manager    │
            │  etcd (data store)     │
            └─────────┬──────────────┘
                      │
      ┌───────────────┴───────────────┐
      │                               │
  ┌─────────┐                     ┌─────────┐
  │  Node 1 │                     │  Node 2 │
  │---------│                     │---------│
  │ Kubelet │                     │ Kubelet │
  │ Kube-   │                     │ Kube-   │
  │ proxy   │                     │ proxy   │
  │ Runtime │                     │ Runtime │
  │ Pods    │                     │ Pods    │
  └─────────┘                     └─────────┘
Control Plane: the brain, decides what happens.

Nodes: the workers, actually run your applications in pods.

Kubelet: talks to control plane to run pods.

Kube-proxy: handles network communication.

---