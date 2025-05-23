---
title: "Disallow hostPorts Range (Alternate)"
category: Pod Security Standards (Baseline)
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    Access to host ports allows potential snooping of network traffic and should not be allowed by requiring host ports be undefined (recommended) or at minimum restricted to a known list. This policy ensures the `hostPort` field, if defined, is set to either a port in the specified range or to a value of zero. This policy is mutually exclusive of the disallow-host-ports policy. Note that Kubernetes Pod Security Admission does not support the host port range rule.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//pod-security/baseline/disallow-host-ports-range/disallow-host-ports-range.yaml" target="-blank">/pod-security/baseline/disallow-host-ports-range/disallow-host-ports-range.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-host-ports-range
  annotations:
    policies.kyverno.io/title: Disallow hostPorts Range (Alternate)
    policies.kyverno.io/category: Pod Security Standards (Baseline)
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    kyverno.io/kyverno-version: 1.6.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.22-1.23"
    policies.kyverno.io/description: >-
      Access to host ports allows potential snooping of network traffic and should not be
      allowed by requiring host ports be undefined (recommended) or at minimum restricted to a known list.
      This policy ensures the `hostPort` field, if defined, is set to either a port in the specified range
      or to a value of zero. This policy is mutually exclusive of the disallow-host-ports policy.
      Note that Kubernetes Pod Security Admission does not support the host port range rule.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: host-port-range
      match:
        any:
        - resources:
            kinds:
              - Pod
      preconditions:
        all:
        - key: "{{ request.operation }}"
          operator: NotEquals
          value: DELETE
      validate:
        message: >-
          The only permitted hostPorts are in the range 5000-6000 or 0.
        deny:
          conditions:
            all:
            - key: "{{ request.object.spec.[ephemeralContainers, initContainers, containers][].ports[].hostPort }}"
              operator: AnyNotIn
              value: 5000-6000
            - key: "{{ request.object.spec.[ephemeralContainers, initContainers, containers][].ports[].hostPort }}"
              operator: AnyNotIn
              value:
              - 0
```
