# Add/remove Stages to/from Project

**Success Criteria**: Keptn supports up/down-scaling the number of stages of a project.

## Short abstract
_In Keptn, a project stage (or just stage) defines a logical space, which has a dedicated purpose for an application in a continuous delivery process. Typically, a project has multiple stages that are ordered depending on the maturity level of the application. For instance, a project can consist of a `deployment`, `hardening`, and `production` stage whereas the maturity level grows from left to right. While a `deployment` stage is used for feature development and initial testing, an application in `production` is bullet-proofed and services production load._

_Due to modern Cloud-native development approaches, a static set of stages is not sufficient anymore. For instance, it should be possible to spin up a stage for chaos-testing an application and to remove it after the test was conducted. Another use case may focus on rolling out a multi/hybrid-Cloud strategy, which requires the need to add new stages for additional execution planes that are running on another Cloud provider._

_Currently, it is not possible to add a new stage after the initial project setup. This KEP addresses this missing capability of adding/removing a stage on demand._ 

## Why
### Target audience / Pain points / Related discussions

*Pain points:*
* After creating a project Keptn, it is not possible to add or remove a stage. The only workaround is re-creating the project with an updated shipyard. 

*Use-case drivers:*
* https://github.com/keptn/keptn/issues/4693 - Add stages after initial project creation: This allows adopting the project due to changed requirements in the software (application) development lifecycle. 
* It should be possible to spin up a temporary test stage, e.g. for functional/unit tests, as part of continuous integration (CI). This would shift Keptn closer to the developer and establish it already as an integral part of CI.  

*Related discussions:*
* n/a

## What

### What it is?

* As mentioned above, stages are ordered from left to right creating a child-parent relationship. The following example shows a setup with four stages. While the stages `dev`, `hardening`, and `production` are in sequential order, the stage `production-remote` is parallel to `production` and has the `hardening` stage as _parent_.
![image](https://user-images.githubusercontent.com/729071/152763695-1362fa90-dd41-4f33-93d6-f35954b1ab81.png)

* Currently, the linking of stages to create a child-parent relationship is based on sequences and their `triggeredOn` configuration. Based on the above example, a shipyard configures the relationship from `hardening` to `dev` as follows: 
  ```
  apiVersion: "spec.keptn.sh/0.2.0"
  kind: "Shipyard"
  metadata:
    name: "shipyard-sockshop"
  spec:
    stages:
      - name: "dev"
        sequences:
          - name: "delivery"
            tasks:
              - name: "deployment"
  
      - name: "hardening"
        sequences:
          - name: "delivery"
            triggeredOn:
              - event: "dev.delivery.finished"
        [...]
  ```
  To configure a *child-parent-relationship* between stages, both stages need to have the same sequence (`delivery`). Besides, the child stage has to have a `triggeredOn` set on the event of the parent stage (`dev.delivery.finished`). 

#### ⭐ Use Case: As a user, I can add a new stage to a project.

This use case comes in two ways: 
1. Add a new stage to extend the stage chain. For example, introduce a new stage `chaos` in-between `dev` and `hardening` for a dedicated testing purpose. > Extend stage chain
2. Add a new stage `quality-assurance` that is parallel to another one. > Clone stage

![image](https://user-images.githubusercontent.com/729071/152803028-6d930aa6-8a72-40a3-b931-4fc356fc0e70.png)

API: **POST** `/project/{project}/stage/{stage}` 

_Extend stage chain:_
To extend the stage chain with `chaos` after `dev`, execute:

```
keptn create stage chaos --project sockshop --follows dev
```

❓ Or should it be: _To extend the deliver line with `chaos` before `hardening`, ..._

_Clone stage:_
To clone `hardening` into `quality-assurance`, execute:  
```
keptn create stage quality-assurance --project sockshop --clone hardening
```

#### ⭐ Use Case: As a user, I can remove a stage from a project.

API: **DELETE** `/project/{project}/stage/{stage}` 

To delete a stage from a project, execute:  

```
keptn delete stage quality-assurance --project sockshop
```

When deleting a stage, the children of the stage need to be updated if they have set a `triggeredOn` to the stage. Given, for example, the following three stages: `dev`, `hardening`, and `production` that are linked. 
```
apiVersion: "spec.keptn.sh/0.2.0"
kind: "Shipyard"
metadata:
  name: "shipyard-sockshop"
spec:
  stages:
    - name: "dev"
      sequences:
        - name: "delivery"
          tasks:
            - name: "deployment"

    - name: "hardening"
      sequences:
        - name: "delivery"
          triggeredOn:
            - event: "dev.delivery.finished"
      [...]

    - name: "production"
      sequences:
        - name: "delivery"
          triggeredOn:
            - event: "hardening.delivery.finished"
      [...]
```
* If the `dev` stage gets deleted, the _delivery_ sequence of `hardening` has no trigger event anymore.
* If the `hardening` stage gets deleted, the _delivery_ sequence of `production` needs to listen on the `dev.delivery.finished` event.
* If the `production` stage gets deleted, no update is needed since there is no child sequence. 

### What it is not?

* to-be-defined

## Open Discussions

* to-be-listed