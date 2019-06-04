![Lyra logo](assets/lyrabanner.png)

Lyra is an open source workflow engine for deploying, configuring, and maintaining cloud and API-driven services. 

If an application spans multiple cloud services -- or runs on Kubernetes, but relies on API-driven services that exist outside of the cloud -- deploying, configuring, and maintaining that application can be complex. Lyra cuts down on that complexity by providing a unified interface for cloud and API-driven services. For example, you can write a Lyra workflow to provision an Amazon Web Services (AWS) Kubernetes cluster, create namespaces, edit or create a security group, and deploy an application to the cluster together with its dependencies. Your workflow can include code to manage your resources. For example, you could write functions to send HTTP requests to an endpoint, send notifications to a messaging application, or remove an instance from a load balancer.

## Using Lyra

Lyra uses steps and workflows to interact with cloud and API-driven resources. 

A workflow is a collection of steps. Each step consumes parameters and, if necessary, produces returns. For example, before you can provision a subnet in AWS, you need a Virtual Private Cloud (VPC). The YAML workflow below contains two steps, one for the VPC, and one for the subnet. The first step creates the VPC and returns an identifier, `vpc_id`:

```
vpc:
    returns: vpc_id
    Aws::Vpc:
      cidr_block: 192.168.0.0/16
      instance_tenancy: default
```      

The second step creates the subnet. The subnet step depends on the parameter $vpc_id from the vpc step above:

```
subnet:
    returns: subnet_id
    Aws::Subnet:
      vpc_id: $vpc_id
      cidr_block: 192.168.1.0/24
      tags:
        name: lyra-subnet-1
        created_by: lyra
```        

You can place steps in any order; Lyra's workflow engine infers the order of the steps in a workflow based on their parameters and returns. 

Much of Lyra's power comes from the ability to mix imperative and declarative steps. Actions are imperative steps that contain a Go or Typescript function. Resources are declarative steps -- written in Go, Typescript, Puppet, or YAML -- that define a desired state. The vpc and subnet steps above are examples of resources. 

Lyra gives you the freedom to decide whether you want to use a resource or an action to solve a particular problem. Because workflows also function as steps, you can string multiple workflows together to perform complex tasks. For example, you could create a workflow to chain together a resource that provisions a Kubernetes cluster on AWS, and an action that sends a Slack notification once the cluster is provisioned. 

Lyra provides a Command Line Interface (CLI) for interacting with your workflows. Alternatively, Lyra can operate persistently in Kubernetes controller mode. This allows Lyra to continuously apply workflows and reconcile Kubernetes events.
