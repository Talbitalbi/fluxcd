- for production use, it is recommended to have a **dedicated GitHub account for Flux** and use fine-grained access tokens with the minimum required permissions.

#### Install FluxCD on local machine

Source Ref : https://fluxcd.io/flux/installation/

```bash
curl -s https://fluxcd.io/install.sh | sudo bash

flux --version

# Output
flux version 2.6.3
```

#### Run `pre-installation check` on the cluster

```bash
flux check --pre

# Output when cluster Up:
► checking prerequisites
✔ Kubernetes 1.32.5+k3s1 >=1.31.0-0
✔ prerequisites checks passed
```
The version match contidions are :

|Kubernetes version	| Minimum required|
|-------------------|-----------------|
|v1.30	            | >= 1.30.0       |
|v1.31	            | >= 1.31.0       |
|v1.32 and later	  | >= 1.32.0       |



#### Install FluxCD on cluster

- ##
```bash
## Duplicate step
helm repo add fluxcd https://fluxcd-community.github.io/helm-charts
helm repo update

helm upgrade -i flux fluxcd/flux2 \
  --namespace flux-system \
  --create-namespace

flux install \
  --namespace flux-system \
  --create-namespace
```

- bootstrap `fluxcd` on cluster and tie to GitHub Repo
```bash
#Token required on prompt
flux bootstrap github \
  --owner=talbitalbi \
  --repository=fluxcd \
  --branch=main \
  --path=infra \
  --personal

```

#### Apply Git Repo to cluster

```bash
kubectl apply -f infra/fluxcd-gitrepo.yaml
```

Now FluxCD on the cluster, is tied to the GitHub Repo, will doenload new content, but won't apply it.

Why : There is no actual Kustomization resource on the cluster, which is what decides what will be applied. Solutions:
  - Have a bootstrap kustomization created when fluxcd is installed
  - Trigger a bootstrap kustomization, and let FlxCD do the next dpeloyments upon new commits

* bootstrap Kustomization :
```bash
flux create kustomization flux-bootstrap \
  --source=GitRepository/flux-project \
  --path="./" \
  --prune=true \
  --interval=5m \
  --namespace=flux-system
```

### state check


```bash
kubectl get ns

# Output

NAME              STATUS   AGE
...
flux-system       Active   7d17h
...
```

```bash
kubectl logs -n flux-system -l app=source-controller
```

```bash
kubectl get gitrepositories -n flux-system

# Output
NAME          URL                                    AGE     READY   STATUS
flux-project   https://github.com/{owner}/{repo}   3m49s   True    stored artifact for revision 'main@sha1:e6ff3d65a41cc0e7b0a9f264878154e5a271df9b'
```

```bash
kubectl describe gitrepository flux-system -n flux-system
```
