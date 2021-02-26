# Validation Apps

This directory uses Kustomize to deploy the `whoami` application and uses a `patchStrategicMerge` resource to update the deployment for `whomai` from 2 replicas to 3.

This is really just a proof of concept to validate that Kustomize works as expected with ArgoCD.
