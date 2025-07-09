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

helm install fluxcd/flux2 \
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

# after a minute:
# This gets flux component deployed : helm-controller, kustomize-controller, notification-controller, source-controller)
flux install


# Or (non recommended fofrce install)
flux install --export > flux-install.yaml
kubectl apply -f flux-install.yaml

```

#### Apply Git Repo to cluster

```bash
kubectl apply -f infra/fluxcd-gitrepo.yaml
```

Now FluxCD on the cluster, is tied to the GitHub Repo, will download (sync) new committed content, but won't apply it.

Why : There is no actual Kustomization resource on the cluster, which is what decides what will be applied. Solutions:
  - Recommended : Execute `fluxcd bootstrap`, wich specifies the Repo to watch. `Flux` will create its own `SSH KEY`, push artifact to the repo and keeps syncing it to cluster 
  - More manual: **1.** Deploy a `GitRepo resource` to `flux-system namespace`, and **2.** Deploy a first kustomization (`Kubectl apply {some kustomization.yaml}`), and so Flux will sync the next deployments upon new commits.

* So, Option 1., Recommended: `bootstrap Kustomization` :
```bash
flux create kustomization flux-bootstrap \
  --source=GitRepository/flux-project \
  --path="./" \
  --prune=true \
  --interval=5m \
  --namespace=flux-system

# Enter GitHub Token Required
# add --personal flag if applicable due to repo type
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

- Get logs of the flux `source controller` (main place to check if GitHub Commits synced)
```bash
kubectl logs -n flux-system -l app=source-controller
# Output

{"level":"info","ts":"2025-07-09T13:19:50.582Z","msg":"no changes since last reconcilation: observed revision 'main@sha1:3ef3090ec6cf0c3f21707de1fe877d165f9aad9d'","controller":"gitrepository","controllerGroup":"source.toolkit.fluxcd.io","controllerKind":"GitRepository","GitRepository":{"name":"flux-system","namespace":"flux-system"},"namespace":"flux-system","name":"flux-system","reconcileID":"117ecf36-e964-424f-a9f6-173f23844721"}
```

- Check what Repo(s) is known to Flux
```bash
kubectl get gitrepositories -n flux-system

# Output
NAME          URL                                    AGE     READY   STATUS
flux-project   https://github.com/{owner}/{repo}   3m49s   True    stored artifact for revision 'main@sha1:e6ff3d65a41cc0e7b0a9f264878154e5a271df9b'
```

```bash
kubectl describe gitrepository flux-system -n flux-system
```
