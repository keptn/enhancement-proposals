# KEP 93: Run Tasks on Evaluation Success or Failure

**State: DRAFTING**

## Motivation
The Keptn Lifecycle Toolkit allows to define tasks which are executed before or after a deployment. Currently, it is not possible to define tasks, which are based on the evaluation result. This is a limitation for users, as it is currently not possible to react on the evaluation result.

## Current Design
Currently, the Lifecycle Toolkit knows following phases which are executed sequentially:

* Application Pre-Deployment-Tasks
* Application Pre-Deployment-Evaluations
* Application Deployment
  * Workload Pre-Deployment-Tasks
  * Workload Pre-Deployment-Evaluations
  * Workload Deployment
  * Workload Post-Deployment-Tasks
  * Workload Post-Deployment-Evaluations
* Post-Deployment-Tasks
* Post-Deployment-Evaluations

Tasks and Evaluations are marked as successful when the respective task or evaluation is executed successfully. If a task or evaluation fails, the whole phase is marked as failed. The phases are executed sequentially, which means that the next phase is only executed if the previous phase was successful. The tasks in a phase are executed in parallel.

## Assumptions / Definitions
* Additional Tasks and Evaluations can be defined on per Application or per Workload basis


## Proposed Design
Whether there are some ways to extend the current design, this KEP will focus on an approach where the user can define on-success and on-failure tasks for KeptnTaskDefinitions and KeptnEvaluations. Therefore, a KeptnTaskDefinition might look as follows afterward:

```yaml
apiVersion: lifecycle.keptn.sh/v1alpha2
kind: KeptnTaskDefinition
metadata:
  name: promote-staging
spec:
  function:
    httpRef:
      url: https://raw.githubusercontent.com/keptn-contrib/klt-tasks/main/trigger-github-action/function.ts
    parameters:
      map:
        nextStage: staging
        username: thschue
        repository: easy-promotion-example
        job: promotion.yaml
        ref: main
    secureParameters:
      secret: github
  on-success: <KeptnTaskDefinition>
  on-failure: <KeptnTaskDefinition>
```

This would allow the user to define a task which is executed when the task was successful. The same applies for the KeptnEvaluationDefinition:

```yaml
apiVersion: lifecycle.keptn.sh/v1alpha2
kind: KeptnEvaluationDefinition
metadata:
  name: app-pre-deploy-eval-1
  namespace: podtato-kubectl
spec:
  objectives:
    - name: available-cpus ## this query should fail
      query: "sum(kube_pod_container_resource_limits{resource='cpu'}) - sum(kube_node_status_capacity{resource='cpu'})"
      evaluationTarget: ">4"
  on-success: <KeptnTaskDefinition>
  on-failure: <KeptnTaskDefinition>
```

This design allows the user to define tasks which are executed when the evaluation was successful or failed. The tasks are executed in parallel to the evaluation. The tasks are executed in the same namespace as the evaluation or the TaskDefinitions.

## Alternative Design
It would be possible to define a new phase after the respective evaluation or task phases. This would lead to the situation in which the author of functions has to deal with the phases and functions would not be as generic as they are now. As an example:

**As a developer, I want to get a notification when the evaluation after a deployment was successful.**

With the proposed design, the function developer would implement the function to send a notification without having to deal with the phases and define this as an on-success action as described above.

With the alternative design, the function developer would have to implement all possible cases (on-success, on-failure) and would also have to take care if the certain task should be executed for the corresponding workload. This would lead to a lot of complexity for the function developer.

## Things to consider
* This would lead to a situation where tasks can be chained. Therefore, it is important to define a maximum number of chained tasks, e.g. it's only possible to run one task after another task. This has to be defined and tracked somewhere.
* The "child" tasks should also be visible in the traces of the parent task. This would allow the user to see the whole execution flow.

## Benefits
* It's possible to define tasks which are executed when the evaluation was successful or failed
* Therefore, it's possible to react on the evaluation result and e.g. trigger a notification or promotion/rollback steps

