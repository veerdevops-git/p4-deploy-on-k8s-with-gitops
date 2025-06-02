

# 🚀 CI/CD Setup Instructions

Follow these steps before running your Jenkins pipeline to ensure a smooth CI/CD process.

---

## 🧩 Jenkins Configuration (Before Running the Job)

### 1. 🔌 Install Required Plugin
- Go to **Jenkins → Manage Jenkins → Plugin Manager**
- Install: `Stage View` plugin (for better pipeline visualization)

### 2. 🔐 Add Credentials in Jenkins

#### ✅ DockerHub Credentials (for pushing images)
- Go to: **Jenkins → Manage Jenkins → Credentials**
- Add:
  - **Kind**: Username and Password
  - **ID**: `dockerhub-credentials-id`
  - **Username**: Your DockerHub username
  - **Password**: Your DockerHub password

#### ✅ GitHub Token (for updating the deployment manifest)
- Go to: **Jenkins → Manage Jenkins → Credentials**
- Add:
  - **Kind**: Secret Text
  - **ID**: `github`
  - **Secret**: Your GitHub [Personal Access Token (PAT)](https://github.com/settings/tokens)

---

## 🎯 CD: ArgoCD Setup

### 1. ⚙️ Deploy ArgoCD on Kubernetes (if not already deployed)
> Refer to [official docs](https://argo-cd.readthedocs.io/en/stable/getting_started/) for ArgoCD install steps.

### 2. 🔑 Enable Token-Based Authentication for Admin

Run the following command to enable **admin token generation** in ArgoCD:

```bash
kubectl patch configmap argocd-cm -n argocd --type merge -p '{"data":{"accounts.admin":"apiKey,login"}}' && kubectl rollout restart deployment argocd-server -n argocd


🧱 Create ArgoCD Application

Apply the following YAML to create the ArgoCD Application resource:

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-application
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/veerdevops-git/p4-deploy-on-k8s-with-gitops.git'
    targetRevision: main
    path: 'manifest_file'
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true


=====


✅ Final Checklist
Task	Status
Stage View Plugin Installed	⬜
DockerHub Credentials Added	⬜
GitHub Token Added	⬜
ArgoCD Deployed on Cluster	⬜
Admin Token Enabled and Created	⬜
ArgoCD Application Created	⬜