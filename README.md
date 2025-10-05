# Homelab Kubernetes Bootstrap

This repository provides a bootstrap setup for a home lab Kubernetes cluster using **ArgoCD** for GitOps and **Cert-Manager** for certificate management. It includes configurations for ingress, certificate management, and OVH DNS integration.

---

## Repository Structure

```
homelab-kubernetes-bootstrap/
├── core
│ ├── argocd
│ │ ├── ingress
│ │  └── argocd-ingress.yaml # Ingress configuration for ArgoCD
│ ├── cert-manager
│ │ ├── certificates
│ │  └── argocd.yaml # Certificate resources for ArgoCD
│ └── cert-manager-ovh-webhook
│ ├── sealed-ovh-credentials.yaml # Sealed secrets for OVH API credentials
│ └── values.yaml # Helm chart values for OVH webhook
├── LICENSE # Repository license
└── README.md # This file
```

---

## Prerequisites

Before deploying, ensure you have:

- A running Kubernetes cluster (any distribution)
- `kubectl` configured to access the cluster
- `helm` installed for managing Helm charts
- `kubeseal` if using sealed secrets
- Access to OVH API credentials (for DNS challenges)

---

## Deployment

1. **Install Cert-Manager**

```bash
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.18.2 --set installCRDs=true
```

2. **Install Sealed Secrets**
```bash
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install sealed-secrets -n kube-system --set-string fullnameOverride=sealed-secrets-controller sealed-secrets/sealed-secrets
```

3.   **Deploy OVH Webhook (for DNS challenge) and its secrets**
```bash
kubectl apply -f ./core/cert-manager-ovh-webhook/sealed-ovh-credentials.yaml
helm repo add cert-manager-webhook-ovh-charts https://aureq.github.io/cert-manager-webhook-ovh/
helm upgrade --install --namespace cert-manager -f ./core/cert-manager-ovh-webhook/values.yaml cm-webhook-ovh cert-manager-webhook-ovh-charts/cert-manager-webhook-ovh
```

4. **Deploy ArgoCD**

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl apply -f ./core/argocd/ingress/argocd-ingress.yaml
kubectl apply -f ./core/cert-manager/certificates/argocd.yaml
```

## Contributing

Contributions are welcome! Please follow standard Git workflow:


## License

This project is licensed under the terms of the LICENSE file.
