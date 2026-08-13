# Models tenants

GitOps manifests for hosting models in Data Science Project namespaces.

## Layout

```text
tenants/models/
  base/                 # shared Namespace, PVC, ServingRuntime, InferenceService
  overlays/staging/     # models-staging
  overlays/prod/        # models-prod
```

## Deploy

```bash
oc apply -k tenants/models/overlays/staging
oc apply -k tenants/models/overlays/prod
```

Or via Argo CD (`rhoai-stable-bm-gpu` cluster-configs ApplicationSet):

- `models-staging` → `tenants/models/overlays/staging`
- `models-prod` → `tenants/models/overlays/prod`

## PVC and model weights

- PVC name: `model-store-gemma` (200Gi, default StorageClass)
- InferenceService `storageUri`: `pvc://model-store-gemma/hub/models--RedHatAI--gemma-4-31B-it-FP8-block/export`

Populate the PVC with model weights before expecting the InferenceService to become Ready. Override `storageClassName` or size in an overlay patch if needed.

## Secrets

Connection / auth secrets are intentionally not managed here yet. Re-add `opendatahub.io/connections` on the InferenceService when secret GitOps is in place.
