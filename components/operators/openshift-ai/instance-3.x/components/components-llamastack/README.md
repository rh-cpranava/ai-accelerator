# components-llamastack

## Purpose

This component enables the Llama Stack Operator and Gen AI Studio:

- Sets `llamastackoperator.managementState: Managed` on the DataScienceCluster
- Sets `spec.dashboardConfig.genAiStudio: true` on `OdhDashboardConfig` so Gen AI Studio appears in the dashboard

Gen AI Studio requires the Llama Stack Operator. See:
https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html-single/experimenting_with_models_in_the_gen_ai_playground/index

## Usage

This component can be added to a base by adding the `components` section to your overlay `kustomization.yaml` file:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

components:
  - ../../components/components-llamastack
```
