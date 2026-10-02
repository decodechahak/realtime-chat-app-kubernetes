# 💬 Realtime Chat App — Deployed on Kubernetes

A full-stack real-time chat application, containerized with Docker and deployed on Kubernetes — built as a hands-on DevOps project covering multi-service orchestration, secrets management, and production-style debugging.

>Based on the open-source full-stack chat app originally created by Burak (MIT licensed), followed through Train With Shubham's tutorial series. This repository adds my own Kubernetes deployment work: PV/PVC storage, Secrets handling, ingress and debugging on a local kind cluster.

---

## 🏗️ Architecture

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│  Frontend   │─────▶│   Backend   │─────▶│   MongoDB   │
│ (React/Vite │      │ (Node.js/   │      │             │
│  + Nginx)   │      │  Express)   │      │             │
└─────────────┘      └─────────────┘      └─────────────┘
     Port 80            Port 5001            Port 27017
```

All three services run as separate Kubernetes Deployments inside a dedicated `chat-app` namespace, communicating over internal ClusterIP Services.

---

## 🛠️ Tech Stack

- **Frontend:** React + Vite, served via Nginx (multi-stage Docker build)
- **Backend:** Node.js + Express, JWT-based authentication
- **Database:** MongoDB (StatefulSet-style Deployment with persistent storage)
- **Containerization:** Docker (multi-stage builds for optimized image size)
- **Orchestration:** Kubernetes (deployed locally via `kind`)
- **Networking:** Ingress with custom local domain routing
- **Secrets Management:** Kubernetes Secrets for JWT and DB credentials

---

## ☸️ Kubernetes Resources

| Resource | Purpose |
|---|---|
| `namespace.yaml` | Isolated `chat-app` namespace |
| `mongodb-deployment.yaml` / `mongodb-pv.yaml` / `mongodb-pvc.yaml` | MongoDB with persistent volume storage |
| `mongodb-service.yaml` | Internal DB service discovery |
| `backend-deployment.yaml` / `backend-service.yaml` | API server, connects to MongoDB via env-injected URI |
| `frontend-deployment.yaml` / `frontend-service.yaml` | Nginx-served static frontend |
| `secrets.yaml` | JWT secret, DB credentials |
| `ingress.yaml` | Custom domain routing (`chatty-ca.com` → services) |

---

## 🚀 Running Locally

### Prerequisites
- Docker Desktop
- [kind](https://kind.sigs.k8s.io/)
- `kubectl`

### Steps

```bash
# 1. Create a kind cluster
kind create cluster --name my-kind-cluster

# 2. Apply all Kubernetes manifests
cd k8s
kubectl apply -f namespace.yaml
kubectl apply -f secrets.yaml
kubectl apply -f mongodb-pv.yaml
kubectl apply -f mongodb-pvc.yaml
kubectl apply -f mongodb-deployment.yaml
kubectl apply -f mongodb-service.yaml
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml

# 3. Verify all pods are running
kubectl get pods -n chat-app

# 4. Port-forward to access locally
kubectl port-forward service/frontend -n chat-app 80:80
kubectl port-forward service/backend -n chat-app 5001:5001
```

Then visit **http://localhost** in your browser.

*(Optional)* For custom domain access via Ingress, add `127.0.0.1 chatty-ca.com` to your system's hosts file and install an [nginx-ingress controller](https://kubernetes.github.io/ingress-nginx/deploy/#kind) for `kind`.

---

## ✅ What's Working

- Full container build pipeline (multi-stage Dockerfiles for frontend and backend)
- Kubernetes deployment across all three tiers
- Backend ↔ MongoDB connectivity via securely injected environment variables
- JWT-based user authentication
- Secrets management (no hardcoded credentials)
- Ingress-based custom domain routing
- Recovered from and debugged real infrastructure failures: `CrashLoopBackOff`, stale Docker images, broken `kind` cluster networking, misconfigured Nginx serving

## 🚧 Known Issues / Roadmap

- **Real-time chat sync between two users is not yet fully functional** — messages and online presence don't reliably reflect across sessions. Currently investigating whether this is a WebSocket/Socket.IO CORS configuration issue between the separately-hosted frontend and backend.
- Ingress controller setup for `kind` requires manual installation (not automated in manifests yet)
- No CI/CD pipeline wired up yet (planned: GitHub Actions for build/push/deploy automation)

---

## 📚 What I Learned

This project was as much about **Kubernetes operations and debugging** as it was about the application itself:

- Diagnosing and fixing `CrashLoopBackOff` pods via `kubectl logs` and `kubectl describe`
- Managing environment variables, Secrets, and ConfigMaps correctly across services
- Understanding the difference between Docker build stages and catching a stale/mismatched image bug
- Recovering a broken `kind` cluster (corrupted Docker networking) via full cluster reset
- Setting up Ingress and local DNS resolution via the Windows hosts file
- The importance of `kubectl logs --previous` and container `exec` for real root-cause debugging, rather than guessing from Kubernetes-level events alone

---

## 📄 License

This project is for educational purposes, adapted from an open-source tutorial repository.
