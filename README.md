# Argocd-Landing

A tiny "static landing page" app packaged as:
- **Docker image** (serves `site/index.html` via NGINX)
- **Helm chart** (Deployment/Service/Ingress)
- **Argo CD Application** manifest (so ArgoCD can sync it)

## What ArgoCD does (and does NOT do)
- ✅ ArgoCD **deploys** Kubernetes manifests (Helm/Kustomize/YAML) from Git.
- ❌ ArgoCD does **not** run your Makefile, and it does **not** build/push Docker images.

So the flow is:
1) Build + push image to DOCR
2) Update `helm/landing/values-prod.yaml` with the image tag
3) Apply `argocd/application.yaml` (or manage it from your infra repo)
4) ArgoCD syncs → app appears in the cluster

---

## 1) Build and push the image to DigitalOcean Container Registry (DOCR)

```bash
REGISTRY="registry.digitalocean.com/guajiro"
IMAGE="landing-static"
TAG="v0.1.0"

doctl registry login
docker build -t ${REGISTRY}/${IMAGE}:${TAG} .
docker push ${REGISTRY}/${IMAGE}:${TAG}
```

---

## 2) Configure the Helm chart for your domain

Edit:
- `helm/landing/values-prod.yaml`

Update at least:
- `image.repository`
- `image.tag`
- `ingress.hosts`
- `ingress.tls[0].hosts`

---

## 3) Create the ArgoCD Application

1) Edit `argocd/application.yaml` and set:
   - `spec.source.repoURL` (your GitHub repo URL)
2) Apply:
```bash
kubectl apply -f argocd/application.yaml
```

---

## 4) Quick checks

```bash
kubectl -n demo get deploy,svc,ingress
kubectl -n demo describe ingress landing
```

If you use cert-manager + external-dns in the cluster, TLS + DNS should work after a few minutes.

---

## Notes
- Namespace: `demo`
- Ingress class: `nginx`
- TLS secret: `landing-tls` (created by cert-manager once validated)
