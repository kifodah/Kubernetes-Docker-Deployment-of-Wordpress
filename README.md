# Kubernetes-Docker-Deployment-of-Wordpress

WordPress + MySQL deployed via Docker Compose (local) and Kubernetes (production).

## Quick Start — Docker Compose

```bash
.env        # fill in your passwords
docker compose up -d
open http://localhost:8080
```

## Quick Start — Kubernetes

```bash
# 1. Encode your passwords and update k8s/wordpress-secret.yml
echo -n 'yourpassword' | base64

# 2. Apply in order
kubectl apply -f k8s/wordpress-secret.yml
kubectl apply -f k8s/wordpress-pvc.yml
kubectl apply -f wordpressdb-deployment.yml
kubectl apply -f wordpressdb-service.yml

# 3. Verify
kubectl get pods,svc,pvc

# 4. Access
# http://<node-ip>:30020
```

## File Structure

```
├── docker-compose.yml            # Local dev stack
├── wordpressdb-deployment.yml    # K8s: MySQL + WordPress Deployments
├── wordpressdb-service.yml       # K8s: WordPress NodePort Service
├── k8s/
│   ├── wordpress-secret.yml      # K8s: credentials (replace base64 values)
│   └── wordpress-pvc.yml         # K8s: persistent storage for DB and uploads
├── .env                          # Local secrets (git-ignored)
├── .env.example                  # Template — safe to commit
└── .gitignore
```
