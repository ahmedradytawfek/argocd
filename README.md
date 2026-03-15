# k8s-argocd-setup

# 🚀 Argo CD on Kubernetes (Accessible from Any Node)

 This repository provides step-by-step instructions to **install and expose Argo CD** in a Kubernetes cluster, allowing access from master or any worker node using **NodePort** (or Ingress, optionally).

---
## 🧠 What is Argo CD?

[Argo CD](https://argo-cd.readthedocs.io/) is a declarative, GitOps continuous delivery tool for Kubernetes.  
It continuously monitors Git repositories and automatically applies changes to your cluster.

---

## ⚙️ Prerequisites

Before installation, make sure you have:
- A running **Kubernetes cluster** (v1.20+)
- `kubectl` access to the cluster
- **Internet access** from cluster nodes
- (Optional) `argocd` CLI installed on your local machine

To install the CLI:

  sudo curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
  
  sudo chmod +x /usr/local/bin/argocd
  
  argocd version

  ### 🪜 Step-by-Step Installation

#### 1️⃣ Create Namespace

kubectl create namespace argocd

#### 2️⃣ Install Argo CD Core Components

Apply the official stable manifest:

  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Check resources:

  kubectl get pods -n argocd

Wait until all are in Running state:

  argocd-application-controller

  argocd-repo-server
  
  argocd-server

  argocd-dex-server

#### 3️⃣ Expose Argo CD via NodePort

By default, argocd-server is a ClusterIP (only internal).
Change it to NodePort so it can be accessed from any node (master or worker):

  kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'


Check the new port:

  kubectl get svc -n argocd argocd-server


Example output:

   NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
  
   argocd-server   NodePort   10.43.126.153   <none>        80:30910/TCP     5m

  ##### ➡️ Here, 30910 is the NodePort.
You can now access Argo CD via:

http://any-cluster-node-ip:30910

#### 4️⃣ Get the Argo CD Admin Password

Argo CD creates a secret for the initial admin user:

   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; ech


#### 5️⃣ Log in to the Argo CD Web UI

Go to your browser:

http://any-node-ip:nodeport


Then:

   Username: admin

   Password: output from previous command
   
  
   
----


## 🔄 Connect Argo CD to GitHub (GitOps Integration)

Argo CD continuously watches your Git repository and applies Kubernetes manifests from it — this is the core of GitOps.

⚙️ Prerequisites

Before you begin:

Requirement	Example

    Argo CD installed	✅ already done
    
    Access to Argo CD Web UI or CLI	via NodePort or argocd CLI
    
    A GitHub repository	e.g. https://github.com/ahmed-devops/my-k8s-app
    
    Repo contains manifests	e.g. /manifests/deployment.yaml, /service.yaml

  ### 🪜 Step 1: Create or use a GitHub repo

Example repo structure:

my-k8s-app/

 ├── manifests/
 │   ├── deployment.yaml
 │   └── service.yaml
 └── README.md


Push this to GitHub:

https://github.com/<your-username>/my-k8s-app

### 🪜 Step 2: Access Argo CD UI

Open Argo CD in your browser:

http://node-ip:nodeport


Login with:

Username: admin
Password: (from `argocd-initial-admin-secret`)

### 🪜 Step 3: Add GitHub Repository to Argo CD
Option 1: via Web UI

Go to Settings → Repositories → Connect Repo

Choose Git

Fill in:

Repository URL:
https://github.com/<your-username>/my-k8s-app.git

Username: (optional if public)

Password / Personal Access Token: (if private repo)

Click Connect

✅ Argo CD will show a green checkmark once connected.


### 🪜  Test GitOps Flow

Make a change in your GitHub repo (e.g., update an image tag)

Commit and push:

  git commit -am "update image version"
  git push


Argo CD will detect the change and:

If auto-sync is enabled → redeploy automatically

If manual → show “Out of Sync” in UI (click Sync)

