# Promotion Phase for GitOps use cases

Add a new lifecycle phase after post deployment dedicated to GitOps promotion tasks.

**State: DRAFTING**

## Motivation

Since one of the main use cases for Keptn is enhancement of GitOps workflows, we should have bigger focus on that
by introducing a dedicated stage after the post Application deployment phase for GitOps promotion tasks.
This will make it much clearer for Keptn users how to set up their GitOps-assisted workflows in multi-stage
environments.

## Explanation

The new promotion phase can be used by adding a list of references to `KeptnTaskDefinition` resources to a new field
called `promotionTasks` inside the `KeptnAppContext` CRD. Keptn will use this list the same way as for other phases
like post-deployment and execute the referenced tasks in parallel using Kubernetes Job resources.

## Trade-offs and mitigations

No real drawbacks come to mind, except some slight slowdown in progression of the lifecycle since the new phase
needs to be executed and the lifecycle-operator needs to check if any tasks need to be run.

## Breaking changes

This KEP would not have any breaking changes. The `KeptnAppContext` CRD would get an additional field called
`PromotionTasks` which contains a list of references to `KeptnTaskDefintion` resources that should be executed in this
phase.

## Prior art and alternatives

The alternative to this KEP is that users re-purpose the post-deployment phase for promotion, which would limit
flexibility since users lose the option to not promote when post-deployment tasks fail.

## Open questions

For now, none.

## Future possibilities

For now, none.
