---
title: Running Lyra in Kubernetes controller mode
layout: default
category: Using Lyra
order: 2
cat_order: 2
---

# Running Lyra in Kubernetes controller mode

Run Lyra in Kubernetes controller mode if you want Lyra to continuously apply a workflow and make sure that your resources exist in a desired state. 

Before you run Lyra in controller mode, make sure you've set up the [Kubernetes command-line tool](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

> **Important:** Deploying a workflow creates real resources and could incur a charge from your cloud provider.

To run Lyra in controller mode:

1. Install the Lyra workflow `CustomResourceDefinition` (CRD): 

   ```
   kubectl apply -f k8s/lyra_v1alpha1_workflow_crd.yaml
   ```

2. Start Lyra in controller mode: 

   ```
   ./build/bin/lyra controller --debug
   ```

3. Open a second Terminal window and create a Workflow resource: 

   ```
   kubectl apply -f k8s/vpc-workflow.yaml
   ```

4. Inspect the resource: 
   
   ```
   kubectl get workflows
   ```

5. (Optional) Delete the resources associated with the workflow:
   
   ```
   kubectl delete workflow vpc-workflow
   ```

> **Note**: The `tag` data for Kubernetes workflows is specified in the data section of `k8s/vpc-workflow.yaml` in your Lyra directory.