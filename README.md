# Conoa-Kargo-Demo

# ArgoCD install
### Skapa namespace för Argo CD
kubectl create namespace argocd

### Installera senaste stabila versionen
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

### Vänta tills alla pods körs
kubectl -n argocd get pods

### Exponera Argo CD server lokalt (port-forward)
kubectl port-forward svc/argocd-server -n argocd 8080:443

### Plocka ut ArgoCD admin lösenord
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo

### Logga in i Argo CD CLI (default-användaren är "admin")
argocd login localhost:8080 --username admin --password <initial-lösenord>





# Argo Rollout install
### Skapa namespace
kubectl create namespace argo-rollouts

### Installera senaste release
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

### Kontrollera status
kubectl -n argo-rollouts get pods




# Installera cert-manager
### Skapa namespace
kubectl create namespace cert-manager

### Installera med den officiella Helm-charten
helm repo add jetstack https://charts.jetstack.io
helm repo update

### Installera cert-manager (inklusive CRDs)
helm install cert-manager jetstack/cert-manager \
  --create-namespace \
  --namespace cert-manager \
  --set installCRDs=true

### Kontrollera status
kubectl -n cert-manager get pods



# Installera Kargo

### Skapa namespace för Kargo
kubectl create namespace kargo

### Installera med Helm (rekommenderat)
helm install kargo \
  oci://ghcr.io/akuity/kargo-charts/kargo \
  --namespace kargo \
  --create-namespace \
  --set api.adminAccount.passwordHash='$2a$10$Zrhhie4vLz5ygtVSaif6o.qN36jgs6vjtMBdM6yrU1FOeiAAMMxOm' \
  --set api.adminAccount.tokenSigningKey=iwishtowashmyirishwristwatch \
  --wait

### Kontrollera status
kubectl -n kargo get pods

### Exponera Kargo server lokalt (port-forward)
kubectl port-forward --namespace kargo svc/kargo-api 3000:443

### Ladda ner Kargo CLI (Mac/Linux exempel)
wget https://github.com/akuity/kargo/releases/download/v1.7.5/kargo-darwin-arm64
chmod +x kargo-darwin-arm64
sudo cp kargo-darwin-arm64 /usr/local/bin/kargo

### Testa
kargo version

### Logga in i Kargo
kargo login --admin --insecure-skip-tls-verify https://localhost:3000/

### Skapa git credentials
kargo create credentials github-creds \
  --project kargo-demo \
  --git \
  --username trappiz \
  --repo-url https://github.com/trappiz/conoa-kargo-demo.git
