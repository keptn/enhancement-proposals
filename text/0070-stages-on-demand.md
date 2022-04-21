# Add/remove Stages to/from Project

**Success Criteria**: Keptn supports adding and removing stages for existing projects.

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

*Related discussions / PR*
* PoC showing required changes in Keptn core: https://github.com/keptn/keptn/pull/6410

## What

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

### What it is?

#### ⭐ Use Case: As a user, I can add a new stage to a project.

This use case comes in two ways: 
1. Add a new stage to extend the stage chain. For example, introduce a new stage `chaos` in-between `dev` and `hardening` for a dedicated testing purpose.
2. Add a new stage `quality-assurance` that is parallel to another one.

_Outcome:_
![image](https://user-images.githubusercontent.com/729071/154692956-020cb941-c5d6-4192-91e0-e59312eaf4cc.png)

Both ways of adding a stage are supported by adjusting the shipyard of that project accordingly: 

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

    - name: "chaos"
      sequences:
        - name: "delivery"
          triggeredOn:
            - event: "dev.delivery.finished"
          tasks:
            - name: "deployment"

    - name: "hardening"
      sequences:
        - name: "delivery"
          triggeredOn:
            - event: "chaos.delivery.finished"
            
   - name: "quality-assurance"
      sequences:
        - name: "delivery"
          triggeredOn:
            - event: "chaos.delivery.finished"
      [...]
```

* After updating the shipyard, a `keptn upgrade project` or API call on PUT `/project/{sockshop}` adds the new stages:
```
keptn update project sockshop --shipyard=new_shipyard.yaml
```
![image](https://user-images.githubusercontent.com/729071/154693162-98973533-835c-4ea5-9a4c-ff282f4daf9c.png)


#### ⭐ Use Case: As a user, I can remove a stage from a project.


* To remove a stage from a project, the user has to modify the shipyard of that project accordingly: 
* After updating the shipyard, a `keptn upgrade project` or API call on PUT `/project/{sockshop}` removes the stages

```
keptn update project sockshop --shipyard=new_shipyard.yaml
```
![image](https://user-images.githubusercontent.com/729071/154693162-98973533-835c-4ea5-9a4c-ff282f4daf9c.png)

#### What it is? Summary:

* Implementation of the PUT `/project/` endpoint allowing a user to add/remove a stage by updating the project.
* CLI / Bridge leverage this endpoint to add/remove stages:
  * `keptn update project sockshop --shipyard=new_shipyard.yaml`
  * In the Bridge, it should be possible to update the shipyard in the *Settings* > *Project* page:
  ![image](https://user-images.githubusercontent.com/729071/154693886-25f8f49f-0fc2-4673-9fc7-c739d55ad39e.png)

### What it is not?

* Implementation of the API endpoints for the *stage* entity:
  * API: **POST** `/project/{project}/stage/{stage}` > Creates a stage for theproject.
  * API: **DELETE** `/project/{project}/stage/{stage}` > Deletes stage from project and updates the shipyard accordingly.

## Open Discussions

* to-be-listed