---
title: "Disallow Capabilities"
category: Pod Security Standards (Baseline)
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    Adding capabilities beyond those listed in the policy must be disallowed.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//pod-security/baseline/disallow-capabilities/disallow-capabilities.yaml" target="-blank">/pod-security/baseline/disallow-capabilities/disallow-capabilities.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-capabilities
  annotations:
    policies.kyverno.io/title: Disallow Capabilities
    policies.kyverno.io/category: Pod Security Standards (Baseline)
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.6.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.22-1.23"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Adding capabilities beyond those listed in the policy must be disallowed.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: adding-capabilities
      match:
        any:
        - resources:
            kinds:
              - Pod
      preconditions:
        all:
        - key: "{{ request.operation || 'BACKGROUND' }}"
          operator: NotEquals
          value: DELETE
      validate:
        message: >-
          Any capabilities added beyond the allowed list (AUDIT_WRITE, CHOWN, DAC_OVERRIDE, FOWNER,
          FSETID, KILL, MKNOD, NET_BIND_SERVICE, SETFCAP, SETGID, SETPCAP, SETUID, SYS_CHROOT)
          are disallowed.
        deny:
          conditions:
            all:
            - key: "{{ request.object.spec.[ephemeralContainers, initContainers, containers][].securityContext.capabilities.add[] }}"
              operator: AnyNotIn
              value:
              - AUDIT_WRITE
              - CHOWN
              - DAC_OVERRIDE
              - FOWNER
              - FSETID
              - KILL
              - MKNOD
              - NET_BIND_SERVICE
              - SETFCAP
              - SETGID
              - SETPCAP
              - SETUID
              - SYS_CHROOT
```
