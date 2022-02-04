# Keptn Declarative Configuration

**Success Criteria**: A Keptn project can be configured by applying a set of custom resources (YAML files).

**Dependency:** The [KEP-67 Keptn <3 GitOps](https://github.com/keptn/enhancement-proposals/pull/67) depends on this KEP. Following illustration shows the current big picture KEP-67 and KEP-69:
![image](https://user-images.githubusercontent.com/38893055/152119636-e989fcea-8c3a-4d10-b216-fd3fb6adc875.png)

## Short abstract
_here comes a short abstract_

## Why
### Target audience pain points

*Pain Points:*
* The configuration of a Keptn project requires a lot of imperative commands, executed in the right order.
* First touch-points with keptn can get time-consuming, even when they are well documented ([Keptn Full Tour on Dynatrace](https://tutorials.keptn.sh/tutorials/keptn-full-tour-dynatrace-011/index.html?index=..%2F..index#0))

### Use-case drivers

* https://github.com/keptn/keptn/issues/4693 - Add stages after initial project creation
* https://github.com/keptn/keptn/issues/3817- Handling of Helm Chart and Configuration Versions
* https://github.com/keptn/enhancement-proposals/issues/63 - keptn apply

### Related Discussions

* [Scheduled Keptn Service Execution · Discussion #5146 · keptn/keptn · GitHub](https://github.com/keptn/keptn/discussions/5146)
* [First Time complexity and learning curve · Discussion #4754 · keptn/keptn · GitHub](https://github.com/keptn/keptn/discussions/4754)

## Prerequisites

* Currently, it is not possible to create a new stage without recreating a project. An approach for a fix of this project is already implemented in a prototype ([experimental: PoC of adding/removing stages via update of the shipyard file by bacherfl · Pull Request #6410 · keptn/keptn · GitHub](https://github.com/keptn/keptn/pull/6410)

## What

*Prototype*: A prototype to demonstrate the declarative approach has been created and is currently stored at [GitHub - keptn-sandbox/keptn-gitops-operator](https://github.com/keptn-sandbox/keptn-gitops-operator). The content of this KEP is based on this prototype.

### What it is?

* The Keptn architecture supports the configuration via Kubernetes custom resource definitions ([Custom Resources | Kubernetes](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/))
* When initializing a Keptn Project, a user can use a blueprint/template
* When recovering/migrating a Keptn installation, the configuration can be recreated using files

#### Use Case: As a user, I want to specify the desired state of a Keptn project and apply this in random order. 

*Assumptions:*
* It is assumed that the Keptn Control Plane is running in Kubernetes, and therefore Custom Resources exist
* Keptn can still be configured in an imperative way

*Kinds/Entities:* Following entities should exist as CRD (or exist in the current prototype):
* KeptnShipyard
* KeptnProject
* KeptnService
* KeptnStage
* KeptnSequence
* KeptnSequenceExecution

Additionally, other Keptn configurations as SLOs (slo.yaml), but also jobs for the job-executor could be implemented as custom resources.

#### Implementation

**Keptn Operator** - A set of controllers, which deal with Keptn entities and their state. It is an addition to Keptn, which watches the Keptn Custom Resources, triggers corresponding Keptn API requests, and ensures that the described state is depicted in the Keptn installation. This leads to the behavior that - for example - a Keptn Project gets created when a new KeptnProject custom resource is created, and the project gets deleted when the custom resource is deleted.

The shipyard is a combination of some entities (Project, Stage, SequenceTemplate). Therefore, the shipyard should be created when one of these entities changes and is pushed to the upstream repo. A thing that might be considered in the future might be that stages/sequences can be created directly (API) without the indirection via the shipyard.

In any way, the work is done by Keptn itself. The operators only orchestrate the API calls, which would be done by the keptn cli currently.

### What it is not?

* to-be-defined

### Open Discussions

* The prototype triggered keptn events via a SequenceExecution custom resource. In the case of deployments, it might be a more valid approach to store the version of the service in the custom resource and let the service-controller trigger the deployment. This might lead to problems when different versions in different stages should be deployed.