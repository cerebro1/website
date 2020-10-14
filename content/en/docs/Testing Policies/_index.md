---
title: Testing Policies 
slug: /testing-policy
description: 
weight: 3
---

The resources definitions for testing are located in the [test](https://github.com/kyverno/kyverno/tree/master/test) directory. Each test contains a pair of files: one is the resource definition, and the second is the Kyverno policy for this definition.

## Test using kubectl

To do this you should [install Kyverno to the cluster](https://kyverno.io/docs/getting-started/).

For example, to test the simplest Kyverno policy for `ConfigMap`, create the policy and then the resource itself via `kubectl`:

````bash
cd test
kubectl create -f policy/policy-CM.yaml
kubectl create -f resources/CM.yaml
````
Then compare the original resource definition in `CM.yaml` with the actual one:

````bash
kubectl get -f resources/CM.yaml -o yaml
````

## Test using Kyverno CLI

The Kyverno CLI allows testing policies before they are applied to a cluster. It is documented at [Kyverno CLI](https://kyverno.io/docs/kyverno-cli/)