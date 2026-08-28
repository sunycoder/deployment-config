# deployment-config

This repository is the GitOps source for Kubernetes deployment configuration.
Application source code remains in SVN; Jenkins builds the application image
and updates the image tag in this repository. Argo CD then synchronizes the
configuration to the cluster and Argo Rollouts performs the Blue-Green switch.

## Repository layout

```text
argocd/test-demo-application.yaml  Argo CD Application definition
test-demo/                         Kubernetes resources for test-demo
  kustomization.yaml              Kustomize entry point and image version
  version.env                     Runtime version/color configuration
  rollout.yaml                     Argo Rollouts Blue-Green strategy
  service.yaml                     Active and preview Services
  ingress.yaml                     Traefik route to active Service
  namespace.yaml                   demo Namespace
```

## Deployment flow

1. Jenkins checks out the application from SVN and runs tests/package.
2. Jenkins builds and pushes `k3d-demo-registry.localhost:5001/demo/test-demo`.
3. Jenkins updates `test-demo/kustomization.yaml` and `test-demo/version.env`.
4. Jenkins commits and pushes those two files to `master`.
5. Argo CD detects the commit and synchronizes the `test-demo` Application.
6. Argo Rollouts starts the preview Pod, waits for readiness, and promotes it.

The external application URL is provided by the cluster's Traefik Ingress
and remains stable while the active Service selector changes:

```text
http://<server-ip>:18183/
```

Do not commit credentials, kubeconfigs, private keys, `.idea`, or build output.

