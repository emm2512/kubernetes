## What is kubernetes?
Kubernetes (K8s) is an open-source container orchestration platform that help to automate the deployment, scaling, and management of containerized applications.

## Why use kubernetes?
- Companies moved from **physical servers → VMs → containers → Kubernetes** to make applications run faster, more efficient (optimize resources usage), and easier to manage at scale.
- companies are moving from **monolithic** app to **microservices** architecture increasing the usage of containers and the need to efficiently manage them in various environments.
## Kubernetes architecture
Kubernetes has a **Control plane** - **Worker** architecture:
### Control plane:
The control plane manages the worker nodes and the pods in the cluster. It contains:
- **API Server** – Entry point for commands (talks to users & other parts).
- **Scheduler** – Decides where to run new containers.
- **Controller Manager** – Handles tasks like scaling & failures.
- **etcd** – Stores all cluster data (like a database).
- **Cloud controller manager**  - ( for cloud cluster. this is in case kubernetes needs to create ressources in the cloud )

### Worker node (nodes):

- **Kubelet** – Talks to the control plane to run and manage containers.
- **Container Runtime** – Runs containers (e.g., Docker, containerd).
- **Kube Proxy** – Manages network communication between containers.

NB: control plane node also has **kubelet, kube-proxy and container runtime** because it runs some system pods 
### Pods
Smallest unit in K8s, holds one or more containers.

## Kubernetes Plugins or Kubernetes Interfaces
   - **CRI** → Manages containers
   - **CNI** → Handles networking
   - **CSI** → Manages storage.

1. CRI (Container Runtime Interface)
- Enables Kubernetes to use different container runtimes (e.g., containerd, CRI-O, Docker).
- Defines how Kubernetes interacts with the runtime to manage containers.
- Decouples Kubernetes from specific runtimes for flexibility.

2. CNI (Container Network Interface)
- Standard for configuring networking in Kubernetes pods.
- Plugins (e.g., Calico, Flannel, Cilium) handle IP allocation, routing, and network policies.
- Ensures pods can communicate within and outside the cluster.

3. CSI (Container Storage Interface)
- Allows Kubernetes to integrate with external storage systems (e.g., AWS EBS, NFS, Ceph).
- Standardizes how storage is provisioned, attached, and mounted to pods.
- Enables dynamic volume provisioning and snapshots.

## Key features of kubernetes
- **Autoscaling** – Increases or decreases containers based on traffic.
- **Self-Healing** – Restarts failed containers, replaces unhealthy ones.
- **Load Balancing** – Distributes traffic across containers for efficiency.
- **Rolling Updates & Rollbacks** – Updates apps without downtime and rolls back if needed (high availability)
- **Service Discovery** – Finds and connects services without manual setup (using IP adress or DNS).
- **Storage Management** – Supports local, cloud, or network storage easily.
- **Multi-Cloud & Hybrid Support** – Runs across AWS, Google Cloud, Azure, and on-prem.
- **Automated Deployments** – Uses YAML files for easy and repeatable deployments.
- **Secrets & Config Management** – Manages configuration and sensitive data securely.
- **Resource Optimization** – Ensures best use of CPU, memory, and storage (automatic bin packing)
- **production Support**– Can be used for production environments

## K8s cluster setup methods
**Note:** In this class, we will use kubernetes playgrounds on Killercoda for testing and deploy our production cluster in AWS EKS. The following are just for more information on various methods available.

### For learning and local development/testing
- **Minikube** – Runs a single-node K8s cluster on your laptop. You can check the [official documentation](https://minikube.sigs.k8s.io/docs/) for minikube just to get more information.
- **Kind (Kubernetes in Docker)** – Uses Docker to run lightweight K8s clusters. 
- **K3s** – A lightweight Kubernetes distribution for low-resource systems.
- **MicroK8s** – A lightweight K8s version from Canonical.
- **Killercoda playgroungs for Kubernetes**: Killercoda is a platform where you get instant access to a real Linux or Kubernetes environment ready to use. **Note: In the cluster setup for this class, you will learn how to use it. We will use it for some practices**

### For production
#### 1. Manual setup (unmanaged)
- using Kubeadm
- Using kubespray
- Using Kops
- From scratch (complex)

#### 2.  Cloud based (managed)
- Amazon EKS – Kubernetes on AWS. **Note: This is what we will use in this class**
- Google GKE – Kubernetes on Google Cloud.
- Azure AKS – Kubernetes on Microsoft Azure.
- Oracle OKE - Kubernetes on Oracle
- LKE - Kubernetes on Linode
- ...

#### 3. Enterprise Kubernetes (K8s distributions)
- Openshift – Red Hat’s enterprise Kubernetes with extra features.
- Rancher – Multi-cluster Kubernetes management.
- Tanzu - vSphere Tanzu Kubernetes Grid (TKG)
- ...

## Interacting with cluster
- **Command Line Interface (CLI)**: using `kubectl` commands
- **User Interface (UI)**: using the native **K8s Dashboard** or tools like **Lens**
- **Kubernetes API**: Using programmatical access (ex. with python scripts, terraform codes etc.)


## How to use this repo
The repo is organized and ordered for you to practice the concepts progressively.

In each folder, you will find a readme file to help you understand the concept and practice.

**Note**: All the files mentioned in the **readme** can be directly found in the same folder. Just use the content of those files if no other specification is given.
