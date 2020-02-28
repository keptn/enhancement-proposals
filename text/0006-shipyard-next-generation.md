# The next generation of Shipyard

*Short (one sentence) summary*

## Motivation

*There's always room for smart enhancements* - the Shipyard specification is no exception.

A shipyard allows a Site Reliability Engineer (SRE) to specify WHAT should happen in the course of an artifact delivery or remediation process. Due to certain use cases, it has been identified that the current specification of the shipyard has limitations and hides details from the SRE. This KEP proposes an enhancement of the *Shipyard* to bring flexibility into the definition of the *WHAT*, but still provides an opinionated approach.

Please read this KEP in combination with the KEP: [First version of a Keptn uniform](); especially regarding the eventing part.

## Explanation

:man:/:blonde_woman: The target persona of this KEP is an *SRE*, who is responsible for processes and workflows applied to automate application delivery or remediation operations. 

Before getting started, a brief reminder of important terms and definitions:

- **Project stage:** A project stage (or just stage) defines a logical space, which has a dedicated purpose for an application in a continuous delivery process. The project stages are ordered.

- **Shipyard:** A shipyard is the declarative means to divide an environment into project stages and to specify workflows for each stage.

- **Workflow:** A workflow declares a set of tasks, which are triggered by an external event (domain event, e.g.: artifact built → deploy artifact, problem occurred → execute remediation, etc.), and executes tasks in the given order. 

- **Task:** A task is the smallest executable unit of a workflow. A task is triggered by an "triggered" event and has a life cycle of a *start* and *finished* event. 

*Current situation and Problem(s):* The current shipyard specification allows dividing an environment into project stages. In each stage, two ways for deploying (direct, blue/green) and testing (functional/performance) an artifact are provided. Besides, an SRE can specify whether a project stage is capable of handling problems by remediation actions. Based on this specification, various issues occur: 

