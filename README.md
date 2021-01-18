# Deploying the cluster

Here are the steps taken to get the cluster from nothing to running

```bash
# Provision all of our hosts
$ ansible-playbook -i inventory.ini rke-deploy-playbook.yaml

# Ensure that the RKE cluster is up
$ rke up

# Install MetalLB
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/namespace.yaml
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/metallb.yaml
$ kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
$ kubectl apply -f manifests/cm.yaml

# Install Longhorn
$ helm repo add longhorn https://charts.longhorn.io
$ helm repo update
$ kubectl create namespace longhorn-system
$ helm install longhorn longhorn/longhorn \
    --namespace longhorn-system \
    --version 1.1.0

# Install Traefik v2
$ helm repo add traefik https://helm.traefik.io/traefik
$ helm repo update
$ helm install traefik traefik/traefik \
    --version 9.12.3

# Install Cert-Manager
$ helm repo add jetstack https://charts.jetstack.io
$ kubectl create namespace cert-manager
$ helm repo update
$ helm install cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --version v1.1.0 \
    --set installCRDs=true

# Set up Cert-Manager DigitalOcean DNS Issuer
$ kubectl apply -f cert-manager/digitalocean-dns01.yaml
```
