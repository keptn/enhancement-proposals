# KEP 89: Context Information in Keptn Tasks 
**State: DRAFTING**

## Dependencies
* This KEP depends on KEP88 which introduces instances of Keptn Workloads and Apps

## Motivation
Currently, the lifecycle toolkit passes some context information (Workload, Application, Version) to the tasks. However, this information is not sufficient for promoting instances. As there should be a Keptn Task for promoting an instance, the context information should be extended to include the instance and a bit more information.

This KEP proposes the extension of passed context information.

## Current Design
At the moment, the context information is passed as a JSON string in the `KEPTN_CONTEXT` environment variable. The JSON string contains the following information:
```golang
  type TaskContext struct {
    WorkloadName    string `json:"workloadName"`
    AppName         string `json:"appName"`
    AppVersion      string `json:"appVersion"`
    WorkloadVersion string `json:"workloadVersion"`
    TaskType        string `json:"taskType"`
    ObjectType      string `json:"objectType"`
  }
```

This information is passed to the tasks as a JSON string in the `KEPTN_CONTEXT` environment variable which can be used in the corresponding tasks.

## Assumptions / Definitions
* It might be easier to pass over the whole KeptnAppVersion or KeptnWorkloadInstance as a JSON string instead of passing over the individual fields.
* There is no sensitive information in the context information

## Proposed Design
The proposed design is to pass over the whole KeptnAppVersion or KeptnWorkloadInstance Object as a JSON string in the `KEPTN_APP_CONTEXT` and `KEPTN_WORKLOAD_CONTEXT` environment variable. This would allow the tasks to access all information about the instance. Furthermore, annotations of the workload could be passed also as a JSON string in the `KEPTN_WORKLOAD_ANNOTATIONS` environment variable.

## Benefits
* The tasks have access to all information about the application and workload
* The tasks can access the annotations of the workload
* Additional information would not have to be defined in the context information

## Drawbacks
* The context information might be too big for some tasks
* The context information might contain sensitive information in the future which has to be filtered out





