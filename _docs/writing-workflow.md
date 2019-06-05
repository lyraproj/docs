---
title: Writing a Lyra workflow in YAML
layout: default
category: Using Lyra
order: 4
---

# Writing a Lyra workflow in YAML

Use a YAML workflow to write steps for declarative resources or reference other workflows.

Before you start writing your workflow, map out the resources you're creating and think about the values each step will return, and the parameters that each step requires before Lyra can apply it.  

## Declaring parameters and returns
<!--
Include:
Workflows and steps can both take parameters and returns. 
Required parameters and `data.yaml` 
-->

Lyra can infer parameter declarations in YAML workflows, so you can omit `parameters` unless you're using a `lookup` key. A step can only be executed when all parameters are available. Those parameters must come from either the top-level workflow parameters or the returns of other steps. Parameters and returns are correlated by name and so must be unique within a workflow.

Lyra workflows require a `parameters` declaration under the following circumstances:
* If a step in your workflow requires a parameter that is not generated as a return by another step in the workflow. For example, if a step in your workflow requires a subnet ID, and you want to use an existing subnet instead of creating a subnet in the workflow, you must declare the subnet parameter using the `parameters` declaration.  
* If you intend to use a `lookup` key to map to a parameter in your `data.yaml` file.


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
    characters:
      type: Hash[String,String]
      lookup: akira.characters
  ```

The `lookup` key corresponds to a key in the `data.yaml` file found in Lyra's root directory. The keys you reference in your `parameters` declaration must exist in the `data.yaml` file before you apply the workflow. The `data.yaml` entry for characters might look like this:

```
Example coming soon
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

## Workflow example

<!-->
Introduce example and explain use of parameters and returns. Also include explanation of foobernetes type and direct users to it's location in lyra repo. 
<-->

```
parameters:
  load_balancer_policy:
    type: String
    lookup: foobernetes.lb_policy
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

<!-- Stuff to add
referencing/linking another workflow
referencing/linking an imperative step? (if it's any different from another workflow) -->
