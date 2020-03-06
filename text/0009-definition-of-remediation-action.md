# Refinement of Spec for Remediation Action

The built-in remediation actions in Keptn 0.6.0 have to become customizable.

## Motivation

Automating operational tasks is a **core use-case** of Keptn. Consequently, means are provided that allow a service switching into a remediation mode when a problem for this service is detected. In this remediation mode, Keptn takes care of executing specified remediation actions. After the execution of one action, Keptn validates the effect of the performed remediation action to verify whether the problem is resolved. If it is not resolved, the next action gets triggered until no action is available. Finally, a not-resolved problem should be escalated. In the implementation of Keptn 0.6.0, this workflow is partly supported and it is not possible to use custom remediation actions. 

To allow the mentioned use-case, this KEP proposes a behavioral change in configuring a remediation action for a service and it proposes a spec change that introduces the property of an action *type*. Latter allows adding a custom remedation action.

## Explanation

:man:/:blonde_woman: The target persona of this KEP is a **Developer**, who is responsible for developing a service of an application. The **Dev** also defines a remediation action that may be built-in the application or is executed by an external tool. Next to the Dev, a **DevOps** engineer is also affected by this KEP because he/she is responsible for the tool that executes the remediation action.   

*Current situation and Problem(s):* 

* Currently, a Dev can specify a remediation action for his/her service by adding a *remediation action* configuration to Keptn using the `keptn add-resource` command. In most cases, a remediation action is part of a new version of an artifact and available when the new version is built. -> Consequently, an action should be shipped along with the new artifact. 

* The current spec does not allow specifying an *action-provider*. An action-provider is a service that is responsible for executing the action. -> Consequently, the spec change has to provide the option to clarify which service can execute the action.

* Remediation actions are currently implemented in the remediation-service and a Dev can only select one of the implemented actions. This also goes along with the problem that it is not possible to use a custom action-provider. 

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
          provider: unleash
          values: 
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
  * **action**: A unique name of the remediation action. 
  * **description**: A short description of the action.
  * **provider | hook:** This property specifies the target unit that executes the remedation action. 
  * **values:** An array of individual `key:value` pairs used for executing the action by the action-provider. 

### Provider | Hook

In this KEP, three types of an action are proposed:

**Provider**: When a user declares a provider, the service that excutes the action is managed by Keptn and part of the Keptn's uniform.
  * Benefit: DevOps has control over the deployed providers. 
  * Disadvantage: The remediation action must be executable by the provider, otherwise a new version of the provider must be deployed. 

**Hook**: When a user declares a hook, then Keptn will call no on-side (Keptn-managed) action provider, but rather calls an endpoint from an external service. This service takes care of executing the remediation action.
  * Benefit: Any service that can execute an action can be triggered.
  * Disadvantage: DevOps has no control over the actions. 

### Functionality

A developer can specify a *remediation action* configuration for his/her service when sending a new-artifact event: 
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
          provider: # TBD
          values: 
            - value: +1
    - problem: Failure rate increase
      actions:
        - action: featuretoggle
          description: Please provide a description for the remediation action.
          provider: unleash
          values: 
            - EnablePromotion: off
```

## Prior art and alternatives

N/A

## Open questions

- How can we model a "rollback" action? Is this the type: *keptn-built-in* and a standard feature of the remedation-service?

## Future possibilities

- This KEP allows supporting multiple remediation actions for a service. 

- Instead of a provider or hook, a `container` (as executable unit) would be a suggestion for future enhancements. Thus, a user can specify a container image that will be launched by Keptn before executing the action. 
