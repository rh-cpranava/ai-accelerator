# cert-manager Operator for Red Hat OpenShift

Install the cert-manager Operator for Red Hat OpenShift.

Do not use the `base` directory directly, as you will need to patch the `channel` based on the version of the operator you want to use.

The current *overlays* available are for the following channels:

* [stable-v1](operator/overlays/stable-v1)

## Usage

```
oc apply -k components/operators/cert-manager/operator/overlays/<channel>
```

As part of a different overlay in your own GitOps repo:

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - components/operators/cert-manager/operator/overlays/<channel>
```
