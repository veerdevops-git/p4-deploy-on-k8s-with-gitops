Before jenkins job build:

Add the dockerhub cred in Jenkins - this is for image upload to docker hub

add the github token in Jenkins - this is deployment update stage


CD:

Deploy the ArgoCD -> then generate the token - > add it on jenkins cred

Run this commond to provide admin to generate the token. 

kubectl patch configmap argocd-cm -n argocd --type merge -p '{"data":{"accounts.admin":"apiKey,login"}}' && kubectl rollout restart deployment argocd-server -n argocd



create application in ArgoCD:

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-application           # Your app name
  namespace: argocd              # ArgoCD namespace
spec:
  project: default               # ArgoCD project, usually "default"
  source:
    repoURL: 'https://github.com/veerdevops-git/p4-deploy-on-k8s-with-gitops.git'  # Your Git repo URL
    targetRevision: main        # Branch or tag to track
    path: 'manifest_file'       # Path inside repo where k8s manifests live
  destination:
    server: 'https://kubernetes.default.svc'  # Kubernetes API server (default cluster)
    namespace: default          # Namespace to deploy your app in
  syncPolicy:
    automated:
      prune: true              # Automatically delete resources that are no longer defined in Git
      selfHeal: true           # Automatically apply changes if drift detected
