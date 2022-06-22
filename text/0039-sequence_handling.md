# Advanced Sequence Handling in Control-plane

**Success Criteria:** Improved UX for Keptn control-plane (for delivery/remediation orchestration)

## Motivation

* *Pain*: A Keptn user triggers sequences/tasks without having a way to stop/pause them afterwards.
* *Target*: Keptn provides basic (but “must-have”) capabilities to control sequence/task execution by the user and by smart timeouts.
* *Driver*: Other delivery/remediation solutions allow stopping/pausing/queuing pipelines. To stay competitive, this basic functionality must be supported.

## User Stories:

#### As a user, I can trigger multiple sequences while Keptn queues the sequences to avoid parallel executions in the same stage.

*Example:* 
```
09:01:30 - keptn trigger delivery --project=sockshop --service=carts --image=docker.io/keptnexamples/carts --tag=0.11.1

09:02:15 - keptn trigger delivery --project=sockshop --service=carts --image=docker.io/keptnexamples/carts --tag=0.11.2
```

*Behavior:* 
* The delivery of `carts:0.11.1` kicks off for the first stage, e.g.: dev.
* The delivery of `carts:0.11.2` is waiting as long as the delivery of `carts:0.11.1` in dev is not finished. 

Bridge:
![image](https://user-images.githubusercontent.com/729071/114506113-a0cc3480-9c31-11eb-857e-118ca8717f7e.png)


#### As a user, I would expect that Keptn terminates a triggered task when no service has reacted after X minutes. 

* If no service reacted after X (e.g., 3) minutes, the task is terminated by the Shipyard-controller. This results in a failed sequence.
* The Bridge explains why the task failed: 
![image](https://user-images.githubusercontent.com/729071/114415061-a41edc00-9baf-11eb-945e-6418ac525dea.png)

#### As a user, I can pause/resume/abort a triggered sequence via the CLI / Bridge.

*CLI:*
```
keptn trigger delivery --project=sockshop --service=carts --image=docker.io/keptnexamples/carts --tag=0.11.1
> Starting to deliver the service carts in project sockshop in version docker.io/keptnexamples/carts:0.11.1
> ID of Keptn context: c217b93c-5d35-45a1-9d83-e5ad9366362a

keptn pause sequence --keptn-context=c217b93c-5d35-45a1-9d83-e5ad9366362a
> The current sequence execution is paused. 
> To resume the sequence execution, execute: keptn resume sequence --keptn-context=c217b93c-5d35-45a1-9d83-e5ad9366362a

keptn resume sequence --keptn-context=c217b93c-5d35-45a1-9d83-e5ad9366362a
> The sequence execution resumes. 

keptn abort sequence --keptn-context=c217b93c-5d35-45a1-9d83-e5ad9366362a
> The sequence execution is aborted. 
```

*Bridge - Option 1:*
![image](https://user-images.githubusercontent.com/729071/114419170-7fc4fe80-9bb3-11eb-836b-7a7f445cb76e.png)

*Bridge - Option 2:*
![image](https://user-images.githubusercontent.com/729071/114507376-5350c700-9c33-11eb-98a7-167619c02c30.png)

*Expected behavior:*
- When clicking the `pause` button  / When executing `keptn pause sequence ...`: 
  - The current (last triggered) task will finish, but the next task in the sequence will not be triggered. 
  - The Bridge shows the `resume` button.
  - The Bridge displays the status of the sequence: *delivery paused*
- When clicking the `resume` button / When executing `keptn resume sequence ...`:
  - The next task in the current sequence will be triggered and the sequence execution continues. 
  - The Bridge displays the status of the sequence: *delivery resumed* 
- When clicking the `abort` button / When executing `keptn abort sequence ...`:
  - The current (last triggered) task will finish, but the next task will not be triggered. Instead, the sequence ends with an aborted status.
  - The Bridge displays the status of the sequence: *delivery aborted* 