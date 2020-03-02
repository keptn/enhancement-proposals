# Definition of Remediation Action

## Motivation



## Explanation

:man:/:blonde_woman: The target persona of this KEP is a **Dev**, who is responsible for developing a service of an application and defines a remediation action that may be built-in.   

*Current situation and Problem(s):* Currently, a **Developer** can specify a remediation action for his/her service by adding a *remediation action* configuration in version 0.1.2 to Keptn using the `keptn add-resource command`. In most cases, a remediation action is part of a new artifact version and available when the new version is built. Thus, this action should be shipped along with the new artifact. Next to the config handling, the current spec does not allow specifying the action-provider responsible for executing the action.

*Solution:* This KEP proposes a behavioral change in configuring a remediation action for a service. Besides, it proposes a spec change that introduces an action *type* and *provider*.

## Internal details

### Specification

This KEP proposes a change of the *remediation action* configuration. To get started, the following example of a *remediation action* is provided: 

```yaml
---
version: 0.2.0
kind: Remediation
metadata:
  name: remedation-service-abc
spec:
  problems: 
  - problem: Response time degradation
    actions:
      - action: togglefeature
        description: Toggle feature flag EnablePromotion from ON to OFF
        type: keptn-service
        provider: unleash
        values: 
          - EnablePromotion: off
```

*Meta-data:*
* **version**: The version of the remedation specification. 
* **kind**: Is set to `Remediation`.
* **metadata:** Contains at least the property *name*, which declares a unique name for the remedation configuration.
* **spec:** Consists of the property `problems`.

*Definition of Problems:*
* **problems:** An array of problems for which remediation actions are available. A *problem* consists of the *problem* property and *actions* array:
  * **problem:** A unique identifier of a problem. 
  * **actions:** An array of *actions* that are executed in the given order.
    * **action**: A unique name of the remedation action. 
    * **description**: A short description of the action.
    * **type:** The type of the action: **keptn-service**, **webhook**.
    * **provider:** If type **keptn-service** is selected, the calling keptn-service is specified by the provider property.
    * **values:** An array of key:value pairs. 

### Action types

**keptn-service**:

**webhook**:

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


## Trade-offs and mitigations


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
        values: 
          - value: +1
  - problem: Failure rate increase
    actions:
      - action: featuretoggle
        description: Please provide a description for the remediation action.
        type: keptn-service
        provider: unleash
        values: 
          - value: EnablePromotion:off
```

## Prior art and alternatives


## Open questions


## Future possibilities

