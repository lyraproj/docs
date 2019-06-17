---
title: YAML workflows
layout: default
category: Reference
order: 4
cat_order: 3
---
# YAML workflows

For an explanation of the semantics of each element, please see [Workflow Semantics](workflow-semantics.md)

## Activities

The YAML workflow is limited to two types of activities, the `workflow` and the `resource`. A hash containing the key `activities` is considered a workflow and a hash containing the key `state` is considered a resource.

### Common step properties
##### parameters

The parameters declaration is a hash with names mapping to hash of type and optional lookup where:

```yaml
parameters:
  i1:
    type: String
  i2:
    type: String
    lookup: some.key
```

declares the named parameters _i1_ and _i2_. The parameter _i2_ will get its value from a lookup.

When using YAML, it is possible to infer the `parameters` delcaration which means that it can often be omitted unless lookup is desired.

#### returns
Can use the same syntax as `parameters` but without the ability to declare `lookup` or:

Single name of a state attribute (resource) or output of contained step (workflow)

```yaml
returns: attr_x
```

List of names of a state attributes (resource) or output of contained activities (workflow)

```yaml
returns: [attr_x, attr_y]
```

An alias is used when it is desirable to give the output variable a different name than the attribute it references:

```yaml
returns:
  alias_x: attr_x
  alias_y: attr_y
```

#### when
a step is considered to have a guard when it declares:

```yaml
when: <guard expression>
```
     
the `<guard expression>` is a string containing a boolean expression consisting of variable names that are combined using the keywords `and` and `or`. The expression can also use parentheses to enforce evaluation order.

#### times collector
A times step declares that a step will be executed multiple times and that the resulting returns will be collected into an array. Example:

```yaml
    nodes:
      times: $ec2_nct
      as: idx
      step:
        <step to be repeated here>
```

#### each collector
An each step declares that a step will be executed once for each value in an array or hash. Example:

```yaml
loadbalancers:
  each:
    - [primary, '10.0.0.1', false]
    - [secondary, '10.0.0.2', true]
  as:
    - role
    - ip
    - replica
  step:
    returns: loadBalancerID
    resource: Foobernetes::Loadbalancer
    value:
      loadBalancerIP: $ip
      location: eu1
      replica: $replica
      webServerIDs: $webServers
      tags:
        team: "lyra team"
        role: $role
```

## Resource

A resource contains a state which must be a key/value hash. Each key must be a string.

### Variable references
An attribute value that starts with a '$' followed by an unqualified name is considered to be a reference to one of the named values available in the workflow, i.e. a parameter. Although an explicit `parameters` declaration can be specified, it is always optional unless there's a desire to declare a lookup.

### Examples

#### Simple resource

```yaml
vpc:
  returns: vpc_id
  resource: Aws::Vpc
  value:
    cidr_block: 192.168.0.0/16
    instance_tenancy: default
    tags: $tags
```

In this example, the inferred `parameters` will be:

```yaml
parameters:
  tags:
    type: Hash[String,String]
```

#### Resources using a collector

This example creates two Aws instances named "lyra-instance-1" and "lyra-instance-2"

```yaml
nodes:
  each: [lyra-instance-1, lyra-instance-2]
  as: name
  step:
    resource: Aws::Instance
    value:
      instance_type: 't2.nano'
      ami: 'ami-f90a4880'
      subnet_id: $subnet_id1
      tags:
        name: $name
        created_by: lyra
```

## Workflow

```yaml
parameters:
  tags:
    type: Hash[String,String]
    lookup: aws.tags
returns:
  vpc_id: String
  subnet_id: String
steps:
  route_table:
    resource: Aws::Route_table
    value:
      vpc_id: $vpc_id
      tags:
        name: lyra-routetable
        created_by: lyra
  vpc:
    returns: vpc_id
    resource: Aws::Vpc
    value:
      cidr_block: 192.168.0.0/16
      instance_tenancy: default
      tags: $tags
  subnet:
    returns:
      subnet_id: subnet_id
    resource: Aws::Subnet
    value:
      vpc_id: $vpc_id
      cidr_block: 192.168.1.0/24
      tags:
        name: lyra-subnet
        created_by: lyra
```
