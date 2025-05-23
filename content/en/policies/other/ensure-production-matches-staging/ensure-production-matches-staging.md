---
title: "Ensure Production Matches staging"
category: Other
version: 1.6.0
subject: Deployment
policyType: "validate"
description: >
    It is common to have two separate Namespaces such as staging and production in order to test and promote app deployments in a controlled manner. In order to ensure that level of control, certain guardrails must be present so as to minimize regressions or unintended behavior. This policy has a set of three rules to try and provide some sane defaults for app promotion across these two environments (Namespaces) called staging and production. First, it makes sure that every Deployment in production has a corresponding Deployment in staging. Second, that a production Deployment uses same image name as its staging counterpart. Third, that a production Deployment uses an older or equal image version as its staging counterpart.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/ensure-production-matches-staging/ensure-production-matches-staging.yaml" target="-blank">/other/ensure-production-matches-staging/ensure-production-matches-staging.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: ensure-production-matches-staging
  annotations:
    policies.kyverno.io/title: Ensure Production Matches staging
    policies.kyverno.io/category: Other
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kyverno-version: 1.7.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/subject: Deployment
    policies.kyverno.io/description: >-
      It is common to have two separate Namespaces such as staging and production
      in order to test and promote app deployments in a controlled manner. In order to ensure
      that level of control, certain guardrails must be present so as to minimize regressions or
      unintended behavior. This policy has a set of three rules to try and provide some sane defaults
      for app promotion across these two environments (Namespaces) called staging and production. First,
      it makes sure that every Deployment in production has a corresponding Deployment in staging. Second,
      that a production Deployment uses same image name as its staging counterpart. Third, that
      a production Deployment uses an older or equal image version as its staging counterpart.
spec:
  validationFailureAction: Enforce
  background: true
  rules:
#######################
# Ensures that each Deployment being created in production
# has an existing Deployment already in staging of the same name.
  - name: require-staging-deployment
    match:
      any:
      - resources:
          namespaces:
          - production
          kinds:
          - Deployment
    preconditions:
      any:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: AnyIn
        value:
        - CREATE        
        - UPDATE
    context:
    - name: deployment_count
      apiCall:
        urlPath: "/apis/apps/v1/namespaces/staging/deployments"
        jmesPath: "items[?metadata.name=='{{ request.object.metadata.name }}'] || `[]` | length(@)"
    validate:
      message: "Every Deployment in production requires a corresponding Deployment in staging."
      deny:
        conditions:
          any:
          - key: "{{deployment_count}}"
            operator: Equals
            value: 0
#######################
# Ensures that each corresponding Deployment in staging and production
# Namespaces uses the same image name (not tag).
  - name: require-same-image
    match:
      any:
      - resources:
          namespaces:
          - production
          kinds:
          - Deployment
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: AnyIn
        value:
        - CREATE        
        - UPDATE
      - key: "{{ deployment_count }}"
        operator: GreaterThan
        value: 0
    context:
    - name: deployment_count
      apiCall:
        urlPath: "/apis/apps/v1/namespaces/staging/deployments"
        jmesPath: "items[?metadata.name=='{{ request.object.metadata.name}}']  || `[]` | length(@)"
    - name: deployment_images
      apiCall:
        urlPath: "/apis/apps/v1/namespaces/staging/deployments/{{ request.object.metadata.name }}"
        jmesPath: "spec.template.spec.containers[].image.split(@, ':')[0]"
    validate:
      message: "Every Deployment in production is required to use the same image(s) as in staging."
      deny:
        conditions:
          any:
          - key: "{{ request.object.spec.template.spec.containers[].image.split(@, ':')[0]  }}"
            operator: AnyNotIn
            value: "{{ deployment_images }}"
#######################
# Ensures that each image version in production is less than or
# equal to its corresponding image version in staging. Uses the container
# name as the basis for comparison. Recommended to pair this policy
# with another policy or rule which enforces only semver image tags, or otherwise extend this rule
# with a precondition which filters out everything else.
  - name: require-same-or-older-imageversion
    match:
      any:
      - resources:
          namespaces:
          - production
          kinds:
          - Deployment
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: AnyIn
        value:
        - CREATE        
        - UPDATE
      - key: "{{ deployment_count }}"
        operator: GreaterThan
        value: 0
    context:
    - name: deployment_count
      apiCall:
        urlPath: "/apis/apps/v1/namespaces/staging/deployments"
        jmesPath: "items[? metadata.name == '{{ request.object.metadata.name }}'  ]  | length(@)"
    - name: deployment_containers
      apiCall:
        urlPath: "/apis/apps/v1/namespaces/staging/deployments/{{ request.object.metadata.name }}"
        jmesPath: "spec.template.spec.containers[]"
    validate:
      message: "Every Deployment in production is required to use an image version less than or equal to the one in staging."
      foreach:
      - list: "request.object.spec.template.spec.containers[]"
        deny:
          conditions:
            any:
            # Uses the container name as the basis for comparing the image tag. Only semver has been tested.
            - key: "{{ element.image.split(@,':')[1] }}"
              operator: GreaterThan
              value: "{{ deployment_containers[?name == '{{element.name}}'].image.split(@,':')[1] | [0] }}"
```
