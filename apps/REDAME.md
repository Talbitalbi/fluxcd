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

```bash
helm repo add fluxcd https://fluxcd-community.github.io/helm-charts
helm repo update

helm upgrade -i flux fluxcd/flux2 \
  --namespace flux-system \
  --create-namespace
```


