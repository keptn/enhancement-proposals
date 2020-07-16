# KEP65 - Keptn Declarative Configuration & GitOps

**Success Criteria:** A keptn project can be configured by either applying a set of custom resources (yaml files) or pointing keptn to a configuration repository.

## Motivation
*Pain Points:*
* The configuration of a keptn project requires a lot of imperative commands, executed in the right order.
* First touch-points with keptn can get time-consuming, even when they are well documented ([Keptn Full Tour on Dynatrace](https://tutorials.keptn.sh/tutorials/keptn-full-tour-dynatrace-011/index.html?index=..%2F..index#0))

*Targets:*
* The keptn architecture supports the configuration via Kubernetes custom resource definitions ([Custom Resources | Kubernetes](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)) and allows the use of developer provided code repositories which can be used by keptn
* A new keptn adopter can bootstrap a keptn project using a blueprint and evaluate the first quality gates 5 minutes after the installation

## Related Issues
* https://github.com/keptn/keptn/issues/4335 - Allow project initialisation via GitOps
* [Support for project/sequence templating · Issue #6334 · keptn/keptn · GitHub](https://github.com/keptn/keptn/issues/6334) - Support for project/sequence templating
* [Add stages after initial project creation · Issue #4693 · keptn/keptn · GitHub](https://github.com/keptn/keptn/issues/4693) - Add stages after initial project creation
* [Handling of Helm Chart and Configuration Versions  · Issue #3817 · keptn/keptn · GitHub](https://github.com/keptn/keptn/issues/3817) - Handling of Helm Chart and Configuration Versions
* [Allow importing a project from an existing Git upstream · Issue #3809 · keptn/keptn · GitHub](https://github.com/keptn/keptn/issues/3809) - Allow importing a project from an existing Git upstream
* [Allow forcing new project creation with existing Git upstream · Issue #3808 · keptn/keptn · GitHub](https://github.com/keptn/keptn/issues/3808) - Allow forcing new project creation with existing Git upstream

## Related Discussions
* [Operator based GitOps approach in Keptn · Discussion #5296 · keptn/keptn · GitHub](https://github.com/keptn/keptn/discussions/5296)
* [Scheduled Keptn Service Execution · Discussion #5146 · keptn/keptn · GitHub](https://github.com/keptn/keptn/discussions/5146)
* [First Time complexity and learning curve · Discussion #4754 · keptn/keptn · GitHub](https://github.com/keptn/keptn/discussions/4754)

## Preceding Work
A first prototype to demonstrate the declarative approach has been created and is currently stored at [GitHub - keptn-sandbox/keptn-gitops-operator](https://github.com/keptn-sandbox/keptn-gitops-operator). The content of this KEP is based on this prototype.

## Use-Case (1)-(2):
**(1 - Declarative Configuration of Keptn)** As a user, I want do specify the desired state of a keptn project and apply this in a random order

*Scenarios:*
* When initializing a Keptn Project,  a user can use a blueprint/template
* When recovering/migrating a Keptn Installation, the configuration can be recreated using files

**(2 - GitOps)** As a user, I want do store the desired state of a keptn project in git and expect keptn to apply the configuration which is stored there

*Scenarios:*
* When bootstrapping a new service/project, a user cans specify a Git Repository containing the configuration for this project

## Technical Details
### Assumptions
* It is assumed, that the Keptn Control Plane is running in Kubernetes and therefore Custom Resources exist
* Keptn can still be configured in an imperative way
* A user decides if he wants to configure keptn via a git repository or “manually” (imperative or declarative)

### Kinds/Entities
Following Entities should exist as CRD (or exist in the current prototype):
* GitRepository (GitOps Operator)
* Shipyard
* Project
* Service
* Stage
* SequenceTemplate
* SequenceExecution
* ScheduledSequenceExecution

### Prerequisites
* Currently, it is not possible to create a new stage without recreating a project. An approach for a fix of this project is already implemented in a prototype ([experimental: PoC of adding/removing stages via update of the shipyard file by bacherfl · Pull Request #6410 · keptn/keptn · GitHub](https://github.com/keptn/keptn/pull/6410)
* When dealing with artefacts (as helm-charts) in keptn / GitOps, it should be ensured that the deployment-combination of the container/helm-chart and corresponding keptn configurations are consistent (https://github.com/keptn/keptn/issues/3817). In the current approach, this is done via a separate service, which assumes that a service-version tag exists on the keptn upstream repository, combines this and creates the “stage-branch” based on this.

### Approach

#### Components
* **GitOps Operator** - Manages the Connection to the Developer Repository, applies Custom Resources and ships configurations/artefacts to the keptn upstream repository
* **Keptn Operator** - A set of controllers, which deal with Keptn Entities and their state.
* **Promotion Service** - Ensures that the right combination of Container/Configuration/Artefacts) is used during deployment (stage branch)

#### Declarative Keptn
In the current approach, the Keptn Operator is an addition to keptn which watches the Keptn Custom Resources, triggers corresponding Keptn API requests and ensures that the described state is depicted in the Keptn Installation. This leads to the behaviour that - for example - a Keptn Project gets created when a new KeptnProject custom resource is created, and the Project gets deleted when the custom resource is deleted.

The shipyard is a combination of some entities (Project, Stage, SequenceTemplate). Therefore, the shipyard should be created when one of these entities change and pushed to the upstream repo. A thing which might be considered in the future might be that stages/sequences can be created directly (API) without the indirection via the shipyard.

In any way, the work is done by Keptn itself, the operators only orchestrate the API calls which would be done by the keptn cli currently.

#### GitOps
When Keptn can be configured declaratively, the GitOps Operator only has to watch on a specified structure in a Git Repository and apply the configurations made there. Artefacts as Helm-Charts or test configurations can be pulled out of this repository and stored in the upstream repository (but it might also be an option to store them in an OCI-compliant registry). When storing them in the upstream repository, a tag containing the service-name and the version should be created. This tag is used by the promotion service as the first step of a sequence in a stage to create a snapshot of the correct configuration for the service in a stage.

### Open Discussions
* There might exist multiple approaches for the structure of a Git Repository
    * A git repository could be a service code-repo which contains a directory `keptn` where the configuration resides. In this case the project and service configuration might be in different repositories
    * There might be a project-wide repository which contains the configuration of a whole project
* The prototype triggered keptn events via a SequenceExecution custom resource. In the case of deployments, it might be a more valid approach to store the version of the service in the custom resource and let the service-controller trigger the deployment. This might lead to problems, when different versions in different stages should be deployed.


