# The next generation of Shipyard

The next generation of Shipyard to allow custom delivery/remediation processes.

## Motivation

*There's always room for smart enhancements* - the Shipyard specification is no exception.

The current specification of Keptn's [Shipyard](https://github.com/keptn/spec/blob/0.1.7/shipyard.md) has several limitations and hides details from end-users. This KEP provides an enhancement of the *Shipyard* format such that it allows end-users to specify *WHAT* should happen in the course of an artifact delivery or remediation process. While it brings flexibility into the definition of the *WHAT*, it is still an opinionated approach with smart defaults.

In addition, please consider [KEP 07: Propose the concept of a Uniform](https://github.com/keptn/enhancement-proposals/pull/7) as it specifies which components responds to certain events/tasks defined within the Shipyard.

## Explanation

:man:/:blonde_woman: For instance, one target persona of this KEP could be an SRE (Site Reliability Engineer), who is responsible for processes and workflows to automate application delivery or remediation operations. 

Before getting started, a brief recap of the important terms and definitions used in this KEP:

- **Project stage:** A project stage (or just stage) defines a logical space in an environment, which has a dedicated purpose for an application in a continuous delivery process. At least one stage must be defined (e.g., for a single stage project), but multiple stages are allowed (e.g., for implementing a staging process using a development, hardening, and production stage). If multiple stages are defined, by default they are ordered according to the shipyard file. However, it is possible to change the default order by defining triggers as mentioned below.

- **Shipyard:** A shipyard is the declarative means to divide an environment into project stages and to specify task sequences for each stage.

- **Sequence:** A sequence declares a set of tasks, which are triggered by an external event (aka. domain event, e.g.: artifact built → deploy artifact, a problem occurred → execute remediation, etc.). Within a sequence, tasks are executed in the given order.

- **Task:** A task is the smallest executable unit of a sequence. A task is kicked-off by a *triggered* event and has a life cycle of a *started*, *status.changed* (optional), and *finished* event. 

*Current situation and Problem(s):* The current shipyard specification allows dividing an environment into project stages. In each stage, two ways for deploying (direct, blue/green) and testing (functional/performance) an artifact are provided. Besides, an SRE can specify whether a project stage is capable of handling problems by remediation actions. This specification has some limitations: 

- Certain tasks are defined implicitly without giving an SRE the possibility to remove/move the task, e.g.:

  - The evaluation always happens after a test execution even though the SRE already knows that no meaningful metric values are available (e.g., functional tests don't provide performance data). 

  - The promotion of a deployed/released artifact happens automatically with no possibility to intervene.

- It is not possible to add a task to a sequence, e.g., a task that takes care of pulling approval from a person in charge. 

- The trigger (domain event) of a task sequence can not be set. 

*Solution:* By enhancing the Shipyard specificiation as detailed below (*target version is: v0.2.0*), the above-mentioned problems are addressed and more flexibility for declaring processes is provided.

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
  - name: hardening
    sequences:
    - name: artifact-delivery
      triggeredOn:   
      - event: dev.artifact-delivery.finished
      tasks:
      - name: deployment
        properties:
          strategy: blue_green
          traffic: 100
      - name: test
        properties:
          kind: functional
      - name: evaluation 
      - name: test
        properties:
          kind: performance
      - name: evaluation
      - name: release 
```

*Meta-data:*
* **version**: The version of the Shipyard specification, e.g., `spec.keptn.sh/0.2.0`
* **kind**: Is set to `Shipyard`.
* **metadata:** Contains at least the property *name*, which declares a name for the shipyard.
* **spec:** Consists of the property *stages*.

*Definition of Stage:*
* **stages:** An array of stages and each stage consists of the properties *name* and *sequences*.
  * **name:** A unique name of the stage.
  * **sequences:** An array of sequences declared by name, triggers, and tasks. 

*Definition of Sequence:*
* **name:** A unique name of the sequence
* **triggeredOn** (`optional`): An array of events that trigger the task sequence.
* **tasks:** An array of tasks executed by the sequence in the declared order.

*Definition of Task:*

* **name**: A unique name of the task
* **properties** *(optional)*: Task properties as individual `key:value` pairs. These properties precise the task and are consumed by the unit that executes the task.

**Note:** Reserved key tasks are: **approval**, **deployment**, **evaluate**, **release**, **remediation**, and **test**,.
 
### Functionality

1. Sequence events: The start and end of a sequence execution is indicated by the mandatory `[stage.name].[sequence.name].started` and a `[stage.name].[sequence.name].finished` event. For the above example, the following events occur: 
    * `hardening.artifact-delivery.started` 
    * `hardening.artifact-delivery.finished` 

1. TriggeredOn: The array *triggeredOn* contains all *domain events*(*) and sequence events with state finished to start this task sequence, but this array is optional. If it is not set, a sequence can be started with an event of type: `[stage.name].[sequence.name].triggered`. Note that only `<stage>.<task-sequence>.finished` events with `data.result` set to `pass` are able to trigger another task sequence.
    * (*) Domain event: *An event that occurred in the business process, written in past tense; [see](https://en.wikipedia.org/wiki/Event_storming)*. This is fired by a human or tool to inform about a certain situation. For example, a `problem.open` event is fired by a monitoring tool when a service runs in a problem mode.

1. Task events: For each task, a `[task].triggered` event is sent by the control plane. Those Keptn-services that have a subscription on this event, will react with a `[task].started` event, perform their functions, and finally confirm their execution with a `[task].finished`. E.g., for the *test* task, the following events occur:
    * `test.triggered`
      * `test.started`
      * `test.finished`

1. Task properties: The additional task properties (e.g., task `test` specifies the property: `kind:functional`) is added to the `[task].triggered` event.

1. Data flow: A task can add any data to the event payload (reserved space, see below). The Keptn's control plane takes care that the data is not lost and delivered to the next task. As more tasks are added to a sequence, as more data can be added to the event stream (= all events occurring in a task sequence).

### Event selector

A sequence has a subscription to certain events that trigger the sequence. However, an SRE not only wants to configure the event type, but also a selector on this event. This selector configures more precisely when the sequence should be triggered. For example, an SRE wants to configure that a rollback sequence should be triggered just when deployment or release failed. Therefore, this KEP proposes the approach of an event selector using the `match` property
For example, the below rollback sequence has a subscription to the event `hardening.deployment.finished`. Further, it specifies a selector that matches the value of the `result` field against the value `fail`.
Thus, the rollback sequence is only triggered iff the selector evaluates to true. Note that in a first step, only selectors which matches a value against the `result` property will be supported.
In a future enhancement additional selectors with more powerful match expressions shall be supported.

```yaml
rollback:
  triggers:
  - event: hardening.deployment.finished
    selector:
      match:
        result: fail
  tasks:
  - name: rollback
```

### Shipyard controller

This KEP also proposes the implementation of a Keptn core component - the so-called *shipyard controller*. This controller implements the functionality described above. Besides, this controller is responsible for keeping sequences atomic. This means that a sequence is executed based on the shipyard, which was configured when the sequence was triggered. In other words, an update of a shipyard (and its containing sequences) does NOT impact ongoing sequences.

### Keptn CloudEvents

Based on the proposed concepts, changes on the Keptn CloudEvents are necessary.

**(1) Reserved space in CloudEvent payload:**

Before the shipyard controller sends out a `[task].triggered` event for a specific task, the controller adds a reserved space to the event payload. This reserved space is named after the task and can be consumed by the unit (Keptn-service) that executes the task. For example, a *deployment* task gets the reserved space of *deployment* in the payload (i.e., data field) of the CloudEvent. Per default, this reserved space is empty. Except for tasks that have properties defined in the shipyard (e.g., `strategy: blue_green`) - for those tasks the properties are added to the reserved space:
  
```
{
  "type": "sh.keptn.event.deployment.triggered",
  ...
  "data": {
    "deployment": {
      "strategy": "blue_green",
      "traffic": "100",
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
        "traffic": "100",
        "deploymentURI":"https://my-service.domain.com/" // Property added by Keptn-service
      }
    }
  }
  ```

* The event selectors for Keptn-services, as proposed in [KEP 07: Propose the concept of a Uniform](https://github.com/keptn/enhancement-proposals/pull/7), work based on the information provided in this reserved space. For example:
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
        "traffic": "100",
        "deploymentURI":"https://my-service.domain.com/" // Property added by Keptn-service
      }
      "test": {

      }
    }
  }
  ```

## Breaking changes

This KEP breaks the implementation of Keptn 0.7.0: 
* All Keptn-services including contributed services (in GitHub organizations: [keptn-contrib](https://github.com/keptn-contrib), [keptn-sandbox](https://github.com/keptn-sandbox)) need to be changed to react on a `[task].triggered` event, and send a `[task].started` → `[task].finished`.
* Keptn CloudEvents must be changed.
* A shipyard migration must take place as outlined below.

### Upgrade path from Shipyard 0.1.x to 0.2.0

The shipyard specification version 0.1.x as used by Keptn versions 0.7.x and older has to be migrated to the new specification. Thus, the following spec: 

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
    sequences:
    - name: artifact-delivery
      tasks:
      - name: deployment
        properties:
          strategy: direct
      - name: test
        properties:
          kind: functional
      - name: evaluation 
      - name: release

  - name: "staging"
    sequences:
    - name: artifact-delivery
      triggeredOn:
      - event: dev.artifact-delivery.finished
      tasks:
      - name: deployment
        properties:
          strategy: blue_green
      - name: test
        properties:
          kind: performance
      - name: evaluation
      - name: release
        
    - name: rollback
      triggeredOn:
      - event: hardening.artifact-delivery.finished
        selector:
          match:
            result: fail
      tasks:
      - name: rollback
    
  - name: "production"
    sequences:
    - name: artifact-delivery 
      triggers:
      - hardening.artifact-delivery.finished
      tasks:
      - name: deployment
        properties:
          strategy: blue_green
      - name: release
      
    - name: remediation
      triggeredOn: 
      - event: production.problem.open            #(smart default)
      - event:  production.remediation.in-progress #(smart default)
      tasks:
      - name: remediation
      - name: test
        properties:
          kind: real_user
      - name: evaluation
```

## Ideas for enhancement

- A task can have a property, which defines if the task should be executed depending on previous tasks: 
  - `Only when all previous tasks have succeeded`
  - `Even if a previous task has failed`
  - `Only when a previous taks has failed`

## Open questions

- Do we need a CloudEvent specification for sequence events? 

## Future possibilities

* A future change this proposal enables is a fix of the Keptn CloudEvent specification. Currently, there are different ways of formulating the CloudEvent `type` property: `sh.keptn.events.evaluation-done` vs `sh.keptn.internal.event.get-sli.done`. 
