# Uniform support and Subscription Management

**Mockup:** https://github.com/keptn/keptn/issues/1280

**Success Criteria:** 

*Note:* The term *Keptn-service* is used as a synonym to a Keptn integration that is maintained in the https://github.com/keptn-contrib organization. A *Keptn-service*  has a subscription to an event and performs a task based on the event type and payload. 

## Motivation

* *Pain*: Currently, it is difficult for a Keptn user to maintain custom Keptn-services since there is no UI support that shows the services and where they are running. Besides, the event subscriptions are not displayed leaving doubt about the execution of a sequence. The only way to check this is by listing the services that are running (on the control- and execution plane) and investigating their configured subscriptions. ​

* *Target*: Keptn supports the first implementation of the Uniform approach by visualizing deployed custom Keptn-services (on the control plane, or on an execution plane) and their subscriptions.​

* *Driver*: To foster the usage of custom integrations in Keptn, there must be UI/UX improvements in handling those integrations. 

## User Stories: 

Preparation steps: 

1. I deploy the custom Keptn-service on a control- or execution plane (in the namespace `keptn-uniform`) using the provided Helm Chart:

```
helm install jenkins-service https://github.com/keptn-contrib/jenkins/releases/download/0.8.0/jenkins-service-0.8.0.tgz -n keptn-uniform --create-namespace --set="distributor.projectFilter=sockshop,distributor.subscription=sh.keptn.event.deployment.triggered"
```

2. I open the Bridge and go to the Uniform screen of my project. 

#### (1) As a user, I want to see the `jenkins-service` as part of the Uniform for this project: 

![image](https://user-images.githubusercontent.com/729071/114532533-36c18880-9c4d-11eb-8561-28ca173b5080.png)

The information displayed for this service shows:
* Name of the service: `jenkins-service`
* Release version: `0.8.1` 
* Name of K8s cluster where the service has been deployed on: `gke_research_us-central1-c_prod-customer-A`
* Name of K8s namespace where the service has been deployed in: `keptn-uniform` 
* Location: ❓ 
* Status: (1) Is my service currently available in the sense that it can process an event, and (2) did a former event processing cause an error? split `healthy` into: `available` and `errored`
* List of active subscriptions: (short name of `sh.keptn.event.deployment.triggered`) -> `deployment.triggered`
* Implemented event spec version: This is not displayed but a requirement for user story (2).
* List of supported event types: This is not displayed but a requirement for user story (3), (5). 

*Details:*
* When deploying a custom Keptn-service, it automatically registers itself at the control plane and provides the above meta-information. 
* Then, a custom Keptn-service updates its status on:
  * (1) a regular basis - (TBD: To not overload the Keptn API this update can happen 2 times a day)
  * (2) after an event execution 
  * (3) after an update of the subscriptions

#### (2) As a user, I want to get notified when the deployed Keptn-service does not work with the current Keptn installation.

Next to the displayed meta-information about the Keptn-service, it should provide the info about the implemented event spec, i.e., the current Keptn event spec version: `0.2.1`. 

The next mockup shows three cases: 
* The implemented event spec is supported by Keptn ✔️ 
* No information about the event spec version available ❌ 
* The implemented event spec is not supported by this Keptn ❌ 

![image](https://user-images.githubusercontent.com/729071/116052148-a4bc7580-a679-11eb-8b70-457d0490f903.png)

#### (3) As a user, I want to add multiple subscriptions to my custom Keptn-service by using the Bridge and then applying the change via `helm upgrade` on the cluster. 

A subscription has: 
* `name`: a name of the subscription for display purposes
* `event`: the event the subscription works on (e.g.: `sh.keptn.event.deployment.triggered`, `sh.keptn.event.release.triggered`)
* `filters`: on the project, stage, and service level.

![image](https://user-images.githubusercontent.com/729071/116056478-21515300-a67e-11eb-8c08-a9b903483f99.png)

*User flow:*
- The button `Add subscription` opens an empty form to set `name` and to specify `event` and `filters`. 
- The drop-down list for the `event` is reduced to the event types supported by the Keptn-service. Consequently, this is additional meta-information that must be provided by the service. 
- When editing the new subscription, the below text field is updated. This text field contains the `helm upgrade` command to update that service. 

*Questions:*
- How to maintain a list of subscriptions? This can't be implemented by the environment variables of the `distributor` but rather by a separate configmap ❓ 
  - By default, a distributor can support just one subscription by its environment variables.
- How does the `helm upgrade` command look like ❓ 

#### (4) As a user, I want to add parameters to my subscription to enrich it with context information. 

* A subscription supports a list of parameters; a parameter is a `key:value` pair. 

*Questions:*
- The parameters won't be added by the shipyard-controller to the `.triggered` event because this is custom information and Keptn core should not expose information of other subscriptions. Hence, can the distributor of a Keptn-service take care of adding the parameters to the event payload before forwarding it to the receiving service? 

![image](https://user-images.githubusercontent.com/729071/116056690-59f12c80-a67e-11eb-8ed9-5e89a1a9e528.png)

❓ How does the `helm upgrade` command look like? 

#### (5) As a user, I can validate whether my shipyard (for this project) is covered by the uniform. If a missing subscription is found, I can add it with two clicks and one `helm upgrade`

*Details / User flow:*
- Since the Keptn core knows the registered Keptn-services and the shipyard for this project, an API endpoint can derive and return the coverage of tasks (events) by subscriptions. 
- (1) The Bridge displays the shipyard and highlights tasks that don't have a subscription. 
- The Bridge displays a drop-down containing all services that support this particular task (event). The Keptn core has to offer an endpoint, to get **all** registered services by their supported event type.  
- (2) - (3) By clicking the `Add subscription` button, an additional subscription - with the event type and stage pre-defined - is created. 
- (4) Finally, the user has to apply the changes (by a `helm upgrade`) to make the subscription working. 

![image](https://user-images.githubusercontent.com/729071/116061684-717ee400-a683-11eb-8b27-241067755a21.png)
