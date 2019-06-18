---
title: Deploying a Lyra workflow
layout: default
category: Using Lyra
order: 1
cat_order: 2
---

# Deploying a Lyra workflow

You can deploy workflows using the Lyra CLI.

Before you deploy a workflow, make sure you've [installed Lyra](./installing-lyra.html). You can write your own workflow to deploy, or use one of the sample workflows found in the [workflows directory](https://github.com/lyraproj/lyra/tree/master/workflows). 

> **Important:** Deploying a workflow creates real resources and could incur a charge from your cloud provider. If you'd like to try Lyra without spending any money, try out Foobernetes, a fictional cloud provider used to illustrate and test the capabilities of Lyra. Fore more information, read the [annotated Foobernetes workflow](https://github.com/lyraproj/lyra/blob/master/workflows/foobernetes.yaml). 

## Deploy a workflow

Use `lyra apply` to deploy a Lyra workflow:

```
./build/bin/lyra apply <YOUR_WORKFLOW>
```
> **Tip**: Leave out the file extension for the workflow.

> **Tip**: Use the `--debug` flag for a more detailed description of what is happening when you perform an action with Lyra.

## Delete a workflow

Use the `delete` command to completely remove the resources created by a workflow:

```
./build/bin/lyra delete <YOUR_WORKFLOW>
```

## Example: Deploying the `aws` workflow

The `aws` workflow example manages several resources on AWS. 

The workflow incorporates external data in the form of AWS `tags` that Lyra loads using a Go implementation of Hiera. The `tags` are specified in the `data.yaml` file found in the root directory of your Lyra installation. Before you apply this workflow, make sure you've [set up AWS credentials](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html) and added them to `~/.aws/credentials`.

To apply the `aws` workflow, use the following command:

```
./build/bin/lyra apply aws --debug
```

To delete the workflow, use the following command::

```
./build/bin/lyra delete aws --debug
```

> **Note**: Currently, the defualt region is hardcoded to "eu-west-1". To use a different region, use `export AWS_DEFAULT_REGION="<YOUR_REGION>"`.