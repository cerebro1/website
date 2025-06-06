---
title: "Restrict runtimeClass in CEL expressions"
category: PSP Migration in CEL
version: 
subject: Pod
policyType: "validate"
description: >
    The runtimeClass field of a Pod spec defines which container engine runtime should be used. In the previous Pod Security Policy controller, defining restrictions on which classes were allowed was permitted. Limiting runtime classes to only those which have been defined can prevent unintended running states or Pods which may not come online. This policy restricts the runtimeClass field to the values `prodclass` or `expclass`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//psp-migration-cel/restrict-runtimeClassName/restrict-runtimeClassName.yaml" target="-blank">/psp-migration-cel/restrict-runtimeClassName/restrict-runtimeClassName.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-runtimeclass
  annotations:
    policies.kyverno.io/title: Restrict runtimeClass in CEL expressions
    policies.kyverno.io/category: PSP Migration in CEL 
    policies.kyverno.io/subject: Pod
    kyverno.io/kyverno-version: 1.12.1
    kyverno.io/kubernetes-version: "1.26-1.27"
    pod-policies.kyverno.io/autogen-controllers: none
    policies.kyverno.io/description: >-
      The runtimeClass field of a Pod spec defines which container engine runtime should be used.
      In the previous Pod Security Policy controller, defining restrictions on which classes were allowed
      was permitted. Limiting runtime classes to only those which have been defined can prevent
      unintended running states or Pods which may not come online. This policy restricts the runtimeClass
      field to the values `prodclass` or `expclass`.
spec:
  validationFailureAction: Enforce
  background: false
  rules:
  - name: prodclass-or-expclass
    match:
      any:
      - resources:
          kinds:
          - Pod
    celPreconditions:
      - name: "operation-should-be-create"
        expression: "request.operation == 'CREATE'"
    validate:
      cel:
        expressions:
          - expression: "!has(object.spec.runtimeClassName) || object.spec.runtimeClassName in ['prodclass', 'expclass']"
            message: Only the runtime classes prodclass or expclass may be used.


```
