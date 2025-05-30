---
title: "Restrict Wildcards in Resources"
category: Security, EKS Best Practices
version: 1.6.0
subject: ClusterRole, Role, RBAC
policyType: "validate"
description: >
    Wildcards ('*') in resources grants access to all of the resources referenced by the given API group and does not follow the principal of least privilege. As much as possible, avoid such open resources unless scoped to perhaps a custom API group. This policy blocks any Role or ClusterRole that contains a wildcard entry in the resources list found in any rule.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/restrict-wildcard-resources/restrict-wildcard-resources.yaml" target="-blank">/other/restrict-wildcard-resources/restrict-wildcard-resources.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-wildcard-resources
  annotations:
    policies.kyverno.io/title: Restrict Wildcards in Resources
    policies.kyverno.io/category: Security, EKS Best Practices
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: ClusterRole, Role, RBAC
    kyverno.io/kyverno-version: 1.7.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/description: >-
      Wildcards ('*') in resources grants access to all of the resources referenced by
      the given API group and does not follow the principal of least privilege. As much as possible,
      avoid such open resources unless scoped to perhaps a custom API group.
      This policy blocks any Role or ClusterRole that contains a wildcard entry in
      the resources list found in any rule.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: wildcard-resources
      match:
        any:
        - resources:
            kinds:
              - Role
              - ClusterRole
      validate:
        message: "Use of a wildcard ('*') in any resources is forbidden."
        deny:
          conditions:
            any:
            - key: "{{ contains(request.object.rules[].resources[], '*') }}"
              operator: Equals
              value: true
```
