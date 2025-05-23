---
title: "Disallow Secrets from Env Vars in CEL expressions"
category: Sample, EKS Best Practices in CEL
version: 
subject: Pod, Secret
policyType: "validate"
description: >
    Secrets used as environment variables containing sensitive information may, if not carefully controlled,  be printed in log output which could be visible to unauthorized people and captured in forwarding applications. This policy disallows using Secrets as environment variables.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/disallow-secrets-from-env-vars/disallow-secrets-from-env-vars.yaml" target="-blank">/other-cel/disallow-secrets-from-env-vars/disallow-secrets-from-env-vars.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: secrets-not-from-env-vars
  annotations:
    policies.kyverno.io/title: Disallow Secrets from Env Vars in CEL expressions
    policies.kyverno.io/category: Sample, EKS Best Practices in CEL 
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod, Secret
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      Secrets used as environment variables containing sensitive information may, if not carefully controlled, 
      be printed in log output which could be visible to unauthorized people and captured in forwarding
      applications. This policy disallows using Secrets as environment variables.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: secrets-not-from-env-vars
    match:
      any:
      - resources:
          kinds:
          - Pod
          operations:
          - CREATE
          - UPDATE
    validate:
      cel:
        expressions:
          - expression: "object.spec.containers.all(container, container.?env.orValue([]).all(env, env.?valueFrom.?secretKeyRef.orValue(true)))"
            message: "Secrets must be mounted as volumes, not as environment variables."
          - expression: "object.spec.containers.all(container, container.?envFrom.orValue([]).all(envFrom, !has(envFrom.secretRef)))"
            message: "Secrets must not come from envFrom statements."
                

```
