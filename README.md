# 🐳 Kubernetes-Docker-Deployment-of-Wordpress

> A production-ready WordPress + MySQL stack deployed with Docker Compose and Kubernetes featuring persistent storage, secret management, and modular service configuration.

![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)
![WordPress](https://img.shields.io/badge/WordPress-21759B?style=flat&logo=wordpress&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat&logo=mysql&logoColor=white)

---

## Overview

This project provisions a fully containerised WordPress application backed by MySQL database. It supports two deployment modes:

- **Docker Compose** — for local development and quick demos
- **Kubernetes** — for scalable, production-grade deployments with persistent volumes and Kubernetes Secrets

---

## Architecture

```
                    ┌─────────────────────────┐
                    │      Ingress / LB        │
                    └──────────┬──────────────┘
                               │
                    ┌──────────▼──────────────┐
                    │   WordPress Deployment   │
                    │   (wordpressdb-deploy)   │
                    └──────────┬──────────────┘
                               │
                    ┌──────────▼──────────────┐
                    │    MySQL Deployment      │
                    │   + PersistentVolume     │
                    └─────────────────────────┘
```

---

## Prerequisites

| Tool | Minimum Version |
|------|----------------|
| Docker | 24.x |
| Docker Compose | v2.x |
| kubectl | 1.27+ |
| Kubernetes cluster | 1.27+ (minikube) |

---

## Quick Start — Docker Compose

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/Kubernetes-Docker-Deployment-of-Wordpress.git
cd Kubernetes-Docker-Deployment-of-Wordpress

# 2. Copy and fill in environment variables
cp .env.example .env
# Edit .env with your values

# 3. Start the stack
docker compose up -d

# 4. Visit WordPress in your browser
open http://localhost:8080
```

---

## Kubernetes Deployment

### 1. Create the Namespace (optional but recommended)

```bash
kubectl create namespace wordpress
```

### 2. Apply Kubernetes Secrets

Never store plain-text passwords in YAML. Use the secret manifest instead:

```bash
# Edit k8s/wordpress-secret.yml with your base64-encoded values first
# Encode a value:  echo -n 'yourpassword' | base64

kubectl apply -f k8s/wordpress-secret.yml -n wordpress
```

### 3. Apply PersistentVolumeClaims

```bash
kubectl apply -f k8s/wordpress-pvc.yml -n wordpress
```

### 4. Deploy the Application

```bash
kubectl apply -f wordpressdb-deployment.yml -n wordpress
kubectl apply -f wordpressdb-service.yml -n wordpress
```

### 5. Verify the Deployment

```bash
kubectl get pods -n wordpress
kubectl get svc -n wordpress
```

---

## Environment Variables

Copy `.env.example` to `.env` and fill in your values before running locally.

| Variable | Description | Example |
|----------|-------------|---------|
| `MYSQL_ROOT_PASSWORD` | MySQL root password | `strongRootPass123` |
| `MYSQL_DATABASE` | Name of the WordPress database | `wordpress` |
| `MYSQL_USER` | MySQL app user | `wp_user` |
| `MYSQL_PASSWORD` | MySQL app user password | `strongUserPass456` |
| `WORDPRESS_DB_HOST` | MySQL service hostname | `mysql:3306` |
| `WORDPRESS_DB_NAME` | Must match `MYSQL_DATABASE` | `wordpress` |
| `WORDPRESS_DB_USER` | Must match `MYSQL_USER` | `wp_user` |
| `WORDPRESS_DB_PASSWORD` | Must match `MYSQL_PASSWORD` | `strongUserPass456` |

> ⚠️ **Never commit `.env` or any file with real credentials to version control.**

---

## Persistent Storage

Database data is stored in a `PersistentVolumeClaim` so it survives pod restarts and rescheduling.

```
k8s/
└── wordpress-pvc.yml    # PVC definitions for MySQL data
```

The MySQL pod mounts the PVC at `/var/lib/mysql`. Without this, all data is lost every time the pod is restarted.

---

## Secret Management

Kubernetes Secrets replace hard-coded passwords in deployment YAML:

```yaml
# k8s/wordpress-secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: wordpress-secret
type: Opaque
data:
  mysql-root-password: <base64-encoded>
  mysql-password: <base64-encoded>
```

Deployments reference secrets via `secretKeyRef` instead of plain `value:` fields, keeping credentials out of source control entirely.

---

## File Structure

```
wordpress-db-application/
├── docker-compose.yml          # Local dev stack
├── wordpressdb-deployment.yml  # Kubernetes Deployment manifest
├── wordpressdb-service.yml     # Kubernetes Service manifest
├── k8s/
│   ├── wordpress-secret.yml    # Kubernetes Secret (do not commit real values)
│   └── wordpress-pvc.yml       # PersistentVolumeClaim for MySQL
├── .env.example                # Template for required environment variables
├── .gitignore                  # Excludes .env and sensitive files
├── LICENSE                     # MIT License
└── README.md
```

---

## Contributing

Contributions are welcome! Please open an issue or pull request.

1. Fork the repository
2. Create a feature branch (`git checkout -b feat/my-feature`)
3. Commit your changes (`git commit -m 'feat: add my feature'`)
4. Push and open a Pull Request

Please ensure you never commit real credentials, use `.env.example` as the template.
