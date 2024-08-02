---
title: Installing ArgoCD on Kubernetes (Core- Non-HA)
weight: 1
---

# ArgoCD Deployment (Core -Non-HA)

## Pre-requisites

### Create TLS Certs

```
kubectl create secret tls argocd-server-tls --cert <certificate.crt> --key <private.key>
```

### Create a Public IP in gCloud

This global IP would be used to resovle the public DNS of the ArgoCD

```
gcloud compute addresses create argocd-ingress-ip  --global --ip-version IPV4
```

### ArgoCD SSO App in Github

To integrate ArgoCD SSO with GitHub, we need to register a new application on GitHub.

Go to the Setting section of your GitHub profile.
Go to the Developer settings.
Click on OAuth Apps, and then click New OAuth Apps.
Now fill in all the details, the callback address should be the /api/dex/callback endpoint of your Argo CD URL (e.g. https://argocd.example.com/api/dex/callback).

![SSO](./argocd-sso.png)

After registering the app, you will receive an OAuth2 client ID and secret.

## Configuring the Basic values.yaml file

Grab the ArgoCD community chart and modify the values.yaml to update the following values.
```text
https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd
```


1. `global.domain`
2. `configs.dex.configs` (Use ClientID and ClientSecret generated in the `ArgoCD SSO App in Github` section )
3. `server.ingress.enabled`
4. `server.ingress.annotations`
    a. Add `kubernetes.io/ingress.global-static-ip-name: argocd-ingress-ip` to use static IP generated in the prerequisite section
    b. Add `kubernetes.io/ingress.class: gce` to tell the gke LB controller to pick up ingress for LB provisioning
5. `server.ingress.hostname`
6. `server.ingress.gke.backendConfig` (Use health check configs)
7. `server.ingress.gke.frontendConfig` (Use config to redirect to https always)

## Deploy

Run following instructions, it will always deploy/upgrade the chart in `argocd` namespace.
```
kubectl get ns argocd || kubectl create ns argocd
helm upgrade --install argocd --namespace argocd .
```


