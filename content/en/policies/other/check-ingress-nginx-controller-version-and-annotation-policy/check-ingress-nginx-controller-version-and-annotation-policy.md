---
title: "Ensure Valid Ingress NGINX Controller and Annotations"
category: Ingress, Security
version: 1.9.0
subject: Ingress, Pod
policyType: "validate"
description: >
    This policy ensures that Ingress resources do not have certain disallowed annotations and that the ingress-nginx controller Pod is running an appropriate version of the image. It checks for the presence of the  `nginx.ingress.kubernetes.io/server-snippet` annotation and disallows its usage, enforces specific values  for `auth-tls-verify-client`, and ensures that the ingress-nginx controller image is of the required version.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/check-ingress-nginx-controller-version-and-annotation-policy/check-ingress-nginx-controller-version-and-annotation-policy.yaml" target="-blank">/other/check-ingress-nginx-controller-version-and-annotation-policy/check-ingress-nginx-controller-version-and-annotation-policy.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: check-ingress-nginx-controller-version-and-annotation-policy
  annotations:
    policies.kyverno.io/title: Ensure Valid Ingress NGINX Controller and Annotations
    policies.kyverno.io/category: Ingress, Security
    policies.kyverno.io/severity: high
    kyverno.io/kyverno-version: 1.11.0
    policies.kyverno.io/minversion: 1.9.0
    kyverno.io/kubernetes-version: "1.28"
    policies.kyverno.io/subject: Ingress, Pod
    policies.kyverno.io/description: >-
      This policy ensures that Ingress resources do not have certain disallowed annotations and that the ingress-nginx
      controller Pod is running an appropriate version of the image. It checks for the presence of the 
      `nginx.ingress.kubernetes.io/server-snippet` annotation and disallows its usage, enforces specific values 
      for `auth-tls-verify-client`, and ensures that the ingress-nginx controller image is of the required version.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: validate-ingress-annotations
    match:
      resources:
        kinds:
        - Ingress
    validate:
      message: "The annotation nginx.ingress.kubernetes.io/server-snippet is not allowed."
      pattern:
        metadata:
          annotations:
            X(nginx.ingress.kubernetes.io/server-snippet): ""
  - name: validate-auth-tls-verify-client
    match:
      resources:
        kinds:
        - Ingress
    validate:
      message: "auth-tls-verify-client annotation must be 'on', 'off', 'optional', or 'optional_no_ca'."
      deny:
        conditions:
          any:
          - key: "{{request.object.metadata.annotations.\"nginx.ingress.kubernetes.io/auth-tls-verify-client\"}}"
            operator: AnyNotIn
            value:
            - "on"
            - "off"
            - "optional"
            - "optional_no_ca"
  - name: ensure-ingress-nginx-controller-version-pattern
    match:
      resources:
        kinds:
          - Pod
    validate:
      message: "The ingress-nginx controller image version must start with v1.11."
      pattern:
        spec:
          containers:
            - name: controller
              image: "registry.k8s.io/ingress-nginx/controller:v1.11.*"

  - name: deny-lower-ingress-nginx-controller-versions
    match:
      resources:
        kinds:
          - Pod
    validate:
      message: "The ingress-nginx controller image version must be v1.11.2 or greater."
      deny:
        conditions:
          - key: "{{ request.object.spec.containers[?(@.name=='controller')].image }}"
            operator: AnyIn
            value:
              - "registry.k8s.io/ingress-nginx/controller:v1.11.0"
              - "registry.k8s.io/ingress-nginx/controller:v1.11.1"
              - "registry.k8s.io/ingress-nginx/controller:v1.10.*"
              - "registry.k8s.io/ingress-nginx/controller:v1.9.*"
              - "registry.k8s.io/ingress-nginx/controller:v1.8.*"
              - "registry.k8s.io/ingress-nginx/controller:v1.7.*"
              - "registry.k8s.io/ingress-nginx/controller:v1.6.*"
              - "registry.k8s.io/ingress-nginx/controller:v1.5.*"
              - "registry.k8s.io/ingress-nginx/controller:v1.4.*"
              - "registry.k8s.io/ingress-nginx/controller:v1.3.*"
              - "registry.k8s.io/ingress-nginx/controller:v1.2.*"
              - "registry.k8s.io/ingress-nginx/controller:v1.1.*"
              - "registry.k8s.io/ingress-nginx/controller:v1.0.*"

```
