# The next generation of Shipyard

The next generation of Shipyard to allow custom delivery/remediation processes.

## Motivation

*There's always room for smart enhancements* - the Shipyard specification is no exception.

A shipyard allows a **Site Reliability Engineer (SRE)** to specify WHAT should happen in the course of an artifact delivery or remediation process. Due to certain use cases, it has been identified that the current specification of the shipyard has limitations and hides details from the SRE. This KEP proposes an enhancement of the *Shipyard* specification to bring flexibility into the definition of the *WHAT*, but still provides an opinionated approach and smart defaults.

Please read this KEP in combination with the KEP: [First version of a Keptn uniform](https://github.com/keptn/enhancement-proposals/pull/7); especially regarding the eventing part.

## Explanation

:man:/:blonde_woman: The target persona of this KEP is an *SRE*, who is responsible for processes and workflows to automate application delivery or remediation operations. 

Before getting started, a brief recap of the important terms and definitions used in this KEP:

- **Project stage:** A project stage (or just stage) defines a logical space in an environment, which has a dedicated purpose for an application in a continuous delivery process. The project stages are ordered, meaning that a stage has at least a previous and/or next stage. 

- **Shipyard:** A shipyard is the declarative means to divide an environment into project stages and to specify workflows for each stage.

- **Workflow:** A workflow declares a set of tasks, which are triggered by an external event (aka. domain event, e.g.: artifact built → deploy artifact, a problem occurred → execute remediation, etc.). A workflow executes the tasks in the given order. 

- **Task:** A task is the smallest executable unit of a workflow. A task is kicked-off by a *triggered* event and has a life cycle of a *start* and *finished* event. 

*Current situation and Problem(s):* The current shipyard specification allows dividing an environment into project stages. In each stage, two ways for deploying (direct, blue/green) and testing (functional/performance) an artifact are provided. Besides, an SRE can specify whether a project stage is capable of handling problems by remediation actions. This specification has some limitations: 

- Certain tasks are defined implicitly without giving an SRE the possibility to remove/move the task, e.g.:

  - The evaluation always happens after a test execution even though the SRE already knows that no meaningful metric values are available (e.g., functional tests don't provide performance data). 

  - The promotion of a deployed/released artifact happens automatically with no possibility to intervene.

- It is not possible to add a task to a workflow, e.g., a task that takes care of pulling approval from a person in charge. 

- The trigger (domain event) of a workflow can not be set. 

*Solution:* A revision of the shipyard specification v0.1.2 addresses the above-mentioned problems and gives SREs more flexibility in declaring processes. 

## Internal details

### Specification 

The KEP proposes a new version for the shipyard specification as explained below. To get started, the following example of a shipyard with one stage is provided: 

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
* **version**: The version of the shipyard specification. 
* **kind**: Is set to `Shipyard`.
* **metadata:** Contains at least the property *name*, which declares a unique name for the shipyard.
* **spec:** Consists of the property *stages*.

*Definition of Stage:*
* **stages:** An array of stages and each stage consists of the properties *name* and *workflows*.
  * **name:** A unique name of the stage.
  * **workflows:** An array of workflows declared by name, listen, and tasks. 

*Definition of Workflow:*
* **name:** A unique name of the workflow
* **listen** (`optional`): An array of events that trigger the workflow.
* **tasks:** An array of tasks executed by the workflow in the declared order.

*Definition of Task:*
* Reserved key tasks are: **update**, **deployment**, **test**, **evaluate**, **release**, **rollback**
* *labels* (`optional`): Task properties as individual `key:value` pairs. These labels precise the task and are consumed by the unit (Keptn-service) that executes the task. 
 
### Functionality

1. Workflow events: The start and end of a workflow execution is indicated by the mandatory `[stage.name].[workflow.name].started` and a `[stage.name].[workflow.name].finished` event. For the above example, the following events occur: 
    * `hardening.artifact-delivery.started` 
    * `hardening.artifact-delivery.finished` 

1. Workflow triggers: The array *listen* contains all *domain events*(*) and workflow events with state finished to start this workflow, but this array is optional. If it is not set, a workflow can be started with an event of type: `[workflow.name].triggered`
    * (*) Domain event: *An event that occurred in the business process, written in past tense; [see](https://en.wikipedia.org/wiki/Event_storming)*. This is fired by a human or tool to inform about a certain situation. For example, a `problem.open` event is fired by a monitoring tool when a service runs in a problem mode.

1. Task events: For each task, a `[task].triggered` event is sent by Keptn's control plane. Those Keptn-services that have a subscription on this event, will react with a `[task].started` event, perform their functions, and finally confirm their execution with a `[task].finished`. E.g., for the *test* task, the following events occurre:
    * `test.triggered`
      * `test.started`
      * `test.finished`

1. Task properties: The additional task properties (e.g., task `test` specifies the property: `kind:functional`) is added to the `[task].triggered` event.

1. Data flow: A task can add any data to the event payload (reserved space, see below). The Keptn's control plane takes care that the data is not lost and delivered to the next task. As more tasks are added to a workflow, as more data can be added to the event stream (= all events occurring in a workflow).

### Event selector

A workflow has a subscription to certain events that trigger the workflow. However, an SRE not only wants to configure the event type, but also a selector on this event. This selector configures more precisely when the workflow should be triggered. For example, an SRE wants to configure that a rollback workflow should be triggered just when deployment or release failed. Therefore, this KEP proposes the approach of an event selector using  `matchLabels` and `matchExpressions`. 

While the `matchLabels` allows configuring a selector on a `key` with exactly one expected `value`, the `matchExpressions` allows configuring a selector on a `key` with multiple value options. For example, the below rollback workflow has a subscription to two events. (1) For the first `hardening.deployment.finished` event a selector is configured based on the label `status:failed`. This means that the rollback workflow gets triggered when the event has the label `status:failed` attached. (2) For the second `hardening.release.finished` event a selector is configured for the key `status`, which can have either the value `warning` or `failed`.

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

### Shipyard controller

This KEP also proposes the implementation of a Keptn core component - the so-called *shipyard controller*. This controller implements the functionality described above. Besides, this controller is responsible for keeping workflows atomic. This means that a workflow is executed based on the shipyard, which was configured when the workflow was triggered. In other words, an update of a shipyard (and its containing workflows) does NOT impact ongoing workflows.

### Keptn CloudEvents

Based on the propsed concepts, changes on the Keptn CloudEvents are necessary.

**(1) Reserved space in CloudEvent payload:**

Before the shipyard controller sends out a `[task].triggered` event for a specific task, the controller adds a reserved space to the event payload. This reserved space is named after the task and can be consumed by the unit (Keptn-service) that executes the task. For example, a *deployment* task gets the reserved space of *deployment* in the payload (i.e., data field) of the CloudEvent. Per default, this reserved space is empty. Expect for tasks that have properties defined in the shipyard (e.g., `strategy: blue_green`) - for those tasks the properties are added to the reserved space:
  
```
{
  "type": "sh.keptn.event.deployment.triggered",
  ...
  "data": {
    "deployment": {
      "strategy": "blue_green"
    }
  }
}
```

*Implementation details:* 
* Keptn-services can add any information to the reserved space. For example:
  ```
  {
    "type": "sh.keptn.event.deployment.triggered",
    ...
    "data": {
      "deployment": {
        "strategy": "blue_green",
        "deploymentURI":"https://my-service.domain.com/" // Property added by Keptn-service
      }
    }
  }
  ```

* The event selectors for Keptn-services, as proposed in [First version of a Keptn uniform](https://github.com/keptn/enhancement-proposals/pull/7), work based on the information provided in this reserved space. For example:
  ```yaml
    - name: deployment-service
      image: keptn/helm-service:0.6.0
      events:
        - deployment.triggered:
          selector:
            matchExpressions:
              - {key: strategy, operator: In, values: [direct, blue_green]}
  ```

* The shipyard controller merges the reserved spaces of a `[task-xyz].triggered`, `[task-xyz].started` and `[task-xyz].finished` event. If, for example, the `deployment.triggered` event contains `strategy:blue_green` and the `deployment.finished` event contains `deploymentURI:xyz` in the reserved space, the shipyard controller merges the the reserved space, before sending the next `[test].triggered`: 
  ```
  {
    "type": "sh.keptn.event.task-abc.triggered",
    ...
    "data": {
      "deployment": {
        "strategy": "blue_green",
        "deploymentURI":"https://my-service.domain.com/" // Property added by Keptn-service
      }
      "test": {

      }
    }
  }
  ```



## Breaking changes

This KEP breaks the implementation of Keptn 0.6.0: 
* Keptn-services (aka. batteries) including contributed services need to be changed to react on a `[task].triggered`, and send a `[task].started` → `[task].finished`.
* Keptn CloudEvents must be changed.
* A shipyard migration must take place as outlined below.

### Upgrade path from Shipyard 0.1.2  to 0.2.0

The shipyard specification version 0.1.2 as used by Keptn 0.5.0 and 0.6.0 (last supported Keptn versions) has to be migrated to the new specification. Thus, the following spec: 

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
metadata:
  name: shipyard
spec:
  stages:
    - name: "dev"
      workflows:
        - name: artifact-delivery: 
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
            - hardening.artifact-delivery.finished:
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
            - production.problem.open            #(smart default)
            - production.remediation.in-progress #(smart default)
          tasks:
            - remediation:
            - test:
                kind: real_user
            - evaluation:
```

## Open questions

- Do we need a CloudEvent specification for workflow events? 

## Future possibilities

* A future change this proposal enables is a fix of the Keptn CloudEvent specification. Currently, there are different ways of formulating the CloudEvent `type` property: `sh.keptn.events.evaluation-done` vs `sh.keptn.internal.event.get-sli.done`. 
