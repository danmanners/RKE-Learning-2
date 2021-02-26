# Deploy Traefik

On your local system, run the following:

```bash
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
helm install traefik traefik/traefik \
    --version 9.14.3 -f Traefik/values.yaml
```

This should get Traefik up and going!

# What are the other files

The `commonMiddlewares.yaml` and `traefik-dashboard.yaml` files are to allow ingress access into the dashboard and to add the `http > https` redirect and `X-Forwarded-For` header middlewares.
