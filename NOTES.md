
### FluxCD key infos

- Monitor FluxCD activity success on the metrics endpoint (Ref : https://fluxcd.io/flux/monitoring/metrics/):

```bash
http://<flux-controller-service>.<namespace>:8080/metrics
```
Key Metrics:

    flux_reconcile_count (total reconciliations)
    flux_reconcile_duration_seconds (how long reconciliations take)
    flux_kustomization_apply_status (success/failure of applies)


- Directories structure:

    - Separate infra and resources dirs: `apps/` `infrastructure` `/clusters`
    - Use environment-based segregation like : `/clusters/staging` and `/clusters/production`

    Example :

    ```text
    ├── apps
    │   ├── base
    │   ├── production 
    │   └── staging
    ├── infrastructure
    │   ├── base
    │   ├── production 
    │   └── staging
    └── clusters
        ├── production
        └── staging
    ```

- **`Kustomize`**

Definition : `Flux CD` leverages `Kustomize`, a tool that allows to customize Kubernetes manifests without modifying the base files.

Ref: https://fluxcd.io/flux/components/kustomize/kustomizations/

The Kustomize controller works between `Kubernetes API` and the `deployments`/`resources`: Ref2 https://fluxcd.io/flux/components/kustomize/

usage:
```yaml
...
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
...
```

- **`Helm`**

In the same way, while using helm through FluxCD, Helm Controler stands between `Kubernetes API` and the `deployments`. Ref : https://fluxcd.io/flux/components/helm/


Additionnal:


- Use a dedicated GitHub Account for FluxDB, with strics access Token. Ref :
- Security Ref: https://fluxcd.io/flux/security/

### FluxCD GitOps Principles

- Single Source of Truth – Everything (manifests, Helm releases, Kustomize patches) must be in Git.

- Releases : Use tagged versions, not latest, for Helm charts and Docker images.

- Automated Sync : Consider using manual approvals on top of auto-sync (Like on new PRs)

- In documentation, add **how to RollBack in emergency**

- `Personal account` vs `Org account`

While launching actions with `FluxCD`, `FluxCD` will assume having **personal rights** on repo, or **Org Level rights** on repo, based on wether the argument `--personal` is used or not:

```bash
# Personal account workflow
flux bootstrap github --owner=yourname --repository=repo --personal
# Requires: PAT with personal repo permissions

# Organization workflow 
flux bootstrap github --owner=orgname --repository=repo
```
- **Bootstrap** means FluxCD will push the Flux manifests to a Git repository and deploy Flux on the cluster. Ref : https://fluxcd.io/flux/cmd/flux_bootstrap/