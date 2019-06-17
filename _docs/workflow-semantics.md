---
title: Workflow semantics
layout: default
category: Reference
order: 3
cat_order: 3
---

# Workflow semantics

This document describes the generic semantics of a workflow in a language neutral way. See [Puppet Workflow DSL](workflow-puppet-dsl.md) and [YAML Workflow](workflow-yaml.md) for language specific declarations.

A Workflow consists of a set of steps that are either declarative or imperative in nature. A `resource` of a certain type, that maps a desired state to to a handler for that state, is an example of a declarative step, whereas an `action` step with a code block, is an example of an imperative step.

A workflow, being a step in its own right, may be used as a step in another workflow.

## Steps

All steps consist of the following elements:

Element|Description|Mandatory
-------|-----------|------
style|one of `action`, `resource`, `state_handler`, or `workflow`|yes
properties|literal hash declaring `input`, `output`, and other properties|yes
iteration|enables multiple invocations of data or code|no
data or code|interpreted differently depending on style|no

### Common step properties
##### parameters
declares a set of parameters. Each parameter has a name and optional `type` and `lookup` expressions.

###### scope
A parameter without a lookup expression must denote a variable known to the containing workflow, i.e. a parameter to the workflow itself or a `returns` variable from sibling step. When a workflow contains other workflows, each workflow acts as its own closure and nested references are not allowed.

The input `type` can be omitted when the parameter maps to an output variable from another step because the type can then be inferred from the output.

A step will have access to the `parameters` that it declares. It will not have access to any other variables. The reason for this is that steps may run in a process different from the workflow engine and will be executed with the presumption that the remote process holds no state. A state must be passed to a step in order for it to execute.

##### returns
Declares the set of named and typed variables that a step will contribute to the workflow.

The `returns` for a `resource` step must always reference named attributes of its state. The `type` of such output can therefore always be inferred from the corresponding attribute and is hence optional.

An `alias` can be used when it is desirable to give the output variable a different name than the attribute it references.

##### when
declares a guard expression that must evaluate to `true` in order for the step to participate in the workflow. A guard expression consists of names of guard steps combined using a language specific notation for `and`, `or`, and `not` and grouped using parenthesis.

Since guards will impose conditional paths in the workflow, the number of guards must be limited or the validation will become very resource consuming. The current limit is 8 guards per workflow which results in a maximum of in 2<sup>8</sup> (256) possible paths of execution. More complex workflows can be created using sub-workflows contained in a parent workflow.

#### Collectors
A collector can be declared to iterate over `parameters` data or literals.

Collector type|Description|Parameters|Variables
--------------|-----------|----------|---------
times|Calls a contained step the number of times given by count|`count` - a positive integer|`index` - an integer that starts with zero and increments up to, but not including count
each|Calls a contained step once with all elements of a hash or an array|`collection` - a hash or array to iterate over|`value` - each element in the array or `key` and `value` to iterate over the associations of a hash

A collector step will always produce an array of values where each value is the result of calling the contained step.

### StateHandler
A stateHandler implements a predefined set of functions (an interface) that performs tasks related to a subject. Lyra currently only defines one such interface, the CRUD, which is implemented by resource handlers and the only reason to write an action, is when there's a desire to implement a handler for a resource state.

It is expected that Lyra will add additional interfaces in the future.

### Action
An action defines an imperative function that performs some task based on its input and provides some output.

### Call
A call is a declaration of an invocation of another publicly available remote step. The call's *parameters* and *returns* declarations can be used to map names used locally to names expected and produced by the remote step.

### Resource
A resource is a declaration of a desired state of some entity external to the workflow. The `<data or code>` block is a hash that defines this state. The block is described by an `Type` which is defined by a `stateHandler`, a step that can perform `create`, `read`, `update`, and `delete` operations on the state.

When the workflow interacts with Resource steps, it delegates the apply of the step to the CRUD layer which in turn consults the `stateHandler` registered for the state's `Type`. The CRUD layer will determine how to use the `stateHandler` depending on how the workflow engine was invoked.

#### Resource Attributes
##### type
A mandatory type name that denotes the resource type.

##### external_id
An optional ID that denotes the external identifier of read-only (unmanaged) resource. The intended use for this is to establish relationships between the infrastructure defined by the workflow and pre-existing infrastructure that the workflow should make no attempt to manage.

### Workflow
A workflow combines a set of steps into a process. The `<data or code>` block is code that declares those steps.
