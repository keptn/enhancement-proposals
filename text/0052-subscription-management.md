# Subscription management for Integrations

**Mockup:** https://github.com/keptn/keptn/issues/1280

**Success Criteria:** Easy integration of Keptn-services into a Shipyard process. 

*Note:* The term *Keptn-service* is used as a synonym to an integration that is maintained in the https://github.com/keptn-contrib organization. A *Keptn-service* (aka. Integration)  has a subscription to an event and performs a task based on the event type and payload. 

## Motivation

* *Pain*: After implementing [KEP45](https://github.com/keptn/enhancement-proposals/pull/45), an integration is listed in the Uniform screen, and subscriptions are displayed. To change the subscription, a user needs to manually adapt the manifest of the deployed Kubernetes services (of this integration). This is not possible when no access to the Kubernetes Cluster is given.

* *Target*: Allow customization of subscriptions to integrate custom Keptn-services (Integrations) into a process definition using the Bridge.

* *Driver*: To foster the usage of custom integrations in Keptn, there must be UI/UX improvements in handling subscriptions of integrations.  

## User Cases: 

### As a user, I want to add multiple subscriptions to my integration by using the Bridge and then manually applying the change on a remote execution plane. 

A subscription has: 
* `name`: Name of the subscription for display purposes
* `event`: Event the subscription works on (e.g.: `sh.keptn.event.deployment.triggered`, `sh.keptn.event.release.triggered`)
* `filters`: On the project, stage, and service level

![image](https://user-images.githubusercontent.com/729071/116056478-21515300-a67e-11eb-8c08-a9b903483f99.png)

*User flow:*
- The button `Add subscription` opens an empty form to set `name` and to specify `event` and `filters`. 
![image](https://user-images.githubusercontent.com/729071/122737979-dc165f80-d281-11eb-9b59-ca2db4262117.png)
- If it is a Keptn-service running on the remote execution plane, the below text field is updated when adding a new subscription or editing an existing one. This text field contains the `helm upgrade` command to update that service including its subscriptions. 
![image](https://user-images.githubusercontent.com/729071/122738183-0831e080-d282-11eb-97e0-ad1f76acc9d0.png)

*open Questions:*
- How to maintain a list of subscriptions? This can't be implemented by the environment variables of the `distributor` but rather by a separate ConfigMap.
  - By default, a distributor can support just one subscription by its environment variables.
- How does the `helm upgrade` command look like?

### As a user, I want to add multiple subscriptions to my integration by using the Bridge and then automatically apply the changes on the control-plane.

*User flow:*
* See the first steps above. 
* If service is running on the control-plane, an `Update subscriptions` button is provided that updates the subscriptions on the cluster when clicking it. 
![image](https://user-images.githubusercontent.com/729071/122738501-53e48a00-d282-11eb-94db-860ca7a66c7f.png)

 ### As a user, I want to add parameters to my subscription to enrich it with context information. 

* A subscription supports a list of parameters; a parameter is a `key:value` pair. 

*open Questions:*
- The parameters won't be added by the shipyard-controller to the `.triggered` event because this is custom information and Keptn core should not expose information of other subscriptions. Hence, can the distributor of the integration take care of adding the parameters to the event payload before forwarding it to the receiving service? 

![image](https://user-images.githubusercontent.com/729071/116056690-59f12c80-a67e-11eb-8ed9-5e89a1a9e528.png)

❓ How does the `helm upgrade` command look like? 

## supporting Use Cases

### As a user, I can validate whether my Shipyard (for this project) is covered by the uniform. If a missing subscription is found, I can add it with two clicks and one `helm upgrade` or click on `Update subscriptions` 

![image](https://user-images.githubusercontent.com/729071/122741316-20efc580-d285-11eb-87f7-07e6fd1ae367.png)

*User flow:*
- Since the Keptn core knows the registered integrations and the Shipyard for this project, an API endpoint can derive and return the coverage of tasks (events) by subscriptions. 
- (1) The Bridge displays the Shipyard and highlights tasks that don't have a subscription. 
- The Bridge displays a drop-down containing all services that support this particular task (event). The Keptn core has to offer an endpoint, to get **all** registered services by their supported event type.  
- (2) - (3) By clicking the `Add subscription` button, an additional subscription - with the event type and stage pre-defined - is created. 
- (4) Finally, the user has to apply the changes (by a `helm upgrade`) to make the subscription working. 

### As a user, I want to get notified when the deployed Keptn-service does not work with the current Keptn installation.

Next to the displayed meta-information about the Keptn-service, it should provide the info about the implemented event spec, i.e., the current Keptn event spec version: `0.2.1`. 

The next mockup shows three cases: 
* The implemented event spec is supported by Keptn ✔️ 
* No information about the event spec version available ❌ 
* The implemented event spec is not supported by this Keptn ❌ 

![image](https://user-images.githubusercontent.com/729071/116052148-a4bc7580-a679-11eb-8b70-457d0490f903.png)