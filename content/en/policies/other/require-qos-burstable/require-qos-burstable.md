---
title: "Require QoS Burstable"
category: Other, Multi-Tenancy
version: 
subject: Pod
policyType: "validate"
description: >
    Pod Quality of Service (QoS) is a mechanism to ensure Pods receive certain priority guarantees based upon the resources they define. When a Pod has at least one container which defines either requests or limits for either memory or CPU, Kubernetes grants the QoS class as burstable if it does not otherwise qualify for a QoS class of guaranteed. This policy requires that a Pod meet the criteria qualify for a QoS of burstable. This policy is provided with the intention that users will need to control its scope by using exclusions, preconditions, and other policy language mechanisms.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/require-qos-burstable/require-qos-burstable.yaml" target="-blank">/other/require-qos-burstable/require-qos-burstable.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-qos-burstable
  annotations:
    policies.kyverno.io/title: Require QoS Burstable
    policies.kyverno.io/category: Other, Multi-Tenancy
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Pod Quality of Service (QoS) is a mechanism to ensure Pods receive certain
      priority guarantees based upon the resources they define. When a Pod has at least
      one container which defines either requests or limits for either memory or CPU,
      Kubernetes grants the QoS class as burstable if it does not otherwise qualify for a QoS class of guaranteed.
      This policy requires that a Pod meet the criteria qualify for a QoS of burstable.
      This policy is provided with the intention that users will need to control its scope by using
      exclusions, preconditions, and other policy language mechanisms.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: burstable
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "At least one container in the Pod must define either requests or limits for either CPU or memory."
      deny:
        conditions:
          all:
          - key: requests
            operator: AnyNotIn
            value: "{{ request.object.spec.containers[].resources.keys(@)[] }}"
          - key: limits
            operator: AnyNotIn
            value: "{{ request.object.spec.containers[].resources.keys(@)[] }}"
```
