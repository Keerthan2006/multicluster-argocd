# Multi-Cluster Kubernetes Deployment using ArgoCD (GitOps)

This project demonstrates a **multi-cluster Kubernetes deployment using ArgoCD and GitOps principles**.
A **hub cluster** runs ArgoCD and manages deployments to multiple **spoke clusters** automatically from a GitHub repository.

The application is deployed using **GitOps**, meaning any changes pushed to GitHub are automatically synchronized to Kubernetes clusters.

---

## 🚀 Project Overview

This project implements a **Hub-Spoke Multi-Cluster Architecture**:

* **Hub Cluster**

  * Runs ArgoCD
  * Controls deployments

* **Spoke Clusters**

  * Run applications
  * Managed by ArgoCD

ArgoCD continuously monitors the GitHub repository and deploys changes automatically.

---

## 🏗 Architecture

```
                GitHub Repository
                        │
                        ▼
                 ArgoCD (Hub Cluster)
                        │
          ┌─────────────┴─────────────┐
          │                           │
          ▼                           ▼
   Spoke Cluster 1             Spoke Cluster 2
     Guestbook App              Guestbook App
```

---

## 🧰 Tech Stack

* Kubernetes (k3d)
* ArgoCD
* GitOps
* Docker
* GitHub
* kubectl

---

## 📂 Project Structure

```
multicluster-argocd
│
├── manifests
│   └── guestbook
│       ├── deployment.yaml
│       ├── service.yaml
│
├── argocd
│   ├── spoke1.yaml
│   └── spoke2.yaml
│
└── README.md
```

---

## ⚙️ Setup Instructions

### 1️⃣ Create Multi-Cluster Environment

Create a shared Docker network:

```
docker network create k3d-multicluster
```

Create clusters:

```
k3d cluster create hub-cluster --servers 1 --agents 1 --network k3d-multicluster

k3d cluster create spoke1-cluster --servers 1 --agents 1 --network k3d-multicluster

k3d cluster create spoke2-cluster --servers 1 --agents 1 --network k3d-multicluster
```

Verify contexts:

```
kubectl config get-contexts
```

---

### 2️⃣ Install ArgoCD (Hub Cluster)

Switch to hub cluster:

```
kubectl config use-context k3d-hub-cluster
```

Create namespace:

```
kubectl create namespace argocd
```

Install ArgoCD:

```
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

### 3️⃣ Configure ArgoCD Access

Enable HTTP access:

```
kubectl edit configmap argocd-cmd-params-cm -n argocd
```

Add:

```
server.insecure: "true"
```

Restart ArgoCD:

```
kubectl rollout restart deployment argocd-server -n argocd
```

Change service type:

```
kubectl edit svc argocd-server -n argocd
```

Change:

```
type: NodePort
```

Access ArgoCD:

```
http://<NODE-IP>:<NODEPORT>
```

---

### 4️⃣ Login to ArgoCD

Get password:

```
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
```

Login:

```
argocd login <NODE-IP>:<NODEPORT> --insecure
```

---

### 5️⃣ Add Spoke Clusters

Register the spoke clusters with ArgoCD:

```
argocd cluster add k3d-spoke1-cluster

argocd cluster add k3d-spoke2-cluster
```

Verify clusters:

```
argocd cluster list
```

---

### ⚠️ Important: Fix kubeconfig API Server Address (If Needed)

Sometimes `argocd cluster add` may fail with an error like:

```
failed to get server version:
Get "https://0.0.0.0:<port>/version"
```

This happens because **k3d kubeconfig uses `0.0.0.0`, which is not reachable from the ArgoCD hub cluster container.**

Since ArgoCD runs inside Kubernetes, it must connect to the spoke clusters using **reachable API server addresses.**

#### Fix Steps

Open kubeconfig:

```
nano ~/.kube/config
```

Find the spoke cluster configuration:

```
name: k3d-spoke1-cluster
```

Change the server address from:

```
server: https://0.0.0.0:<PORT>
```

to:

```
server: https://k3d-spoke1-cluster-server-0:6443
```

Do the same for:

```
k3d-spoke2-cluster
```

After updating kubeconfig, run the commands again:

```
argocd cluster add k3d-spoke1-cluster
argocd cluster add k3d-spoke2-cluster
```

This ensures ArgoCD can communicate with the Kubernetes API servers in each spoke cluster.


## 🔐 ArgoCD Cluster Access

When a cluster is added, ArgoCD automatically creates:

* ServiceAccount → **argocd-manager**
* ClusterRole
* ClusterRoleBinding

The **argocd-manager ServiceAccount** allows ArgoCD to deploy applications into spoke clusters.
A ServiceAccount provides an identity for applications like ArgoCD to authenticate with the Kubernetes API and perform operations such as deployments and synchronization.

---

## 🚀 Deploy Applications

Deploy Guestbook app to Spoke Cluster 1:

```
kubectl apply -f argocd/spoke1.yaml
```

Deploy Guestbook app to Spoke Cluster 2:

```
kubectl apply -f argocd/spoke2.yaml
```

Verify deployment:

```
kubectl get pods --context k3d-spoke1-cluster

kubectl get pods --context k3d-spoke2-cluster
```

---

## 🔄 GitOps Workflow

```
Developer Push → GitHub → ArgoCD → Kubernetes Clusters
```

1. Code is pushed to GitHub
2. ArgoCD detects changes
3. Applications automatically sync
4. Clusters are updated

---

## ⭐ Key Features

* Multi-cluster Kubernetes deployment
* Hub-Spoke Architecture
* GitOps deployment using ArgoCD
* Automatic synchronization
* Self-healing applications
* Shared cluster network
* Real-time deployment updates

---

## 📊 Learning Outcomes

This project demonstrates:

* Kubernetes Multi-Cluster Architecture
* ArgoCD GitOps workflows
* Cluster registration in ArgoCD
* Kubernetes networking
* Automated deployments


---

## 👨‍💻 Author

Keerthan

DevOps Enthusiast