- Certain tasks are defined implicitly without giving an SRE the possibility to remove/move the task, e.g.:

  - The evaluation always happens after a test execution even though we already know that no meaningful metric values are available (e.g., functional tests don't provide performance data) 

  - The promotion of a deployed/released artifact happens automatically with no possibility to intervene.

- It is not possible to add a task to a workflow. 

- The trigger (domain event) of a workflow can not be set. 

*Solution:* A revision of the shipyard specification 0.1.2 addresses the above-mentioned problems and gives SREs the flexibility they are looking for. 

## Internal details

### Specification 

The KEP proposes a new version for the shipyard specification as explained below. To start the explanation, the following example of a shipyard with two stages is provided: 

```yaml
---
version: 0.2.0
kind: Shipyard
metadata:
  name: shipyard-abc
spec:
  stages:
    - name: "hardening"
      workflows:
        - name: artifact-delivery
          listen:   
            - dev.artifact-delivery.finished
          tasks:
            - update:
            - deployment:
                strategy: blue_green
            - test:
                kind: functional
            - evaluation: 
            - test:
                kind: performance
            - evaluation:
            - release: 
```

*Meta-data:*
* **version**: The version of the uniform specification. 
* **kind**: Is set to `Shipyard`.
* **metadata:** Contains at least the property *name*, which declares a unique name for the shipyard.
* **spec:** Consists of the property *stages*.

*Definition of Stage:*
* **stages:** An array of stages and each stage consists of the properties *name* and *workflows*.
* **name:** A unique name of the stage.
* **workflows:** An array of workflows declared by name, listen, and tasks. 

*Definition of Workflow:*
* **name:** A unique name of the workflow
* **listen:** (optional) An array of events that trigger the workflow.
* **tasks:** An array of tasks executed by the workflow.

*Definition of Task:*
* Reserved key tasks are: **update**, **deployment**, **test**, **evaluate**, **release**, **rollback**
* (+) (optional) task properties as individual key:value pairs. 
 
### Functionality

1. The start and end of a workflow execution is communicated by a `[stage.name].[workflow.name].started` and a `[stage.name].[workflow.name].finished` event.

1. Workflow triggers: The array *listen* lists all *domain events*(*) and `[stage.name].[workflow.name].finished` events that kick off the workflow, but this array is optional. If it is not set, a workflow can be started with an event of type: [workflow.name].triggered
    * (*) Domain event: This is produced by a human or tool to inform about a certain situation. For example, a `problem.open` event is fired by a monitoring tool when a service runs in a problem mode.

1. For each task, a `[task].triggered` event is sent by Keptn's control plane. Those Keptn-services that have a subscription on this event, will react with a `[task].started` event, perform their functions, and finally confirm their executionn with a `[task].finished`. E.g., for the *test* task, the following events are fired:
    * test.triggered
      * test.started
      * test.finished

1. The additional task properties (e.g., task `deployment` specifies the property: `strategy: blue_green`) is added to the `[task].triggered` event.

1. A task can add any information to the event payload. The Keptn's control plane takes care that this information is not lost and delivered to the next task. (As more task are added to a workflow, as more data can be added to the event stream (= all events occuring in a workflow).)

### Event selector

A workflow has a subscription to certain events that trigger the workflow. However, a SRE not only wants to configure the event type, but also a selector on this event that states more precisely when the workflow should be triggered. For example, a SRE wants to configure that a rollback workflow should be triggered just when a deployment or release failed. Therefore, this KEP proposes the approach of an event selector using  `matchLabels` and `matchExpressions`. 

While the `matchLabels` allows configuring a selector on a `key` with exactly one expected `value`, the `matchExpressions` allows configuring a selector on a `key` with multiple value options. For example, the below rollback workflow has a subscription on two events. (1) For the first event `hardening.deployment.finished` a selector is configured based on the label `status: failed`. This means that the rollback workflow gets triggered when the event has the label `status: failed` attached. (2) For the second event `hardening.release.finished` a selector is configured for the key `status`, which can have either the value `warning` or `failed`.

```yaml
rollback:
  listen:
  - hardening.deployment.finished:
      selector:
        matchLabels:
          status: failed
  - hardening.release.finished:
      selector:
        matchExpressions:
          - {key: status, operator: In, values: [warning, failed]}
  tasks:
  - rollback:
```

## Trade-offs and mitigations

TBD

## Breaking changes

There is no 

### Upgrade path from Shipyard 0.1.2  to 0.2.0

The shipyard specification version 0.1.2 as supported by Keptn 0.5.0 and 0.6.0 (last supported versions) has to be migrated to the new specification. The following example: 

```yaml
stages:
  - name: "dev"
    deployment_strategy: "direct"
    test_strategy: "functional"
  - name: "staging"
    deployment_strategy: "blue_green_service"
    test_strategy: "performance"
  - name: "production"
    deployment_strategy: "blue_green_service"
    remediation_strategy: "automated"
```

... migrates to:

```yaml
---
version: 0.2.0
kind: Shipyard
spec:
  stages:
    - name: "dev"
      workflows:
        artifact-delivery: 
          listen:   # from
            - ### config changed
          tasks:
          - update:
          - deployment:
              strategy: direct
          - test:
              kind: functional
          - evaluation: 
          - release: 

    - name: "hardening"
      workflows:
        artifact-delivery: 
          listen:
          - dev.artifact-delivery.finished
          tasks:
          - update:
          - deployment:
              strategy: blue_green
          - test:
              kind: performance
          - evaluation:
          - release:
        
        rollback:
          listen:
          - hardening.release.finished:
              selector:
                matchLabels:
                  status: failed
          tasks:
          - rollback:
    
    - name: "production"
      workflows:
        artifact-delivery: 
          listen:
          - hardening.artifact-delivery.finished
          tasks:
          - update:
          - deployment:
              strategy: blue_green
          - release:
      
        remediation:  ## Subject to change
          listen: 
          - production.problem.open #(smart default)
          - production.remediation.in-progress #(smart default)
          tasks:
          - approval:
              kind: manual
          - remediation:
          - validation:
              kind: real_user
```

## Prior art and alternatives

N/A

## Open questions

What are some questions that you know aren't resolved yet by the KEP? These may be questions that could be answered through further discussion, implementation experiments, or anything else that the future may bring.


- How to handle rollback
- How to add remediation workflow

## Future possibilities

What are some future changes that this proposal would enable?
