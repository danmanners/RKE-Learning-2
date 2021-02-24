# Deploying the cluster

Here are the steps taken to get the cluster from nothing to running

```bash
# Ensure you install the required Ansible Galaxy Modules
$ ansible-galaxy collection install ansible.posix
$ ansible-galaxy collection install community.general
$ ansible-galaxy install geerlingguy.docker

# Provision all of our hosts
$ ansible-playbook -i inventory.ini rke-deploy-playbook.yaml

# Get the correct RKE Binary
$ sudo wget https://github.com/rancher/rke/releases/download/v1.2.5/rke_linux-amd64 \
    -O /usr/local/bin/rke
$ sudo chmod a+x /usr/local/bin/rke

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
$ kubectl apply -f Longhorn/ingress.yaml

# Install Traefik v2.4.5
$ helm repo add traefik https://helm.traefik.io/traefik
$ helm repo update
$ helm install traefik traefik/traefik \
    --version 9.14.3 -f Traefik/values.yaml

# Install Cert-Manager
$ helm repo add jetstack https://charts.jetstack.io
$ kubectl create namespace cert-manager
$ helm repo update
$ helm install cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --version v1.2.0 \
    --set installCRDs=true

# Set up Cert-Manager DigitalOcean DNS Issuer
$ kubectl apply -f cert-manager/digitalocean-dns01.yaml

# Install Jenkins
$ kubectl apply -f Jenkins/sts.yaml -f Jenkins/rbac.yaml -f Jenkins/ingress.yaml
$ kubectl exec -it jenkins-ce-0 -- /bin/cat /var/jenkins_home/secrets/initialAdminPassword

# Install Nexus-OSS
$ kubectl apply -f Nexus-OSS/sts.yaml -f Nexus-OSS/ingress.yaml
$ kubectl exec -it nexus-oss-0 -- /bin/cat /nexus-data/admin.password
```

After running through the initialization steps in the [GitLab README](Gitlab/README.md) and the [SonarQube README](Sonarqube/README.md), you can also install GitLab-EE and SonarQube!

```bash
# Installing Gitlab & SonarQube through Argo
$ kubectl apply -f ArgoCD/projects.yaml
```
