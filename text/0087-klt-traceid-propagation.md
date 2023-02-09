# KEP0087 - Lifecycle Toolkit Parent Trace ID in Keptn Apps 

**State: DRAFTING**

## Motivation
When promoting apps to other instances, an end user would like to see the whole flow across the stages in a single trace. This KEP proposes the usage of the [W3C trace-id](https://www.w3.org/TR/trace-context/#trace-id) to create links between traces across stages.

## Current Design
Currently, the Keptn Lifecycle Toolkit (Controller) starts a new trace whenever a new version/revision of a KeptnAppVersion is created and there is no way to pass over a TraceId to an Application. As a result, the Trace ends when an application is deployed and there is no Link to the next stage. 

## Proposed Design
This KEP proposes to add a new field to the KeptnApp CRD called `previousTraceId`. This field is optional and can be used to pass over a TraceId to the KeptnApp. The main use case for this field is to pass over the TraceId of the previous stage to the next stage when promoting an application and therefore keep an end to end view of the whole flow.

Therefore:
* If the `previousTraceId` is not set in the KeptnApp, the Keptn Lifecycle Toolkit (Controller) will create a new trace and leave the `previousTraceId` empty.
* if the `previousTraceId` is set in the KeptnApp, the Keptn Lifecycle Toolkit (Controller) will create a new trace and link the new trace to the `previousTraceId`.

## Benefits
Using this mechanism, a link between traces across stages can be created and therefore an end to end view of the whole flow can be achieved. This enables users to trace the whole flow of an application across stages.


