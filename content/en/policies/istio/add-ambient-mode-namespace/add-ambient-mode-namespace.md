---
title: "Add Istio Ambient Mode"
category: Istio
version: 1.6.0
subject: Namespace
policyType: "mutate"
description: >
    In order for Istio to include namespaces in ambient mode, the label `istio.io/dataplane-mode`  must be set to `ambient`. As an alternative to rejecting Namespace definitions which don't already  contain this label, it can be added automatically. This policy adds the label `istio.io/dataplane-mode` set to `ambient` for all new Namespaces.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//istio/add-ambient-mode-namespace/add-ambient-mode-namespace.yaml" target="-blank">/istio/add-ambient-mode-namespace/add-ambient-mode-namespace.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-ambient-mode-namespace
  annotations:
    policies.kyverno.io/title: Add Istio Ambient Mode
    policies.kyverno.io/category: Istio
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.8.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.24"
    policies.kyverno.io/subject: Namespace
    policies.kyverno.io/description: >-
      In order for Istio to include namespaces in ambient mode, the label `istio.io/dataplane-mode` 
      must be set to `ambient`. As an alternative to rejecting Namespace definitions which don't already 
      contain this label, it can be added automatically. This policy adds the label `istio.io/dataplane-mode`
      set to `ambient` for all new Namespaces.
spec:
  rules:
  - name: add-ambient-mode-enabled
    match:
      any:
      - resources:
          kinds:
          - Namespace
    mutate:
      patchStrategicMerge:
        metadata:
          labels:
            istio.io/dataplane-mode: ambient

```
