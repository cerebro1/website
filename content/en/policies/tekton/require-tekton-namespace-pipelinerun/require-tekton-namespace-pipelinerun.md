---
title: "Require Namespace for Tekton PipelineRun"
category: Tekton
version: 1.7.0
subject: PipelineRun
policyType: "validate"
description: >
    A Namespace is required for a PipelineRun and may not be set to `default`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//tekton/require-tekton-namespace-pipelinerun/require-tekton-namespace-pipelinerun.yaml" target="-blank">/tekton/require-tekton-namespace-pipelinerun/require-tekton-namespace-pipelinerun.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-tekton-namespace-pipelinerun
  annotations:
    policies.kyverno.io/title: Require Namespace for Tekton PipelineRun
    policies.kyverno.io/category: Tekton
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: PipelineRun
    kyverno.io/kyverno-version: 1.7.2
    policies.kyverno.io/minversion: 1.7.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/description: >- 
      A Namespace is required for a PipelineRun and may not be set to `default`.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: check-pipelinerun-namespace
    match:
      any:
      - resources:
          kinds:
          - PipelineRun
    preconditions:
      all:
      - key: "{{ request.operation || 'BACKGROUND' }}"
        operator: Equals
        value: CREATE
    validate:
      message: "A namespace is required and may not be set to default."
      pattern:
        metadata:
          namespace: "!default"
```
