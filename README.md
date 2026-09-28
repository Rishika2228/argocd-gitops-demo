# GitOps Workflow using ArgoCD on Kubernetes

This project is a simple hands-on implementation of **GitOps using ArgoCD and Kubernetes**.

The main idea behind the project is to manage Kubernetes deployment files through GitHub and let ArgoCD automatically apply the changes to the Kubernetes cluster.

For this project, I used a basic **Nginx application** and deployed it on a local **Minikube Kubernetes cluster**.

## Tools Used

- Kubernetes
- Minikube
- ArgoCD
- Git & GitHub
- Docker
- Nginx
- PowerShell

## How the Project Works

The workflow is basically:

```text
Developer
    ↓
GitHub
    ↓
ArgoCD
    ↓
Kubernetes / Minikube
    ↓
Nginx Application
```

The Kubernetes configuration is stored inside the GitHub repository. ArgoCD monitors the repository and keeps the Kubernetes cluster synchronized with the configuration stored in Git.

So instead of manually running `kubectl apply` every time there is a change, I can make the change in Git and let ArgoCD handle the deployment.

## Project Structure

```text
argocd-gitops-demo/
│
├── k8s/
│   ├── deployment.yaml
│   ├── namespace.yaml
│   └── service.yaml
│
└── README.md
```

### `namespace.yaml`

Creates a separate namespace called `gitops-demo` for the application.

### `deployment.yaml`

Contains the Nginx deployment configuration.

The current deployment runs **4 replicas** of Nginx.

```yaml
replicas: 4
```

### `service.yaml`

Creates a Kubernetes NodePort service so that the Nginx application can be accessed from the browser.

## Setting Up the Kubernetes Cluster

I used Minikube to create a local Kubernetes environment.

```powershell
minikube start
```

Then I checked that the Kubernetes node was running:

```powershell
kubectl get nodes
```

The node was in the `Ready` state.

## Installing ArgoCD

First, I created the ArgoCD namespace:

```powershell
kubectl create namespace argocd
```

Then installed ArgoCD:

```powershell
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

After installation, I checked the ArgoCD pods:

```powershell
kubectl get pods -n argocd
```

Once all the pods were running, I accessed the ArgoCD dashboard using port forwarding:

```powershell
kubectl port-forward svc/argocd-server -n argocd 8081:443
```

Then I opened:

```text
https://localhost:8081
```

## ArgoCD Application

I created an ArgoCD application named:

```text
gitops-demo
```

The application uses this GitHub repository:

```text
https://github.com/Rishika2228/argocd-gitops-demo.git
```

The configuration is:

```text
Branch: main
Path: k8s
Namespace: gitops-demo
```

I enabled automatic synchronization along with:

- Automatic Sync
- Prune
- Self Heal

After creating the application, ArgoCD showed the application as:

```text
Healthy
Synced
```

## Testing the GitOps Workflow

The most important part of this project was testing whether a change made in Git would automatically reach Kubernetes.

Initially, the deployment had:

```yaml
replicas: 3
```

I changed it to:

```yaml
replicas: 4
```
<img width="484" height="398" alt="Screenshot 2026-09-28 210924" src="https://github.com/user-attachments/assets/9984390e-821b-403a-a59d-ece500988e1c" />


Then I committed and pushed the change to GitHub:

```powershell
git add k8s/deployment.yaml
git commit -m "Scale nginx deployment to 4 replicas"
git push
```

The new commit was:

```text
3a8964b
```
<img width="959" height="448" alt="Screenshot 2026-09-28 210011" src="https://github.com/user-attachments/assets/360f9012-b3c0-4d55-be0f-afe2c87de4b3" />


After ArgoCD detected the new commit, it synchronized the application automatically.

I then checked the Kubernetes pods:

```powershell
kubectl get pods -n gitops-demo
```

The result showed four Nginx pods running.

```text
NAME                     READY   STATUS
nginx-xxxxxxxxxx-xxxxx   1/1     Running
nginx-xxxxxxxxxx-xxxxx   1/1     Running
nginx-xxxxxxxxxx-xxxxx   1/1     Running
nginx-xxxxxxxxxx-xxxxx   1/1     Running
```
<img width="920" height="255" alt="Screenshot 2026-09-28 210051" src="https://github.com/user-attachments/assets/fea713a4-c839-4760-8434-40fda963d1bb" />


I did not manually run `kubectl apply` for the replica change.

This was the main demonstration of the GitOps workflow.

## Accessing the Application

The Nginx application was exposed using a NodePort service.

I checked the service using:

```powershell
kubectl get svc -n gitops-demo
```

Then I opened the application using:

```powershell
minikube service nginx -n gitops-demo
```

The browser displayed the default Nginx page:

```text
Welcome to nginx!
```
<img width="959" height="466" alt="Screenshot 2026-09-28 210120" src="https://github.com/user-attachments/assets/d5193330-b0bd-4a96-8ed6-d0d141539212" />


## What I Learned

While working on this project, I got hands-on experience with:

- Kubernetes deployments and services
- Minikube
- Git and GitHub
- ArgoCD
- GitOps concepts
- Automated synchronization
- Kubernetes namespaces
- Scaling applications using Git
- ArgoCD self-healing and reconciliation

One of the main things I understood from this project is that **Git can be used as the source of truth for Kubernetes deployments**, while ArgoCD takes care of keeping the actual cluster state in sync with it.

## GitOps Workflow

The final workflow looks like this:

```text
Change Kubernetes YAML
        ↓
     Git Commit
        ↓
      GitHub
        ↓
     ArgoCD
        ↓
 Automatic Synchronization
        ↓
    Kubernetes
        ↓
  Nginx Application
```

## Repository

GitHub repository:

https://github.com/Rishika2228/argocd-gitops-demo

## Author

**Rishika**

This project was built as a hands-on learning project to understand how GitOps works with Kubernetes and ArgoCD.
