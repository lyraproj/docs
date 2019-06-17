---
title: Building blocks
layout: default
category: Reference
order: 2
cat_order: 3
---

# Building blocks

Consists of a Provider Framework, a Lookup Framework, a Workflow Engine, and polyglot Language front-ends.

<p align="center"><img src="buildingblocks.png" alt="Lyra"></p>

## Workflow
A Workflow describes a collection of Steps and must be declared in such a way that all input requirements of all contained activities have the potential to be fulfilled within the workflow itself.
The workflow engine loads this declaration, validates it, and invokes its steps in an order determined by their *parameters* and *returns* declarations.

## Step
A Step is an abstract executable unit, analog to “a function in a program”, that consumes named parameters and produces named return values. The workflow engine triggers its execution when all its parameter requirements are met. When several steps are grouped into a workflow, the execution in turn produces returns available as parameters to other steps which then get triggered. In essence, this means that all ordering of steps and all concerns regarding parallel or sequential execution, can be expressed as parameter declarations and return values that fulfills them.

A Step may have a Guard. A Guard is an expression consisting of named variables,  the boolean operators “!”, “and”, and “or”. It may also contain parentheses to enforce order of evaluation.

A Step may be embedded in a workflow, in which case it can only be seen by the workflow that contains it, or be public, in which case it can be referenced from anywhere. A CLI command and `Call` step will typically invoke a public workflow.

A Step can be a `Resource`, an `Action`, a `Call`, a `State Handler`, or a `Workflow`.

## Resource
A special form of declarative step that defines a desired state. A resource is always managed by a `State Handler` responsible for setting or mutating its state.

## Action
A step that contains a function written in a supported programming language. The code is evaluated by the corresponding language front-end.

## Call
A step that calls executes another step that is publicly available (can be found by the loader).

## Handler
A step that contains CRUD (Create, Read, Update, Delete) functions written in a supported programming language. The workflow engine selects the functions to be evaluated and the evaluation is then performed by the corresponding language front-end.
