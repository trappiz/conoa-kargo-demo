# Conoa-Kargo-Demo

## ArgoCD install
# Skapa namespace för Argo CD
kubectl create namespace argocd

# Installera senaste stabila versionen
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Vänta tills alla pods körs
kubectl -n argocd get pods

# Exponera Argo CD server lokalt (port-forward)
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Plocka ut ArgoCD admin lösenord
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo

# Logga in i Argo CD CLI (default-användaren är "admin")
argocd login localhost:8080 --username admin --password <initial-lösenord>

## Installera Kargo

# Skapa namespace för Kargo
kubectl create namespace kargo

# Generera secrets
pass=$(openssl rand -base64 48 | tr -d "=+/" | head -c 32)
echo "Password: $pass"
hashed_pass=$(htpasswd -bnBC 10 "" $pass | tr -d ':\n')
signing_key=$(openssl rand -base64 48 | tr -d "=+/" | head -c 32)

# Installera med Helm (rekommenderat)
helm install kargo \
  oci://ghcr.io/akuity/kargo-charts/kargo \
  --namespace kargo \
  --create-namespace \
  --set api.adminAccount.passwordHash=$hashed_pass \
  --set api.adminAccount.tokenSigningKey=$signing_key \
  --wait
# Kontrollera status
kubectl -n kargo get pods

# Ladda ner Kargo CLI (Mac/Linux exempel)
curl -sL https://github.com/akuity/kargo/releases/latest/download/kargo-$(uname -s)-amd64 -o /usr/local/bin/kargo
chmod +x /usr/local/bin/kargo

# Testa
kargo version
