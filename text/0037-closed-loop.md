# Closed loop remediation with Shipyard v0.2

**Success Criteria:** A remediation process is modeled in the Shipyard of a project

## Motivation

* *Target*: As a user, I can model my custom remediation process in the Shipyard. The rem. process follows the closed-loop principle; meaning that it loops as long as the problem is not fixed and actions are available. 

* *Pain*: Currently, our opinionated closed-loop rem. process is hard-coded in the remediation-service. This limits a user in tailoring it. + It is not possible (!) to register action-providers to action.triggered events since they are not managed by the shipyard-controller.

* *Driver*: The use-case of automating remediations should work like the delivery (quality-gates) use-case.

## User Stories

* As a user, can define a remediation process (sequence) in the Shipyard file.
  - Depending on the number of actions defined in the remediation.yaml, this sequence gets triggered except for a passed evaluation (that considers closed problems)
* As a user (who is running an action-provider on the exec. plane), I can subscribe my action-provider to an `action.triggered` event. 
  - See issue: https://github.com/keptn/keptn/issues/3498
* *Changed behavior* As a user, I would like to define the mapping between problem and remedation action based on the root cause (spec change: add ProblemDetailsJSON) and not the problem title. (Consider updates of examples / tutorials)

## Remediation in Shipyard v0.2

```
- name: "production" 
  sequences: 
  - name: remediation
      triggeredOn: 
      - event: production.remediation.finished
          selector:
            match:
              evaluation.result: fail
      tasks:
    - name: get-action 
    - name: action
    - name: evaluation
      triggeredAfter: "10m"
      properties:
        timeframe: "5m"
```

## Open Question

- How can we stop the loop when, for example, the action returned `result=fail`?
