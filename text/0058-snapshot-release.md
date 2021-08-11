# Snapshot releases

## Problem statement

**Problem 1:** Currently, Keptn is mainly designed for the deployment and promotion of - more or less independent - microservices without dependencies. This behavior might become a challenge when multiple services are handled and should be tested together. Using the current approach, each service has to be promoted individually and therefore, each stage might run with version combinations that are not tested together.

**Problem 2:** Keptn assumes that each sequence is running in the scope of a service. As a result, it is not possible to define tasks that run once per project/stage which leads to the problem that every task that is taken by a sequence has to be designed in a way that it does not interfere with other ones. If there would be such - *more global* - scoped tasks, it would be possible to, e.g. create infrastructure per project but also run global tests, which might be evaluated on a per-service basis.

## Success Criteria

In the future, it should be possible to deploy larger-scale applications/platforms using Keptn. Therefore, Keptn should be able to deal with a set of services, but also to take tasks on project/stage scope.

As a result, the following success criteria are defined:

* A set of services can be defined in Keptn
* Services can be added and removed to/from the set
* The current set of services in a stage can be promoted in one step to the next stage
* Tasks can be taken on the scope of the set instead of services

## Explanation

### General overview

![PlatformDeployment (1)](https://user-images.githubusercontent.com/729071/125741167-99e6d196-280d-4664-ad71-d3768d229789.png)
Image source: <>

### (proposed) Entity model

![image](https://user-images.githubusercontent.com/729071/126744703-41489b40-3e44-478b-8965-3d37d7f1d086.png)
Image source: 

* Snapshot - A set of Keptn services
* Service - A Keptn service
* Service Sequence Instance (Service-SI) - Execution of a sequence (e.g., delivery) for one service
* Snapshot Sequence Instance (Snapshot-SI) - Execution of a sequence (e.g., delivery) for an entire snapshot
* `service version` - value from CI build
* `snapshot version` - defined by Keptn

## Use Cases

Overview:
1. As a platform operator, I want to define a set of services to test and deploy them together in further stages
1. As a platform operator, I want to deploy all services together to ensure that they are operating together properly
1. As a developer, I want to ensure that my service is tested to ensure that it is running properly on the platform
1. As a platform operator, I want to run tests involving all of the services to ensure that the platform runs properly
1. As a developer/platform operator, I want to know which service causes problems related to the tests to be able to find out where problems are
1. As a platform operator, I’d like to promote the whole group to the next stage to ensure that the preceding tests are still valid

#### As a platform operator, I want to define a set of services to test and deploy them together in further stages

It should be possible to add a service to a group to treat them as one in the future sequences. This grouping could be done at a project level, therefore a *promotion strategy* (grouped or single-service) would have to be added to the Shipyard file. If this is set, all services in the project are part of the group. 

A **snapshot** represents a configuration of a group for a specific point in time. When operating in a grouped mode, it would be possible to run sequences on a *snapshot* scope.

#### As a platform operator, I want to deploy all services together to ensure that they are operating together properly

it should be possible to deploy all services (snapshot) in a stage. Obviously, this doesn’t apply to the first stage, where the services will be added to the platform. The deployment could be done on a project-level (deploy everything with one artifact) or on a per-service level.

- *Project-Level Deployment*: In an operator-managed deployment, it might be intended that all services are deployed by applying one custom resource to the target environment while having the possibility to evaluate every services quality criteria on its own. In this case, the operator would have to pass the artifact for the project-deployment as well as the configuration needed for the services to Keptn.
- *Service-Level Deployment*: This is the current way of dealing with deployments. But instead of deploying every service on its own, Keptn would iterate over all services in the project and deploy them in the previously specified version.

#### As a developer, I want to ensure that my service is tested to ensure that it is running properly on the platform

After the deployment, tests scoped to the service will be executed. 

#### As a platform operator, I want to run tests involving all of the services to ensure that the platform runs properly

There might be tests, which are scoped to the snapshot (end-to-end tests). These tests should be executed once and might span across multiple services.

#### As a developer/platform operator, I want to know which service causes problems related to the tests to be able to find out where problems are

After all, the results of the tests done before should be done on a per-service basis. Therefore, it should make no difference if the tests are triggered on a per-service or on a per-snapshot basis.

#### As a platform operator, I would like to promote the whole group (snapshot) to the next stage to ensure that the preceding tests are still valid

Last but not least, the platform operator should be able to promote the service - either automatically or manually - if all of the tests ran successfully. Furthermore, the operator should be able to promote a specific set of previously successfully deployed and tested versions to the next stage

## Implementation

### Prerequisites

* Version is mandatory in Keptn-events
  * Should be a human-readable name
  * Provided by the CI
  * Could be the Git-commit (from a technical point of view)

### Overview

![Untitled-2021-07-22-1555](https://user-images.githubusercontent.com/729071/126651325-c4ed6616-4bc9-43c6-a898-d4e48ce844cb.png)

### Possible Shipyard

**Option A**

```
apiVersion: "spec.keptn.sh/0.2.2"
kind: "Shipyard"
metadata:
  name: "shipyard-sockshop"
spec:
  promotionStrategy: [snapshot | service ]  < default is `service`
  stages:
    - name: "development"
      sequences:
        - name: "delivery"
          scope: service
          tasks:
            - name: "deployment"
            - name: "test"
            - name: "evaluation"
            - name: "release"

    - name: "hardening"
      sequences:
      
        - name: "snapshot-delivery"
          tasks:
            - name: "monaco"
            - ref: "delivery"
            - name: "test"
              properties:
                teststrategy: "e2e-platform"
            - name: "evaluation"                         < uses SLO from stage-level

        - name: "delivery"
          scope: service
          tasks:
            - name: "deployment"
            - name: "test"
            - name: "evaluation"
            - name: "release"
```

![Untitled (1)](https://user-images.githubusercontent.com/729071/126660584-04e2d636-3205-4d48-996d-69641171b1d6.png)

```
title This is a title

DEVELOPER->dev.delivery:sh.keptn.event.dev.delivery.triggered (service A, v1) 
note left of snapshot: {\n"snapshotVersion": 1,\n"stages": [dev],\n"services": [\n{"service A": "shkeptncontext"}\n]\n}
dev.delivery->dev.delivery: .delivery.finished

DEVELOPER->dev.delivery:sh.keptn.event.dev.delivery.triggered (service B, v1) 
note left of snapshot: {\n"snapshotVersion": 2,\n"stages": [dev],\n"services": [\n{"service A": "shkeptncontext"},\n{"service B": "shkeptncontext"}\n]\n}
dev.delivery->dev.delivery: .delivery.finished 


DEVELOPER->dev.delivery:sh.keptn.event.dev.delivery.triggered (service C, v1)
note left of snapshot: {\n"snapshotVersion": 3,\n"stages": [dev],\n"services": [\n{"service A": "shkeptncontext"},\n{"service B": "shkeptncontext"},\n{"service C": "shkeptncontext"}\n]\n}
dev.delivery->dev.delivery: .delivery.finished 

DEVELOPER->dev.delivery:sh.keptn.event.dev.delivery.triggered (service A, v2) 
note left of snapshot: {\n"snapshotVersion": 4,\n"stages": [dev],\n"services": [\n{"service A": "shkeptncontext"},\n{"service B": "shkeptncontext"},\n{"service C": "shkeptncontext"}\n]\n}
dev.delivery->dev.delivery: .delivery.finished

DEVELOPER->hardening.snapshot-delivery:sh.keptn.event.hardening.snapshot-delivery.triggered [snapshotVersion:3]
note left of snapshot: {\n"snapshotVersion": 3,\n"stages": [dev, hardening],\n"services": [\n{"service A": "shkeptncontext"},\n{"service B": "shkeptncontext"},\n{"service C": "shkeptncontext"}\n]\n}

hardening.snapshot-delivery->hardening.snapshot-delivery: .monaco.triggered

hardening.snapshot-delivery->hardening.delivery: hardening.delivery.triggered parallel for: [service A] & [service B] & [service C]

hardening.delivery->hardening.delivery: deployment.triggerd
hardening.delivery->hardening.delivery: test.triggerd
hardening.delivery->hardening.delivery: evaluation.triggerd
hardening.delivery->hardening.delivery: release.triggerd

hardening.snapshot-delivery<-hardening.delivery: hardening.delivery.finished for: [service A] & [service B] & [service C]


hardening.snapshot-delivery->hardening.snapshot-delivery: .test.triggered

hardening.snapshot-delivery->hardening.snapshot-delivery: .evalution.triggered

hardening.snapshot-delivery->hardening.snapshot-delivery:sh.keptn.event.hardening.snapshot-delivery.finished

```

**Option B**

```
apiVersion: "spec.keptn.sh/0.2.2"
kind: "Shipyard"
metadata:
  name: "shipyard-sockshop"
spec:
  promotionStrategy: [snapshot | service ]  < default is `service`
  stages:
    - name: "development"
      sequences:
        - name: "delivery"
          scope: service
          tasks:
            - name: "deployment"
            - name: "test"
            - name: "evaluation"
            - name: "release"

    - name: "hardening"
      sequences:
      
        - name: "delivery"             < to trigger: sh.keptn.event.hardening.delivery.triggered {snapshotVersion: 3} (see below)
          tasks:
            - name: "monaco"
              scope: snapshot
            - name: "deployment"       < triggers a deplyoment for each service (context) as provided in the snapshotVersion
            - name: "test"             < triggers a test -||- 
            - name: "evaluation"       < triggers a evaluation -||- 
            - name: "test"
              scope: snapshot
              properties:
                teststrategy: "e2e-platform"
            - name: "evaluation"                   < uses SLO from stage-level
              scope: snapshot

```
![Untitled (2)](https://user-images.githubusercontent.com/729071/126660896-9f6f10c1-5ed5-40b7-9274-674c29ffb26e.png)

https://sequencediagram.org/
```
title This is a title

DEVELOPER->dev.delivery:sh.keptn.event.dev.delivery.triggered (service A, v1) 
note left of snapshot: {\n"snapshotVersion": 1,\n"stages": [dev],\n"services": [\n{"service A": "shkeptncontext"}\n]\n}
dev.delivery->dev.delivery: .delivery.finished

DEVELOPER->dev.delivery:sh.keptn.event.dev.delivery.triggered (service B, v1) 
note left of snapshot: {\n"snapshotVersion": 2,\n"stages": [dev],\n"services": [\n{"service A": "shkeptncontext"},\n{"service B": "shkeptncontext"}\n]\n}
dev.delivery->dev.delivery: .delivery.finished 

DEVELOPER->dev.delivery:sh.keptn.event.dev.delivery.triggered (service C, v1)
note left of snapshot: {\n"snapshotVersion": 3,\n"stages": [dev],\n"services": [\n{"service A": "shkeptncontext"},\n{"service B": "shkeptncontext"},\n{"service C": "shkeptncontext"}\n]\n}
dev.delivery->dev.delivery: .delivery.finished 

DEVELOPER->dev.delivery:sh.keptn.event.dev.delivery.triggered (service A, v2) 
note left of snapshot: {\n"snapshotVersion": 4,\n"stages": [dev],\n"services": [\n{"service A": "shkeptncontext"},\n{"service B": "shkeptncontext"},\n{"service C": "shkeptncontext"}\n]\n}
dev.delivery->dev.delivery: .delivery.finished

DEVELOPER->hardening.delivery: sh.keptn.event.hardening.delivery.triggered [snapshotVersion:3]
note left of snapshot: {\n"snapshotVersion": 3,\n"stages": [dev, hardening],\n"services": [\n{"service A": "shkeptncontext"},\n{"service B": "shkeptncontext"},\n{"service C": "shkeptncontext"}\n]\n}

hardening.delivery->hardening.delivery: .monaco.triggered

hardening.delivery->hardening.delivery: .deployment.triggered [serviceA: shkeptncontext]

hardening.delivery->hardening.delivery: .deployment.triggered [serviceB: shkeptncontext]

hardening.delivery->hardening.delivery: .deployment.triggered [serviceC: shkeptncontext]

hardening.delivery->hardening.delivery: .test.triggered [serviceA: shkeptncontext]

hardening.delivery->hardening.delivery: .test.triggered [serviceB: shkeptncontext]

hardening.delivery->hardening.delivery: .test.triggered [serviceC: shkeptncontext]

hardening.delivery->hardening.delivery: .evaluation.triggered [serviceA: shkeptncontext]

hardening.delivery->hardening.delivery: .evaluation.triggered [serviceB: shkeptncontext]

hardening.delivery->hardening.delivery: .evaluation.triggered [serviceC: shkeptncontext]

hardening.delivery->hardening.delivery: .test.triggered

hardening.delivery->hardening.delivery: .evalution.triggered

hardening.delivery->hardening.delivery: sh.keptn.event.hardening.delivery.finished
```

```
{
  "snapshotVersion": 3,
  "stages": [dev], 
  "services": [
    {"serviceA": "shkeptncontext"},
    {"serviceB": "shkeptncontext"},
    {"serviceC": "shkeptncontext"}
  ]
}
```

### Grouping of Services

As described above, the Shipyard file should contain the information if the promotion of the services should happen on a per-group (snapshot) or on a per-service level.

- ❔ What if a user wants to change the promotionStrategy in the future?
- ❔ Might it be an option that snapshot scoped sequences are ignored then?
- ❔ What if services are removed or added?

If a user decides to use promote on a per-snapshot level, the ability to promote everything to the next stage is only “open” when all services are successfully deployed and tests/evaluations are in a success/warning state.

#### Enabling this feature

* To enable this feature, set `promotionStrategy` in the Shipyard to `snapshot`.

#### Group/Project Context

If the project should be promoted on a per-snapshot level, every deployment triggered event creates a new version of the snapshot.

An example:

- A project consists of service A, service B, and service C, each in version 1.0
  - the snapshot version will be `3` (for each deployment, a snapshot was created. Hence, we are already at `3`)
- Service A should be upgraded and the version is set to 1.1, Service B and C remain at version 1
  - the snapshot version will be `4`
- Service C is updated to version 1.1, therefore service A in version 1.1, service B in version 1.0, and service c in version 1.1 
  - the snapshot version will be `5`

#### Storing the Information

As each version change will result in a new snapshot version, the version should be incremented when a new version of a service arrives at the Keptn API. Furthermore, the information about the currently deployed service context ids (in the current stage) should be stored in the database (“Snapshot”). Therefore a JSON object containing this information might look as follows (example):

```
{
  "snapshotVersion": 1,
  "stages": [development], 
  "services": [
    {"serviceA": "shkeptncontext"},
    {"serviceB": "shkeptncontext"},
    {"serviceC": "shkeptncontext"}
  ]
}
``` 
- ❔ Would it be more useful to store the KeptnContext (Service) instead of the Version?
  - Yes, since it references the version

To keep track, it might be useful to make the version number unique across all stages. Therefore it can be ensured that snapshot version 1 in a dev stage is the same one as in a hardening stage.

#### Maintaining the Group Context

Normally, each sequence in each stage could be triggered by a Keptn event. To ensure that this functionality does not break the snapshots (e.g. by sending a new version of a service directly to the next stage), deployment events from “the outside” might only be triggered in the first stage of the Shipyard, when the `promotionStrategy` is set to an on snapshot-level.

### Sequences

As described above, it might be necessary to have sequences that are scoped to a project/stage (snapshot) or a service level. While snapshot-scoped sequences are a new concept in Keptn, service-scoped sequences change their behavior when used with a snapshot-based promotion strategy.

Proposal: Whenever no scope is specified in a sequence, it is assumed that it is scoped to a service

#### service-scoped sequences

When running a service-scoped sequence, the specified tasks are running with every service in the snapshot instance scope. At first, the current project version is detected. Afterwards, Keptn iterates over all of the services in the current project version and triggers the events for them. In further consequence, the state of the current task is watched (shipyard controller?). After all services finished the execution of this task, an additional event ()

❔ Where do we find the current platform version in a stage?

### Wireframe

- In the Bridge, I can click on a **>** button that shows a list of available snapshots. (can be represented as time-line) 
- I select a snapshot (e.g., `snapshotVersion: 3`) and press deploy 