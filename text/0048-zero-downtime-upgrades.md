# Zero-downtime Upgrades (ZDU)

**Success Criteria:** Keptn can be upgraded without downtime for the end-user.

## Motivation

* *Pain*: Upgrading a Keptn installation causes downtime of Keptn of approximately 2-3 minutes. 
* *Target*: The Keptn architecture supports upgrading Keptn without any downtime or data loss.
* *Driver*: To operate Keptn at scale, supporting hotfixes/upgrades/downgrades without end-user impact.

## Use Case: 

**Zero-downtime Upgrades: As a user, I want to be able to upgrade (or downgrade) my Keptn installation without losing access to the API and without losing CloudEvents / Tasks / Sequences (both, currently running sequences/tasks as well as new sequences/tasks).**

*Scenarios:*
* While upgrading the Keptn installation, a user triggers an evaluation
* While upgrading the Keptn installation, a user triggers a sequence with a task with `triggeredAfter: 15m` (essentially shipyard-controller needs to sleep during that time)  (*Note:* shipyard-controller already works stateless in this regard) 
* While upgrading a Keptn installation, a problem is triggered by Dynatrace and Keptn needs to perform a remediation action
* While upgrading a Keptn installation, an external execution-plane service tries to access a file from configuration-service (via api-gateway-nginx)

## Technical Details:

Keptn control-plane Services / Deployments / Pods are:
 
|   Services / Deployments / Pods 	|  estimate on Replicas >1 support |
|---	|---	|
|  `api-gateway-nginx`	|  easy 	|
|  `api-service`  	|  easy 	|
|  `approval-service`  	| easy  	|
|   `bridge`	|   easy	|
|  `remediation-service` 	|   easy	|
|  `secret-service` 	|  easy 	|
|  `keptn-nats-cluster` 	|  ? 	|
|   `statistics-service`	|  ?	|
|  `lighthouse-service` 	|  ? 	|
|  `mongodb` 	|  ?	|
|  `mongodb-datastore`	|  ? 	|
|   `configuration-service`	| difficult	|
|  `shipyard-controller` 	|   difficult |

* ~~`helm-service`~~
* ~~`jmeter-service`~~

#### Know limitations: 
The following technical limitations on zero-downtime upgrades are known or should be taken under consideration:
* Messaging bus (NATS) is not configured to allow multiple event recipients: *Research on NATS queue groups* 
* Can NATS support in any aforementioned scenarios/use-cases? *Research on NATS Streaming*  
* Keptn control-plane services don't implement graceful shutdown, but rather kill the running processes.
* Does the `lighthouse-service` maintain a state for the `get-sli` events it sends out?
* Which architectural changes are needed for `shipyard-controller` & `configuration-service`?

#### Resource footprint: 
While highly available systems come with an increased resource footprint, the resource usage should stay at the minimum: 
* Identity which Keptn control-plane Services / Deployments / Pods should run with replicas >1 and which not (e.g., the approval-service could run with replicas = 1)  

## Notes:
* `api-gateway-nginx` is a user-facing component. If it's down, the end-user loses access to Keptn. 
* NATS is a very central component. If it's down, events are lost. 
* `shipyard-controller` is our control-plane. If the `shipyard-controller` is down, sequences/tasks are not triggered. 
