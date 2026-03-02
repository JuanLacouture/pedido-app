# Pedido-App Parcial I

## Helm Manual
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install pedido-dev ./charts/pedido-app -n pedido-dev --create-namespace -f values-dev.yaml

Endpoints:
- Frontend: minikube service pedido-dev-pedido-app-frontend
- Backend API: http://pedido.local/api
- DB: postgresql://pedidos_user:userpass123@pedido-dev-postgresql

## ArgoCD GitOps
Apply application-dev.yaml/prod.yaml en ArgoCD UI/CLI.

Demo: Cambia réplicas en values-dev.yaml → git push → ArgoCD auto-sync.
