# Deploy Traefik

On your local system, run the following:

```bash
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
helm install traefik traefik/traefik \
    --version 9.14.3 -f Traefik/values.yaml
```

This should get Traefik up and going!
