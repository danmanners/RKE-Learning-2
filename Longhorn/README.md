# Install Longhorn

If you have added ArgoCD, you can simply add the `longhorn.yaml` argoproject file. This will install Longhorn. You can then optionally add the other two files.

```bash
kubectl apply -f {longhorn,ingress,storageclass}.yaml
```
