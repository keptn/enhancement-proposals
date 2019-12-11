# Behavioral change of JMeter

Behavioral change of the testing service in Keptn (jmeter-service) to not send an evaluation-done event for failed tests, but rather a test-finished event with a test result set to *fail*.

## Motivation

In the current implementation of the jmeter-service, the service sends a `sh.keptn.events.evaluation-done` event in case a test execution failed. In the data block of this event the property `result` is set to `fail`. However, sending this event type creates confusion because the jmeter-service did not execute an actual evaluation based on SLO or SLI configurations like the lighthouse-service does. Besides, the corresponding `sh.keptn.events.tests-finished` event is missing, and no status of the test execution is provided.

## Explanation

Instead of sending a `sh.keptn.events.evaluation-done` event for a failed test execution, this KEP proposes to send a `sh.keptn.events.tests-finished` event with the property `result`, which is set to `fail`. The following example shows the structure for this particular case:

```json
{
 "data": {
    "project": "sockshop",
    "stage": "staging",
    "service": "carts",
    "testStrategy": "performance",
    "deploymentStrategy": "direct",
    "start": "2019-09-01 12:00:00",
    "end": "2019-09-01 12:05:00",
    "result": "fail"
  },
  "type": "sh.keptn.events.tests-finished",
  "specversion": "0.2",
  "source": "https://github.com/keptn/keptn/jmeter-service",
  "id": "f2b878d3-03c0-4e8f-bc3f-454bc1b3d79d",
  "time": "2019-06-07T07:02:15.64489Z",
  "contenttype": "application/json",
  "shkeptncontext": "08735340-6f9e-4b32-97ff-3b6c292bc509"
}
```

## Internal details

* *How the change would impact and interact with existing functionality?*: Since the jmeter-service does not send the `sh.keptn.events.evaluation-done`, the lighthouse-service is responsible for this step.

## Trade-offs and mitigations

There is no known trade-off. 

## Breaking changes

There is no breaking change since this KEP just adds a property to an event, but does not remove or rename a property. Also the `sh.keptn.events.evaluation-done` event is sent throughout the workflow meaning that services, which listen to it, still receive it.

## Open questions

The semantic of the `result` property in the `sh.keptn.events.tests-finished` event may lead to misunderstanding because this property can be used to not only inform about the result of the test execution, but rather about the status of the service execution itself. For example, an implementation of a testing service could also use this property to inform about a service call that could not be executed due to an internal error (test file not valid/found). However, to address this open question a new KEP is required that proposes a way of informing Keptn's control plane about a failed service execution.

## Future possibilities

To optimize the evaluation workflow after a test execution, the lighthouse-service could receive the test result. If the result is `fail`, the lighthouse-service could skips the process of querying the SLI-provider since no monitoring data from the test would be available. 
