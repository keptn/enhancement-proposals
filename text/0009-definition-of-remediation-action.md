# Refinement of Spec for Remediation Action

The built-in remediation actions in Keptn 0.6.0 have to become customizable.

## Motivation

Automating operational tasks is a core use-case of Keptn. Consequently, means should be provided to
trigger a multi-step remediation workflow. In this remediation workflow, Keptn takes care of executing specified remediation actions. After the execution of one action, Keptn validates the effect of the performed remediation action to verify whether the problem is resolved. If it is not resolved, the next action gets triggered until no action is available. Finally, a not-resolved problem should be escalated. In the implementation of Keptn 0.6.0, this workflow is partly supported and it is not possible to use custom remediation actions. 

To allow the mentioned use-case, this KEP proposes a behavioral change in configuring a remediation action for a service and it proposes a spec change. Latter allows adding a custom remedation action.

## Explanation

:man:/:blonde_woman: The target persona of this KEP is a **Developer**, who is responsible for developing a service of an application. The **Dev** also defines a remediation action that may be built-in the application or is executed by an external tool. Next to the Dev, a **DevOps** engineer is also affected by this KEP because he/she is responsible for the tool that executes the remediation action.   

*Current situation and Problem(s):* 

* Remediation actions are currently implemented in the `remediation-service` and a Dev can only select one of the implemented actions. This also goes along with the problem that it is not possible to use a custom action-provider. (An action-provider is a service that is responsible for executing the action.)

* The `remediation-service` currently only allows to execute one action per problem. -> It should be possible to execute multiple actions as long as the problem is unresolved.

* In order to write the remediation file, the user has to know upfront which problems may occur. Even more difficult, he/she needs to know the exact name of the problem in order to formulate the remediation file. -> The matching from the problem to the remediation action should be more generic.

* The remediation action always requires a problem name. -> The user has no means to formulate a remediation action regardless of the problem, i.e., make a rollback for any problem.

## Internal details

### Specification

This KEP proposes a spec change of the *remediation action* configuration. To get started, the following example of a *remediation action* is provided: 

```yaml
---
version: 0.2.0
kind: Remediation
metadata:
  name: remediation-service-abc
spec:
  problems: 
    - problem: Response time degradation
      actions:
        - action: togglefeature
          description: Toggle feature flag EnablePromotion from ON to OFF
          values: 
            - EnablePromotion: off
    - problem: *
        actions:
          - action: rollback
            description: Roll my service back
            values:
              - Version: LastStable
```

*Meta-data:*
* **version**: The version of the remediation specification. 
* **kind**: Is set to `Remediation`.
* **metadata:** Contains at least the property *name*, which declares a unique name for the remediation configuration.
* **spec:** Consists of the array `problems`. This array lists all problems for which a remediation action is provided.

*Definition of a Problem:*
* **problem:** A unique identifier of the problem or a regular expression (RegEx) to capture different problem types.
* **actions:** An array of *actions* that are executed in the given order.
  * **action**: A unique name of the remediation action. This name will be used for sending out an event, to which an action-provider has subscripted to.
  * **description**: A short description of the action.
  * **hook:** This property is optional. A hook allows to specify a custom endpoint to send the problem event to. If the response of this hook has a response code between 200 and 300, the action execution is considered as successful. In all other cases, it is considered as failed.  
  * **values:** An array of individual `key:value` pairs used for executing the action by the action-provider. 

### Behaviour of Action providers and Hooks

**Default**: A provider gets triggered by an event and executes the remediation action.
More precisely, a provider listens to `sh.keptn.event.remediation.[action].triggered` events and when the event is received, it has to execute the remediation action. All providers are managed by the Keptn's uniform.
  * *Benefit*: DevOps have control over the deployed providers. 
  * *Disadvantage*: The remediation action has to register to Keptn events and has to send a `started` and `finished` event, which may be an overkill for simple, short-running remediation actions.

**Hook**: Remediation actions can be triggered using hooks. When a user declares a hook, Keptn calls this endpoint and waits for the HTTP response code. If the response code is between 200 and 300, the action is considered to be successful executed. Note that these hooks are not managed by Keptn.
  * *Benefit*: Any service that can execute an action can be triggered. Synchronous communication using HTTP POSTs.
  * *Disadvantage*: DevOps have no control over the webhooks. 

### Functionality

A developer can specify a *remediation action* configuration for his/her service when sending a new-artifact event: 
    ```console
    keptn send event new-artifact --project=PROJECTNAME --service=SERVICENAME --image=docker.io/keptnexamples/carts --tag=0.9.1
            --remediation-actions=action.yaml
    ```
    > This way overrides the current *remediation action* configuration. For now, it is an override and no merge. In other words, a Developer can delete a remediation action by removing it from the config when sending a new artifact event.

### Refactoring

The current implementation of the `remediation-service` has built-in actions, like the toggling of a feature flag, or scaling a pod. The feature flag toggling, for example, need to be extracted into a service (`unleash-service`) that is then able to handle the action.

## Trade-offs and mitigations

N/A

## Breaking changes

This KEP breaks the implementation of Keptn 0.6.0:

* The functionality of toggling a feature flag as implemented in the `remediation-service` (a Keptn battery) must be extracted into a separate service, the `unleash-service`.

* A migration of the *remediation action* configuration must be conducted.

### Upgrade path from Remediation Spec 0.1.2 to 0.2.0

The *remediation action* specification version 0.1.2 as used by Keptn 0.5.0 and 0.6.0 (last supported Keptn versions) has to be migrated to the new specification. Thus, the following spec:

```yaml
remediations:
- name: "Response time degradation"
  actions:
  - action: scaling
    value: +1
- name: "Failure rate increase"
  actions:
  - action: featuretoggle
    value: EnablePromotion:off
```

migrates to ...

```yaml
version: 0.2.0
kind: Remediation
metadata:
  name: remedation-service-abc
spec:
  problems: 
    - problem: Response time degradation
      actions:
        - action: scaling
          description: Please provide a description for the remediation action.
          values: 
            - value: +1
    - problem: Failure rate increase
      actions:
        - action: featuretoggle
          description: Please provide a description for the remediation action.
          values: 
            - EnablePromotion: off
```

## Prior art and alternatives

N/A

## Open questions

- How can we model a "rollback" action? Is this the type: *keptn-built-in* and a standard feature of the remedation-service?
- How can the user write remediation actions for unknown problems?

## Future possibilities

- This KEP allows supporting multiple remediation actions for a service. 

- Instead of a provider or hook, a `container` (as executable unit) would be a suggestion for future enhancements. Thus, a user can specify a container image that will be launched by Keptn before executing the action. 
