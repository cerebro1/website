---
title: "Require aws-node DaemonSet use IRSA"
category: AWS, EKS Best Practices
version: 1.6.0
subject: DaemonSet
policyType: "validate"
description: >
    According to EKS best practices, the `aws-node` DaemonSet is configured to use a role assigned to the EC2 instances to assign IPs to Pods. This role includes several AWS managed policies that effectively allow all Pods running on a Node to attach/detach ENIs, assign/unassign IP addresses, or pull images from ECR. Since this presents a risk to your cluster, it is recommended that you update the `aws-node` DaemonSet to use IRSA. This policy ensures that the `aws-node` DaemonSet running in the `kube-system` Namespace is not still using the `aws-node` ServiceAccount.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//aws/require-aws-node-irsa/require-aws-node-irsa.yaml" target="-blank">/aws/require-aws-node-irsa/require-aws-node-irsa.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-aws-node-irsa
  annotations:
    policies.kyverno.io/title: Require aws-node DaemonSet use IRSA
    policies.kyverno.io/category: AWS, EKS Best Practices
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: DaemonSet
    kyverno.io/kyverno-version: 1.8.2
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.24"
    policies.kyverno.io/description: >-
      According to EKS best practices, the `aws-node` DaemonSet is configured to use
      a role assigned to the EC2 instances to assign IPs to Pods. This role includes
      several AWS managed policies that effectively allow all Pods running on a Node
      to attach/detach ENIs, assign/unassign IP addresses, or pull images from ECR.
      Since this presents a risk to your cluster, it is recommended that you update
      the `aws-node` DaemonSet to use IRSA. This policy ensures that the `aws-node` DaemonSet
      running in the `kube-system` Namespace is not still using the `aws-node` ServiceAccount.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: validate-node-daemonset-irsa
    match:
      any:
      - resources:
          kinds:
          - DaemonSet
          names: 
          - aws-node
          namespaces:
          - kube-system
    validate:
      message: "Update the aws-node daemonset to use IRSA."
      pattern:
        spec:
          template:
            spec:
              serviceAccountName: "!aws-node"
```
