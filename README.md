# home-k3s
Kubernetes configuration for my home setup

# Initial setup of K3s nodes

Install Ubuntu 24.04 server minimal on the nodes.

## Install K3s
On the first node (master):
```
echo "net.core.rmem_max=2500000" | sudo tee -a /etc/sysctl.conf  # For cloudflared
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=traefik" sh -
sudo cat /var/lib/rancher/k3s/server/node-token
```

On the other nodes (workers):
```
curl -sfL https://get.k3s.io | K3S_URL=https://<master-node>:6443 K3S_TOKEN=<token from above> sh -
```

You can get the Kubeconfig from the master node:
```
sudo cat /etc/rancher/k3s/k3s.yaml
```

## Install ArgoCD
```bash
sudo kubectl create namespace argocd
sudo kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Create Cloudflared Tunnel

From your local machine:
```bash
cloudflared login
cloudflared tunnel create stoddinet
```

Install the tunnel credentials as a Kubernetes secret:

```bash
kubectl create secret generic cloudflared-tunnel --from-file=credentials.json=<file show when creating tunnel>
```

## Bootstrap ArgoCD Applications
```bash
sudo kubectl apply -f bootstrap/application.yaml
```