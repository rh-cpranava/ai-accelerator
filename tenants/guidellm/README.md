# GuideLLM Benchmarking

This tenant deploys [GuideLLM](https://github.com/vllm-project/guidellm) as a CronJob to benchmark the vLLM model serving endpoint on the cluster.

## What it does

GuideLLM runs a sweep benchmark against the `gemma-4-31b` InferenceService, producing JSON and HTML reports with latency, throughput, and saturation metrics.

## Structure

```
tenants/guidellm/
├── base/
│   ├── kustomization.yaml
│   ├── namespace.yaml
│   ├── pvc-guidellm-results.yaml
│   └── cronjob-guidellm-benchmark.yaml
└── overlays/
    └── staging/
        └── kustomization.yaml
```

## Running a benchmark

The CronJob is **suspended by default**. Trigger it manually:

```bash
oc create job guidellm-run-$(date +%s) --from=cronjob/guidellm-benchmark -n guidellm-staging
```

To enable a recurring schedule, set `suspend: false` in the overlay. The default schedule is weekly (Monday 02:00 UTC).

## Configuration

Key parameters in `cronjob-guidellm-benchmark.yaml`:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--target` | `https://gemma-4-31b-predictor.models-staging.svc.cluster.local` | The model serving endpoint URL |
| `--model` | `gemma-4-31b` | Model name to benchmark |
| `--data` | `{"prompt_tokens":256,"output_tokens":128}` | Synthetic workload shape |
| `--rate-type` | `sweep` | Load profile (sweep, concurrent, poisson) |
| `--max-seconds` | `300` | Maximum benchmark duration |
| `--backend-kwargs` | `{"verify": false}` | Disables TLS verification for self-signed certs |

## Retrieving results

After the Job completes, results are stored on the `guidellm-results` PVC:

```bash
oc run pvc-inspector --image=registry.access.redhat.com/ubi9/ubi-minimal \
  --restart=Never -n guidellm-staging \
  --overrides='{"spec":{"containers":[{"name":"inspector","image":"registry.access.redhat.com/ubi9/ubi-minimal","command":["sleep","3600"],"volumeMounts":[{"name":"results","mountPath":"/results"}]}],"volumes":[{"name":"results","persistentVolumeClaim":{"claimName":"guidellm-results"}}]}}'

oc exec -n guidellm-staging pvc-inspector -- ls /results
oc cp guidellm-staging/pvc-inspector:/results/benchmark-results.json ./benchmark-results.json
oc cp guidellm-staging/pvc-inspector:/results/benchmark-results.html ./benchmark-results.html
```

## Notes

- The InferenceService has `enable-auth: "true"` — if auth tokens are required, add a `secretKeyRef` env var for `OPENAI_API_KEY` to the CronJob spec.
- Adjust `--target` URL if the KServe predictor service uses a different naming convention in your cluster.

