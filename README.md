# GKE Kubernetes Manifests

Kubernetes manifests for deploying the Color application on Google Kubernetes Engine with Kustomize overlays for `dev` and `prod`.

## Structure

```text
.
├── color-api/          # API deployment, service, ingress, certificates, and policies
├── color-db/           # MongoDB StatefulSet, service, init script, and policies
├── namespaces/         # Namespace manifests for dev and prod
└── shared-config/      # Shared network policies and generated MongoDB secrets
```

## Environments

- `dev` deploys to the `dev` namespace.
- `prod` deploys to the `prod` namespace.

Each application has a shared `_base` directory and environment-specific overlays:

```text
color-api/dev
color-api/prod
color-db/dev
color-db/prod
shared-config/dev
shared-config/prod
```

## Prerequisites

- A configured GKE cluster context.
- `kubectl` with Kustomize support.
- Static IP addresses for the API ingresses:
  - `color-api-dev`
  - `color-api-prod`
- DNS records pointing to the ingress IPs:
  - `dev.gke.appcraft.ing`
  - `gke.appcraft.ing`
- Environment files for generated MongoDB secrets:
  - `shared-config/dev/.env.root-creds.dev`
  - `shared-config/dev/.env.colordb-creds.dev`
  - `shared-config/prod/.env.root-creds.prod`
  - `shared-config/prod/.env.colordb-creds.prod`

## Deploy

Create namespaces first:

```bash
kubectl apply -f namespaces/dev.yaml
kubectl apply -f namespaces/prod.yaml
```

Deploy shared config, database, and API for dev:

```bash
kubectl apply -k shared-config/dev
kubectl apply -k color-db/dev
kubectl apply -k color-api/dev
```

Deploy shared config, database, and API for prod:

```bash
kubectl apply -k shared-config/prod
kubectl apply -k color-db/prod
kubectl apply -k color-api/prod
```

## Validate

```bash
kubectl get all -n dev
kubectl get ingress -n dev
kubectl get managedcertificate -n dev

kubectl get all -n prod
kubectl get ingress -n prod
kubectl get managedcertificate -n prod
```

## Notes

- The API is exposed through GCE ingress and GKE managed certificates.
- MongoDB credentials are generated from environment files by Kustomize secret generators.
- Shared network policies are applied before workloads so namespace traffic rules are in place.
