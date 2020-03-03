# Refinement of Spec for Remediation Action

The built-in remediation actions in Keptn 0.6.0 have to become customizable.

## Motivation

Automating operational tasks is a core use case of Keptn. Consequently, means are provided that allow a service switching into a remediation mode when a problem for this service is detected. In this remediation mode, Keptn takes care of executing specified remediation actions. After the execution of one action, Keptn validates the effect of the performed remediation action to verify whether the problem is resolved. If it is not resolved, the next action gets triggered until no action is available. Finally, a not-resolved problem should be escalated. In the implementation of Keptn 0.6.0, this workflow is not supported and it is not possible to use custom remediation actions. 

To address this gap, this KEP proposes a behavioral change in configuring a remediation action for a service and it proposes a spec change that introduces the property of an action *type*. Latter allows adding a custom remedation actions.

## Explanation

:man:/:blonde_woman: The target persona of this KEP is a **Developer**, who is responsible for developing a service of an application. The **Dev** also defines a remediation action that may be built-in the application or is executed by an external tool. Next to the Dev, a **DevOps** engineer is also affected by this KEP because he/she is responsible for the tool that executes the remediation action.   

*Current situation and Problem(s):* 

* Currently, a Dev can specify a remediation action for his/her service by adding a *remediation action* configuration to Keptn using the `keptn add-resource` command. In most cases, a remediation action is part of a new version of an artifact and available when the new version is built. -> Consequently, an action should be shipped along with the new artifact. 

* The current spec does not allow specifying an action-provider. An action-provider is the service that is responsible for executing the action. -> Consequently, the spec change has to provide the option to clarify which service can execute the action.

* Remediation actions are currently implemented in the remediation-service and a Dev can only select one of the implemented actions. This also goes along with the problem that it is not possible to use an individual action-provider. 

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
        type: keptn-service
        provider: unleash
        parameters: 
          - EnablePromotion: off
```

*Meta-data:*
* **version**: The version of the remediation specification. 
* **kind**: Is set to `Remediation`.
* **metadata:** Contains at least the property *name*, which declares a unique name for the remediation configuration.
* **spec:** Consists of the array `problems`. This array lists all problems for which a remediation action is provided.

*Definition of a Problem:*
* **problem:** A unique identifier of the problem. 
* **actions:** An array of *actions* that are executed in the given order.
  * **action**: A unique name of the remedation action. 
  * **description**: A short description of the action.
  * **type:** The type of the action: **keptn-service**, **webhook**, (**container**).
  * **provider | hook:** If type *keptn-service* is selected, the service to call is specified by the **provider** property. If type *webhook* is selected, the endpoint to call is specified by the **hook** property.
  * **parameters:** An array of individual `key:value` pairs used for executing the action by the action-provider. 

### Action types

In this KEP, two action types are proposed:

**1. keptn-service**: An action of type *keptn-service* has a corresponding action-provider, which is deployed and managed by Keptn. This action-provider takes care of executing the remediation action.
  * Benefit: DevOps has control over the deployed action-providers. 
  * Disadvantage: The remediation action must be executable by the action-provider, otherwise a new version of the action-provider must be deployed. 

**2. webhook**: A action of type *webhook* has no on-side (Keptn-managed) action-provider, but rather calls an endpoint from an external service. This service takes care of executing the remediation action.
  * Benefit: Any service that can execute an action can be triggered.
  * Disadvantage: DevOps has no control over the actions. 

### Functionality

A developer can specify a *remediation action* configuration for his/her service in two ways:

1. When onboarding a service:
    ```console
    keptn onboard service xyz --project=PROJECTNAME --service=SERVICENAME --chart=helm_chart.tgz
            --remediation-actions=action.yaml
    ```
    > This way provides the inital version of a *remediation action* configuration.

2. When sending a new-artifact event: 
    ```console
    keptn send event new-artifact --project=PROJECTNAME --service=SERVICENAME --image=docker.io/keptnexamples/carts --tag=0.9.1
            --remediation-actions=action.yaml
    ```
    > This way overrides the current *remediation action* configuration. For now, it is an override and no merge. In other words, a Developer can delete a remediation action by removing it from the config when sending a new artifact event.

### Refactoring

The current implementation of the remediation-service has built-in actions, like the toggling of a feature flag, or scaling a pod. The feature flag toggling, for example, need to be extracted into a service (unleash-service) that is then able to handle the action.

## Trade-offs and mitigations

N/A

## Breaking changes

This KEP breaks the implementation of Keptn 0.6.0:

* The feature of toggling a feature flag as implemented in the *remediation-service* (a Keptn battery) must be extracted into a separate service, the *unleash-service*.

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
        type: keptn-service
        provider: # TBD
        parameters: 
          - value: +1
  - problem: Failure rate increase
    actions:
      - action: featuretoggle
        description: Please provide a description for the remediation action.
        type: keptn-service
        provider: unleash
        parameters: 
          - EnablePromotion: off
```

## Prior art and alternatives

N/A

## Open questions

- The name of action type *keptn-service* is subject to change. What is a better naming?

## Future possibilities

- This KEP allows supporting multiple remediation actions for a service. 
