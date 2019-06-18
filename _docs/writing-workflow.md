---
title: Writing a Lyra workflow in YAML
layout: default
category: Using Lyra
order: 1
cat_order: 2
---

# Writing a Lyra workflow in YAML

Use a YAML workflow to write steps for declarative resources or reference other workflows.

Before you start writing your workflow, map out the resources you're creating and think about the parameters that each step requires. Look at the typesets in Lyra's build/types directory to get an idea of the required attributes for the type you're using.  

## Declaring parameters and returns

Each step in a workflow consumes parameters and, if necessary, produces returns for other steps in the workflow to use. Lyra can only execute a step when all of it's required parameters are available. If you need to use parameters that are generated from outside of a workflow, you can declare these as top-level workflow parameters. Similarly, you can declare top-level workflow returns -- the end result of a workflow -- for use in other workflows. Parameters and returns are correlated by name and so must be unique within a workflow.


Lyra workflows require a top-level `parameters` declaration under the following circumstances:
* If a step in your workflow requires a parameter that is not generated as a return by another step in the workflow. For example, if a step in your workflow requires a subnet ID, and you want to use an existing subnet instead of creating a subnet in the workflow, you must declare the subnet parameter using the `parameters` declaration.  
* If you've defined parameter in your `data.yaml` file and need to use a `lookup` key to map the parameter.


A `parameters` declaration can include the following:
* The name of an attribute. For example:
  
  ```
  parameters: kaneda
  ```

* An array of names. For example:
  
  ```
  parameters: [kaneda, tetsuo]
  ```

* A hash with a name mapping to a hash of type and an optional `lookup` key. For example: 

  ```
  parameters:
    akira_characters:
      type: Hash[String,String]
      lookup: akira.tetsuo
  ```

The `lookup` key corresponds to a key in the `data.yaml` file found in Lyra's root directory. The `lookup` keys you reference in your `parameters` declaration must exist in the `data.yaml` file before you apply the workflow. The `data.yaml` entry for characters looks like this:

```
Akira:
  characters:
    tetsuo: "Tetsuo Shima"
```

Use the `returns` key to map the expected output values of your workflow. For example, you could write a workflow that provisions two load balancers and returns two load balancer IDs. Contrary to `parameters`, you cannot declare the type for a return, as Lyra always infers the type. A `returns` declaration can include the following:
* The name of a return attribute from a step in the workflow, for example 
  
  ```
  returns: kaneda
  ```

* A list of names of return attributes from steps in the workflow. For example: 
  
  ```
  returns: [kaneda, tetsuo]
  ```

* A hash of aliases for return attributes. Use an alias to give a return attribute a different name to the attribute it references. For example:
  
  ```
  returns:
    biker: kaneda
    psychic: tetsuo 
  ```

## Writing steps

Steps make up the body of a workflow and define the workflow's behavior. Each step is represented by a hash under the parent `Steps` hash. There is no need to place your steps in a particular order. Lyra infers the order of the steps based on their `parameters` and `returns`. 

The body of a typical Lyra workflow uses the following syntax:

```
Steps:
  my_step:
    returns: return_name
  Namespace::type
    required_key: value 
    optional_key: value
    parameters: [$parameter1, $parameter2]  
```

A step hash consists of the following:
* The name of the step.
* An optional `returns` key. Use this key if you expect the step to return a value for another step in the workflow to consume as a parameter. As with workflow returns, step returns take a single name, a list of names, or a hash of aliases.
* A type declaration in the form `Namespace::Type` with a hash of required and optional keys, including any parameters that you're passing in from another step. Prepend parameters with a dollar sign (`$`).

The `Namespace` in your type declaration must match a typeset from Lyra's `build/types` directory. The `type` must match a type in that namespace. For example, to provision a Google Cloud Project (GCP) cluster, find the `Google.pp` file in `build/types` directory and search for the container cluster type: `Container_cluster`. The resulting type declaration for a GCP cluster is `Google::Container_cluster`.  

The typesets in `build/types` also present a list of optional and required parameters for each type. This extract from the Container_cluster type shows three optional attributes -- `master_version`, `min_master_version`, and `monitoring_service`, and -- and one required attribute, `name`.

```
attributes => {
…
  'master_version' => Optional[String],
  'min_master_version' => Optional[String],
  'monitoring_service' => Optional[String],
  'name' => String,
…
}
```
## Using a workflow as a step

Use the `call` key to link to a workflow as a step. 
<!-- This should work, but I'm getting a type error. I'll edit it once I figure out what I'm doing wrong --> 

The workflow in the following example calls the workflow `my_database`. The `my_database` workflow produces the `databaseID` needed by the `app-server` step.

```
…
steps:
  app-server:
    returns:
      appServerID: instanceID
    Foobernetes::Instance:
      location: eu1
      image: lyra::application
      config:
        name: app-server1
        databaseID: $databaseID
      cpus: 4
      memory: 8G

  database:
    call: my_database
    returns: databaseID  
```

The `my_database` workflow looks like this:

```  
returns: [ databaseID ]    

steps:

  database:
    returns:
      databaseID: instanceID
    Foobernetes::Instance:
      location: eu1
      image: "lyra::database"
      cpus: 16
      memory: 64G  
``` 

## Workflow example

This example is based on the Foobernetes, a fictional cloud provider used to illustrate and test the capabilities of Lyra. 

The workflow deploys a fictional application consisting of a database, an application server, a web server, and a load balancer. Each step in this workflow is a resource step -- a declarative step that defines the desired state for a resource.


```
parameters:
  load_balancer_policy:
    type: String
    lookup: foobernetes.policies
returns: [ loadBalancerID ]    

steps:

  web-servers:
    returns: webServerID
    Foobernetes::Webserver:
      port: 8080
      appServers: [ $appServerID ]

  loadbalancer:
    returns: loadBalancerID
    Foobernetes::Loadbalancer:
      loadBalancerIP: 10.0.0.1
      location: eu1
      replica: false
      webServerIDs: [ $webServerID ]
      tags:
        team: "lyra team"
        role: primary

  app-server:
    returns:
      appServerID: instanceID
    Foobernetes::Instance:
      location: eu1
      image: lyra::application
      config:
        name: app-server1
        databaseID: $databaseID
      cpus: 4
      memory: 8G

  database:
    returns:
      databaseID: instanceID
    Foobernetes::Instance:
      location: eu1
      image: "lyra::database"
      cpus: 16
      memory: 64G
```

After you apply the workflow, Lyra orders and executes the steps based on their parameters and returns. In this case, Lyra executes the `database` step first as it doesn't require any implicit parameters from other steps in the workflow to provision the resource. Lyra provisions the database and returns `$databaseID`. Because the `$databaseID` return is now available, Lyra runs the `app-server` step and passes in `$databaseID` as a parameter. This process continues until Lyra provisions the load balancer and returns `loadBalancerID`. 

Instead of creating real resources, applying this workflow in Lyra produces a .JSON representation of the deployed fictional resources in Lyra's root directory.

Fore more information on testing with Foobernetes, read the [annotated Kubernetes workflow](https://github.com/lyraproj/lyra/blob/master/workflows/foobernetes.yaml).
