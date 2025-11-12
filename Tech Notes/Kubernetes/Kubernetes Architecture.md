## What is Kubernetes Architecture?

At its core, a **Kubernetes cluster** consists of two main parts:

- **Control Plane**: The brain of the cluster, responsible for managing and coordinating everything.
- **Worker Nodes**: The workforce that executes the actual application workloads.

![[kubernetes-cluster-architecture.svg]]

## Control Plane: The Management Hub

The control plane is responsible for making global decisions—such as scheduling new workloads—and responding to events, like restarting a failed container. It’s composed of several key components, each with a distinct role:

### 1. kube-apiserver

What it does: The kube-apiserver is the entry point to the Kubernetes cluster. It exposes the Kubernetes API, allowing users, tools (like kubectl), and other components to interact with the cluster.

Details: It validates and processes API requests, such as creating a pod or scaling a deployment. It’s designed to scale horizontally, meaning you can run multiple instances for redundancy and load balancing.

### 2. etcd

What it does: etcd is a distributed key-value store that holds all cluster data, like configuration, state, and metadata.

Details: Highly consistent and available, etcd ensures the cluster’s state is preserved. If it fails, you lose the cluster’s “memory,” so backups are critical.

### 3. kube-scheduler

What it does: The kube-scheduler assigns newly created pods to nodes based on resource needs, constraints, and policies.

Details: It considers factors like CPU/memory requests, node affinity, and workload interference to optimize placement.

### 4. kube-controller-manager

What it does: This runs various controllers through *apiserver* that monitor the cluster and maintain its desired state.

**Details**: Examples include:
- Node controller: Responsible for noticing and responding when nodes go down.
- Job controller: Watches for Job objects that represent one-off tasks, then creates Pods to run those tasks to completion.
- EndpointSlice controller: Populates EndpointSlice objects (to provide a link between Services and Pods).
- ServiceAccount controller: Create default ServiceAccounts for new namespaces.


## Node components

Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

- A **node** is a worker machine (physical or virtual) in the Kubernetes cluster that provides the compute resources (CPU, memory, storage) to execute workloads.
- A **pod** is the smallest deployable unit in Kubernetes, typically containing one or more containers that share resources like networking and storage. Pods are scheduled and managed by Kubernetes to run on nodes.

### 1. kubelet

What it does: The kubelet is an agent on each node that ensures containers within pods are running and healthy.
Details: It takes pod specifications (PodSpecs) from the API server and manages the container lifecycle, but it doesn’t handle non-Kubernetes containers.