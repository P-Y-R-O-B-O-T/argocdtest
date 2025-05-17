## Creation of KIND cluster
* Install docker
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install packages
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-compose
```

* Install KIND
```bash
mkdir temp-install-kind
cd temp-install-kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo cp ./kind /usr/local/bin/kind
cd ..
```

* Create kind cluster with 3 worker nodes and 1 control plane node
```bash
kind create cluster --config=kind-conf.yaml --name=test-cluster
```

## Install kubectl
```bash
curl -LO https://dl.k8s.io/release/v1.30.0/bin/linux/amd64/kubectl
chmod +x kubectl
sudo cp ./kubectl /usr/bin/kubectl
sudo rm kubectl
```

## Install ArgoCD
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm install my-argo-cd argo/argo-cd --version 8.0.3 -f argocd-values.yaml
```

## Creating ArgoCD Deployment

* Create the `NodePort` service by patching the existing one
```bash
kubectl patch service my-argo-cd-argocd-server -p '{"spec":{"type": "LoadBalancer"}}'
```

* Now do run `kubectl get svc` and seek for the service and see which ports it is on running, the range is 30000-32767 and create a `port-forward`
```bash
kubectl port-forward svc/my-argo-cd-argocd-server NODE-PORT:80 --address 0.0.0.0
```

* Get ArgoCD credentials
```bash
kubectl get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

* Access ArgoCD at address `http://localhost:NODE-PORT`

### Steps to create CI/CD 
1. Create app
2. Application options as follows
    - `Application Name`: test 
    - `Project Name`: default
    - `Sync Policy`: manual
    - `Sync Options`: `auto create namespace` and `apply out of sync only`
