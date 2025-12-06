# Argo CD Local GitOps Configuration (Minikube)

This guide explains how to install, configure, and access **Argo CD locally** using **Minikube**.  
All steps have been validated and follow best practice.


## Prerequisites

Before starting, confirm you have:

- ✅ Minikube installed and working
- ✅ kubectl installed and configured
- ✅ Internet access
- ✅ Administrator / sudo access


## Step 1: Start Minikube and Create Namespace

```bash
minikube start
kubectl create namespace argocd
````

> The `argocd` namespace isolates all Argo CD components.


## Step 2: Install Argo CD

### 2.1 Install Argo CD Components

```bash
kubectl apply -n argocd -f \
https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait for pods to be ready:

```bash
kubectl get pods -n argocd
```


### 2.2 Install Argo CD CLI (Linux)

```bash
sudo curl -sSL -o /usr/local/bin/argocd \
https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

sudo chmod +x /usr/local/bin/argocd

argocd version
```

> For macOS or Windows, download the appropriate binary from the Argo CD releases page.


## Step 3: Expose Argo CD Locally

```bash
kubectl port-forward svc/argocd-server -n argocd 8081:443
```

Argo CD will now be available at:

```
https://localhost:8081
```

## Step 4: Get Initial Admin Password

### Recommended Method

```bash
argocd admin initial-password -n argocd
```

### Alternative Method

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d; echo
```

> Save this password securely. It is required for first login.


## Step 5: Login and Change Password

### 5.1 Login via CLI

```bash
argocd login localhost:8081 \
--username admin \
--password <initial-password> \
--insecure
```

### 5.2 Update Password

```bash
argocd account update-password
```

You will be prompted for:

* Current password
* New password
* Confirm new password


## Step 6: Login via Web UI

Open your browser and navigate to:

```
https://localhost:8081
```

Login:

* **Username:** `admin`
* **Password:** `<your-new-password>`

> You will receive a browser warning due to a self-signed certificate.
> **Proceed anyway.**


## Troubleshooting

Check pod status:

```bash
kubectl get pods -n argocd
```

Try another port if 8081 is blocked:

```bash
kubectl port-forward svc/argocd-server -n argocd 8085:443
```

Check Minikube:

```bash
minikube status
```

Restart Argo CD:

```bash
kubectl rollout restart deployment argocd-server -n argocd
```