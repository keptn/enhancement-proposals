# KEP 92: Separation of Concerns for Tasks and MetricProviders

**State: IMPLEMENTED**

## Motivation
The Lifecycle Toolkit provides Custom Resources to define tasks and metric providers. At the moment, it assumes that KeptnTaskDefinitions, as well as Metric Providers are defined in the same namespace as the Application/Workload which gets deployed. For some reasons, this might not be beneficial for the user. 

It might be desired that platform admins/sre specify KeptnTaskDefinitions and MetricProviders, but also metrics in a central namespace and provide them to all developers. The developers might want to use these objects in their own Application and Workload definitions.

## Current Design
At the moment, it is assumed that KeptnTaskDefinitions as well as MetricProviders are deployed in the same namespace as the Application/Workload which gets deployed. Therefore, the Task Controller will look for a Metric Provider in the same namespace to compose the Task to be executed, the same is happening for the metrics with the metrics provider.

## Assumptions / Definitions
* A separation of concerns for tasks and metric providers is desired
* If an object is defined in the same namespace as the Application/Workload, this is preferred over the global objects

## Proposed Design
The current design will be extended to support the separation of concerns. Therefore, the Task and Metrics Controller will try to find the respective objects in the same namespace as the Applications/Workloads and if this is the case, the Controllers will use this. If the object is not found in the same namespace, The Controllers will look for the object in the global namespace, where the Keptn installation is running.

## Possible Extensions
* The Keptn installation can be configured to use a different namespace for the global objects
* The TaskDefinitions/MetricProviders may be configured in a <namespace>/<object> way, to allow the user to specify the namespace where the object is located

## Benefits
* The separation of concerns is supported
* It can be ensured that only allowed users can define tasks and metric providers (RBAC)



