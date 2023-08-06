# Group Keptn Tasks and Evaluations as Hooks

Proposal to change the way the project refers to Keptn Tasks and Evaluations. Instead, refer to both as Keptn Hooks.

## Motivation

* Make it simpler for a user to understand the core concepts of Keptn Lifecycle Toolkit

## Explanation

Hooks are a fairly standard and well understood software mechanism. So rather than referring to tasks and evaluations - then attempting to explain the difference.

Instead we start referring to Keptn hooks. Keptn hooks are a way to interject into the lifecycle of a running Keptn process.

With this description, the explanation becomes easy. There are two possible Keptn hooks:

- Evaluations
- Tasks

Keptn hooks can be configured to fire at any of these times:

- Pre-deployment at a workload level
- Pre-deployment at an application level
- Pre-deployment at a workload level
- Post-deployment at a workload level

## Internal details

Note: This KEP does not mean changing anything about the current implementation: KeptnTasks and KeptnEvaluations remain as-is.

This is simply a way to all agree on how we message and refer to these capabilities.

Ergo, no additional coding is expected (apart from documentation rewording).

## Trade-offs and mitigations


## Breaking changes

None

## Prior art and alternatives


## Open questions

## Future possibilities
