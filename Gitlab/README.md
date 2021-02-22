# Commands to run prior to ArgoCD Kickoff

All of these will need to be run. Referenced and copied from [here](https://gitlab.com/gitlab-org/charts/gitlab/-/blob/master/doc/installation/secrets.md).

```bash
# Create the Gitlab Shell Hostkeys
mkdir -p hostKeys
ssh-keygen -t rsa  -f hostKeys/ssh_host_rsa_key -N ""
ssh-keygen -t dsa  -f hostKeys/ssh_host_dsa_key -N ""
ssh-keygen -t ecdsa  -f hostKeys/ssh_host_ecdsa_key -N ""
ssh-keygen -t ed25519  -f hostKeys/ssh_host_ed25519_key -N ""

# Upload the files
kubectl -n gitlab create secret generic gitlab-gitlab-shell-host-keys \
    --from-file hostKeys

# Create the gitlab-issuser certs
mkdir -p certs
openssl req -new -newkey rsa:4096 -subj "/CN=gitlab-issuer" \
    -nodes -x509 -keyout certs/registry-certs.key \
    -out certs/registry-certs.crt

# Create the registry-secrets
kubectl -n gitlab create secret generic gitlab-registry-secret \
    --from-file=registry-auth.key=certs/registry-certs.key \
    --from-file=registry-auth.crt=certs/registry-certs.crt

# Load in the LetsEncrypt CA Files
curl -s https://letsencrypt.org/certs/isrgrootx1.pem https://letsencrypt.org/certs/lets-encrypt-r3.pem > certs/le-chain.pem
kubectl -n gitlab create secret generic gitlab-wildcard-tls-ca \
    --from-file=custom-ca-certificates=certs/le-chain.pem

# Initial Root Password
kubectl -n gitlab create secret generic gitlab-gitlab-initial-root-password \
    --from-literal=password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 32)

# Redis Password
kubectl -n gitlab create secret generic gitlab-redis-secret \
    --from-literal=secret=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

# GitLab Shell Secret
kubectl -n gitlab create secret generic gitlab-gitlab-shell-secret \
    --from-literal=secret=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

# Gitaly Secret
kubectl -n gitlab create secret generic gitlab-gitaly-secret \
    --from-literal=token=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

# Praefect Secret
kubectl -n gitlab create secret generic gitlab-praefect-secret \
    --from-literal=token=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

# GitLab Rails
cat << EOF > secrets.yml
production:
  secret_key_base: $(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 128)
  otp_key_base: $(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 128)
  db_key_base: $(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 128)
  encrypted_settings_key_base: $(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 128)
  openid_connect_signing_key: |
$(openssl genrsa 2048 | awk '{print "    " $0}')
  ci_jwt_signing_key: |
$(openssl genrsa 2048 | awk '{print "    " $0}')
EOF

kubectl -n gitlab create secret generic gitlab-rails-secret \
    --from-file=secrets.yml

# GitLab Workhorse
kubectl -n gitlab create secret generic gitlab-gitlab-workhorse-secret \
    --from-literal=shared_secret=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 32 | base64)

# GitLab Runner
kubectl -n gitlab create secret generic gitlab-gitlab-runner-secret \
    --from-literal=runner-registration-token=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
    --from-literal=runner-token="5oTLJS47SrUAuL5Lznc1DL0IwzcqbHLkq9PLwi0xdJ0PiVSjjq0M9js5tmMwOysn"

kubectl -n gitlab create secret generic gitlab-gitlab-runner-secret \
    --from-literal=runner-registration-token=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
    --from-literal=runner-token="5oTLJS47SrUAuL5Lznc1DL0IwzcqbHLkq9PLwi0xdJ0PiVSjjq0M9js5tmMwOysn"

# GitLab Registry HTTP Secret
kubectl -n gitlab create secret generic gitlab-registry-httpsecret \
    --from-literal=secret=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) 

# GitLab KAS Secret
kubectl -n gitlab create secret generic gitlab-gitlab-kas-secret \
    --from-literal=kas_shared_secret=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 32 | base64)

# Minio Secret
kubectl -n gitlab create secret generic gitlab-minio-secret \
    --from-literal=accesskey=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 20) \
    --from-literal=secretkey=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

# Postgres Password
kubectl -n gitlab create secret generic gitlab-postgresql-password \
    --from-literal=postgresql-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
    --from-literal=postgresql-postgres-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

# GitLab Pages Secret
kubectl -n gitlab create secret generic gitlab-gitlab-pages-secret \
    --from-literal=shared_secret=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 32 | base64)

# Registry HTTP Secret
kubectl -n gitlab create secret generic gitlab-registry-httpsecret \
    --from-literal=secret=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64 | base64)

# Praefect DB Password
kubectl -n gitlab create secret generic gitlab-praefect-dbsecret \
    --from-literal=secret=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)
```

## Troubleshooting GitLab Runner

[See this link for up-to-date troubleshooting steps](https://gitlab.com/gitlab-org/charts/gitlab/-/blob/master/doc/troubleshooting/index.md#included-gitlab-runner-failing-to-register). It's most likely that you need to simply update the runner token.
