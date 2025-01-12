# Patches for knative charts

Knative is not hosted via a helm repository upstream and the official recommended way to install is to either using another tool or using raw YAML manifests - https://knative.dev/docs/install/. Based on this, we don't use `helm dependency update` to update the subcharts here but instead use raw YAML manifests from upstream to update the chart here. Following lists the manual patches that needs to be applied whenever charts are bumped. 

## Simple Chart Bump

Before following the patch steps below, and before fetching the latest release yaml files from Knative, simply bump the appVersion in the `Chart.yaml` files.
These values are used in the templates to get the latest version, and barring a major change in the upstream yaml files, this should be sufficient.

Small errors in the apiVersion can also be fixed simply by amending the offending value.

Pulling the new yaml files from Knative and applying the patch below should only be done if the above steps are insufficient.

## Patch 1: Relax the PodDisruptionBudget policies

Developed in order to fix https://d2iq.atlassian.net/browse/D2IQ-96590

When fetching `serving-core.yaml` from [upstream](https://github.com/knative/serving/releases/download/knative-v1.10.0/serving-core.yaml) apply the following patch to it (if the latest version has moved the resources or has changed the fields on PDB please update this patch accordingly).

1. Get the latest file
```bash
wget https://github.com/knative/serving/releases/download/knative-v1.9.3/serving-core.yaml
```

2. Create the patch kustomization

```yaml
cat <<EOF >>kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- serving-core.yaml
patches:
  - target:
      kind: PodDisruptionBudget
      group: policy
      version: v1
    patch: |
      - op: remove
        path: /spec/minAvailable
      - op: add
        path: /spec/maxUnavailable
        value: 1
EOF
```

3. Apply the patch kustomization

```bash
kustomize build . > serving-core-patched.yaml
```

And then use the `serving-core-patched.yaml` to apply the changes to `charts/serving/templates/serving-core.yaml`

<!-- More patches can be added here in future as needed. -->